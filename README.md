# NOELLE artifact evaluation

This repository includes the evaluation materials for the NOELLE CGO 2022 paper: "NOELLE Offers Empowering LLVM Extensions".

## Artifact

This artifact generates three set of results.
- MINIMAL: Data that supports the version of the paper that was submitted in September excluding the few benchmarks from SPEC CPU2017 that requires several days each (4 days)
- SUBMISSION: Data about the few benchmarks from SPEC CPU2017 that requires several days each (extra 12 days)
- FINAL: New results that were not included in the submitted version of the paper, but will be added to the final version of the paper (extra 5 days)
Next you can find the instructions to reproduce all the above results.

### Prerequisites 

The artifact is available as a docker image.
The artifact will generate the results when invoking the script ```./run_me.sh``` included in the directory ```CGO_2022_artifact_evaluation```.
Which results will be generated depends on the envionment variables set (see below).

We open sourced NOELLE more than a year ago.
We also open sourced the infrastructure we built to evaluate NOELLE on several benchmark suites (e.g., PARSEC, MiBench, SPEC CPU2017).
Therefore, we decided to not include these frameworks within the artifact.
Instead, the script ```run_me.sh``` will clone the open sourced git repository (from GitHub).
So please make sure to have a network connection when you run the artifact.

### Experiments and results

Next we describe the three set of experiments and results that can be generated with this artifact.

Some results differ slighlty from the plots shown in the submitted paper because we found a few minor bugs in one of the alias analyses we relied on.
We fixed these bugs and we noticed two conseguences:
- a few more dependences now exist in the PDG of a few benchmarks. These dependences do not actually exist, but the current alias analyses aren't able to prove it because of our conservarive fix. These dependences have reduced the speedups obtained for a few benchmarks like streamcluster of PARSEC.
- a few dependences have been removed from the PDG. This allows NOELLE to have higher speedups than the submitted version of the paper in a few benchmarks like blackscholes of PARSEC.
Moreover, the changes to dependences had a minor impact to the number of invariants and loop dependences.
Finally, all these changes are minimal and do not change the claims made in the paper.


#### MINIMAL
This set of experiments and results are about all benchmarks included in the submitted version of the paper with the only exception of five SPEC CPU2017 benchmarks (omnetpp_r, perlbench_r, x264_r, blender_r, parest_r).
This is because these five benchmarks require a significant amount of time so we decided to keep them separate from the minimal set; these benchmarks are included in the SUBMISSION set.

To generate the MINIMAL results, then do the following:
```
cd CGO_2022_artifact_evaluation ;
unset NOELLE_SUBMISSION ;
unset NOELLE_FINAL ;
./run_me.sh
```

Please look at the output of the script to know how to check the current state.
Finally, results will be stored in ```results/current_machine```.


#### SUBMISSION

A few SPEC CPU2017 benchmarks are evaluated for this set of experiments/results (omnetpp_r, perlbench_r, x264_r, blender_r, parest_r).
Warning: each of these benchmarks will take several days to compile and run for all configurations required by the NOELLE paper (total of 12 days).

To generate the SUBMISSION results, then first generate MINIMAL, and then do the following:
```
cd CGO_2022_artifact_evaluation ;
unset NOELLE_FINAL ;
export NOELLE_SUBMISSION=1;
./run_me.sh ;
```


#### FINAL
Since NOELLE is an ongoing project, we did not stop working on it after submission.
We improved NOELLE after submission to the point that we can now target more benchmarks and we can perform more evaluations.
This artifact also includes the capability to generate these extra evaluations and benchmarks.
Finally, these new evaluations and benchmarks will be included in the final version of the paper.

To generate the FINAL results, then first generate MINIMAL, and then do the following:
```
cd CGO_2022_artifact_evaluation ;
unset NOELLE_SUBMISSION ;
export NOELLE_FINAL=1 ;
./run_me.sh ;
```


## Data collection
In this section, we will collect the data required to reproduce the figures in the paper.

### Run all evaluations (Estimated time: ? hours)

To run the artifact to generate all data needed to support the submitted version of the paper, run within the docker image
```
cd ~ ;
./run_me_submission.sh ;
```

To (optionally) run the artifact to generate all data needed to support the final version of the paper, run within the docker image
```
cd ~ ;
./run_me_final.sh ;
```

### Step-by-Step Guide for running a single evaluation
TODO

#### Generating Figure 3 (Estimated time: ? hours)
TODO

#### Generating Figure 4 (Estimated time: ? hours)
TODO

#### Generating Figure 5 (Estimated time: ? hours)
TODO

## Data organization
All the generated data can be found under `results`.
Data we generated in our machine can be found under `results/authors_machine`.
Data that is generated by running the artifact can be found under `results/current_machine`.

Both `results/authors_machine` and `results/current_machine` are organized in the same way.
They have a subdirectory per benchmark suite (e.g., `results/current_machine/PARSEC3` includes all data generated for the PARSEC-3.0 benchmarks).
Each benchmark suite has three sub-directories: 
- `dependences`: this sub-directory includes information about the dependences of the benchmark. This data is used to generate Figure 3. 
- `loops`: this sub-directory includes information about loops like their induction variables or they loop invariants. This data is used to generate Figure 4 and it will be used to generate a new figure in the final version of the paper. This new figure will compare the number of induction variables (per benchmark) detected by LLVM and those detected by NOELLE.
- `speedups`: this sub-directory includes execution times collected by running the benchmarks when compiled using vanilla `clang` and when compiled when using `clang` adding NOELLE optimizations in its middle-end. This data is used to generate Figure 5.
- `compilation`: this sub-directory includes new information that wasn't part of the original submission of the paper. This data is about the compilation time and memory consumption of the most important tools built upon NOELLE.

### Dependences
For each benchmark in a benchmark suite you will find a text file in the benchmark suite directory (e.g., `results/current_machine/PARSEC3/blackscholes.txt` )

### Loops
TODO

### Speedups
TODO

### (Optional) Induction variables
TODO
