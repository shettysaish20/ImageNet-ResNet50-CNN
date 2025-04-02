[![made-with-pytorch](https://img.shields.io/badge/Made%20with-PyTorch-red.svg)](https://pytorch.org/)
[![Open Source Love](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)]()

# ResNet-50 training on Imagenet-1k

This is a PyTorch implementation of training a ResNet50 model from scatch on the Imagenet-1k dataset on AWS EC2 instance, as part of The School of AI ERA-V3 course Session 9.

[ResNet50 ImageNet AWS Demo: HF Spaces](https://huggingface.co/spaces/saish-shetty/ResNet50-Imagenet-AWS-Demo)

Huge shoutout to **Rohan Shravan** sir, **Rakesh Raushan** and the students of ERA-V3 who helped me through all the issues I faced while executing this project.

## About ImageNet

ImageNet is a large dataset of images with over 14 million images and 22,000 categories. It is used for training and testing deep learning models, particularly for image classification tasks.
For this project, I used ImageNet-1k is a subset of ImageNet with 1000 categories that has become a standard in computer vision.

[Stanford Dawn ImageNet Training Leaderboard](https://dawnd9.sites.stanford.edu/imagenet-training)

![Imagenet-1k dataset](images/imagenet_images.png)
*ImageNet sample classes and images*

### ImageNet-1k Dataset: Pros and Cons

#### Pros:

-   **Standard Benchmark:** Widely used for evaluating image classification models.
-   **Large Scale:** Contains a significant number of images and diverse categories.
-   **Rich Annotations:** Provides accurate labels for training supervised models.
-   **Pre-trained Models:** Many pre-trained models are available, facilitating transfer learning.
-   **Research Advancement:** Has driven significant advancements in computer vision research.

#### Cons:

-   **Complexity:** Training from scratch can be computationally expensive and time-consuming.
-   **Bias:** May contain biases due to the data collection process.
-   **Annotation Errors:** Some images may have inaccurate or ambiguous labels.
-   **Not Fully Representative:** May not cover all real-world scenarios or domains.
-   **Licensing and Access:** Access to the full dataset may require specific permissions or fees.

## About ResNet50

ResNet50 is a deep convolutional neural network architecture that was proposed by Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun in their 2015 paper "Deep Residual Learning for Image Recognition." It is a variant of the ResNet family of models, which are known for their ability to train very deep networks effectively.

![ResNet 50 Architecture](images/resnet-arch.webp)
*Residual Networks (ResNet) model architecture*

## Steps

### 1. Data Download and Extracttion

- #### ImageNet-Mini
  - Size: **~4 GB**
  - Source: (https://www.kaggle.com/datasets/ifigotin/imagenetmini-1000)
  - Download via Kaggle CLI:
    ```bash
    kaggle datasets download ifigotin/imagenetmini-1000
    ```

- #### ImageNet-1k
  - Size: **156 GB**
  - Source 1: (https://www.kaggle.com/c/imagenet-object-localization-challenge/)
    - Download via Kaggle CLI:
    ```bash
    kaggle competitions download -c imagenet-object-localization-challenge
    ```

  - Source 2: (https://www.kaggle.com/datasets/mayurmadnani/imagenet-dataset)
    - Download via Kaggle CLI:
    ```bash
    kaggle datasets download mayurmadnani/imagenet-dataset
    ```
    - Download via cURL
    ```bash
    curl -L -o dataset.zip --header "Authorization: Bearer $(jq -r .key ~/.kaggle/kaggle.json)" "https://www.kaggle.com/api/v1/datasets/download/mayurmadnani/imagenet-dataset"
    ```

- #### Issues with Kaggle CLI command
  - EC2 instances attempted:
    - g4dn.2xlarge
    - g4dn.8xlarge
    - t2.xlarge
    - g4dn.12xlarge
  - The kaggle command would run for sometime and then either the EC2 server would crash or the command would get killed. This happened multiple times
  ![Dataset download issue](Training_log_images\EC2_dataset_download_issue.png)
  *Kaggle download command issue*

  - After multiple attempts and serious debugging, I figured out the reason. Kaggle API was downloading the dataset **first to Memory** and not writing it to disk, hence causing my server to crash due to OOM issue. Even using a 192GB memory g4dn.12xlarge EC2 instance was not enough as the kaggle API was consuming more than 155 GB (dataset volume), possibly doing some more operations before writing to disk
  ![EC2 Memory usage](Training_log_images\EC2_memory_usage_3.png)
  *EC2 memory monitoring using htop*

  - Using both Wget/cURL on Source 2 dataset was allowing concurrent writing operations to Disk as the data was getting downloaded, putting less pressure on Server's memory

- #### Unzipping the dataset
  ```bash
  unzip dataset.zip
  ```

### 2. Training

  

