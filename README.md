Helps performance evaluation experiments automation.

# Introduction
Performance evaluation experiments are hard to be done manually by people. It's
basically repetitive and time consuming. To be practical, most people automate
the experiments. Because each experiments have particular characterizations and
limitations, However, most automation code could not be reused easily.

This project aims to

1. generalize those experiments and
2. develop reusable, useful tools and structure for automation.

# Usage
`$ run_exps.py <experiments specification file>`

## Experiments Specification File
Text file which specifies what experiments should be done.
Consists with zero or more experiments. Each experiment seperated by one or
more blank line.

Experiment consists with zero or more bash commands which should be done as
part of experiment. Each command have specific type which specifies when and
how the command should be executed.

If your experiments are too big and repetitive, consider to write your own
experiments specification file generator rather than write it down manually in
miserable way. For detail, see below section, [Experiments Specification File
Generator](#Experiments Specification File Generator).

### Type of commands
 * start: Commands which should be done before experiment start.
   (e.g., kernel module loading or program build)
 * main: Main workload. Experiments will be end after command(s) of this type
   end. If multiple main workloads exist, they will be executed concurrently
   and workloads terminated earlier will be executed repeatedly until slowest
   workload be terminated.
 * back: Background command which should be run while main type commands run.
   (e.g., profiling or system stressing task)
 * end: Commands which should be done after experiment end.
   (e.g., kernel modul unloading or meta files cleanup)
 * check: Check whether experiment was successful. Commands be executed after
   end commands end and notifies check result using return code. Return code 0
   means sucess, other values means failure. If any one of check command says
   failed, the experiment be executed again until success up to 10 times.

### Example
Below is an example for *experiments specification file*
```
start: git checkout v1.0
start: make -j
start: insmod sjp.ko
main: ./workload/workload1 > work1.out
main: ./workload/workloadtogether > work2.out
back: perf record -o perfout.data -a
back: vmstat 2 > vmstatout.data
end: rmmod sjp
end: make clean
check: grep success work1.out
check: grep success work2.out

start: git checkout v1.1
start: make -j
start: insmod sjp.ko
main: ./workload/workloadalone
back: perf record -o perfout.data -a
back: vmstat 2 > vmstatout.data
end: rmmod sjp
end: make clean
end: git checkout master
```

# Experiments Specification File Generator
For simplicity, we recomment to write experiment specification file in linearly
interpretable way rather than procedural way. Using `loop` or `goto` inside
specification file would be complex to be read.

However, constraining people to write down repeating big content manually is
crime against humanity. For the reason, users are recommended to develop and
use their own experiments specification file generator. To help the process,
simple stub generator, `generate_exp_conf.py` provided. It contains essential
code for automated experiments specification file generator.

## Usage of the stub
1. Copy the stub
2. Edit main loop of experiments generation as you need
3. Run modified copy and redirect stdout to appropriate file
4. Use the file for `run_exps.py`


# License
GPL v3

# Author
SeongJae Park (sj38.park@gmail.com)
