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

The VGG-RRC is the previously proposed model that uses a pre-initialized VGG16 base net. The VGG-RRC is written using Caffe framework (https://github.com/xiaohaoChen/rrc_detection). ResNet-RRC, on the other hand, uses a pre-initialized ResNet18 base net. The remaining layers of ResNet-RRC are the same as those of VGG-RRC.

The connection pattern between base layers and rolling layers of ResnNet-RRC can be found out by analyzing such connection pattern of VGG-RRC. In the VGG16 base net part of the VGG-RRC, conv4_3 is the final convolution layer of the second-last convolutional block, and fc7 is the second-last non-pooling layer of all VGG16 layers. conv4_3 is connected to conv4_3r, the first rolling layer, and fc7 is connected to fc7r, the second rolling layer. Here, fc7_relu which is the layer that performs ReLU activation on the output of fc7 is branched out and connected to conv6_1, the first intermediate layer. From this, we can infer that res4b, res5b, and res5b_relu can be used for connection. res4b is connected to conv4_3r since it is the final convolution layer of the second-last convolutional block, and res5b is connected to fc7r since it is the second-last non-pooling layer of all Resnet18 layers. res5b_relu is the layer that performs ReLU activation on the output of res5b, so it is branched out and connected to conv6_1, the first intermediate layer.

The most intuitive way to write down code for Resnet-RRC is to first refer to the VGG-RRC git repository (https://github.com/xiaohaoChen/rrc_detection), which uses Caffe framework with user-defined layers. After cloning the git repository and using it as the workspace, we need to modify it so that it constructs and trains Resnet-RRC instead of VGG-RRC. Here, `examples/car/rrc_kitti_car.py` constructs the VGG-RRC for training. It has multiple occurrences of `VGGNetBody()` which instantiates VGG16 base layers. We can replace these function calls to `ResnetBody()` so that it instantiates Resnet18 base layers instead. Note that `ResnetBody()` needs to be defined in `python/caffe/model_libs.py` of the cloned git repository. Then, we need to modify `AddExtraLayers()` function, defined in `examples/car/rrc_kitti_car.py`, so that it appends the proper intermediate and rolling layers to Resnet18. Pre-trained weights of Resnet18 can also be downloaded as well (https://github.com/HolmesShuan/ResNet-18-Caffemodel-on-ImageNet). To use the pre-trained model, line 107 of `examples/car/rrc_kitti_car.py` needs to be modified so that it points to the correct pathname of the pre-trained Resnet18. After the source code modification is finished, the instructions on the VGG-RRC git repository web page can be followed to compile the Caffe source, prepare datasets, build the Resnet-RRC, and then train or test the Resnet-RRC.

If Resnet-RRC is to be used for detecting taillights instead of cars, the files in `data/KITTI-car` needs to be modified as well so that taillight labels of input dataset are recognized correctly. `data/KITTI-car/labelmap_voc.prototxt` should contain label map entry for taillights, and `data/KITTI-car/extract_car_label.sh` should be modified so that it extracts taillight labels instead of car labels. After the file modification is finished, the instructions on the VGG-RRC git repository web page can be followed to compile the Caffe source, prepare datasets, build the Resnet-RRC, and then train or test the Resnet-RRC on datasets with taillight labels.

It should be noted that if Resnet-RRC is to be implemented this way, the Caffe framework C/C++ source to be compiled from the workspace only. All other distributions of Caffe source will **NOT** work since they lack the user-defined layers necessary for the VGG-RRC or Resnet-RRC. In addition to this, Resnet-RRC is highly dependent on system environment which needs to be manually configured one by one. Besides the hardware and software specifications mentioned in our paper, the Resnet-RRC is tested and found to be working properly only if the following versions of tools and libraries are given:

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
