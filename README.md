# Storage Tuner Benchmark
Quick but meaningful storage benchmark utility

## Introduction
Performance measurements on storage systems or even local disks are difficult to "get right". More than often blog or forum posts, news articles and other sources of information will quote flawed tests or incomparable numbers.

Storage Tuner Benchmark aims to change that by being easy to use (no config) and covering a good range of application needs in a well defined series of "Tests". It uses the excellent "fio" as a backend and will likely be extended in the future to cover more scenarios by adding more tests and eventually forming test groups. Individual tests will stay unchanged for as long as possible to stay comparable.

## Requirements
 - \*nix (tested on Debian GNU/Linux, Open Indiana and FreeBSD)
 - fio (auto-installed with apt-get, yum or pkg when not present)
 - 1GB of free disk space for testing
 - SSH for remote execution (optional)

## Installation
Download storage-tuner-benchmark directly from Github and mark executable:
```bash
wget https://raw.githubusercontent.com/Korkman/storage-tuner-benchmark/master/storage-tuner-benchmark
chmod a+x storage-tuner-benchmark
```
Install "fio" or execute storage-tuner-benchmark with root privileges to have it installed automatically.

## Usage
```
storage-tuner-benchmark 2.0.0

Quick but meaningful storage benchmark utility

Usage: storage-tuner-benchmark <target> [path|cleanup [testname|cleanup] ]

target        Required. Any of:
              here            run on local system
              user@hostname   run on remote system via ssh

path          Optional. Base directory where a new directory will 
              be created containing test files.
              Default: home directory

testname      Optional. Name of a specific test in quotes.
              Default: run all tests

cleanup       Optional. Deletes test files directory. Run this
              when done tuning.

Pre-requisites: fio, the benchmark tool, will be installed via apt-get
or yum when missing.
```
## Examples
```
./storage-tuner-benchmark here /mnt
Creates "stb-testdir" in /mnt and runs all tests.

./storage-tuner-benchmark here /mnt "1 file, 1 thread, seq 1M writes"
Repeats the test named "1 file, 1 thread, seq 1M writes".

./storage-tuner-benchmark here /mnt cleanup
Will remove the test directory.
```
## Results example
Results are boiled down to essential, human readable information.
```
storage-tuner-benchmark version 2.0.0
Testgroup "current"
===   1 file series   ===
1 file, 1 thread, seq 1M writes, simple: 87 write iops
1 file, 1 thread, rnd 16k writes, simple: 98 write iops
1 file, 16 threads, rnd 4k writes, posixaio: 107 write iops
1 file, 16 threads, rnd 8k writes, posixaio: 104 write iops
1 file, 16 threads, rnd 16k writes, posixaio: 103 write iops
1 file, 16 threads, rnd 16k writes, posixaio: 103 write iops
===  16 file series   ===
16 files, 1 thread each, seq 1M writes, simple: 92 write iops
16 files, 1 thread each, rnd 16k writes, simple: 254 write iops
16 files, 1 thread each, rnd 16k writes, posixaio: 253 write iops
16 files, 16 threads each, rnd 16k writes, posixaio: 264 write iops
===   O_SYNC series   ===
1 file, 1 thread, rnd 16k writes, simple, o_sync: 190 write iops
1 file, 16 threads, rnd 16k writes, posixaio, o_sync: 201 write iops
16 files, 1 thread each, rnd 16k writes, simple, o_sync: 542 write iops
16 files, 16 threads each, rnd 16k writes, posixaio, o_sync: 491 write iops
===    read series    ===
1 file, 1 thread, seq 1M reads, simple: 93 read iops
1 file, 16 threads, rnd 16k reads, posixaio: 309 read iops
16 files, 1 thread each, seq 1M reads, simple: 78 read iops
16 files, 1 thread each, rnd 16k reads, posixaio: 1975 read iops
16 files, 16 threads each, rnd 16k reads, posixaio: 1980 read iops
=== native aio series ===
1 file, 16 threads, rnd 16k writes, native aio: 87 write iops
16 files, 16 threads each, rnd 16k writes, native aio: 250 write iops
1 file, 16 threads, rnd 16k reads, native aio: 306 read iops
16 files, 16 threads each, rnd 16k reads, native aio: 1997 read iops
```
## Should I run only tests I care about?
No, not in general. Selective testing is only meant for bottleneck resolving. After resolving a bottleneck, always compare full runs from before and after to identify potentially negative impacts on other tests. Also, when sharing results with others, always share full results. Trade-offs in storage tuning are totally legit, but they need to be understood and visible.

