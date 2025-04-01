# ResNet-50 training on Imagenet-1k

This is a PyTorch implementation of training a ResNet50 model from scatch on the Imagenet-1k dataset on AWS, as part of The School of AI ERA-V3 course Session 9.

## About ImageNet

ImageNet is a large dataset of images with over 14 million images and 22,000 categories. It is used for training and testing deep learning models, particularly for image classification tasks.
For this project, I used ImageNet-1k is a subset of ImageNet with 1000 categories that has become a standard in computer vision.

[Stanford Dawn ImageNet Training Leaderboard](https://dawnd9.sites.stanford.edu/imagenet-training)

![Imagenet-1k dataset](images/imagenet_images.png)
*ImageNet sample classes and images*

## About ResNet50

ResNet50 is a deep convolutional neural network architecture that was proposed by Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun in their 2015 paper "Deep Residual Learning for Image Recognition." It is a variant of the ResNet family of models, which are known for their ability to train very deep networks effectively.

![ResNet 50 Architecture](images/resnet-arch.webp)
*Residual Networks (ResNet) model architecture*

## Objective

### 1. AWS EC2 Instance setup

- Attempt 1:
  - Machine: AWS EC2 g4dn.2xlarge

  # TO BE COMPLETED 



### 2.Data Download and Extracttion

- #### ImageNet-Mini
  - Source: (https://www.kaggle.com/datasets/ifigotin/imagenetmini-1000)
  - Download via Kaggle CLI:
    ```bash
    kaggle datasets download ifigotin/imagenetmini-1000
    ```

- #### ImageNet-1k 
  - Source: (https://www.kaggle.com/c/imagenet-object-localization-challenge/)
  - Download via Kaggle CLI:
  ```bash
  kaggle competitions download -c imagenet-object-localization-challenge
  ```

  - Download via TBD

- #### Extraction
  - 