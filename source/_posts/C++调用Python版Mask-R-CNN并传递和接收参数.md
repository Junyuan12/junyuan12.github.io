---
title: C++调用Python版Mask-R-CNN并传递和接收参数
date: 2019-04-14 12:00:00
categories: C++
tags:
     - C++
     - python
     - mask rcnn
---

想做一个用Mask R-CNN只检测人的demo，发现github已经有人用Python实现了，但后处理用的是C++，所以想直接用C++器调用python，再将需要的结果返回给C++。

<!-- more -->

**运行环境：**

Visual Studio 2015，Python3.6（Anaconda），OpenCV3.4.2（C++），tensorflow1.12，Keras2.0.8

### C++环境配置

- **配置OpenCV**

  1. 新建一个C++工程；

  2. 右键属性$\rightarrow$C++目录$\rightarrow$包含目录$\rightarrow$点击编辑，将

     ```
     D:\Programs\opencv\build\include
     D:\Programs\opencv\build\include\opencv
     D:\Programs\opencv\build\include\opencv2
     ```

     加入包含目录，**注意配置的方案平台版本和当前运行的一致**；

  3. 右键属性$\rightarrow$C++目录$\rightarrow$库目录$\rightarrow$点击编辑，将

     ```
     D:\Programs\opencv\build\x64\vc14\lib
     ```

     加入库目录，注意VS的版本，2017改用vc15的目录；

  4. 右键属性$\rightarrow$链接器$\rightarrow$输入$\rightarrow$点击编辑，将

     ```
     opencv_world342.lib
     ```

     加入其中。

+ **配置Python环境**

  1. 右键属性$\rightarrow$C++目录$\rightarrow$包含目录$\rightarrow$点击编辑，将

     ```
     D:\Programs\Anaconda3\include
     D:\Programs\Anaconda3\Lib\site-packages\numpy\core\include
     ```

     加入到包含目录中；

  2. 右键属性$\rightarrow$C++目录$\rightarrow$库目录$\rightarrow$点击编辑，将

     ```
     D:\Programs\Anaconda3\libs
     D:\Programs\Anaconda3\Lib\site-packages\numpy\core\lib
     ```

     加入库目录;

  3. 右键属性$\rightarrow$链接器$\rightarrow$输入$\rightarrow$点击编辑，将

     ```
     python36.lib
     ```

     加入其中，注意只有python36，没有python36d，所以调试的时候要用relese的版本，debug的版本会报错。

### 调用Python

+ **从C++输入图片数据**

  由于C++用OpenCV读取的图片格式和Mask R-CNN的输入格式不同，所以需要先用C++处理好图片，传递给Python。

  ```c++
  Mat img = imread("./image.jpg");
  auto sz = img.size();
  int x = sz.width;
  int y = sz.height;
  int z = img.channels();
  uchar *CArrays = new uchar[x*y*z];
  int iChannels = img.channels();
  int iRows = img.rows;
  int iCols = img.cols * iChannels;
  if (img.isContinuous())
  {
      iCols *= iRows;
      iRows = 1;
  }
  uchar* p;
  int id = -1;
  for (int i = 0; i < iRows; i++)
  {
      // get the pointer to the ith row
      p = img.ptr<uchar>(i);
      // operates on each pixel
      for (int j = 0; j < iCols; j++)
      {
          CArrays[++id] = p[j];//连续空间
      }
  }
  npy_intp Dims[3] = {y, x, z}; //注意这个维度数据！
  PyObject *PyArray = PyArray_SimpleNewFromData(3, Dims, NPY_UBYTE, CArrays);
  ```

+ **调用Mask R-CNN**

  ```c++
  //初始化python
  Py_Initialize();
  import_array();
  
  PyRun_SimpleString("import sys");
  	PyRun_SimpleString("sys.path.append(r'C:\\Users\\Administrator\\Documents\\Keypoints-of-humanpose-with-Mask-R-CNN-master')");
  
  //定义python类型的变量
  PyObject *pModule = NULL;
  PyObject *pFunc = NULL;
  PyObject *pArg = NULL;
  PyObject *result = NULL;
  PyObject *pDict = NULL;
  
  //直接运行python代码
  PyRun_SimpleString("print('python start')");
  
  PyObject *ArgList = PyTuple_New(1);
  .
  .
  .
  PyTuple_SetItem(ArgList, 0, PyArray);
  
  //引入模块
  pModule = PyImport_ImportModule("human_detected");
  
  if (!pModule)
  {
      cout << "Import Module Failed" << endl;
      system("pause");
      return 0;
  }
  
  //获取模块字典属性
  pDict = PyModule_GetDict(pModule);
  
  //直接获取模块中的函数
  pFunc = PyObject_GetAttrString(pModule, "detect");
  .
  .
  .
  ```

  

