# Storage Tuner Benchmark
Quick but meaningful storage benchmark utility

## Introduction
Performance measurements on storage systems or even local disks are difficult to "get right". More than often blog or forum posts, news articles and other sources of information will quote flawed tests or incomparable numbers.

Storage Tuner Benchmark aims to change that by being easy to use (no config) and covering a good range of application needs in a well defined series of "Tests". It uses the excellent "fio" as a backend and will likely be extended in the future to cover more scenarios by adding more tests and eventually forming test groups. Individual tests will stay unchanged for as long as possible to stay comparable.

## Requirements
 - Linux (tested on Debian first)
 - Bash
 - Fio (auto-installed with apt-get or yum when not present)
 - 1GB of free disk space for testing
 - Optional SSH for remote execution

It should be easy to adapt to other \*nix, and possible to adapt to Windows as well. Patches are welcome.

## Installation
Download storage-tuner-benchmark directly from Github and mark executable:
```bash
wget https://raw.githubusercontent.com/Korkman/storage-tuner-benchmark/master/storage-tuner-benchmark
chmod a+x storage-tuner-benchmark
```
Install fio (apt-get/yum install fio) or execute storage-tuner-benchmark with root privileges to have it installed automatically.

## Usage
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
## Examples
```
./storage-tuner-benchmark here /mnt
Creates "stb-testdir" in /mnt and runs all tests.

./storage-tuner-benchmark here /mnt "Many files seq write"
Repeats the test named "Many files seq write".

./storage-tuner-benchmark here /mnt cleanup
Will rm -rf /mnt/stb-testdir
```
## Results example
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
## Should I run only tests I care about?
No, not in general. Selective testing is only meant for bottleneck resolving. After resolving a bottleneck, always compare full runs from before and after to identify potentially negative impacts on other tests. Also, when sharing results with others, always share full results. Trade-offs in storage tuning are totally legit, but they need to be understood and visible.

## Tests and their meaning
All tests share the following fio flags:
```
--fallocate=none --group_reporting --verify=0 --disable_lat=1 --disable_clat=1 --disable_slat=1 --clat_percentiles=0 --refill_buffers --randrepeat=0
```
They disable some unused statistics, prevent file allocation at startup and set fio up to generate incompressible random data. 

All but the initialization tests are time limited to 5 seconds.
```
--runtime=5 --time_based
```
None of those tests are meant to write or read more data than fits into memory or other caches, but instead give a quick indication whether things are cached or not and if not, how well the storage system performs.

## Many files

All "Many files" tests share the following fio flags:
```
--size=32M --numjobs=16
```
They define "many" as 16, with 32 MB each. The goal here is not to thrash the filesystem with a huge amount of files, but to show scaling effects when compared to single file tests.

### Test "Many files seq write"
```
... --ioengine=psync  --iodepth=1 --direct=0 --rw=write --end_fsync=1 --bs=4M
```
Writes a set of files in parallel using multiple processes and large blocksize, performing an fsync at the end. No fancy IO engine. This is mainly to initialize the files but also gives an idea how fast files can be written on the filesystem.

### Test "Many files seq read direct"
```
... --ioengine=libaio --iodepth=1 --direct=1 --rw=read --bs=4M
```
Reads back the set of files in parallel using multiple processes and large blocksize, accessing them in direct mode to work around operating system cache. Note that in some cases direct mode is still accessing cache (think FUSE), so if it is too fast to be true, it probably is.

### Test "Many files rnd 4k read direct"
```
... --ioengine=libaio --iodepth=1 --direct=1 --rw=randread --bs=4k
```
Just like "Many files seq read direct", but with 4k block size. 4k block size is a worst-case scenario for hard drives and many other system components as well. Again, this is in direct mode, which should work around the OS cache. It won't make hard drive caches disappear, let alone RAID controller caches or ZFS ARC.

