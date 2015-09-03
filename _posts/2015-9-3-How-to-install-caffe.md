---
layout: post
title: How to install caffe on ubuntu 14.04(without GPU)
published: true
---




I've been playing with caffe for a while. Installing it on unbuntu is easy thanks to the apt-get command. However, due to my bad memory, I'd better write down the steps as well as problems I encountered while installing it. 
## Steps:
**1. Get ubuntu.**
I've been playing with CentOS at work and Ubuntu at home. From my personal experience, ubuntu seems to offer more on-shelf softwares than CentOS. So it might be a little nicer for newbees like me.

**2. Installing dependencies.** 
The following commands are used to install [protobuf](https://developers.google.com/protocol-buffers/docs/overview), [leveldb](http://leveldb.org/), [snappy](https://github.com/google/snappy), [opencv](http://opencv.org/), [hdf5](https://www.hdfgroup.org/HDF5/), protobuf compiler and [boost](http://www.boost.org/):

        sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler

        sudo apt-get install --no-install-recommends libboost-all-dev

Keep going to install dependencies, the commands below install [gflags](https://github.com/gflags/gflags), [glogs](https://code.google.com/p/google-glog/) ,[lmdb](https://lmdb.readthedocs.org/en/release/) and [atlas](http://math-atlas.sourceforge.net/).

        sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
        sudo apt-get install libatlas-base-dev

This is why I like linux more than windows or mac os.

**3. Download Caffe**
Git might be the easiest way to download caffe. Of course you will first apt-get install git and then clone caffe to your current folder.

        git clone git://github.com/BVLC/caffe.git

**4. Compile Caffe**
Compiling caffe is like walking on the shells because this is where I encountered most of the errors. To compile caffe, first use cmd to enter the caffe directory. For me, I put it in the Documents/caffe directory:

        cd /Documents/caffe/

caffe gives as an example of Makefile in Makefile.config.example, we need to copy this file to Makefile.config and modify some configurations according to my machine and system. 

        cp Makefile.config.example Makefile.config

Since I don't have a Navidia GPU so I will only use the CPU mode to compile and run caffe. Thus I will uncomment the # CPU_ONLY := 1 line, this is the only change I made to the Makefile.config file. When I worked on CentOS, I used Openblas rather then atlas, so I also pointed out where to find Openblas in the Makefile.config file. But with ubuntu, all paths of the dependencies are handled by the apt-get, so I only need to change one line in Makefile.config. 

        ~~# CPU_ONLY := 1~~

        CPU_ONLY := 1

But if the make command tells you that it can't find some dependencies, a convinient(maybe not best) way is to find out where dependencies are and add their paths to the INCLUDE_DIRS and LIBRARY_DIRS, I used this to solve a lot of missing dependencies on CentOS.

        # Whatever else you find you need goes here.
        INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include
        LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib

Next, we come the exciting part, compiling caffe, just enter make all to compile it:
        make all

If you are lucky and get no errors, we can make the tests:
        make test

Again, if no errors pop out, we run the test to see if everything is installed correctly:
        make runtest

If you saw the image below, congratulations! You have caffe installed properly!

![runtest-success][1]
[1]: https://raw.githubusercontent.com/sunshineatnoon/sunshineatnoon.github.io/master/images/runtest-success.png
