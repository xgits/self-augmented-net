# Modeling Surface Appearance from a Single Photograph using Self-augmented Convolutional Neural Networks

The major contributors of this repository include [Xiao Li](http://home.ustc.edu.cn/~pableeto), [Yue Dong](http://yuedong.shading.me), [Pieter Peers](www.cs.wm.edu/~ppeers/) and [Xin Tong](https://www.microsoft.com/en-us/research/people/xtong/).

## Introduction

This repository provided source code for SIGGRAPH 2017 paper "Modeling Surface Appearance from a Single Photograph using Self-augmented Convolutional Neural Networks".

## Citation
If you find our code helpful for your research, please consider citing:

```
@article{Li:2017:MSA, 
 author = {Li, Xiao and Dong, Yue and Peers, Pieter and Tong, Xin},
 title = {Modeling Surface Appearance from a Single Photograph using Self-Augmented Convolutional Neural Networks},
 month = {July},
 year = {2017},
 journal = {ACM Transactions on Graphics},
 volume = {36},
 number = {4},
 article = {45},
 }
```

----------------------------------------------------------------
## Usage:

### System Requirements
   - Windows or Linux system (We have tested our code on Windows 10 and Ubuntu 12.04. Mac OSX are not supported yet)
   - A nvidia GPU (We have tested our code on Titan X and GTX 1080)
   - Python 2.7 (Python 3.x is not supported)

     We strongly recommend users install Anaconda2, which would include most libraries used in this code. Still, you might need following packages:
     * NumPy (Tested with 1.10.4, newer version should be fine)
     * OpenCV (Tested with 3.1.0)
     * PyCUDA (Tested with 2016-1-2 version)
     * Matplotlib (Tested with 1.5.1)
     * skimage
     * jinja2
   - Caffe with Python support (Tested with both CUDA 7.5 + cuDNN 5.0 and CUDA 8.0 + cuDNN 5.1)

### Installation
After install all pre-requisities (see above), please download (or git clone) our code. Note to running our training or test you may also need to download our datasets (includes training/test patches and our collected lighting maps) as below.

### Preparing data
We provide our dataset including both training and test data for people who want to quickly test or reproduce our method.

The dataset could be downloaded from our project website: http://msraig.info/~sanet/sanet.htm. Since all rendered image patch are too large, we rather provided original SVBRDF maps generated by artist, our collected lighting env. maps, and python scripts for generate all training images on your local machine.

- After download dataset (contains SVBRDF-Net maps and lighting), please open ./BRDFNet/folderPath.txt (for BRDF-Net) and/or ./BRDFNet/folderPath_SVBRDF.txt

- To generate training and test data for BRDF-Net, running:

    python ./BRDFNet/RenderBRDFNetData.py $GPUID$ $BRDF_NET_DATA_FOLDER$

- To generate training and test data for SVBRDF-Net, running:

    python ./SVBRDFNet/RenderSVBRDFDataset.py $SVBRDF_NET_DATA_FOLDER$ $CATEGORY_TAG$ $GPUID$ $RENDERTYPE$ -1 -1 $RENDERTEST$

Meaning of paramaters:

**$BRDF_NET_DATA_FOLDER$:** output folder of BRDF-Net dataset.

**$SVBRDF_NET_DATA_FOLDER$**: folder contains provided SVBRDF Map (downloaded from our website). It also serves as output folder of SVBRDF-Net dataset.

**$CATEGORY_TAG$**: one of "wood" "metal" or "plastic".

**$RENDERTYPE$**: 0 - only render PFM images; 1 - only render JPG images; 2 - render both. (0 or 2 are usually OK)

**$RENDERTEST$**: "train" - only render training images; "test" - only render testing images; "all" - render both training and testing data.

### Test our trained model
We provide our trained model of SA-SVBRDF-Net on Wood, Metal and Platic dataset, as well as our SA-BRDF-Net model, which are avaliable on our project website: http://msraig.info/~sanet/sanet.htm. We also provided our test dataset which correspond to results in our paper and supp. materials.

To test our model on our provided SVBRDF dataset, running: 

    python ./SVBRDFNet/TestSVBRDF.py $MODELFILE$ $TESTSET$ $GPUID$
    
Meaning of paramaters:

**$MODELFILE$**: caffe model for test

**$TESTSET$**: test dataset. often it's **$SVBRDF_NET_DATA_FOLDER$\Test_Suppmental\$DATA_TAG$\list.txt**

For BRDF-Net, by default, our training script will automaticlly generate test reports after training finished. If you want to manually test our model on BRDF dataset and generate reports, running:

    python ./BRDFNet/TestBRDF.py $MODELFILE$ $TESTCONFIG$ $GPUID$

Meaning of paramaters:

**$MODELFILE$**: caffe model for test

**$TESTCONFIG$**: a config ini file for testing. usually it is generated by our training.

The test result and reports (a html file) will generated at the folder which contains trained model. 

**Note**: For running those testing, you might want to download and generate some test data first (see section 2 above)

If you want test on your own images, here are some tips:

 - All our model are in caffe format.
 - The input of our SVBRDF-Net model is an image (in [0, 1] range) with size 256*256. The output of our model is an albedo map and a normal map with the same size as input, a 3-channal vector represents RGB specular albedo and a float value represents roughness. 
 - It is easy to write a simple python script that load our model, read an image and generate the results; such examples are not hard to find on the internet:).

