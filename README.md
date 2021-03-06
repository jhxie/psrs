## Overview
MPI-based program that implements the parallel sorting by regular sampling
algorithm.

## Dependency
* C Compiler with ISO C99 Support (GCC **4.3+**).
* [CMake](https://cmake.org/) build system (**3.5+**).
* [Matplotlib](http://matplotlib.org/) plotting library.
* [MPICH](http://www.mpich.org/) message passing library (**3.2+**).
* [Python](https://www.python.org/) interpreter (**3.5+**).

**Ubuntu** (>= 16.04)  
```bash
sudo apt-get install build-essential cmake cmake-extras extra-cmake-modules libmpich-dev mpich python3-matplotlib
```
A script that can automate the above process is named
[prepare.sh](./tools/prepare.sh); to run the script:
```bash
bash ./tools/prepare.sh
```

## Build Instructions
Change working directory to where the source directory resides and then issue:
```bash
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make
```
Note the **"CMAKE_BUILD_TYPE=Release"** cache entry definition is necessary
if debug information is not needed (necessary for the
[plot.py](./tools/plot.py) script to work properly).

## Getting Started
To get usage help from the compiled program obtained from the last section:
```bash
cd src
mpiexec -n 1 ./psrs -h
```
For example, to obtain the mean sorting time and standard deviation of one
process quick sort to be used as a baseline for comparison:
```bash
mpiexec -n 1 ./psrs -l 10000000 -r 7 -s 10 -w 5
```
The *1* followed by the *-n* flag stands for how many processes to be launched;
the *10000000* followed by the *-l* flag stands for the length of the generated
array; the *7* that comes after *-r* stands for how many runs are required in
order to obtain the average; the *10* is the seed value supplied to the
*PRNG* of the array generation function; the last *5* argument for *-w*
flag denotes the window size for calculating moving average (number of runs
must not be less than window size; otherwise moving average can not be
calculated).

If needed, the mean sorting time along with its standard deviation (when both
*-r* and *-w* are set to *strictly greater* than 1, otherwise standard
deviation would always be 0) can be printed in binary form (double precision
floating-point value) by giving an extra *-b* flag - this is mostly useful for
piping the output tuple directly into another program.

The program also supports output statistics about per-phase running time by
giving an extra *-p* flag; depends on whether *-b* flag is given, the output
would be either a *4-element* tuple recording (moving-averaged) per-phase
running time in binary or text form of the following format:

> PHASE1,PHASE2,PHASE3,PHASE4

**NOTE**:
For simplicity of implementation, the author has made a decision that length
of the generated array must be *divisible* by the number of processes.

The program would print related warning message if this constraint is not
satisfied.

## Speedup Comparison
**NOTE**:  
Please refer to [REPORT.pdf](./doc/REPORT.pdf) for the detailed exposition.

In order to get the speedup comparison graph using both array length and number
of processes as independent variables, run the python script resides in *tools*
subdirectory:
```bash
python3 tools/plot.py -e ./build/src/psrs -d deviation.png -r runtime.png -p pie.png -s speedup.png -t table.png

```
The argument for *-e* flag is the path pointing to the *psrs* executable
compiled previously; *deviation.png* is the name of the standard deviation
table; *runtime.png* is the name of the bar chart visualization for comparing
runtime grouped by array length; *pie.png* is the base name (2 *png*s would be
written with *0* and *1* appended) for per-phased run time pie chart;
*speedup.png* is name of the speedup graph; *table.png* is the name of the
summary table for actual runtime.

To run the python script on a *RAC* cloud, two changes to the 'mpi_prefix'
variables need to be made; the old rvalues need to be replaced by
"mpiexec -n {process} -f /home/ubuntu/psrs/tools/mpi_hosts ".

Note that speedup ratios are calculated based on a collection of moving-average
runtimes with window size of 5 and 7 runs in total, which are the runtime
values shown in the figure ("Running Time in Moving Average") below.

A sample speedup graph obtained by following the above instructions:
![speedup](./doc/speedup.png)

The result is obtained from the following input table:

| Array Size | Number of Processes |
|:----------:|:-------------------:|
| 2¹⁹        | 1 2 4 8             |
| 2²¹        | 1 2 4 8             |
| 2²³        | 1 2 4 8             |
| 2²⁵        | 1 2 4 8             |

The actual sorting time is recorded in the following table:
![table](./doc/table.png)

And a table that records standard errors of those sorting times (last 5 runs of
7 runs in total):
![deviation](./doc/deviation.png)

To get a bird's eye view of how those sorting times differ group-wise, a 3-d
bar chart is drawn using both array size and number of processes as categorical
variables:
![runtime](./doc/runtime.png)

Two per-phase sorting-time comparison pie charts are also drawn to see what is
the dominant phase; note the second phase of the 2²⁷ case is *0%*, which could
be caused by floating point rounding errors as per-phase run time is divided by
the total run time to get the percentage value shown:
![pie0](./doc/pie0.png)

![pie1](./doc/pie1.png)

## Credit
* The Matplotlib CMake module is borrowed from the source repository of
[FreeCAD](
https://github.com/FreeCAD/FreeCAD/blob/master/cMake/FindMatplotlib.cmake).
* The paper illustrates the *Parallel Sorting by Regular Sampling* algorithm
can be found [here](./doc/PSRS.pdf).
* The reference book that provides guidelines on the proper use of MPI
interface is titled
*Using MPI: Portable Parallel Programming with the Message-Passing Interface*;
its official [website](http://wgropp.cs.illinois.edu/usingmpiweb/) has lots of useful resources about MPI in general.

## License
Copyright © 2016 - 2017 Jiahui Xie  
Licensed under the [BSD 2-Clause License][BSD2].  
Distributed under the [BSD 2-Clause License][BSD2].  

[BSD2]: https://opensource.org/licenses/BSD-2-Clause
