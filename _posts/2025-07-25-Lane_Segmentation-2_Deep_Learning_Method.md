---
layout: post
title: Part 2 Deep Learning Approach to Lane Segmentation
category: A_Study_on_the_Trade-offs_Between_Performance,_Robustness_and_Efficiency_of_Computer_Vision_Systems
---

## From Rule-Based to Learning-Based Lane Detection

To handle more complex scenarios encountered in real-world driving, I turned to semantic segmentation using deep learning based models. 

This modern machine learning technique makes use of a pretrained model. Models such as LaneNet, SCNN, DeepLabV3+, CLRN, are usually used for lane detection tasks. Similar to the choice of dataset, the choice of model must suit the task to be accomplished. In this project, the DeepLabV3 + model is used as it demonstrates strong performance in pixel-level semantic segmentation. It uniquely combines Atrous Spatial Pyramid Pooling (ASPP) and Encoder-Decoder Architecture. ASPP enhances the model’s ability to capture objects of different sizes and thus, both fine details and large structures can be recognized. The encoder uses a backbone model such as ResNet or Xception to extract features such as edges, textures and shapes. The decoder then refines the segmentation map by integrating low-level spatial details from early layers with high-level semantic information. Finally, a classifier assigns class probabilities to each pixel, and the class with the highest probability is selected for each pixel to generate the final segmentation mask. [1]. The DeepLabV3+ model has been trained on Common Objects in Context (COCO) train2017, on the 20 categories that are present in the Pascal VOC dataset [2]. The input data to the model should consist of RGB images and the target should consist of single-channel grayscale images where each pixel represents an integer class label. In this project, this pretrained model is trained on the KITTI Road dataset and the hyperparameters are
finetuned using Optuna.

## Data Preprocessing

Before using the KITTI dataset segmentation masks for training the DeepLabV3+ model, the RGB masks need to be preprocessed. These RGB masks should be converted into single-channel grayscale masks. This conversion is achieved by mapping each RGB pixel in a mask to its corresponding class ID, as specified in the color map of the dataset. The resulting masks contains pixel values ranging from 0 to 12, which may not be very visible to the naked eye due to the low contrast between these values. The KITTI dataset is split into training, validation and testing sets with a ratio of 60:20:20.

## Architecture Overview

The UML class diagram below illustrates the relationships between core components like the dataset loader, lane segmentation model, and training pipeline.

<div align="center">
  <img src="{{ site.baseurl }}/images/lane-segmentation/uml_class_diagram.png" alt="UML class diagram">
</div>


## Training

### Hardware used for training

The model was trained using a GPU to accelerate the training process. GPUs from the AI Developer Cloud, Lambda [3], were used. PyTorch was configured to use CUDA. GPUs accelerate operations such as convolutions and backpropagation as required for processing large volumes of high-resolution images.

### Conda environment setup

A Conda environment was set up to manage the project’s dependencies, including PyTorch, torchvision,OpenCV and other necessary libraries. The environment was created and exported to a ’.yml’ file. This environment was activated and used on each instance.

### Dataset download

The KITTI dataset was downloaded using a Bash script which automated the process by fetching the required zip file from its download URL and extracting the contents into organized folders, simplifying the dataset acquisition process on each instance.

### Training configuration

The trainer consists of loading the DeepLabV3+ model with ResNet101 backbone from Pytorch. It is initialized with pretrained weights from COCO dataset. The classifier layer was modified to include 13 neurons, each corresponding to a class as defined in the color map. 

The input images and preprocessed masks are transformed and loaded into train loader, validation loader and test loader using PyTorch. Cross-entropy loss is used to evaluate the performance of the model on the training set. The Adam optimizer is used for gradient backpropagation. The model and data loaders are moved to the GPU to accelerate training.

For each epoch, the losses on the training set are calculated and minimized. Then the loss on the validation set is calculated. The training and validation losses for each epoch are saved to training logs (.txt files) for post-processing analysis. The model weights are saved at the end of each epoch, enabling recovery of the best model if needed. The hyperparameters include the batch size, the learning rate, and the number of epochs. 

The training process is summarized in the following sequence diagram.

<div align="center">
  <img src="{{ site.baseurl }}/images/lane-segmentation/uml_sequence_diagram.png" alt="UML Sequence Diagram">
</div>



## Hyperparameter Finetuning with Optuna

Optuna is used for hyperparameter fine-tuning. It is an open-source hyperparameter optimization algorithm that is used to automate hyperparameter search [4]. It requires as input an upper bound and a lower bound for each hyperparameter and the number of trials. This method was chosen over conventional methods such as Grid Search or Random Search since Optuna dynamically adjusts its search strategy based on past trials and refines its search over time. On the other hand, Grid Search tests all combinations systematically, requiring large computing resources, and Random Search might take more time since arbitrary values for hyperparameters are chosen for each experiment. Optuna offers the option of stopping unpromising trials
early using pruners. Its exploration-exploitation balance ensures a more broader, more efficient search. The
range chosen for the hyperparameters is as follows:

• The batch size is chosen from the set 8, 16, 32, 64.

• The learning rate is chosen from the range [1e-5, 1e-1].

