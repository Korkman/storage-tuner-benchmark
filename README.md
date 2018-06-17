# Storage Tuner Benchmark 2.1.0
Quick but meaningful storage benchmark utility

## Introduction
Performance measurements on storage systems or even local disks are difficult to "get right". More than often blog or forum posts, news articles and other sources of information will quote flawed tests or incomparable numbers.

Storage Tuner Benchmark aims to change that by being easy to use (no config) and covering a good range of application needs in a well defined series of "Tests". It uses the excellent "fio" as a backend and will likely be extended in the future to cover more scenarios by adding more tests and eventually forming test groups. Individual tests will stay unchanged for as long as possible to stay comparable.

## Table of contents
- [Usage](#usage)
- [FAQ](#faq)
- [Test series explained](#test-series-explained)

# Usage

## Requirements
 - \*nix (tested on Debian GNU/Linux, Open Indiana and FreeBSD)
 - fio (auto-installed with apt-get, yum or pkg when not present)
 - 1GB of free disk space for testing
 - SSH for remote execution (optional)


## Installation
- Install "fio" with your package manager or let storage-tuner-benchmark do it for you (requires root privileges).
- Download storage-tuner-benchmark directly from Github and mark executable:
```bash
wget https://raw.githubusercontent.com/Korkman/storage-tuner-benchmark/master/storage-tuner-benchmark
chmod a+x storage-tuner-benchmark
```

## Invocation
```
# ./storage-tuner-benchmark --help
storage-tuner-benchmark 2.1.0
Quick but meaningful storage benchmark utility

Usage: storage-tuner-benchmark <target> [path|cleanup [testname|cleanup] ]

target        Required. Any of:
              here            run on local system
              user@hostname   run on remote system via ssh

path          Optional. Base directory where a new directory will
              be created containing test files.
              Default: home directory

testname      Optional. Name of a specific test in quotes. Requires a full run before.
              Default: run all tests

cleanup       Optional. Deletes test files directory. Run this
              when done tuning.

Pre-requisites: fio, the benchmark tool, will be installed via apt-get
or yum when missing.
```
## Examples
```
# ./storage-tuner-benchmark here /mnt
Creates "stb-testdir" in /mnt and runs all tests.

# ./storage-tuner-benchmark here /mnt "16 files, 1 thread each, rnd 16k writes, simple, o_sync"
Repeats the test named "16 files, 1 thread each, rnd 16k writes, simple, o_sync".

# ./storage-tuner-benchmark here /mnt cleanup
Will remove the test directory.
```
Replace "here" with ```user@your.hostname``` to have storage-tuner-benchmark ssh into the remote machine (root user only required when installing fio automatically).

## Results example
Results are boiled down to essential, human readable information.
```
Running tests in "/root/stb-testdir" on your.hostname @ 2018-06-17 13:13:07 ...
storage-tuner-benchmark version 2.1.0
Testgroup "current"
===   1 file series   ===
1 file, 1 thread, seq 1M writes, simple: 191 write iops (105% HDD)
1 file, 1 thread, rnd 16k writes, simple: 409 write iops (57% HDD)
1 file, 1 thread, rnd 16k writes, simple, take 2: 294 write iops (44% HDD)
1 file, 16 threads, rnd 4k writes, posixaio: 452 write iops (128% HDD)
1 file, 16 threads, rnd 8k writes, posixaio: 454 write iops (107% HDD)
1 file, 16 threads, rnd 16k writes, posixaio: 357 write iops (79% HDD)
1 file, 16 threads, rnd 16k writes, posixaio, take 2: 318 write iops (63% HDD)
===  16 file series   ===
16 files, 1 thread each, seq 1M writes, simple: 113 write iops (75% HDD)
16 files, 1 thread each, rnd 16k writes, simple: 544 write iops (83% HDD)
16 files, 1 thread each, rnd 16k writes, simple, take 2: 428 write iops (68% HDD)
16 files, 1 thread each, rnd 16k writes, posixaio: 323 write iops (69% HDD)
16 files, 16 threads each, rnd 16k writes, posixaio: 308 write iops (57% HDD)
16 files, 16 threads each, rnd 16k writes, posixaio, take 2: 322 write iops (64% HDD)
===   O_SYNC series   ===
1 file, 1 thread, rnd 16k writes, simple, o_sync: 78 write iops (50% HDD)
1 file, 16 threads, rnd 16k writes, posixaio, o_sync: 174 write iops (72% HDD)
16 files, 1 thread each, rnd 16k writes, simple, o_sync: 472 write iops (93% HDD)
16 files, 16 threads each, rnd 16k writes, posixaio, o_sync: 298 write iops (58% HDD)
===    read series    ===
1 file, 1 thread, seq 1M reads, simple: 1541 read iops (41% HDD)
1 file, 16 threads, rnd 16k reads, posixaio: 131 read iops (76% HDD)
16 files, 1 thread each, seq 1M reads, simple: 8222 read iops (203% HDD)
16 files, 1 thread each, rnd 16k reads, posixaio: 665 read iops (141% HDD)
16 files, 16 threads each, rnd 16k reads, posixaio: 849 read iops (171% HDD)
=== native aio series ===
1 file, 16 threads, rnd 16k writes, native aio: 460 write iops (75% HDD)
16 files, 16 threads each, rnd 16k writes, native aio: 518 write iops (103% HDD)
1 file, 16 threads, rnd 16k reads, native aio: 119 read iops (90% HDD)
16 files, 16 threads each, rnd 16k reads, native aio: 757 read iops (153% HDD)
Tests complete on your.hostname @ 2018-06-17 13:15:28.
Files remain. To clean up, add argument "cleanup".
```
# FAQ

## What is "x% HDD" for?
It's a baseline comparison to give a quick indication of how well your solution compares to a "naive" non-redundant, XFS formatted 6 TB SATA HDD (HGST HUS726060ALE610) on Linux 4.9.88 mounted with options ```rw,noexec,nodev,noatime,nodiratime,largeio,inode64,nobarrier```, having write-cache enabled and I/O scheduler set to default CFQ.

## Should I run only tests I care about?
No, not in general. Selective testing is only meant for bottleneck resolving. After resolving a bottleneck, always compare full runs from before and after to identify potentially negative impacts on other tests. Also, when sharing results with others, always share full results. Trade-offs in storage tuning are totally legit, but they need to be understood and visible.

## I have insanely high values in read tests! Is something broken?
No. Read tests are expected to run into caches that cannot be flushed (which is attempted), so they are only interesting when they show low values.

## Should my storage solution always score higher than HDD?
Depends on your redundancy setup and of course your application requirements (latency and / or throughput). In RAID6 without battery-backed write-cache for example, single threaded random writes are expected to score less compared to non-redundant HDD. In RAID10 on the other hand, something may be broken when that happens. Also, with distributed filesystems, network is a natural barrier that is to be expected and cannot compete with locally attached disks in throughput (unless very expensive networking is in use).

## Why remote execution? SSH? apt-get?
So storage-tuner-benchmark can be run from a central management server and gather benchmarks from remote machines comfortably without the need to keep versions of the tool in sync. Have a new machine up and running? Just log into your management server, forward SSH agent and execute. That's the idea. You can ignore it and use "here" to execute locally.

## Why is there no O_DIRECT series?
O_DIRECT is not part of a sane overall solution. Some applications can be configured to open their files with O_DIRECT, and if they do perform better, the developers probably did not account for the drawbacks, like write durability, risking your data. On a local system, you are essentially connecting the application directly to the disk cache, with the filesystem just pointing to the sector locations. You lose write merges, cache flushes, page cache, read-ahead, multitask fairness (the OS I/O scheduler is skipped) and so on. In Linus Torvalds words:

```
The right way to do it is to just not use O_DIRECT. 

The whole notion of "direct IO" is totally braindamaged. Just say no.

	This is your brain: O
	This is your brain on O_DIRECT: .

	Any questions?

I should have fought back harder. There really is no valid reason for EVER 
using O_DIRECT. You need a buffer whatever IO you do, and it might as well 
be the page cache. There are better ways to control the page cache than 
play games and think that a page cache isn't necessary.

So don't use O_DIRECT. Use things like madvise() and posix_fadvise() 
instead. 

		Linus
```

# Test series explained
## 1 file series
```
1 file, 1 thread, seq 1M writes, simple: 191 write iops (105% HDD)
1 file, 1 thread, rnd 16k writes, simple: 409 write iops (57% HDD)
1 file, 1 thread, rnd 16k writes, simple, take 2: 294 write iops (44% HDD)
```
Working on a single file, writing in single thread, sequentially and random. Watch out for low throughput in the first test (it uses 1M blocksize so you can easily read IOPS as megabytes per second) or low random writes in the next. Take 2 repeats the same test to show variation over time.
```
1 file, 16 threads, rnd 4k writes, posixaio: 452 write iops (128% HDD)
1 file, 16 threads, rnd 8k writes, posixaio: 454 write iops (107% HDD)
1 file, 16 threads, rnd 16k writes, posixaio: 357 write iops (79% HDD)
1 file, 16 threads, rnd 16k writes, posixaio, take 2: 318 write iops (63% HDD)
```
Adding threads, solutions with multiple disks should show positive scaling effects. Starting with smallest real-life blocksize 4k and ramping up to 16k, a typical database blocksize, mostly SSDs will shine here. A take 2 for the most important test shows variation over time.

##  16 file series
```
16 files, 1 thread each, seq 1M writes, simple: 113 write iops (75% HDD)
16 files, 1 thread each, rnd 16k writes, simple: 544 write iops (83% HDD)
16 files, 1 thread each, rnd 16k writes, simple, take 2: 428 write iops (68% HDD)
16 files, 1 thread each, rnd 16k writes, posixaio: 323 write iops (69% HDD)
```
This is basically the same as 16 threads in 1 file series, but in different files. Distributed filesystems can show scaling effects here.
```
16 files, 16 threads each, rnd 16k writes, posixaio: 308 write iops (57% HDD)
16 files, 16 threads each, rnd 16k writes, posixaio, take 2: 322 write iops (64% HDD)
```
Deep queue test, with a total of 16 * 16 = 256 parallel writers. Big distributed filesystems can show-off large total IOPS here.

## O_SYNC series
```
1 file, 1 thread, rnd 16k writes, simple, o_sync: 78 write iops (50% HDD)
1 file, 16 threads, rnd 16k writes, posixaio, o_sync: 174 write iops (72% HDD)
16 files, 1 thread each, rnd 16k writes, simple, o_sync: 472 write iops (93% HDD)
16 files, 16 threads each, rnd 16k writes, posixaio, o_sync: 298 write iops (58% HDD)
```
O_SYNC puts the requirement on the storage stack that each write actually hits "stable storage". There are varying interpretations of what "stable storage" is across operating systems and how to force data onto a disk with write-cache turned on. An interesting test here is to turn off write-cache and see the numbers rise, because cache flushes interfere with parallelism.

## read series
```
1 file, 1 thread, seq 1M reads, simple: 1541 read iops (41% HDD)
1 file, 16 threads, rnd 16k reads, posixaio: 131 read iops (76% HDD)
16 files, 1 thread each, seq 1M reads, simple: 8222 read iops (203% HDD)
16 files, 1 thread each, rnd 16k reads, posixaio: 665 read iops (141% HDD)
16 files, 16 threads each, rnd 16k reads, posixaio: 849 read iops (171% HDD)
```
Reads are next to impossible to compare across storage solutions. In many cases, all attempts from fio to invalidate the cache before reading are in vain and you end up measuring RAM bandwidth. Nevertheless, for distributed filesystems, low values indicate problems.

## native aio series
```
1 file, 16 threads, rnd 16k writes, native aio: 460 write iops (75% HDD)
16 files, 16 threads each, rnd 16k writes, native aio: 518 write iops (103% HDD)
1 file, 16 threads, rnd 16k reads, native aio: 119 read iops (90% HDD)
16 files, 16 threads each, rnd 16k reads, native aio: 757 read iops (153% HDD)
```
Native asynchronous IO is the "neat thing to do" when dealing high IOPS on a single system. It is available on Linux and OpenIndiana, and pushes load into kernel space, lowering CPU usage in the demanding application.
