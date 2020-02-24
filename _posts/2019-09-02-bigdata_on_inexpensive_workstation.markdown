---
layout: post
title:  "Processing a Billion Row Dataset on an Inexpensive Workstation"
date:   2019-09-02 00:00:00
categories: performance
---

It is interesting to consider the problem of processing large datasets
given limited resources such as time and money.  How to approach the
problem?  What is needed in terms of hardware?  Is it expensive?  Is
it slow?  In this post we will show how to do fast and reliable
parallel computing of on a less-than-$1000 machine.

We will show that, using simple examples with correspondingly simple
Python programs, it is possible to work in an agile way with datasets
exceeding one billion lines.  Two examples (details will follow):

  1. String matching.  Our simple example can process a 100 billion
     (100.000.000.000) line dataset in about 20 minutes!  All in
     Python, using a program that is only thirteen non-blank lines, on
     a $1000 computer.

  2. Creating histograms of Gaussian distributed 64-bit floats runs at
     more than 20 million lines per second.  We can process a billion
     line dataset in 45 seconds!


We process data using the Accelerator, a tool released by eBay, that
has its origins in a startup company running large data driven
projects with limited resources.  The Accelerator makes efficient use
of available hardware, and many applications will therefore perform
well on a standard laptop.  A more powerful computer will execute
programs faster and shorten design iteration time.



### The Accelerator Framework

The Accelerator is a minimalistic framework with a small footprint
that runs on anything from 32-bit Raspberry Pies to 64 bit multi core
rack servers.  One of many features is that it provides simple
parallel processing that makes good use of modern multi core computer
hardware.  The Accelerator is programmed in Python, making is easy to
use and get familiar with.

The Accelerator has been used in many projects, including being
back-end for recommendation systems running live to millions of
customers, since 2012.




### Hardware for the Accelerator

What is a good hardware platform for data processing?  As mentioned
earlier, a laptop may do, but if we are free to choose our computer,
which parameters are most important?

- More cores is better.  Many data processing and analysis
applications can be parallelised.  Therefore, performance improve with
the number of available CPU cores.

- Plenty of RAM is beneficial for two main reasons.  First,
unallocated memory is used for disk buffering of intermediate and
previous results, which results in a general speedup.  Second, more
RAM makes it possible to solve larger problems directly in memory, and
RAM is significantly faster than any disk.

- Large disks are needed to work on large datasets.

We believe that memories with error detection and correction (ECC) and
file systems with integrity checking and redundancy (such as zfs) are
necessary to fulfill reproducibility requirements.  If we just want to
play around with a dataset, this is not the first priority, but as
soon as we start doing anything more serious, our results depend on
reproducible computing.  Data is a valuable resource, and any error in
the data should at least be detected (and preferably corrected).



### An Example:  Recycling a Lenovo D20 Workstation from 2011

This machine is cheap, reliable, and powerful, and actually compares
well to modern machines, which we will show in a future post.  We
bought our machine from an Internet auction site in order to try it
out.  The machine is actually from **2009**, but the CPUs in our
example were manufactured in 2011.


<p align="center"><img src="{{ site.url }}/assets/d20-eol.jpg" height="300"> </p>
<p align="right"> (Image from lenovo.com) </p>

The workstation is equipped two CPUs providing in total 12 cores (24
"threads"), has 108GB RAM, and 2TB solid state disk.  Approximate
price for a machine like this is below $1000.

```
Details:
  CPU:   12 cores, 2 x Intel Xeon X5675 @ 3.07GHz
  RAM:   DDR3, 108GB
  DISK:  SSD, 2*960GB
```

Despite its age, this machine outperforms modern laptops, mainly
because it has more cores, less problem with heat dissipation, and
three independent memory channels per CPU chip that speeds up memory
accesses.

Clearly, this machine, in its current configuration, has too little
disk for data heavy applications, but there is room and connectors for
eight drives in the box.  We have measured the disk controllers to
transfer data at a sustained rate of 265MB/s between memory any of the
SSDs.  (Using more SSDs in zfs raidz configuration, we have measured
continuous writes exceeding 1.2GB/s.)



### Performance Testing