• The number of epochs is an integer chosen from the range [10, 50].

The number of trials performed is 20 due to constraints in computational resources. The Optuna study consists of defining an objective function which finds the set of hyperparameters which minimizes the loss on the validation set on the last training epoch. This results in the optimization history below, where the validation loss of the last epoch for each trial is plotted.

<div align="center">
  <img src="{{ site.baseurl }}/images/lane-segmentation/optuna_history.png" alt="Optuna history">
</div>


The hyperparameters that minimize the final validation IoU is from trial 12:

• Batch size = 8

• Learning rate = 1.46e-4

• Number of epochs = 31

The evolution of training loss, validation loss and IoU on the validation set during the training of the best model is shown below.

<div align="center">
  <img src="{{ site.baseurl }}/images/lane-segmentation/loss_history.png" alt="Loss">
</div>


The validation loss is higher than the training loss but both decrease steadily as the number of epochs increases. The IoU on the validation set improves over time, eventually stabilizing at 0.84. Model training and finetuning were done on the NVIDIA A10 GPU (24 GB VRAM), with 226 GiB RAM and 1.3 TiB SSD, rented from Lamda, at a cost of USD 0.75/hour. Training of a model was completed in approximately 11 minutes while the hyperparameter tuning study lasted for around 2.5 hours.


## Results

The mean IoU for the lane class on the test set is 0.870 for the deep-learning based model. Overall, the modern method demonstrates good performance. It achieves high accuracy and shows significant
potential in diverse conditions. The lane area is consistently segmented and classified correctly. However, the boundaries between classes present in the predicted segmentation masks are not as sharp as the true masks.

## Comparison of Classical and Deep Learning Approaches

<figure style="text-align: center; margin-bottom: 30px;">
  <img src="{{ site.baseurl }}/images/lane-segmentation/comparison_input.png" alt="Input image" style="max-width: 100%; height: auto;">
  <figcaption style="margin-top: 8px; font-style: italic;">Input Image</figcaption>
</figure>

<figure style="text-align: center; margin-bottom: 30px;">
  <img src="{{ site.baseurl }}/images/lane-segmentation/comparison_classical.png" alt="Classical output" style="max-width: 100%; height: auto;">
  <figcaption style="margin-top: 8px; font-style: italic;">Classical Lane Detection Output</figcaption>
</figure>

<figure style="text-align: center; margin-bottom: 30px;">
  <img src="{{ site.baseurl }}/images/lane-segmentation/comparison_modern.png" alt="Modern output" style="max-width: 100%; height: auto;">
  <figcaption style="margin-top: 8px; font-style: italic;">Modern Deep Learning-Based Detection Output</figcaption>
</figure>

<figure style="text-align: center; margin-bottom: 30px;">
  <img src="{{ site.baseurl }}/images/lane-segmentation/comparison_true.png" alt="Ground Truth" style="max-width: 100%; height: auto;">
  <figcaption style="margin-top: 8px; font-style: italic;">Ground Truth Mask</figcaption>
</figure>

The modern, deep-learning based method achieves better performance compared to the classical image processing pipeline previously explored. It is capable of handling occlusions, complex road conditions and diverse road layouts. This is an area where the classical method struggles. It has limited generalization capabilities. 

To improve the performance of the classical method, further fine-tuning of parameters could be wxplored,for example, the filter size of the Gaussian blur, the coordinates of the region of interest, the upper and lower bounds for Canny Edge detection and the threshold for the Hough Line transform. For the modern method, increasing training epochs and optimizing hyperparameters may yield more precise segmentation with clearer boundaries, provided additional computational resources are available. 

However, the limitations of the dataset used must be considered. The KITTI dataset might contain biases. Since it was collected from roads in Karlsruhe, Germany, it inherently contains geographical bias in terms of the weather conditions experienced only in Germany, and the road layouts there. Furthermore, biases might be introduced because of pictures collected in urban and suburban environments, clear weather, and good lighting. Sensor characteristics from the cameras used might also lead to biases. Besides, it is a relatively small dataset, containing only 200 images. To improve generalization, the model could be further trained on alternative datasets, such as A2D2, or a combination of multiple datasets. The use of larger datasets might improve the model performance.

## Conclusion

In practical applications, while the modern approach is more accurate, choosing between classical and modern methods depends on several factors. The classical approach is quicker to implement and more cost-effective, while the modern method requires extensive computational resources and training time. However, its superior accuracy and optimized inference speed make it a compelling choice for high-performance lane segmentation. Ultimately, the best approach depends on the use case while balancing accuracy, cost and computation power. This balance is crucial for developing deployable lane segmentation technologies that contribute to safer and more efficient intelligent transport systems.

## References

[1] L.-C. Chen, Y. Zhu, G. Papandreou, F. Schroff, and H. Adam, “Encoder-decoder with atrous separable convolution for semantic image segmentation,” arXiv, 2018.

[2] “Deeplabv3 — pytorch.org,” https://pytorch.org/hub/pytorch vision deeplabv3 resnet101/.

[3] Lambda, “Gpu compute for ai.” [Online]. Available: https://lambda.ai/

[4] Optuna. [Online]. Available: https://optuna.org/