### Test "Many files rnd 16k read direct"
```
... --ioengine=libaio --iodepth=1 --direct=1 --rw=randread --bs=16k
```
The same as "Many files rnd 4k read direct" with a more real-world block size encountered when working with databases.

### Test "Many files rnd 4k write direct"
```
... --ioengine=libaio --iodepth=1 --direct=1 --fdatasync=1 --rw=randwrite --bs=4k
```
Another worst case scenario, writing the dreaded 4k blocks in 16 processes and most importantly: fdatasync'ing every write. This will push the write to "stable storage". For most users that would be the actual hard disk drive if all caches flush properly and no accelerator sits in-between.

### Test "Many files rnd 16k write direct"
```
... --ioengine=libaio --iodepth=1 --direct=1 --fdatasync=1 --rw=randwrite --bs=16k
```
Just like "Many files rnd 4k write direct" with a more realistic 16k block size. This one matters the most for databases, and will limit their performance when they run out of buffering log space (which is sequentially written, see innodb log size).

## Single file tests

All "Single file" tests share the following in addition to the globally shared flags:
```
... --size=512M --numjobs=1
```
Same total size as "Many files" series, but in one file.

### Test "Single file and thread seq write"
```
... --ioengine=psync    --iodepth=1  --direct=0 --rw=write --end_fsync=1 --bs=4M
```
This test will initialize the file for the "Single file" series and show sequential write performance for a single writer. This is important for copying large files, journalling, database transaction logs ...

### Test "Single file and thread seq read direct"
```
... --ioengine=libaio   --iodepth=1  --direct=1 --rw=read  --bs=4M
```
Reading back the just written file sequentially in direct mode, trying to work around OS cache. Same limitations as with "Many files seq read direct".

### Test "Single file rnd 16k write"
```
... --ioengine=psync    --iodepth=16 --direct=0 --fdatasync=1 --rw=randwrite --bs=16k
```
Database typical writes in 16 threads NOT using direct mode. Similar to having a database inside a VM guest with qemu set to writeback/threads.

### Test "Single file rnd 16k write direct"
```
... --ioengine=libaio   --iodepth=16 --direct=1 --fdatasync=1 --rw=randwrite --bs=16k
```
Database typical writes in 16 threads using direct mode. Similar to having a database inside a VM guest with qemu set to native/none.

### Test "Single file rnd 16k readwrite direct"
```
... --ioengine=libaio   --iodepth=16 --direct=1 --fdatasync=1 --rw=randrw    --bs=16k
```
Mixing reads and writes for a more realistic database load.

### Test "Single file rnd 16k write mmap"
```
... --ioengine=mmap     --iodepth=16 --direct=1 --fdatasync=1 --rw=randwrite --bs=16k
```
Some databases use mmap for IO. So this switches the engine to mmap. Otherwise same as "Single file rnd 16k write direct"

### Test "Single file rnd 16k write posixaio"
```
... --ioengine=posixaio --iodepth=16 --direct=1 --fdatasync=1 --rw=randwrite --bs=16k
```
And another clone of "Single file rnd 16k write direct" with posixaio as engine.

### Test "Single file and thread rnd 16k write"
```
... --ioengine=psync   --iodepth=1 --direct=0 --fdatasync=1 --rw=randwrite --bs=16k
```
Again 16k random write IOPS, but now single-threaded and not in direct mode. Performance bottleneck for databases with few users, filesystem metadata heavy operations.

### Test "Single file and thread rnd 16k write direct"
```
... --ioengine=libaio   --iodepth=1 --direct=1 --fdatasync=1 --rw=randwrite --bs=16k
```
Again 16k random write IOPS, single-threaded and in direct mode. Performance bottleneck for databases with few users, filesystem metadata heavy operations.

### Test "Single file and thread rnd 16k readwrite direct"
```
... --ioengine=libaio   --iodepth=1 --direct=1 --fdatasync=1 --rw=randrw    --bs=16k
```
Mixing in reads with the single threaded database load to get a more realistic picture.
