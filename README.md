# Storage Tuner Benchmark
Quick but meaningful storage benchmark utility

# Introduction
Performance measurements on storage systems or even local disks are difficult to "get right". More than often blog or forum posts, news articles and other sources of information will quote flawed tests or incomparable numbers.

Storage Tuner Benchmark aims to change that by being easy to use (no config) and covering a good range of application needs in a well defined series of "Tests". It uses the excellent "fio" as a backend and will likely be extended in the future to cover more scenarios by adding more tests and eventually forming test groups. Individual tests will stay unchanged for as long as possible to stay comparable.

# Requirements
 - Linux (tested on Debian first)
 - Bash
 - Fio (auto-installed with apt-get or yum when not present)
 - 1GB of free disk space for testing
 - Optional SSH for remote execution

It should be easy to adapt to other \*nix, and possible to adapt to Windows as well. Patches are welcome.

# Installation
Download storage-tuner-benchmark directly from Github and mark executable:
```bash
wget https://raw.githubusercontent.com/Korkman/storage-tuner-benchmark/master/storage-tuner-benchmark
chmod a+x storage-tuner-benchmark
```
Install fio (apt-get/yum install fio) or execute storage-tuner-benchmark with root privileges to have it installed automatically.

# Usage
```
storage-tuner-benchmark 1.0.0

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
# Examples
```
./storage-tuner-benchmark here /mnt
Creates "stb-testdir" in /mnt and runs all tests.

./storage-tuner-benchmark here /mnt "Many files seq write"
Repeats the test named "Many files seq write".

./storage-tuner-benchmark here /mnt cleanup
Will rm -rf /mnt/stb-testdir
```
# Results example
Results are boiled down to essential, human readable information. Color is supported and disabled when redirected.
```
yourhost  2018-06-01 13:19:21  Running tests in "/mnt/stb-testdir" ...
yourhost  2018-06-01 13:19:21  storage-tuner-benchmark version 1.0
yourhost  2018-06-01 13:19:21  Test "Many files seq write"
yourhost  2018-06-01 13:19:26  write: io=524288KB, bw=107041KB/s, iops=26, runt=  4898msec
yourhost  2018-06-01 13:19:26  Test "Many files seq read direct"
yourhost  2018-06-01 13:19:43  read : io=22684MB, bw=1382.9MB/s, iops=345, runt= 16404msec
yourhost  2018-06-01 13:19:43  Test "Many files rnd 4k read direct"
yourhost  2018-06-01 13:19:49  read : io=1518.1MB, bw=310643KB/s, iops=77660, runt=  5007msec
yourhost  2018-06-01 13:19:49  Test "Many files rnd 16k read direct"
yourhost  2018-06-01 13:19:54  read : io=5703.2MB, bw=1139.3MB/s, iops=72912, runt=  5006msec
yourhost  2018-06-01 13:19:54  Test "Many files rnd 4k write direct"
yourhost  2018-06-01 13:20:00  write: io=31680KB, bw=6308.3KB/s, iops=1577, runt=  5022msec
...
yourhost  2018-06-01 13:21:02  Test "Single file and thread rnd 16k readwrite direct"
yourhost  2018-06-01 13:21:07  read : io=6400.0KB, bw=1279.3KB/s, iops=79, runt=  5003msec
yourhost  2018-06-01 13:21:07  write: io=5808.0KB, bw=1160.1KB/s, iops=72, runt=  5003msec
yourhost  2018-06-01 13:21:07  Tests complete.
```
# Tests and their meaning
They should be self explanatory to you :-) Detailed description and interesting numbers for comparison will be added soon.
