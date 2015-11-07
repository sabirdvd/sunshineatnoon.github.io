---
layout: post
title: Implementation a Single Layer CNN on MNIST
published: true
---

Regarding CNN(Convolutional Neural Network) as a black box is easy and convinient. And luckily there are many CNN frameworks such as [Caffe](http://caffe.berkeleyvision.org) or [Torch](http://torch.ch). But for the sake of curiosity and integrity, in this post, I would like to write down my notes of implementing a single layer CNN on [MNIST](https://en.wikipedia.org/wiki/MNIST_database) in MATLAB. Readers might be expected to know basics about CNN. If not, I strongly recommend this [tutorial](http://neuralnetworksanddeeplearning.com).
*****
##Convolution and Pooling
Convolution and pooling are two basic operations. 

