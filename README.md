# NOELLE artifact evaluation for CGO 2022

This repository includes the evaluation materials for the NOELLE CGO 2022 paper: "NOELLE Offers Empowering LLVM Extensions".

## Artifact

This artifact generates three sets of results.
Adding or not SPEC CPU2017 to each set of results is optional (see the section "Experiments and results" for more details).
- MINIMAL: Data that supports the version of the paper that was submitted in September excluding the few benchmarks from SPEC CPU2017 that requires several days each (6 days when SPEC CPU2017 is included while excluding the five benchmarks mentioned, 2 days otherwise).
- SUBMISSION: Data about the few benchmarks from SPEC CPU2017 that requires several days each (extra 12 days).
- FINAL: New results that were not included in the submitted version of the paper, but will be added to the final version of the paper (extra 5 days).

Next you can find the instructions to reproduce all the above results.

### Prerequisites 

The artifact is available as a docker image and can be downloaded at
```
http://www.cs.northwestern.edu/~simonec/files/Software/Artifacts/Noelle.tar
```
The artifact will generate the results when invoking the script ```./run_me.sh``` included in the directory ```CGO_2022_artifact_evaluation```.
Which results will be generated depends on the envionment variables set (see below).

We open sourced NOELLE, VIRGIL, and the SCAF alias analysis framework more than a year ago.
We also open sourced the infrastructure we built to evaluate NOELLE on several benchmark suites (e.g., PARSEC, MiBench, SPEC CPU2017, PolyBench).
We decided to only include NOELLE in the artifact, everything else will be downloaded automatically.
In more details, the script ``run_me.sh`` will clone the open sourced git repositories (from GitHub) that are not included within the docker image.
So please make sure to have a network connection when you run the artifact.

### Experiments and results

Next we describe the three set of experiments and results that can be generated with this artifact.

Some results might differ slighlty from the plots shown in the submitted paper (the claims made in the paper are still valid).
Changes are created because we found a few minor bugs in one of the alias analyses NOELLE relies on (they are not part of NOELLE).
We fixed these bugs and we noticed two conseguences:
- a few more dependences now exist in the PDG of a few benchmarks. These dependences do not actually exist, but the current alias analyses aren't able to prove it because of our conservarive fix. These dependences have reduced the speedups obtained for a few benchmarks like streamcluster of PARSEC.
- a few dependences have been removed from the PDG. This allows NOELLE to have higher speedups than the submitted version of the paper in a few benchmarks like blackscholes of PARSEC.

The above changes to dependences had a minor impact to the number of loop invariants and dependences in the PDG.
Finally, all these changes are minimal and do not change the claims made in the paper.

#### SPEC CPU2017

Because SPEC CPU2017 cannot be shared, this artifact enable/disable this suite by having or not the environment variable `NOELLE_SPEC`. 
This environment variable is not set by default and therefore SPEC benchmarks will not run by default.
To include the SPEC CPU2017 benchmarks, you need to:
- copy the SPEC CPU2017 archived and compressed using gzip into the file `CGO_2022_artifact_evaluation/benchmarkSuites/SPEC2017.tar.gz` 
- set the environment variable `NOELLE_SPEC` to an integer value (e.g., `export NOELLE_SPEC=1`)
- Run the other steps as described next (e.g., MINIMAL)

#### Results

Time results are generated by running each benchmark 5 times by default. The median is computed from these runs.
You can customize how many runs you want to generate by setting the environment variable `NOELLE_RUNS`.
For example, if you want to run each time-experiment 11 times, then run
```
export NOELLE_RUNS=11;
```
and then generate one of the three set of results (see below).
If you do not set ```NOELLE_RUNS```, then each time-sensitive result is generated 5 times.

Execution times might vary depending on the platform.
We tuned the parallelization techniques for our platform, the one described in the NOELLE paper submitted in September.
We noticed exeution times vary significantly for HELIX depending on the core-to-core latencies.
Also, execution times vary significantly for DSWP depending on the core-to-core bandwidth.

Results need to be generated in an equivalent platform as the one described in the NOELLE paper.
Turbo boost and hypter-threading needs to be disabled (they only impact the execution times).
Furthermore, because of HELIX and DSWP are sensitive to either latency or bandwidth between cores, it is important to keep all threads running on the same NUMA zone.
Also, all the experiments need to be contained within the same CPU socket.
Finally, the Intel-based multicore needs to have at least 8 physical cores in the same CPU where the experiments run.

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
We improved NOELLE after submission to the point that we can now target more benchmarks and we can perform more evaluations compared to when we submitted the paper.
This artifact also includes the capability to generate these extra evaluations and benchmarks.
Finally, these new evaluations and benchmarks will be included in the final version of the paper.

To generate the FINAL results, then first generate MINIMAL, and then do the following:
```
cd CGO_2022_artifact_evaluation ;
unset NOELLE_SUBMISSION ;
export NOELLE_FINAL=1 ;
./run_me.sh ;
```


## Data organization
All the generated data can be found under `results`.
Data we generated in our machine can be found under `results/authors_machine`.
Data that is generated by running the artifact can be found under `results/current_machine`.