### Training from scratch
#### Training SA-BRDF-Net:
Open the text file **./BRDFNet/BRDF_Net_Config.ini**. This file contains all the settings w.r.t training. change the following rows:

    dataset = $BRDF_NET_DATA_FOLDER$/train_envlight/train_full.txt
    unlabelDataset = $BRDF_NET_DATA_FOLDER$/train_envlight/train_full.txt
    testDataset = $BRDF_NET_DATA_FOLDER$/test_envlight/test_full.txt

These rows setup the path for training/test data; **$BRDF_NET_DATA_FOLDER$** are folder of your BRDF-Net data.

By default, our training of SA-BRDF-Net are configured as only use corner labeled data, leaving rest as unlabeled data. 
This behavior is defined via **albedoRange**, **specRange** and **roughnessRange** parameters in BRDF_Net_Config.ini. Feel free to change them if you want to change the distribution of labeled/unlabeled data. Please note that albedoRange and specRange are in [0, 9], while roughnessRange are in [0, 14].

To train our SA-BRDF-Model, running:

    python ./BRDFNet/BRDFNetTraining.py BRDF_Net_Config.ini $OUT_TAG$ $RESTORE_TAG$ $GPUID$ $RENDERGPUID$ $AUTOTEST$
    
Meaning of paramaters:

**$OUT_TAG$**: name your training.

**$RESTORE_TAG$**: 0 - training from scratch

**$RENDERGPUID$**: for current release version please set it same as **$GPUID$**

**$AUTOTEST$**: 1 - running a full test and generate reports after training.

By default, the training snapshot and results are saved at **./TrainedResult/$OUT_TAG$** (relative to root of code folder).
You could change it by open **./BRDFNet/folderPath.txt** and change the first line.

#### Training SA-SVBRDF-Net
Open the text file **./SVBRDFNet/SVBRDF_Net_Config.ini**. This file contains all the settings w.r.t training. change the following rows:

    dataset = $SVBRDF_NET_DATA_FOLDER$/$CATAGORY_TAG$/Labeled/trainingdata.txt
    unlabelDataset = $SVBRDF_NET_DATA_FOLDER$/$CATAGORY_TAG$/unlabeled.txt
    testDataset = $SVBRDF_NET_DATA_FOLDER$/$CATAGORY_TAG$/Test/test.txt
    
These rows setup the path for training/test data; **$SVBRDF_NET_DATA_FOLDER$** are folder of your SVBRDF-Net data. 
**$CATAGORY_TAG$** should be one of "wood", "metal" or "plastic"

    lightPoolFile = lightPool_$CATAGORY_TAG$.dat
    autoExposureLUTFile = lightNormPool_$CATAGORY_TAG$.dat
    
These rows setup the path for pre-defined lighting rotations and pre-computed auto-exposure factors.

To train our SA-SVBRDF-Model, running:
    
    python ./SVBRDFNet/SVBRDFNetTraining.py SVBRDF_Net_Config.ini $OUT_TAG$ $RESTORE_TAG$ $GPUID$ $RENDERGPUID$
    
Meaning of paramaters:    
**$OUT_TAG$**: name your training.

**$RESTORE_TAG$**: 0 - training from scratch

**$RENDERGPUID$**: for current release version please set it same as **$GPUID$**

By default, the training snapshot and results are saved at **./TrainedResult/$OUT_TAG$** (relative to root of code folder). You could change it by open **./SVBRDFNet/folderPath_SVBRDF.txt** and change the first line.