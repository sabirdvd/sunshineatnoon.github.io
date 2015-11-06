---
layout: post
title: How to train fast rcnn on imagenet
published: true
---

I've been playing with [fast-rcnn](https://github.com/rbgirshick/fast-rcnn) for a while. This amazing and wonderful project helps me understand more about deep learning and its beautiful power. However, there's only a pre-trained fast rcnn model for pascal voc with 20 classes. To use this project in real applications, I need to train a model on the [ImageNet](http://www.image-net.org/) detection dataset( For time's sake, I only chose two classes out of 200 classes). So this blog records what to be done to train a fast rcnn on ImangeNet.
## Prepare Dataset
The organization of my dataset is like this:

```
imagenet
|-- data
    |-- train.mat
    |-- Annotations
         |-- *.xml (Annotation files)
    |-- Images
         |-- *.JPEG (Image files)
    |-- ImageSets
         |-- train.txt
```
- train.mat: This is the selective search proposals file
- Annotations: This folder contains all annotation files of the images
- Images: This folder contains all images
- ImageSets: This folder only contains one file--trian.txt, which contains all the names of the images. It looks like this:

```
n02769748_18871
n02769748_2379
n02958343_4294
...
```
## Construct IMDB File
We need to create a file imagenet.py in the directory `$FRCNN_ROOT/lib/datasets`. This file defines some functions which tell fast rcnn how to read ground truth boxes and how to find images on disk. I mainly changed these functions:
#### __init__(self,image_set,devkit_path):
This function is easy to modify, only two lines need to be changed:

```
self._classes = ('__background__','n02958343','n02769748')
self._image_ext = '.JPEG'
```
These two lines specify classes and image extentions. For the sake of time, I only chose 2 classes out of 200 classes of the dataset. We need to pay attention to the names of these two classes because in our annotation files, the groud truth class is its number in the imagenet dataset such as n02958343 or n02769748, not its real name such as car or bakcpack.
#### _load_imagenet_annotation
This is an important function for our training, it tells fast rcnn how to read annotation files. But the imagenet annotation files are much like ones in pascal voc
