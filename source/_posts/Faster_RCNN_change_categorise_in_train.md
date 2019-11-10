---
title: Tensorflow Faster RCNN修改训练的类别数量
date: 2019-11-10 17:00:00
categories: 深度学习
tags:
     - 深度学习
     - python
     - tensorflow
     - Faster RCNN
---

本文主要基于COCO数据集，需要某一类别或某几个类别的检测结果，所以针对特定的类别专门训练一个Faster RCNN的模型，后续可以针对特定的类别设置`ancher`和`scale`的大小以达到更好的效果。

 <!-- more -->

源码地址：[tf faster rcnn]( https://github.com/endernewton/tf-faster-rcnn )

### 训练

要修改的文件：`./tf-faster-rcnn/lib/datasets/coco.py`

**第39行：**

`cats = self._COCO.loadCats(self._COCO.getCatIds())`

```python
COCO.getCatIds?
Signature: COCO.getCatIds(self, catNms=[], supNms=[], catIds=[])
Docstring:
filtering parameters. default skips that filter.
:param catNms (str array)  : get cats for given cat names
:param supNms (str array)  : get cats for given supercategory names
:param catIds (int array)  : get cats for given cat ids
:return: ids (int array)   : integer array of cat ids
File:      d:\programs\anaconda3\envs\tensorflow1.x\lib\site-packages\pycocotools\coco.py
Type:      function
```

通过设置`getCatIds`的参数改变训练的类别，比如本例只训练一个检测`person`的模型，就改成`cats = self._COCO.loadCats(self._COCO.getCatIds(catIds=[1]))`，把需要训练的类别对应的`id`作为参数传入。

原始类别：

```python
coco.loadCats(coco.getCatIds())

[{'supercategory': 'person', 'id': 1, 'name': 'person'},
 {'supercategory': 'vehicle', 'id': 2, 'name': 'bicycle'},
 {'supercategory': 'vehicle', 'id': 3, 'name': 'car'},
 {'supercategory': 'vehicle', 'id': 4, 'name': 'motorcycle'},
 {'supercategory': 'vehicle', 'id': 5, 'name': 'airplane'},
 {'supercategory': 'vehicle', 'id': 6, 'name': 'bus'},
 {'supercategory': 'vehicle', 'id': 7, 'name': 'train'},
 {'supercategory': 'vehicle', 'id': 8, 'name': 'truck'},
 {'supercategory': 'vehicle', 'id': 9, 'name': 'boat'},
 {'supercategory': 'outdoor', 'id': 10, 'name': 'traffic light'},
 {'supercategory': 'outdoor', 'id': 11, 'name': 'fire hydrant'},
 {'supercategory': 'outdoor', 'id': 13, 'name': 'stop sign'},
 {'supercategory': 'outdoor', 'id': 14, 'name': 'parking meter'},
 {'supercategory': 'outdoor', 'id': 15, 'name': 'bench'},
 {'supercategory': 'animal', 'id': 16, 'name': 'bird'},
 {'supercategory': 'animal', 'id': 17, 'name': 'cat'},
 {'supercategory': 'animal', 'id': 18, 'name': 'dog'},
 {'supercategory': 'animal', 'id': 19, 'name': 'horse'},
 {'supercategory': 'animal', 'id': 20, 'name': 'sheep'},
 {'supercategory': 'animal', 'id': 21, 'name': 'cow'},
 {'supercategory': 'animal', 'id': 22, 'name': 'elephant'},
 {'supercategory': 'animal', 'id': 23, 'name': 'bear'},
 {'supercategory': 'animal', 'id': 24, 'name': 'zebra'},
 {'supercategory': 'animal', 'id': 25, 'name': 'giraffe'},
 {'supercategory': 'accessory', 'id': 27, 'name': 'backpack'},
 {'supercategory': 'accessory', 'id': 28, 'name': 'umbrella'},
 {'supercategory': 'accessory', 'id': 31, 'name': 'handbag'},
 {'supercategory': 'accessory', 'id': 32, 'name': 'tie'},
 {'supercategory': 'accessory', 'id': 33, 'name': 'suitcase'},
 {'supercategory': 'sports', 'id': 34, 'name': 'frisbee'},
 {'supercategory': 'sports', 'id': 35, 'name': 'skis'},
 {'supercategory': 'sports', 'id': 36, 'name': 'snowboard'},
 {'supercategory': 'sports', 'id': 37, 'name': 'sports ball'},
 {'supercategory': 'sports', 'id': 38, 'name': 'kite'},
 {'supercategory': 'sports', 'id': 39, 'name': 'baseball bat'},
 {'supercategory': 'sports', 'id': 40, 'name': 'baseball glove'},
 {'supercategory': 'sports', 'id': 41, 'name': 'skateboard'},
 {'supercategory': 'sports', 'id': 42, 'name': 'surfboard'},
 {'supercategory': 'sports', 'id': 43, 'name': 'tennis racket'},
 {'supercategory': 'kitchen', 'id': 44, 'name': 'bottle'},
 {'supercategory': 'kitchen', 'id': 46, 'name': 'wine glass'},
 {'supercategory': 'kitchen', 'id': 47, 'name': 'cup'},
 {'supercategory': 'kitchen', 'id': 48, 'name': 'fork'},
 {'supercategory': 'kitchen', 'id': 49, 'name': 'knife'},
 {'supercategory': 'kitchen', 'id': 50, 'name': 'spoon'},
 {'supercategory': 'kitchen', 'id': 51, 'name': 'bowl'},
 {'supercategory': 'food', 'id': 52, 'name': 'banana'},
 {'supercategory': 'food', 'id': 53, 'name': 'apple'},
 {'supercategory': 'food', 'id': 54, 'name': 'sandwich'},
 {'supercategory': 'food', 'id': 55, 'name': 'orange'},
 {'supercategory': 'food', 'id': 56, 'name': 'broccoli'},
 {'supercategory': 'food', 'id': 57, 'name': 'carrot'},
 {'supercategory': 'food', 'id': 58, 'name': 'hot dog'},
 {'supercategory': 'food', 'id': 59, 'name': 'pizza'},
 {'supercategory': 'food', 'id': 60, 'name': 'donut'},
 {'supercategory': 'food', 'id': 61, 'name': 'cake'},
 {'supercategory': 'furniture', 'id': 62, 'name': 'chair'},
 {'supercategory': 'furniture', 'id': 63, 'name': 'couch'},
 {'supercategory': 'furniture', 'id': 64, 'name': 'potted plant'},
 {'supercategory': 'furniture', 'id': 65, 'name': 'bed'},
 {'supercategory': 'furniture', 'id': 67, 'name': 'dining table'},
 {'supercategory': 'furniture', 'id': 70, 'name': 'toilet'},
 {'supercategory': 'electronic', 'id': 72, 'name': 'tv'},
 {'supercategory': 'electronic', 'id': 73, 'name': 'laptop'},
 {'supercategory': 'electronic', 'id': 74, 'name': 'mouse'},
 {'supercategory': 'electronic', 'id': 75, 'name': 'remote'},
 {'supercategory': 'electronic', 'id': 76, 'name': 'keyboard'},
 {'supercategory': 'electronic', 'id': 77, 'name': 'cell phone'},
 {'supercategory': 'appliance', 'id': 78, 'name': 'microwave'},
 {'supercategory': 'appliance', 'id': 79, 'name': 'oven'},
 {'supercategory': 'appliance', 'id': 80, 'name': 'toaster'},
 {'supercategory': 'appliance', 'id': 81, 'name': 'sink'},
 {'supercategory': 'appliance', 'id': 82, 'name': 'refrigerator'},
 {'supercategory': 'indoor', 'id': 84, 'name': 'book'},
 {'supercategory': 'indoor', 'id': 85, 'name': 'clock'},
 {'supercategory': 'indoor', 'id': 86, 'name': 'vase'},
 {'supercategory': 'indoor', 'id': 87, 'name': 'scissors'},
 {'supercategory': 'indoor', 'id': 88, 'name': 'teddy bear'},
 {'supercategory': 'indoor', 'id': 89, 'name': 'hair drier'},
 {'supercategory': 'indoor', 'id': 90, 'name': 'toothbrush'}]
```

根据需要的类别的`id`设置输出参数，本例输出：

```python
coco.loadCats(coco.getCatIds(catIds=[1]))
[{'supercategory': 'person', 'id': 1, 'name': 'person'}]
```

**第42行：**

`self._class_to_coco_cat_id = dict(list(zip([c['name'] for c in cats],
                                               self._COCO.getCatIds())))`

修改为

`self._class_to_coco_cat_id = dict(list(zip([c['name'] for c in cats],
                                               self._COCO.getCatIds(catId=[1]))))`

同样加入训练的类别参数。

**第75行：**

`image_ids = self._COCO.getImgIds()`

修改为

`image_ids = self._COCO.getImgIds(catIds=[1])`

只获取和`person`有关的图片。感觉这里不是非必要的，因为后续获取`anno`的信息也会传入类别。

**第133行：**

`annIds = self._COCO.getAnnIds(imgIds=index, iscrowd=None)`

修改为

`annIds = self._COCO.getAnnIds(imgIds=index, catIds=[1], iscrowd=None)`

获取和`person`有关的标注信息，用于生成`roidb`训练`RPN`网络。

改完这四处就可以跑训练了。

```
iter: 27300 / 490000, total loss: 0.481602
 >>> rpn_loss_cls: 0.035242
 >>> rpn_loss_box: 0.018606
 >>> loss_cls: 0.139855
 >>> loss_box: 0.159181
 >>> lr: 0.001000
speed: 0.645s / iter
iter: 27320 / 490000, total loss: 0.867995
 >>> rpn_loss_cls: 0.085730
 >>> rpn_loss_box: 0.067610
 >>> loss_cls: 0.204145
 >>> loss_box: 0.381795
 >>> lr: 0.001000
speed: 0.645s / iter
```

**Note：**

我是用`COCO2017`训练的，如果要用`COCO`2017，还需要改几处。

+ factory.py中加入COCO2017信息
+ 修改coco.py中image_path_from_index函数获取的图片路径，`COCO2017`图片直接就是`id`，没有别的字符。

删除以前生成的文件和模型。

```
rm -rf output
rm data/cache/*.pkl
```

### 测试

修改demo文件，`./tf-faster-rcnn/tools/demo.py`

**第33行：**

`CLASS`修改为自己训练设置的类别，比如我的

`CLASSES = ('__background__', 'person')`

**第40行：**

`NET`中模型名改成自己的。

**可能遇到的问题：**

` tensorflow.python.framework.errors_impl.InvalidArgumentError: Assign requires shapes of both tensors to match. lhs shape= [x] rhs shape= [y] `

**第141行：**

1. `net.create_architecture("TEST", 21, tag='default', anchor_scales=[8, 16, 32])`，将21改成自己类别的数量，本例`__background__`和`person`两个类别，就改成2。

2. `anchor`和`scale`设置引起的维度不匹配错误，训练和测试的`anchor`和`scale`要对应。