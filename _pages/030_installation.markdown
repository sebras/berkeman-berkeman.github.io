---
layout: page
title: Installation
permalink: /installation/
---


### Build and Runtime Environment

The Accelerator is built, tested, and runs on:

 - Ubuntu18.04 and 16.04,
 - Debian 9,
 - FreeBSD 11.1

but is in no way limited to these systems or versions.


### Installation

We are working on a PyPi installation, but for now, please use this
two step guide to install the Accelerator using git:

1.  Install dependencies. On Debian and Ubuntu
```bash
  sudo apt-get install build-essential python-dev python3-dev zlib1g-dev git virtualenv
```

2.  Clone the [https://github.com/eBay/accelerator-project_skeleton](https://github.com/eBay/accelerator-project_skeleton) repository.  
Run the setup script
```bash
cd accelerator-project_skeleton
  ./init.py
```
Please read and modify this script according to your needs.

Done. The Accelerator is now ready for use.

For a more comprehensive installation guide, please see the [skeleton
installation manual](https://berkeman.github.io/pdf/acc_install.pdf).

The init.py script will clone both the accelerator-gzutil and the main
accelerator repositories. The gzutil library will be set up in virtual
environments for Python2 as well as Python3, and the Accelerator will
be set up as a git submodule to the `project_skeleton` repository.