+ **Mask R-CNN接受参数并返回值**

  Mask R-CNN检测人项目地址：https://github.com/Junyuan12/Mask_RCNN_Humanpose

  Mask R-CNN的配置就不细说了，网上有很多教程，主要是windows下配置pycocotools可能会有问题，

  windows下pycocotools安装教程：https://blog.csdn.net/qq_29592829/article/details/82877494

  在Mask R-CNN中新建一个py文件，`human_detected.py`

  ```python
  import os
  import sys
  import random
  import math
  import numpy as np
  import skimage.io
  import matplotlib
  import matplotlib.pyplot as plt
  
  import coco
  import utils
  import model as modellib
  import visualize
  from model import log
  import cv2
  
  def detect(image):
  	# print(image)
  	ROOT_DIR = os.getcwd()
  	# Directory to save logs and trained model
  	MODEL_DIR = os.path.join(ROOT_DIR, "mylogs")
  
  	# Local path to trained weights file
  	COCO_MODEL_PATH = os.path.join(ROOT_DIR, "mask_rcnn_coco_humanpose.h5")
  	# Download COCO trained weights from Releases if needed
  	if not os.path.exists(COCO_MODEL_PATH):
  	    utils.download_trained_weights(COCO_MODEL_PATH)
  
  	class InferenceConfig(coco.CocoConfig):
  	    GPU_COUNT = 1
  	    IMAGES_PER_GPU = 1
  	    KEYPOINT_MASK_POOL_SIZE = 7
  
  	inference_config = InferenceConfig()
  
  	# Recreate the model in inference mode
  	model = modellib.MaskRCNN(mode="inference", 
  	                          config=inference_config,
  	                          model_dir=MODEL_DIR)
  
  	# Get path to saved weights
  
  	model_path = os.path.join(ROOT_DIR, "mask_rcnn_coco_humanpose.h5")
  	assert model_path != "", "Provide path to trained weights"
  	print("Loading weights from ", model_path)
  	model.load_weights(model_path, by_name=True)
  
  	# COCO Class names
  	#For human pose task We just use "BG" and "person"
  	class_names = ['BG', 'person']
  
  	#BGR->RGB
  	image = image[:,:,::-1]
  	#print(np.shape(image))
  
  	# Run detection
  	results = model.detect_keypoint([image], verbose=1)
  	r = results[0] # for one image
  
  	log("rois",r['rois'])
  	log("keypoints",r['keypoints'])
  	log("class_ids",r['class_ids'])
  	log("keypoints",r['keypoints'])
  	log("masks",r['masks'])
  	log("scores",r['scores'])
  
  	visualize.display_instances(image, r['rois'], r['masks'], r['class_ids'], 
  	                            class_names, r['scores'])
  	flat_roi = r['rois'].flatten()
  	print(flat_roi)
   
  	return flat_roi
  ```

  其中包括一个函数`detect(image)`，用于接受C++传进来的图片，并最终返回每个人的`roi`。

+ **C++接受返回参数**

  python将结果展开成一列，C++接受返回的一维数组并输出：

  ```c++
  // 调用直接获得的函数,并传递参数
  PyObject *pReturn = PyObject_CallObject(pFunc, ArgList);
  
  //获取python程序的返回结果
  //以下是对返回的一维数组结果进行处理
  if (pReturn)
  {
      //将结果类型转换成数组对象类型
      PyArrayObject *pyResultArr = (PyArrayObject *)pReturn;
  
      //从Python中的PyArrayObject解析出数组数据为c的double类型。
      int *resDataArr = (int *)PyArray_DATA(pyResultArr);
      int dimNum = PyArray_NDIM(pyResultArr);//返回数组的维度数，此处恒为1
      npy_intp *pdim = PyArray_DIMS(pyResultArr);//返回数组各维度上的元素个数值
  
      //以下是对返回结果的输出显示
      for (int i = 0; i < dimNum; ++i)
      {
          for (int j = 0; j < pdim[0]; ++j)
              cout << resDataArr[i * pdim[0] + j] << ",";
      }
          cout << endl;
  }
  ```

