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
#### \_\_init__(self,image_set,devkit_path):
This function is easy to modify, only two lines need to be changed:
```
self._classes = ('__background__','n02958343','n02769748')
self._image_ext = '.JPEG'
```
These two lines specify classes and image extentions. For the sake of time, I only chose 2 classes out of 200 classes of the dataset. We need to pay attention to the names of these two classes because in our annotation files, the groud truth class is its number in the imagenet dataset such as n02958343 or n02769748, not its real name such as car or bakcpack.
#### \_load\_imagenet_annotation
This is an important function for our training, it tells fast rcnn how to read annotation files. But the imagenet annotation files are much like ones in pascal voc, so one can easily figure out this function. Though special attention should be paid to these four lines:

```
x1 = float(get_data_from_tag(obj, 'xmin')) - 1
y1 = float(get_data_from_tag(obj, 'ymin')) - 1
x2 = float(get_data_from_tag(obj, 'xmax')) - 1
y2 = float(get_data_from_tag(obj, 'ymax')) - 1
```
This is because in the pascal voc dataset, all coordinates start from one, so in order to make them start from 0, we need to minus 1. But this is not true for imagenet, so we should not minus 1. The same goes for `box_list.append(raw_data[i][:, (1, 0, 3, 2)]-1) `. So we need to modify these lines to:

```
x1 = float(get_data_from_tag(obj, 'xmin'))
y1 = float(get_data_from_tag(obj, 'ymin'))
x2 = float(get_data_from_tag(obj, 'xmax'))
y2 = float(get_data_from_tag(obj, 'ymax')) 
box_list.append(raw_data[i][:, (1, 0, 3, 2)])
```
I didn't do any change to other functions. My imagenet.py file can be found [here](https://github.com/sunshineatnoon/fast-rcnn/blob/master/lib/datasets/imagenet.py) for a reference.
#### factory\.py
This file is easy to change, just add these lines to it:

```
# Set up inria_<split> using selective search "fast" mode
import datasets.imagenet
imagenet_devkit_path = '/path/to/imagenet'
for split in ['train', 'test']:
    name = '{}_{}'.format('imagenet', split)
    __sets[name] = (lambda split=split: datasets.imagenet(split, imagenet_devkit_path))
```
This tells fast rcnn where we put our dataset.
#### \_\_init__\.py
The only thing we need to change for this file is to import imagenet:

```
from .imagenet import imagenet
```
## Run Selective Search
Since I don't have MATLAB installed, so instead I use [dlib](http://dlib.net/)'s slective search. This is a fast and convinient library for many computer vision algorithms. I use the file [generate_bbox.py](https://github.com/sunshineatnoon/fast-rcnn/blob/master/tools/generate_bbox.py) to generate the train.mat file which gives all object proposals.  To use this file, you need to specify the image path and image name path in the file:

```
imagenet_path = '/path/to/imagnet/data/Images/'
names = '/path/to/imagenet/data/ImageSets/test.txt'
```
To run this file, just run `python generate_bbox.py`. Then you will find a train.mat file in the same folder, just copy the train.mat file to imagenet/data/.
## Modify Prototxt Files
Sine we only have 3 classes(including background class), we need to change the network structure. For me, I trained this model on the pre-trained caffenet model. So I need to change `$FRCNN_ROOT/models/CaffeNet/train.prototxt` to fit my dataset.
- For the input layer, we need to change input class to 3: `param_str: "'num_classes': 3"`
- For the cls_score layer, we need to change output class to 3: `num_output: 3`
- For the bbox_pred layer, we need to change output to 3*4=12: `num_output: 12`

See the [train.prototxt](https://github.com/sunshineatnoon/fast-rcnn/blob/master/models/CaffeNet/train.prototxt) file for reference.
## Train
Training is easy, just run this command under `$FRCNN_ROOT`:

```
./tools/train_net.py --gpu 0 --solver models/CaffeNet/solver.prototxt --weights data/imagenet_models/CaffeNet.v2.caffemodel --imdb imagenet_train
```
For me, it took me about 0.44s for each iteration for 40000 iterations. So the total time for training is about 5 hours. 
## See Training Result
I don't really want to test the trained model for an accuracy, but instead view its performance intuitively. So just copy the trained model in `$FRCNN_ROOT/output/default/train/caffenet_fast_rcnn_iter_40000.caffemodel` to `/data/fast_rcnn_models/`(Don't forget to backup the old one). Then run `./tools/demo.py` under `$FRCNN_ROOT` to see how our trained model works. Of course, changes need to be made in demo.py file to use dlib's selective search and the trained model. You can find mine [here](https://github.com/sunshineatnoon/fast-rcnn/blob/master/tools/demo.py) for a reference.
