---
layout: post
title:  "Accelerator Installation"
date:   2019-10-30 00:00:00
categories: example
---


This HOWTO shows how to install and test the Accelerator.

#### [TL;DR](https://en.wikipedia.org/wiki/Wikipedia:Too_long;_didn%27t_read)
```sh
# install
pip install accelerator
# setup new project
mkdir myproject
cd myproject
ax init
# check/modify the "accelerator.conf" file according to your needs
ax server &
ax run tests
```



### Install Using pip

Using `pip`, the Accelerator is installed just like any other package.
Installation can be global, local, or in a virtual environment.
Here we show how to set up using a virtual environment.

We create a virtual environment using Python's `venv` package

```shell
python3 -m venv accvenv
```

and activate it like this

```shell
source accvenv/bin/activate
```

Then we install the Accelerator

```shell
pip install accelerator
```

This command will download, compile, and install the Accelerator to
the active virtual environment, `accvenv`.



### Set up a Simple Project

Now that the Accelerator is installed, let's create a simple project.
Each project should have its own directory, so we use `mkdir` to
create a new directory, and then we `cd` into it

```shell
mkdir myproject
cd myproject
```
and run the Accelerator's init command
```shell
ax init
```

The `init` command will set up everything needed to work on the new
project.  **At this point we are done**, but please read on to learn
more about the project setup, how to tweak the configuration file, and
how to run the built-in tests.



### What's in the Project Directory

If we do a `ls -F` in the `myproject` directory, we'll find the
following files and directories that are of key importance

```text
accelerator.conf         # The project's configuration file
dev/                     # The project's source code goes here
workdirs/                # All jobs and corresponding output is written here
```
and these that are optional
```text
results/                 # Optional central place for computed results
urd.db/                  # Optional urd database directory
```

The file `accelerator.conf` contains the project's configuration.
Accelerator commands look for this file in the current directory or
directories directly above the current work directory, so it is
required to have the _current working directory_ inside the project
directory when running any command.



#### The Configuration File

The configuration file comes with pretty detailed inline documentation
that we've chosen to not show here for brevity reasons.  Instead,
we'll walk through the file and comment the things that are important
at this stage.


##### slices

First, the file specifies the number of _slices_.  This number
stipulates how many processes the Accelerator will run in parallel.
The variable is initiated to the number of available CPU cores
(including threads):

```text
slices: 8
```


##### workdirs

```text
workdirs:
        dev /home/ab/myproject/workdirs/dev
```

A workdir is where jobs and their output is stored.  Workdirs can be
located anywhere in the file system, and they can be shared between
users.

Any number of workdirs can be specified in the configuration file.  In
this case a singe workdir `dev` is defined.  Note that it is possible
to use shell environment variables, such as `${HOME}`, in the
definitions.



##### target workdir

```text
target workdir: dev
```

The `target workdir` specifies in which workdir jobs are stored by
default.  It is not mandatory, but it is good practice to use it.


##### method packages

```text
method packages:
        dev
        accelerator.standard_methods
        accelerator.test_methods
```
This is a declaration of which
[packages](https://docs.python.org/3/tutorial/modules.html#packages)
that should be accessible in the project:

Three packages are defined here, first the `dev` package (which
corresponds to our recently created `myproject/dev` directory), and
then two packages from the Accelerator installation:
`standard_methods` and `test_methods`.


##### result and input directories
```text
result directory: /home/ab/myproject/results
input directory: # /some/path where you want import methods to look.
```

This specifies the `results` and `source` directories.

 - The `result directory` is a directory where the user may chose to
store important output files for convenient access.

 - The `input directory` is the directory where data files to be
   _imported_ by the project are stored.  Import methods look here by
   default.



### Starting the Server Process

When we are satisfied with the configuration file, it is time to start
the Accelerator server process.  The Accelerator is based on a
"client-server" architecture, meaning that there is a server process
that is constantly running, and we issue commands to it in order to
execute programs.

Make sure the virtual environment is active, and do

```shell
ax server
```

to start the server.  It is highly recommended to keep the server
running in a separate terminal, so when issuing commands to it, we do
that from a new terminal.



### Running the Built-in Tests

If you have not done so already, please launch a new terminal ([gnu
screen](https://www.gnu.org/software/screen/) or
[tmux](https://github.com/tmux/tmux/wiki) is suggested), initiate the
virtual environment and `cd` to the project's directory:

```shell
source accvenv/bin/activate
cd myproject
```

and run the built-in tests like this

```shell
ax run tests
```

The built-in tests reside in the `accelerator.test_methods` package.
The `tests` script will perform a large number of tests.  Among them,
a number of tests relate to character encodings, since the Accelerator
is designed to handle in practice any character encoding scheme.  The
tests _may_ print a warning message if there is insufficient support
of _locales_ installed on the system.  If this is important for the
application, please read the message for information on how to fix
this and make sure all character encoding tests are run successfully.



### Running Arbitrary programs:  The dev package

New project code is put in the `dev/` directory.  The `init` command
has already put a minimal example there that can be examined and
executed directly.  Here is the contents of the `dev/` directory as created by the `init` command

```text
dev/
    __init__.py
    a_example.py
    build.py
    methods.conf
```

The `dev/` directory is actually a Python package, that and thus it
contains an `__init__.py` file.

The directory contains one example method `a_example.py` and one
example build script `build.py`.  It also contains the mandatory
`methods.conf` file which specifies which methods in the package that
should be executable.  In this case it c contains the single line

```text
example
```

indicating that `a_example.py` is indeed executable in the current
project.

In order to run the example, type

```shell
ax run
```



### Install from GitHub

It is also possible to install directly from the git repository.

Clone the repository
```sh
git clone https://github.com/eBay/accelerator
```
and install
```sh
cd accelerator
./setup.py build
./setup.py install
```

On a clean Debian-based system, install dependencies using
```shell
apt-get install python3-ujson python3-cffi 
```
