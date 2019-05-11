# graph500
### Moved to: https://github.com/UniHD-CEG/gpugraph500
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Requirements](#requirements)
- [Download](#download)
  - [Using git](#using-git)
  - [Using gz](#using-gz)
- [Build](#build)
- [Run](#run)
- [Scripts](#scripts)
  - [Script r-ify.sh](#script-r-ifysh)
  - [Script r-compare.sh](#script-r-comparesh)
  - [Script check-all.sh (requires SLURM)](#script-check-allsh-requires-slurm)
- [Profiling](#profiling)
- [Current Limitations](#current-limitations)
  - [Out-Of-Memory errors and CUDA memory size limitations:](#out-of-memory-errors-and-cuda-memory-size-limitations)
- [Troubleshooting](#troubleshooting)
- [Author](#author)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# Requirements

- C compiler. C++ Compiler with c++11 support.
- To use CUDA-BFS or CUDA-compression: CUDA 6+ support.
- To use SIMD compression: SSE4 support.
- To use SIMD+ compression SSE2 support.


# Download

- Create an account in [https://github.com](https://github.com)


## Using git

- Use your Bitbucket username account name as  __Your_GITHUB__

- [https://github.com/__Your_GITHUB__/UniHD-CEG/graph500.git](https://github.com/___Your_GITHUB___/UniHD-CEG/graph500.git)

- install `git`

- Fork the main repository, into your account 
- Clone from your account

```
$ # REPLACE __Your_GITHUB__
$ git clone https://github.com/__Your_GITHUB__/UniHD-CEG/graph500.git
$ cd bfs_multinode
```

## Using tar

- Download from:
[latest .gz](https://github.com/UniHD-CEG/graph500/archive/master.zip)

- Decompress:

```
$ tar -xzvf architectural_tuning.tar.gz
$ cd architectural_tuning
```

# Build

The code to compile is in the folder `cpu_2d/`. to build the binary:

```
./configure
make
```

for options, run `./configure --help`

# Run

Change to folder `eval/`

The tests are (valid with SLURM)

- o4p2n-roptim.rsh [SCALE_FACTOR]
- o4p2n-coptim.rsh [SCALE_FACTOR]
- o4p2n-noptim.rsh [SCALE_FACTOR]
- o9p8n.rsh [SCALE_FACTOR]
- o16p8n.rsh [SCALE_FACTOR]

```
sbatch o16p8n.rsh 21
```

runs a test with 16 proccesses in 8 nodes, using Scale Factor 21


# Scripts

* r-ify.sh
* r-compare.sh


Scripts are located under the `scripts/` folder

To install:

change to the `eval/` directory

```
$ ln -s ../scripts/r-ify.sh r-ify.sh
$ ln -s ../scripts/r-compare.sh r-compare.sh
$ chmod u+x *.sh
```


## Script r-ify.sh

It uses the execution traces to generate R-code. This R-code (once run) shows the time measurements of the Phases of the BFS code. This script uses result files with same Scale Factor. The results are represented as a Barplot.


```
$ ./r-ify.sh 423 424 425

-> File slurm-423.out. Validation passed (Tasks: 4, GPUS/Task: 1, Scale Factor: 21).
-> File slurm-424.out. Validation passed (Tasks: 4, GPUS/Task: 1, Scale Factor: 21).
-> File slurm-425.out. Validation passed (Tasks: 4, GPUS/Task: 1, Scale Factor: 21).
Enter new labels for the X-Axe? (y/n) [n] y
Enter a total 3 label(s) between quoutes. Separate them with spaces: "4p2n-Roptim" "4p2n-Coptim" "4p2n-Noptim"
-> Created file "file-423-424-425.r".
-> R-Code successfully generated.
```

Open the file `file-423-424-425.r` with your R editor and run the code.

## Script r-compare.sh

As the previous script, this also uses the execution traces to generate R-code. This differs from the previous one in that it can compare several files from several Scale Factors. Results are visualized as a Lineplot.


```
$ ./r-compre.sh JOBID1 JOBID2 ...
```

## Script check-all.sh (requires SLURM)

This script automatizes the execution of tests for different Scale Factors.

```
$ check-all.sh 15 30
```

This will run the tests with format `o*.rsh` in the `eval/` folder for Scale Factors 15 to 30. Process is shown in ncurses-like format.


# Profiling

This BFS application allows the code to be instrumented in Zones using Score-P with very low overhead. This requires Score-P and Scalasca to be installed in the system. The results may be analyzed either visually (using CUBE) or through console using `scorep-score`.

These tools may be installed locally (no priviledged user is needed) using the external-apps-installer.sh aforementioned.

The names of the instrumentable Zones are listed below. Other Zones may be added if needed.

```
BFSRUN_region_vertexBroadcast
BFSRUN_region_nodesTest
BFSRUN_region_localExpansion
BFSRUN_region_testSomethingHasBeenDone
BFSRUN_region_columnCommunication
BFSRUN_region_rowCommunication
```

The first step is to update the system variables. This may be done either on .bashrc or in a separate script.

Update the paths in the variables below

```
$ cat >> ~/.bashrc << EOF
export G500_ENABLE_RUNTIME_SCALASCA=yes

export SCOREP_CUDA_BUFFER=48M
export SCOREP_CUDA_ENABLE=no
export SCOREP_ENABLE_PROFILING=true
export SCOREP_ENABLE_TRACING=false
export SCOREP_PROFILING_FORMAT=CUBE4
export SCOREP_TOTAL_MEMORY=12M
export SCOREP_VERBOSE=no
export SCOREP_PROFILING_MAX_CALLPATH_DEPTH=330

export LD_LIBRARY_PATH=$HOME/cube/lib:$LD_LIBRARY_PATH
export PATH=$HOME/cube/bin:$PATH

export LD_LIBRARY_PATH=$HOME/scorep/lib:$LD_LIBRARY_PATH
export PATH=$HOME/scorep/bin:$PATH

export LD_LIBRARY_PATH=$HOME/scalasca/lib:$LD_LIBRARY_PATH
export PATH=$HOME/scalasca/bin:$PATH

export MPI_PATH=/home/jromera/openmpi
export PATH=$MPI_PATH/bin:$CUDA_PATH/bin:$PATH

export LD_LIBRARY_PATH=$MPI_PATH/lib:$CUDA_PATHo/lib64:$CUDA_PATHo/lib64/stubs:$CUDA_PATHo/lib:$CUDA_PATHo/extras/CUPTI/lib64:$CUDA_PATHo/extras/CUPTI/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$HOME/scorep/lib:$LD_LIBRARY_PATH
export PATH=$HOME/scorep/bin:$PATH
EOF
```

The variable `G500_ENABLE_RUNTIME_SCALASCA` set to yes will enable the required runtime instrumentor of Scalasca.

Results will be stored on a folder with format `scorep-*` in the `/eval` folder.

To instrument graphically with CUBE run:

```
$ cd eval/scorep-____FOLDER_NAME____
$ cube profile.cubex
```

To instrument through the console run:

```shell
$ cd eval/scorep-____FOLDER_NAME____
$ scorep-score -r profile.cubex
```

# Current Limitations

## Out-Of-Memory errors and CUDA memory size limitations:

For some high Score-Factors (e.g: 22, as of the day of writing this guide), the resulting Slurm trace will be:

```
Using SCALE-FACTOR 22
Tue Jul 14 15:42:51 CEST 2015
 ========================   JOB MAP   ========================

 Data for node: creek01 Num procs: 2
        Process OMPI jobid: [1404,1] Process rank: 0
        Process OMPI jobid: [1404,1] Process rank: 2

 Data for node: creek02 Num procs: 2
        Process OMPI jobid: [1404,1] Process rank: 1
        Process OMPI jobid: [1404,1] Process rank: 3

 =============================================================
row slices: 2, column slices: 2
graph_generation:               7.749108 s
Input list of edges genereted.
6.710886e+07 edge(s) generated in 8.499692s (7.895447 Medges/s on 4 processor(s))
Adjacency Matrix setup.
2.956432e+06 edge(s) removed, because they are duplicates or self loops.
1.283049e+08 unique edge(s) processed in 18.308235s (7.008041 Medges/s on 4 processor(s))
[../b40c/graph/bfs/csr_problem_2d.cuh, 697] CsrProblem cudaMalloc frontier_queues.d_values failed (CUDA error 2: out of memory)
[cuda/cuda_bfs.cu, 486] Reset error. (CUDA error 2: out of memory)
MPI_ABORT was invoked on rank 0 in communicator MPI_COMM_WORLD
with errorcode 1.
```


# Troubleshooting
- Problem: In the .out file of Slurm/ Sbatch execution I get the text:

```
S=C=A=N: Abort: No SCOREP instrumentation found in target ../cpu_2d/g500
```

- Solution:

The instrumentation is activated for the runtime execution (i.e: the binary is being run prefixed with scalasca).

Disable it with:

```
$ export G500_ENABLE_RUNTIME_SCALASCA=no
```

# Author

Computer Engineering Group at Ruprecht-Karls University of Heidelberg

# License

Copyright (c) 2016, Computer Engineering Group at Ruprecht-Karls University of Heidelberg, Germany. All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

- Redistributions of source code must retain the above copyright notice, this
 list of conditions and the following disclaimer.

- Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.

- Neither the name of <project> nor the names of its
    contributors may be used to endorse or promote products derived from
     this software without specific prior written permission.

     
THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS” AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE

DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