The Accelerator installation repository comes with a simple
performance testing script that we have used for the testing.  The
details can be found on page 3 of [this
document](https://berkeman.github.io/pdf/acc_install.pdf), and the
code is found on
[github.com/eBay](https://github.com/eBay/accelerator-project_skeleton/tree/master/example_perf).
The test will create a dataset of **one billion lines**, and is
composed of several columns with random data of different types.  This
dataset will be read, written, and processed in different ways while
execution time is measured.

The size of the one billion line dataset is 79 GiB (36 GiB compressed).


### Test Results

Here is the complete output from the build script.

```
      operation                       exec time         rows/s

1.    csvexport                         686.775      1,456,081

2.    reimport total                    912.549      1,095,832
         csvimport                      591.277      1,691,256
         type                           321.272      3,112,623

3.    sum
        small number                      4.192    238,556,448
        small integer                     3.156    316,887,361
        large number                     10.907     91,688,275
        gauss number                      5.519    181,178,210
        gauss float                       4.345    230,175,033

4.    sum positive
        small number                     11.814     84,643,880
        small integer                    10.683     93,603,000
        large number                     18.803     53,181,776
        gauss number                     14.636     68,322,432
        gauss float                      13.615     73,450,887

5.    histogram
        number                           45.894     21,789,231
        float                            44.249     22,599,181

6.    find string                        12.304     81,275,676

7.    Total running time               3235.222
      Total test time                  1112.67

Example size is 1,000,000,000 lines.
Number of slices is 23.
```

The majority of the execution time is spent in generating the
synthetic dataset CVS file that is later used for testing, while 1113
seconds (less than 20 minutes) are used to import the dataset and
perform the actual tests.  During the tests, the one billion dataset
is read and processed in various ways multiple times.  Total execution
time, including data generation, is less than one hour (3235 seconds).
Let us take a closer look at what happens during testing:


#### 1. csvexport

The script starts by creating a one-billion line dataset in the
Accelerator's internal dataset format.  All this data is then fed to
the `csvexport` method that writes it to a compressed CSV file on
disk.  Generating this file takes about 11 minutes (687 seconds), and
runs at a data rate of almost 1.5 million rows per second, or 115
MB/s.  The resulting output is a 36GiB compressed CSV file.


#### 2. import and typing

The CSV file is then imported again using the `csvimport` method,
followed by typing of the data using `dataset_type`.  Importing runs
at 1.7 million rows per second while typing runs at 3.1 million rows
per second.  (The typing jobs continuously process about 250MB/s.)
The time spent importing and typing the data corresponds approximately
to a coffee-break.  To be specific, it takes **15 minutes (913
seconds)** to import and type the complete dataset.


#### 3. sum

When imported, the dataset is ready for calculations.  The testing
begins with adding all values in single columns together.  The test
result highlights the difference in speed between different datatypes.
For example, **the Accelerator adds 230 million Gaussian distributed
64-bit floats per second**.  The "small integer" dataset has less
entropy and therefore runs slightly faster, 317 million rows per
second.  Note, these numbers are for a program written in Python
operating on data on disk!

There are four main reasons for this high performance.

- The Accelerator only reads the columns needed for the current work,
and not all the data.

- Data is compressed on disk, which makes it possible to "go beyond
the IO bandwidth" between disk and memory. 317 million 64-bit floats
per second corresponds to a data rate of **2.5GB/s**, which is far
above the disk to memory transfer rate.

- The Python programs run in parallel on all available CPU cores.
(Actually, in this example, on 23 of the 24 available CPU "threads".)

- Data is streamed continuously, rather than accessed randomly, from
  disk to CPU cores.  Such a deterministic access pattern takes
  advantage of cache prefetching and minimises seek times.


#### 4. sum positive

This test adds a data dependency.  Numbers are added only if they are
positive.  These tests are significantly slower, but still they
consume data at rates between 50-100 million lines per second.


#### 5. histogram

This part uses the Python
[Counter](https://docs.python.org/3/library/collections.html) class to
store distributions of each data column.  Although it uses a very
high-level language construct, it runs at more than 20 million rows
per second, **creating a histogram of a billion 64-bit floats in about
45 seconds!**


#### 6. find string

Check for existence of a four character string in a billion random
strings of length 10.  This runs at a rate **exceeding 80 million rows
per second** (i.e. 800MB/s).  (The number of strings found is less
than one in a million, which is in line with theory.)


#### 7. Total Test Time

Assuming the input data was available on disk in a CSV-like format,
**importing the data and running all the tests took less than 20
minutes**, where each test took between 3 and 46 seconds.  Total
execution time, including generating the one billion line dataset, was
close to one hour.  On a faster machine, execution times will be even
lower.



### Conclusion

The Accelerator can handle large datasets at high speed on inexpensive
hardware.  Being able to create a histogram of a billion values in
less than a minute (45 seconds) is a great advantage in any data
science application.  (Full disclosure: an import of the complete
dataset takes 15 minutes, and this has to be carried out first, but
only once.)  For reasons like this, the Accelerator is an ideal
framework for a lot of data intensive applications.
