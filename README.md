B-Link Tree and YCSB Benchmark Reproducibility Guide

This repository contains the benchmarking setup used to evaluate different B-Link Tree synchronization strategies under YCSB workloads A–E. The benchmarks are based on the btree-benchmarks framework and were executed using YCSB-generated workloads.

This guide describes exactly how to reproduce all results.

Required dependencies:

sudo apt update
sudo apt install -y cmake g++ clang libnuma-dev python3 openjdk-21-jdk


Verify Java:

java -version

2. Build the Project

From the root of the repository:

cmake .
make


This will build the following binaries:

blinktree_thread_spinlock
blinktree_thread_rwlock
blinktree_thread_olfit
bwtree_thread
olc_btree_thread


They will appear in:

build/bin/

3. Generate YCSB Workloads (A–E)

We use the built-in Python generator script.

From the project root:

python3 scripts/generate_ycsb a randint
python3 scripts/generate_ycsb b randint
python3 scripts/generate_ycsb c randint
python3 scripts/generate_ycsb d randint
python3 scripts/generate_ycsb e randint


After these commands finish, verify that the workload files were generated:

wc -l workloads/fill_randint_workloada
wc -l workloads/mixed_randint_workloada

wc -l workloads/fill_randint_workloadb
wc -l workloads/mixed_randint_workloadb
...


Expected behavior:

Workload A, B, C, D, E must NOT be 0 lines

Example:

50000000 workloads/fill_randint_workloadc
10000000 workloads/mixed_randint_workloadc

4. Editing Workload Sizes (Important)

We manually adjusted the workload sizes inside:

workloads_specification/


Each workload file (workloada, workloadb, etc.) contains parameters like:

recordcount=1000000
operationcount=1000000


We modified these values to match our experiment scale:

Typical values we used:

fill phase: 1,000,000 or 50,000,000 inserts

transaction phase: 1,000,000 to 10,000,000 operations

Example edit:

nano workloads_specification/workloadc


Then adjust:

recordcount=10000000
operationcount=10000000


After editing any workload, you must regenerate it:

python3 scripts/generate_ycsb c randint

5. Running the Benchmarks (1–16 Threads)

All benchmarks were run from:

cd build/bin

Spinlock
./blinktree_thread_spinlock 1:16 -s 1 \
-f ../../workloads/fill_randint_workloadc \
../../workloads/mixed_randint_workloadc \
| tee ../../ycsb_c_spinlock.txt

Reader–Writer Lock
./blinktree_thread_rwlock 1:16 -s 1 \
-f ../../workloads/fill_randint_workloadc \
../../workloads/mixed_randint_workloadc \
| tee ../../ycsb_c_rwlock.txt

OLFIT (Optimistic)
./blinktree_thread_olfit 1:16 -s 1 \
-f ../../workloads/fill_randint_workloadc \
../../workloads/mixed_randint_workloadc \
| tee ../../ycsb_c_olfit.txt


The same commands apply for workloads A, B, D, E by replacing:

fill_randint_workloadc
mixed_randint_workloadc


with:

fill_randint_workloada
mixed_randint_workloada
fill_randint_workloadb
mixed_randint_workloadb
fill_randint_workloadd
mixed_randint_workloadd
fill_randint_workloade
mixed_randint_workloade

6. Converting Output to Clean CSV

After running a benchmark:

awk '$3==1 {print $1 "," $6}' ycsb_c_spinlock.txt > ycsb_c_spinlock.csv
awk '$3==1 {print $1 "," $6}' ycsb_c_rwlock.txt  > ycsb_c_rwlock.csv
awk '$3==1 {print $1 "," $6}' ycsb_c_olfit.txt   > ycsb_c_olfit.csv


This extracts:

THREADS,THROUGHPUT


Example output:

1,3.12598e+06
2,4.39560e+06
3,6.04595e+06
...
16,6.25e+07


These CSV files were then imported directly into Excel and Python for plotting.

7. Bw-tree and OLC Tree

These two binaries are also available:

./bwtree_thread
./olc_btree_thread


They use the same workload files and are executed with the same format as above.

8. Important Notes

If you see inf or nan throughput:

Your workload file is empty or incorrect

Re-run:

python3 scripts/generate_ycsb <workload> randint


If locale errors appear:

export LC_ALL=C
export LANG=C


The OLFIT variant consistently produces the best scalability

9. Framework Reference

This implementation is based on:

btree-benchmarks
Mühlig, J. (2021). GitHub Repository
https://github.com/jmuehlig/btree-benchmarks
