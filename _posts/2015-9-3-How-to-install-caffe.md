---
layout: post
title: How to install caffe on ubuntu 14.04(without GPU) and python interface
published: true
---




I've been playing with caffe for a while. Installing it on unbuntu is easy thanks to the apt-get command. However, due to my bad memory, I'd better write down the steps as well as problems I encountered while installing it. 

## Get ubuntu.

I've been playing with CentOS at work and Ubuntu at home. From my personal experience, ubuntu seems to offer more on-shelf softwares than CentOS. So it might be a little nicer for newbees like me.

## Installing dependencies.

1.  The following commands are used to install [protobuf](https://developers.google.com/protocol-buffers/docs/overview), [leveldb](http://leveldb.org/), [snappy](https://github.com/google/snappy), [opencv](http://opencv.org/), [hdf5](https://www.hdfgroup.org/HDF5/), protobuf compiler and [boost](http://www.boost.org/):

        sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler

        sudo apt-get install --no-install-recommends libboost-all-dev

2.  Keep going to install dependencies, the commands below install [gflags](https://github.com/gflags/gflags), [glogs](https://code.google.com/p/google-glog/) ,[lmdb](https://lmdb.readthedocs.org/en/release/) and [atlas](http://math-atlas.sourceforge.net/).

        sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
        sudo apt-get install libatlas-base-dev

This is why I like linux more than windows or mac os.

## Download Caffe

Git might be the easiest way to download caffe. Of course you will first apt-get install git and then clone caffe to your current folder.

        git clone git://github.com/BVLC/caffe.git

## Compile Caffe

1.  Compiling caffe is like walking on the shells because this is where I encountered most of the errors. To compile caffe, first use cmd to enter the caffe directory. For me, I put it in the Documents/caffe directory:

        cd /Documents/caffe/

2.  caffe gives as an example of Makefile in Makefile.config.example, we need to copy this file to Makefile.config and modify some configurations according to my machine and system. 

        cp Makefile.config.example Makefile.config

3.  Since I don't have a Navidia GPU so I will only use the CPU mode to compile and run caffe. Thus I will uncomment the # CPU_ONLY := 1 line, this is the only change I made to the Makefile.config file. When I worked on CentOS, I used Openblas rather then atlas, so I also pointed out where to find Openblas in the Makefile.config file. But with ubuntu, all paths of the dependencies are handled by the apt-get, so I only need to change one line in Makefile.config. 

        CPU_ONLY := 1

    But if the make command tells you that it can't find some dependencies, a convinient (maybe not best) way is to find out where dependencies are and add their paths to the INCLUDE_DIRS and LIBRARY_DIRS, I used this to solve a lot of missing dependencies complaints on CentOS.

        # Whatever else you find you need goes here.
        INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include
        LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib

4.  Next, we come the exciting part, compiling caffe, just enter make all to compile it:

        make all

5.  If you are lucky and get no errors, we can make the tests:

        make test

6.  Again, if no errors pop out, we run the test to see if everything is installed correctly:

        make runtest

7.  If you saw the image below, congratulations! You have caffe installed properly!

![runtest-success][1]

## Compile python interface for caffe

Python is easy and convinient, especially when you want to use caffe for some projects. In order to use the python interface of caffe, we need to compile it. All the commands below are supposed to be done in the caffe root directory.

1.  First make sure you get python in your system. By simply running the command python, you can check if you have python installed.
2.  Next, we need to install [Pip][3]. Pip is a package management system used to install and manage software packages written in Python. On ubuntu, just use apt-get to install it and <b>update it</b>:
   
        sudo apt-get install python-pip python-dev build-essential 
        sudo pip install --upgrade pip
3.  Then we can use pip to install all the essential requirements in order to compile the python interface:
   
        sudo pip install -r ./python/requirements.txt
    When taking a further look into the requirements.txt file, the command above installs these packages:

        Cython>=0.19.2

        numpy>=1.7.1
        
        scipy>=0.13.2
        
        scikit-image>=0.9.3
        
        matplotlib>=1.3.1
        
        ipython>=3.0.0
        
        h5py>=2.2.0
        
        leveldb>=0.191
        
        networkx>=1.8.1
        
        nose>=1.3.0
        
        pandas>=0.12.0
        
        python-dateutil>=1.4,<2
        
        protobuf>=2.5.0
        
        python-gflags>=2.0
        
        pyyaml>=3.10
        
        Pillow>=2.3.0
        
        six>=1.1.0
Here I encountered a problem when pip tries to install numpy:

        UnicodeDecodeError: 'ascii' codec can't decode byte 0xe2 in position 72: ordinal not in range(128)
I found answer on [Stackoverflow][4], according to that answer, I need to install python-numpy manually by apt-get rather than numpy by pip:

        apt-get install python-numpy
The problem is that we need to modify the requirements.txt file now since we already used apt-get to install numpy rather than pip. So I delete the first two rows of the requirements.txt and run 

        sudo pip install -r ./python/requirements.txt 
to install other requirements. Maybe a better way to install all the requirements is to use pip manually and install them one by one, this is inconvinient but you can solve the complaints immediately, knowing which one has been successfully installed.

4.  Finally, we can compile the python interface:

        make pycaffe
        
## Training LeNet on MNIST with Caffe

A quick start with caffe is to run mnist using caffe since this data set is small and easy to get. 

1.  To get this dataset, just run the command below under caffe root directory:

        ./data/mnist/get_mnist.sh
        ./examples/mnist/create_mnist.sh
        
2.  Since I only use CPU here, I need to change solver_mode in caffe/examples/mnist/lenet_solver.prototxt to solver_mode: CPU.

        solver_mode: CPU
        
3.  Then we start to train lenet model:

        ./examples/mnist/train_lenet.sh
        
This will take quiet a while. You can take the time to install the python interface in another terminal. After the training, I got nearly 99.06% accuracy as below.

![lenet-result][2]

[1]: https://raw.githubusercontent.com/sunshineatnoon/sunshineatnoon.github.io/master/images/runtest-success.png
[2]: https://raw.githubusercontent.com/sunshineatnoon/sunshineatnoon.github.io/master/images/lenet-result.png
[3]: https://pip.pypa.io/en/stable/index.html
[4]: http://stackoverflow.com/questions/26473681/pip-install-numpy-throws-an-error-ascii-codec-cant-decode-byte-0xe2
