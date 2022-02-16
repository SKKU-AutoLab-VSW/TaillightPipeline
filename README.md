# TaillightPipeline

This repository contains the demo videos and additional implementation details of the following paper:

-    H.-J. Jeon et al., "A Deep Learning Framework for Robust and Real-time Taillight Detection Under Various Road Conditions"

To implement the taillight detection pipeline for reproducing results, you can refer to the following papers:
-	[21] H.-J. Jeon et al., “High-speed car detection using resnet-based recurrent rolling convolution,” in Proceedings of the IEEE conference on systems, man, and cybernetics, Oct. 2018, pp. 286–291.
-	[22] J. Ren et al., “Accurate single stage detector using recurrent rolling convolution,” in Proceedings of the IEEE conference on computer vision and pattern recognition, 2017, pp. 5420–5428.
-	[26] T. T. Duong et al., “Near real-time ego-lane detection in highway and urban streets,” in 2016 IEEE International Conference on Consumer Electronics-Asia (ICCE-Asia), 2016, pp. 1–4.


## Implementing Lane Module

The Lane Module can be implemented by following the paper of Duong et al. [26]. The OpenCV library provides numerous functions for simple but powerful operations such as image cropping, colorspace conversion, binary morphological operations, and thresholding, all of which are sufficient for manifesting what is specified as mathematical equations in the paper of Duong et al. .

## Implementing Car and Taillight Modules

To write down code for these two models, you first need to know how to write down Resnet-RRC. According to the paper written by Jeon et al. [21], Resnet-RRC was written on top of VGG-RRC [22]. The two figures below show the VGG-RRC and the ResNet-RRC, respectively.

<p align="center">
  <img src="https://github.com/SKKU-AutoLab-VSW/TaillightPipeline/blob/main/vggRRC-vs-resnetRRC.png">
</p>

The VGG-RRC is the previously proposed model that uses a pre-initialized reduced VGG16 base net, and ResNet-RRC is the proposed model of this paper that uses a pre-initialized ResNet18 base net. The remaining layers are the same. Although conv4_3, FC7, and FC7_pool of the reduced VGG16 are nonexistent names in terms of the ResNet18 layers, in the VGG16, conv4_3 is the second-last convolution layer and FC7 is the layer right after the last layer, excluding pooling layer. Plus, the rest of the layers were newly appended. Judging from this, we hypothesized that such pattern can be similarly applied for the ResNet18 by binding additional feature maps on Res4b and Res5b. This results in a pre-trained layer set without any fully-connected layers before the RRC rolling layers.

The most intuitive way to write down code for Resnet-RRC is to first refer to the VGG-RRC git repository (https://github.com/xiaohaoChen/rrc_detection), which uses Caffe framework with user-defined layers. Using the code from there, it is possible to replace the VGG16 code part to Resnet18 code part, thereby switching the base net from VGG16 to Resnet18. Pre-trained weights of Resnet18 can also be downloaded as well (https://github.com/HolmesShuan/ResNet-18-Caffemodel-on-ImageNet). After the base net replacement is finished, the instructions on the VGG-RRC git repository web page can be followed to compile the Caffe source, prepare datasets, build the Resnet-RRC, and then train or test the Resnet-RRC.

It should be noted that if Resnet-RRC is to be implemented this way, the Caffe framework C/C++ source to be compiled from the VGG-RRC git repository only. All other distributions of Caffe source will **NOT** work since they lack the user-defined layers necessary for the VGG-RRC or Resnet-RRC. In addition to this, Resnet-RRC is highly dependent on system environment which needs to be manually configured one by one. Besides the hardware and software specifications mentioned in our paper, the Resnet-RRC is tested and found to be working properly only if the following versions of tools and libraries are given:

-	GCC 7 or lower
-	CUDA 9.0 or lower
-	cuDNN 7.0 or lower
-	libboost 1.65.1, with C++11 flag enabled on build
-	Ubuntu 17.10
-	LMDB 0.9.24-1
-	Google Protobuf 3.5.1

VGG-RRC does not seem to be implemented in other deep learning frameworks. Instead, there are many implementations of Single Shot Detector (SSD) on various deep learning frameworks, which is the predecessor model of VGG-RRC. The following websites can be referred:

-	https://github.com/NVIDIA/DeepLearningExamples/tree/master/PyTorch/Detection/SSD
-	https://github.com/qfgaohao/pytorch-ssd
-	https://github.com/balancap/SSD-Tensorflow
-	https://github.com/pierluigiferrari/ssd_keras
-	https://towardsdatascience.com/implementing-ssd-in-keras-part-i-network-structure-da3323f11cff

Note that the structures of SSD and VGG-RRC are very similar. One of the only differences between the two models is whether the up-convolution operation on feature maps is used. Therefore, following the details mentioned in the websites above, it is possible to get an idea on how to write down VGG-RRC or Resnet-RRC from SSD using any one of well-known deep learning frameworks such as Pytorch or Tensorflow.
