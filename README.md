[![made-with-pytorch](https://img.shields.io/badge/Made%20with-PyTorch-red.svg)](https://pytorch.org/)
[![Open Source Love](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)]()

# ResNet-50 training on Imagenet-1k using AWS

This is a PyTorch implementation of training a ResNet50 model from scatch on the Imagenet-1k dataset on AWS EC2 instance, as part of The School of AI ERA-V3 course

[ResNet50 ImageNet AWS Demo: HF Spaces](https://huggingface.co/spaces/saish-shetty/ResNet50-Imagenet-AWS-Demo)

Huge shoutout to **Rohan Shravan** sir, [**Rakesh Raushan**](https://github.com/Rakesh-Raushan) and the students of ERA-V3 who helped me throughout this project.

## About ImageNet

ImageNet is a large dataset of images with over 14 million images and 22,000 categories. It is used for training and testing deep learning models, particularly for image classification tasks.
For this project, I used **ImageNet-1k**, which is a subset of ImageNet with 1000 categories that has become a standard in computer vision.

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

![ResNet 50 Architecture](images/ResNet_50_architecture.png)
*Residual Networks (ResNet) model architecture - multiple versions*

ResNet50 consists of 48 Convolutional layers, 1 MaxPool layer, and 1 Average Pool layer, followed by a fully connected layer. The model uses skip connections to solve the vanishing gradient problem.

Key components:
- Input: 224x224x3 images
- Output: 1000 classes (ImageNet-1K)
- Total Parameters: 25.6M


## Steps

### 1. Dataset Download and Extraction

- #### ImageNet-Mini
  - Size: **~4 GB**
  - Source: https://www.kaggle.com/datasets/ifigotin/imagenetmini-1000
  - Download via Kaggle CLI:
    ```bash
    kaggle datasets download ifigotin/imagenetmini-1000
    ```

- #### ImageNet-1k
  - Size: **156 GB**
  - Source 1: https://www.kaggle.com/c/imagenet-object-localization-challenge/
    - Download via Kaggle CLI:
    ```bash
    kaggle competitions download -c imagenet-object-localization-challenge
    ```

  - Source 2: https://www.kaggle.com/datasets/mayurmadnani/imagenet-dataset
    - Download via Kaggle CLI:
    ```bash
    kaggle datasets download mayurmadnani/imagenet-dataset
    ```
    - Download via cURL
    ```bash
    curl -L -o dataset.zip --header "Authorization: Bearer $(jq -r .key ~/.kaggle/kaggle.json)" "https://www.kaggle.com/api/v1/datasets/download/mayurmadnani/imagenet-dataset"
    ```
    - Download via wget
    ```bash
    wget --header="Authorization: Bearer $(jq -r .key ~/.kaggle/kaggle.json)" "https://www.kaggle.com/api/v1/datasets/download/mayurmadnani/imagenet-dataset"
    ```

- #### Issues with Kaggle CLI command
  - Storage: 500 GB EBS Volume attached and mounted to an EC2 instance
  - EC2 instances attempted:
    - g4dn.2xlarge
    - g4dn.8xlarge
    - t2.xlarge
    - g4dn.12xlarge
  - The kaggle command would run for sometime and then either the EC2 server would crash or the command would get killed. This happened multiple times
  ![Dataset download issue](Training_log_images/EC2_dataset_download_issue.png)
  *Kaggle download command issue*

  - After multiple attempts and serious debugging, I figured out the reason. Kaggle API was downloading the dataset **first to Memory** and not writing it to EBS Volume, hence causing my server to crash due to OOM issue. Even using a 192GB memory g4dn.12xlarge EC2 instance was not enough as the kaggle API was consuming more than 155 GB (dataset size), possibly doing some more operations before writing to disk
  ![EC2 Memory usage](Training_log_images/EC2_memory_usage_3.png)
  *EC2 Server Memory Monitoring using htop, moments before it crashed*

  - Using either wget/cURL on Source 2 dataset was allowing concurrent writing operations to EBS Volume as the data was getting downloaded, putting less pressure on Server's memory

- #### Unzipping the dataset
  ```bash
  unzip dataset.zip
  ```

### 2. Training

  - #### Single GPU Training
    Infrastructure: **My Local Computer**
      - GPU: NVIDIA RTX 4060 with 16 GB RAM and 8 GB VRAM
      - Storage: 1 TB Local SSD Storage
      - Training Time: **~1.2 hours** per epoch
      - Total Training time: **~60 hours** for 50 epochs
      - Optimizations used:
        - Mixed Precision Training
        - Checkpointing
        - Data Loading Optimizations (Pin Memory and multi-worker data loading)
        - Learning Rate Scheduling using OneCycle LR policy
        - Data Augmentations
      - Metrics:<br><br>
        After 50 epochs
        - Training 
          - Top-1 Accuracy: **71.45%** ✅
          - Top-5 Accuracy: **88.92%** ✅
        - Testing 
          - Top-1 Accuracy: **73.77%** ✅
          - Top-5 Accuracy: **91.87%** ✅
      
      - Logs:
      ![Local ResNet training](Training_log_images/local_training_logs_1.png)

  - #### Multi-GPU Distributed Training
    Infrastructure: **AWS g5.12xlarge EC2 Instance**
    - Was able to create this EC2 instance after getting approved for **48 vCPUS** of "All G and VT Spot Requests" in AWS Mumbai (ap-south-1) region
    - GPU: 4 x NVIDIA A10G Tensor Core GPUs
      - Storage: 500 GB EBS Volume attached and mounted to the EC2 instance
      - Training Time: **~12 minutes** per epoch
      - Total Training time: **~10 hours** for 50 epochs
      - Optimizations used:
        - Distributed Data Parallel Training (to effectively use multiple GPUs at once)
        - Distributed Data Sampling
        - Efficient Data Loading
        - Learning Rate Scheduling using OneCycle LR policy
        - Memory Management
        - Process and Error Management
        - tmux for creating multiple terminal sessions and disconnecting to the server without losing progress 
      - Metrics:<br><br>
        After 50 epochs 
          - Top-1 Accuracy: **76.15%** ✅
          - Top-5 Accuracy: **91.26%** ✅
      - Cost:
        - Dataset download and unzipping: ~6$
        - Training: ~25$
        - Total: **~31$**
      
      - AWS Training Logs 
        ![AWS ResNet Training](Training_log_images/Training_logs_pic_6.png)
        ![AWS ResNet Training](Training_log_images/Training_logs_pic_7.png)

      - Training Monitoring using TensorFlow Board

        ![LR log](Training_log_images/TF_Board/LR_log.png)
        *Learning Rate Log*

        ![Test Accuracy Log](Training_log_images/TF_Board/test_accuracy_log.png)
        *Test Accuracy (Top-1 & Top-5) Log*

### 3. Model Deployment and Inferencing

The model is deployed on HuggingFace and can be inferenced through HF Spaces App: [ResNet50 ImageNet Demo](https://huggingface.co/spaces/saish-shetty/ResNet50-Imagenet-AWS-Demo)

![Demo Image](Inferencing_images/Dog_demo.png)

      