+ **完整代码如下**

  ```c++
  #include <Python.h>
  #include <iostream>
  #include <string>
  #include <numpy/arrayobject.h>
  #include <opencv2/highgui/highgui.hpp>
  #include <opencv2/imgproc/imgproc.hpp>
  #include <opencv2/core/core.hpp>
  
  using namespace std;
  
  int main(int argc, char* argv[])
  
  {
  	cv::Mat img = cv::imread("./image.jpg");
  	//初始化python
  	Py_Initialize();
  	import_array();
  
  	PyRun_SimpleString("import sys");
  	PyRun_SimpleString("sys.path.append(r'C:\\Users\\Administrator\\Documents\\Keypoints-of-humanpose-with-Mask-R-CNN-master')");
  	//PyRun_SimpleString("print(sys.path)");
  
  	//定义python类型的变量
  
  	PyObject *pModule = NULL;
  	PyObject *pFunc = NULL;
  	PyObject *pArg = NULL;
  	PyObject *result = NULL;
  	PyObject *pDict = NULL;
  
  	//直接运行python代码
  	PyRun_SimpleString("print('python start')");
  
  	PyObject *ArgList = PyTuple_New(1);
  
  	auto sz = img.size();
  	int x = sz.width;
  	int y = sz.height;
  	int z = img.channels();
  	uchar *CArrays = new uchar[x*y*z];
  	int iChannels = img.channels();
  	int iRows = img.rows;
  	int iCols = img.cols * iChannels;
  	if (img.isContinuous())
  	{
  		iCols *= iRows;
  		iRows = 1;
  	}
  
  	uchar* p;
  	int id = -1;
  	for (int i = 0; i < iRows; i++)
  	{
  		// get the pointer to the ith row
  		p = img.ptr<uchar>(i);
  		// operates on each pixel
  		for (int j = 0; j < iCols; j++)
  		{
  			CArrays[++id] = p[j];//连续空间
  		}
  	}
  
  	npy_intp Dims[3] = { y, x, z }; //注意这个维度数据！
  	PyObject *PyArray = PyArray_SimpleNewFromData(3, Dims, NPY_UBYTE, CArrays);
  	PyTuple_SetItem(ArgList, 0, PyArray);
  
  	//引入模块
  	pModule = PyImport_ImportModule("human_detected");
  
  	if (!pModule)
  	{
  		cout << "Import Module Failed" << endl;
  		system("pause");
  		return 0;
  	}
  
  	//获取模块字典属性
  	pDict = PyModule_GetDict(pModule);
  
  	//直接获取模块中的函数
  	pFunc = PyObject_GetAttrString(pModule, "detect");
  
  	// 调用直接获得的函数,并传递参数
  	PyObject *pReturn = PyObject_CallObject(pFunc, ArgList);
  
  	//获取python程序的返回结果
  	//以下是对返回的一维数组结果进行处理
  	if (pReturn) {
  		//将结果类型转换成数组对象类型
  		PyArrayObject *pyResultArr = (PyArrayObject *)pReturn;
  
  		//从Python中的PyArrayObject解析出数组数据为c的double类型。
  		int *resDataArr = (int *)PyArray_DATA(pyResultArr);
  		int dimNum = PyArray_NDIM(pyResultArr);//返回数组的维度数，此处恒为1
  		npy_intp *pdim = PyArray_DIMS(pyResultArr);//返回数组各维度上的元素个数值
  
  												   //以下是对返回结果的输出显示
  		for (int i = 0; i < dimNum; ++i) {
  			for (int j = 0; j < pdim[0]; ++j)
  				cout << resDataArr[i * pdim[0] + j] << ",";
  		}
  		cout << endl;
  	}
  
  	PyRun_SimpleString("print('python end')");
  
  	////释放python
  	Py_Finalize();
  
  	system("pause");
  	return 0;
  }
  ```

  

  

