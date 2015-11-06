---
layout: post
title: How to train fast rcnn on imagenet without matlab
published: true
---

I've played with rcnn and fast rcnn for a long time. The amazing works of [Ross Girshick](https://github.com/rbgirshick) have excited my curiosity towards Deep Learning. This is the very first time I enjoyed and was amazed by the power of Convolutional Netoworks.

The [fast rcnn code](https://github.com/rbgirshick/fast-rcnn) comes with an already trained model on pascal voc and needs [selective search code](https://github.com/rbgirshick/rcnn/tree/master/selective_search) to do detections. For me, these two necessities are very inconvinient. So I modified the code to train on ImageNet(two classes) and use pure python. 