Both `results/authors_machine` and `results/current_machine` have the same structure.
They both have one sub-directory per benchmark suite; for example, `results/current_machine/PARSEC3` includes all data generated for the PARSEC-3.0 benchmarks.
Each benchmark suite has three sub-directories: 
- `dependences`: this sub-directory includes information about the dependences of the benchmark. This data is used to generate Figure 3. 
- `loops`: this sub-directory includes information about loops like their induction variables or their loop invariants. This data is used to generate Figure 4. This data also includes new results that we will add to the final version of the paper. In more detail, we will add a new figure in the final version of the paper to compare the number of induction variables (per benchmark) detected by LLVM and those detected by NOELLE.
- `time`: this sub-directory includes execution times collected by running the benchmarks when compiled using the unmodified middle-end of `clang` and when compiled using NOELLE transformations. This data is used to generate Figure 5.
- `IR`: this sub-directory includes all the IR files generated by the different NOELLE configurations (e.g., DOALL, HELIX, DSWP). This is only useful to cache the results of compilations enabling the user of this artifact to avoid re-compiling benchmarks.

### Speedups
Each benchmark is run using the vanilla `clang` compilation pipeline (called baseline) as well as using DOALL, HELIX, and DSWP included in NOELLE.

Baseline results can be found in `results/current_machine/time/BENCHMARK_SUITE/baseline`.
For example, the execution times of blackscholes from PARSEC can be found in `results/current_machine/time/PARSEC3/baseline/blackscholes.txt`.

DOALL results can be found in `results/current_machine/time/BENCHMARK_SUITE/DOALL`.

HELIX results can be found in `results/current_machine/time/BENCHMARK_SUITE/HELIX`.

Finally, DSWP results can be found in `results/current_machine/time/BENCHMARK_SUITE/DSWP`.

The speedups of DOALL over the baseline can be found in `results/current_machine/time/BENCHMARK_SUITE/DOALL.txt`.
The speedups of HELIX over the baseline can be found in `results/current_machine/time/BENCHMARK_SUITE/HELIX.txt`.
The speedups of DSWP over the baseline can be found in `results/current_machine/time/BENCHMARK_SUITE/DSWP.txt`.
These speedups (DOALL, HELIX, DSWP) are used to generate Figure 5 of the paper.

### Invariants
The invariants of all loops of all benchmarks of a benchmark suite can be found in `results/current_machine/loops/BENCHMARK_SUITE/invariants.txt`.
For example, the invariants of PARSEC benchmarks can be found in the file `results/current_machine/loops/PARSEC3/invariants.txt`.

Each benchmark has one line in the related `invariants.txt`. 
This file is organized in three columns.
The first column is the name of the benchmark.
The second column is the number of invariants accumulated over all loops of a given benchmark that are detected by LLVM.
The third column is the number of invariants accumulated over all loops of a given benchmark that are detected by NOELLE.
The IR that is analyzed to generate this information is the result of all NOELLE transformations that run before a parallelization scheme.
This is the IR file `baseline_with_metadata.bc` of a given benchmark, which can be found in `results/current_machine/IR/BENCHMARK_SUITE/BENCHMARK`.
For example, for `blackscholes` of PARSEC, the IR file that is analyzed to generate the invariants is 
```
results/current_machine/IR/BENCHMARK_SUITE/blackscholes/baseline_with_metadata.bc
```

### Dependences
The number of memory dependences in the PDG computed by using only the LLVM alias analyses and those computed by adding other alias analyses included in NOELLE can be found in `results/current_machine/dependences/BENCHMARK_SUITE/absolute_values.txt` and `results/current_machine/dependences/BENCHMARK_SUITE/relative_values.txt`.
The first file includes the absolute numbers of dependences and the second file includes the fraction of dependences declared by LLVM and NOELLE.

The file `absolute_values.txt` has the following structure.
One row per benchmark and four columns for each row.
The first column is the name of the benchmark.
The second column is about NOELLE.
The third column is about LLVM.
The fourth column is the total number of memory dependences computed assuming all memory instructions depend on each other.

The file `relative_values.txt` has the following structure.
One row per benchmark and three columns for each row.
The first column is the name of the benchmark.
The second column is about NOELLE.
The third column is about LLVM.

### (Optional) Induction variables
The induction variables of all loops of all benchmarks of a benchmark suite can be found in `results/current_machine/loops/BENCHMARK_SUITE/induction_variables.txt`.
For example, the induction variables of PARSEC benchmarks can be found in the file `results/current_machine/loops/PARSEC3/induction_variables.txt`.

Each benchmark has one line in the related `induction_variables.txt`. 
This file is organized in three columns.
The first column is the name of the benchmark.
The second column is the number of induction variables accumulated over all loops of a given benchmark that are detected by LLVM.
The third column is the number of induction variables accumulated over all loops of a given benchmark that are detected by NOELLE.
The IR that is analyzed to generate this information is the result of all NOELLE transformations that run before a parallelization scheme (this code is the IR file `baseline_with_metadata.bc` of a given benchmark).
