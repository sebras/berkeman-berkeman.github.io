---
layout: post
title:  "The Accelerator and Performance Bottlenecks on Modern Computers"
date:   2019-06-05 00:00:00
categories: performance
---


This post will discuss the performance bottlenecks of a modern
computer and explain how the Accelerator is optimized towards maximum
speed given these hardware constraints.






### What is Limiting the Execution Time?

Assume that we have a large dataset stored on disk, and that we want
to process it as fast as possible.  From a performance perspective, we
are interested in the _bottlenecks_ of the whole processing system.
From a high level perspective, it makes sense to start with these two
questions:

 - How fast can we read the data from disk?
 - how fast can we process the data, assuming it is available?

If we understand the bottlenecks, we might improve performance.  This
is the key performance design principle for the Accelerator, and in
the next sections we'll take a closer look at the problems.



#### Random Access Versus Streaming

First, we note that the disk accessing pattern matters.  There is a
significant average access time associated with accessing a random
data block on a standard rotating hard drive.  This is because we have
to wait for both the disks to rotate and the heads to move to the
position where the data we need is stored.

Hard disk access time is in the range of **10 milliseconds**.  This
implies that we can access about 100 random data chunks per second and
drive.  Clearly, this is not much.

In the future, solid state storage will be more common, but it is
still a much more expensive technology.



#### Access Times Matters

Traditional standard databases are designed for random access.  A lot
of effort has been put in to predict, cache, partition, and by other
means minimize the average access time per record.

If we just access a small random set of our data once in a while, 10
milliseconds might not matter and databases are fine.  But if we are
to process a large chunk of the data, the random access approach will
be very slow.  In order to speed it up, significant effort has to be
spent tuning the database for the particular dataset and application.
Even then, throughput will be far from its theoretical optimum.  We
believe that it is better to work on the actual problem than to spend
time and resources tuning particular software systems.

If we are to process a large portion of the dataset, it is much more
efficient to stream all the data and filter out and work on the
relevant parts only.  If streaming is, say 1000 times faster than
random access, streaming is better if we process as little as 0.1% of
the data or more!



### Disk Throughput:  How Fast can we Read from Disk?

The minimalistic way to read data very fast from disk is like this:
```bash
  cat file > /dev/null
```

This command will, as fast as possible and with minimal overhead, read
data from disk and discard it.  There is no faster way to read data
from disk, and if the CPU is fast enough, **this will saturate the
disk to memory bandwidth**.

On a "standard" computer, we reach a few 100MB/s throughput.



#### Maximizing Throughput

With the disk bandwidth saturated, we can transfer more useful
_information_ per second by compressing the data on disk:
```bash
  zcat file.gz > /dev/null
```

Decompression is a lightweight operation on a modern CPU.  **The
compression typically increases disk bandwidth by 5 to 10 times** on a
realistic dataset, so this is a favorable trade-off.

When storing data in a compressed format, we can stream in the
ballpark of 1000MB/s from disk to CPU.



#### Can we go Faster?

Well, as soon as we start to process the data, the CPU load will go
up, and we realize that processing speed of the CPU will be the new
bottleneck.  We have turned a data-intensive problem into a
processing-intensive one.

Fortunately, modern computers comes with several CPU
cores, so we can read and decompress several files in parallel,
thereby increasing the computational power per data item while
maintaining the high disk to memory bandwidth:

```bash
  zcat file1.gz | processing_program &
  zcat file2.gz | processing_program &
  ...
```

It is not possible to do faster, but please read on.



#### RAM is Faster than Disk

It makes sense to have a computer with lots of RAM in it.  Operating
systems such as Linux or FreeBSD use unallocated RAM for disk buffers
/ cache, which means that when a file is read from or written to disk,
it also stays in RAM to allow for much faster access should it be
accessed again.

As for disks, RAM does not get much faster over time, but it gets
cheaper, i.e. you get more for the same price.  It is unnecessarily
old-fashioned to use machines with little RAM.  AWS EC2 instances with
[4TB
RAM](https://aws.amazon.com/blogs/aws/now-available-ec2-instances-with-4-tb-of-memory/)
and even [12TB
RAM](https://aws.amazon.com/blogs/aws/now-available-amazon-ec2-high-memory-instances-with-6-9-and-12-tb-of-memory-perfect-for-sap-hana/)(!).
are available just a few mouse clicks away.  In general, more RAM is
better, but even a small increase in RAM may improve performance.

The nice thing is that more RAM may increase performance without any
extra work (due to the disk buffering).  In addition, data intensive
algorithms such as collaborative filtering execute much much faster if
the source data fits into RAM.





### The Accelerator Approach

The important tricks that we've just seen are used by the Accelerator to
maximize data read and write performance, in particular:

  - Data is stored in a compressed format, and
  - Data is split into several files, _slices_, and processed in
    parallel.

In addition, the Accelerator

  - splits data into one file _per column_ and _slice_, and
  - spreads data into _independent_ slices using a hashing algorithm.



#### One File Per Column

A dataset may typically have several columns containing different
information.  Each processing/analysis task typically uses only a few
of them simultaneously.

Therefore it makes sense to separate the columns into independent
files, so that the read bandwidth is only occupied with data that is
_necessary_ for the processing task at hand.



#### Data Partitioning using a Hash Function
Using a hash function to determine which data rows that goes into
which slices is a simple way to open up for entirely independent
parallel processing.

If we hash our dataset on the `user` variable, for example, we will
have data from each particular user in only one slice, meaning the we
will never have to merge any results between slices/CPUs.

This is the opposite of _Map-Reduce_, where we _partition by_hash
before processing_ instead of _merging everything after processing_.
Hashing a dataset also happens to be a very fast operation.



### What About Networking?

The standard network connection today delivers about 100MB/s, which is
a bit less than what we can get from a single hard drive.  There are
much faster networking technology available, but it is expensive.
Before partitioning a problem onto several computers, one have to do
the math to see where the break-even point is: how many networked
computers is needed to outperform a single powerful machine?  And then,
one has to consider the issues that comes with cluster computing, like
administration (orchestration?) and availability (redundancy at all
levels).

One of the reasons that The Accelerator was designed using a
client-server solution was that it should be able to run in parallel
on several machines.  We have worked on several large projects, with
tens of billions of records, but never seen the need to go beyond one
computer.  We believe that it is better to support software functions
that are actually used rather than including lots of features that are
not so well covered, so for that reason the Accelerator does not come
with multi machine support.


### Summary

The Accelerator is using low level basic techniques such as
compression, slicing, partitioning and parallel execution to mitigate
the computer's bottlenecks and maximize performance.
