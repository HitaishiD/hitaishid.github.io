---
layout: post
title: Part 1 Lane Segmentation with Classical Image Processing Techniques
category: A_Study_on_the_Trade-offs_Between_Performance,_Robustness_and_Efficiency_of_Computer_Vision_Systems
---

## Introduction

This project focuses on lane detection, a critical task in computer vision which involves identifying the lane area in images or video. Effective lane detection must account for various challenging conditions such as poor lighting, glare, complex road layouts, diverse scenes and varying weather. Lane segmentation plays a vital role in automotive engineering, robotics and intelligent transport systems. The primary motivation behind accomplishing this task is its application in Advanced Driver Assistance Systems (ADAS) and autonomous vehicles. By performing lane detection accurately, these algorithms enhance road safety for humans and increases the reliability of self-driving vehicles.

There exist three main types of image segmentation: semantic segmentation, instance segmentation and panoptic segmentation. This is illustrated below. 

1. **Semantic segmentation** involves classifying each pixel in an image to a specific class or category (e.g., car, road, building, etc.) without differentiating between individual instances. All pixels of the same class are assigned the same label.
2. **Instance segmentation** not only labels pixels but also distinguishes between different objects of the same class. In an image with multiple cars, instance segmentation can differentiate each car as a separate entity, even though all are part of the "car" class.
3. **Panoptic segmentation** combines both semantic and instance segmentation. It assigns a semantic label to every pixel, while also distinguishing between different instances of the same class. Thus, panoptic segmentation offers a unified view that captures both the object class and its individual instances.


![Types of Image Segmentation|500](/images/lane-segmentation/segmentation_types.jpg)
<div style="text-align: center;">Types of Image Segmentation [1] </div>

## Dataset 

There exists various open source datasets used for benchmarking models for lane segmentation tasks. Among those, the datasets explored for use in this project are A2D2, CULane, AppolloScape, CityScapes and KITTI Road. Since the datasets provide different types of annotations such as pixel-level semantic segmentation masks, lane boundaries and lane markings, the choice of the dataset for this project depends on the desired task to be performed by the model. In this project, the KITTI Road dataset is used [2]. The semantic segmentation dataset consists of 200 training images and the corresponding semantic segmentation masks in ’png’ format, with a size of approximately 30 GB. The training images are taken from urban roads in
Karlsruhe, Germany. Below is an example of a training image and its corresponding segmentation mask.

<div align="center">
  <img src="{{ site.baseurl }}/images/lane-segmentation/dataset_input_eg.png" alt="Input image" style="display:inline-block; margin: 0 10px;">
  <img src="{{ site.baseurl }}/images/lane-segmentation/dataset_output_eg.png" alt="Segmented output" style="display:inline-block; margin: 0 10px;">
</div>


The KITTI Road dataset consists of 34 defined classes with their corresponding RGB color and class
ID (or label). However, only 13 of the predominant classes are used in this project, among which is the
’road’ class, to facilitate computations. The color map used is given below. It must be noted that class 0
corresponds to background or unlabeled regions.

#### Color Map Dictionary

```python
color_map = {
    (128, 64, 128): 1,  # Road
    (244, 35, 232): 2,  # Sidewalk
    (70, 70, 70): 3,    # Building
    (102, 102, 156): 4, # Wall
    (190, 153, 153): 5, # Fence
    (107, 142, 35): 6,  # Vegetation
    (152, 251, 125): 7, # Terrain
    (70, 130, 180): 8,  # Sky
    (220, 20, 60): 9,   # Person
    (0, 0, 142): 10,    # Car
    (119, 11, 32): 11,  # Bicycle
    (0, 0, 230): 12,    # Motorcycle
    (0, 0, 0): 0,       # Background
}
```

## Evaluation Metric
The metric used to evaluate model performance is Intersection over Union (IoU). IoU, also known as the Jaccard Index, is a common evaluation metric used for tasks such as segmentation, object detection and tracking. It is calculated using the equation below: 

$$
\text{IoU} = \frac{\text{Intersection of Predicted and Ground Truth Pixels}}{\text{Union of Predicted and Ground Truth Pixels}}
$$

Since lane detection is the task performed in this project, the IoU is calculated specifically for the lane class. The numerator represents the total number of pixels where the predicted lane class matches the true lane class. The denominator is the sum of all pixels classified as the lane class in both the prediction and ground truth. The IoU metric is computed for each image and the mean IoU (mIoU) across the test set is obtained by averaging these values. It ranges from 0 to 1, with higher values indicating better performance. 

## Classical Approach

### Overview
We first explore a classical approach using traditional image processing techniques applied sequentially using OpenCV. Traditional image segmentation techniques include k-means clustering and thresholding [3]. However, these techniques come with inherent limitations, for example, the threshold and other parameters must be manually chosen. Therefore, a two-step process was implemented: 
1. Lane boundary detection is performed using a combination of computer vision techniques sequentially applied such as color space conversion, smoothing, edge detection, and Hough transform. This was proposed in [4].
2. Once the lane boundaries are detected, the left and right lines bounding the lane area are joined to form a polygon. This area is filled to represent the detected lane area.

This method does not inherently output a segmentation mask where each pixel is classified into a class. Instead, the above process is used to detect lane boundaries only and subsequently the lane area. 

### Methodology  


The classical method is outlined as follows: 

**Load the RGB image:** The RGB image is loaded with OpenCV using its file path.
<div align="center">
  <img src="{{ site.baseurl }}/images/lane-segmentation/0-input-image.png" alt="Input image">
</div>

**Color space conversion:** The image is converted from RGB to HSL (Hue, Saturation, Lightness). The HSL color space helps to better isolate colors like yellow and white which are commonly used for lane marking. 
<div align="center">
	<img src="{{ site.baseurl }}/images/lane-segmentation/1-hsl.png" alt="Input image">
</div>

**Masks application:** The range of yellow and white colors is chosen and masks are created for these pixels. These masks are combined and applied to the original image. This helps to filter out irrelevant colors from the image such as road surface, sky, and to focus on the lane area while reducing distractions. It retains spatial information while emphasizing on the lanes.
<div align="center">
	<img src="{{ site.baseurl }}/images/lane-segmentation/2-mask.png" alt="Input image">
</div>

**Grayscale conversion:** Image processing techniques such as edge detection are easier and more effective on grayscale images. This is because it simplifies the calculation of gradients and intensity changes. Therefore, the processing pipeline consists of this step of converting masked images to grayscale.
<div align="center">
	<img src="{{ site.baseurl }}/images/lane-segmentation/3-grayscale.png" alt="Input image">
</div>

**Gaussian blur:** Gaussian blur is a linear filter which helps to smooth the image, by reducing noise and small details that could result in superfluous edges detected further on. This results in the removal of low intensity edges in road images. A Gaussian filter of size 5x5 is used. 
<div align="center">
	<img src="{{ site.baseurl }}/images/lane-segmentation/4-gaussian-blur.png" alt="Input image">
</div>

**Canny edge detection:** This step detects edges by finding regions with significant changes in intensity. It can be used after a 5x5 Gaussian filter has been applied on an image. It requires a high and a low threshold value so that edges with intensity higher than the upper threshold are classified as edges with certainty and those below the lower threshold are discarded. Edges lying between the two thresholds are classified as edges or non-edges based on their connectivity.
<div align="center">
	<img src="{{ site.baseurl }}/images/lane-segmentation/5-detect-edges.png" alt="Input image">
</div>

**Region of Interest (ROI):** The ROI focuses on the area of the image where the lanes most likely appear. This is a step filter and we use an ROI of size (0.5 x height) x width, assuming the camera is mounted on a fixed spot on the car such that the ROI is manually validated. 
<div align="center">
	<img src="{{ site.baseurl }}/images/lane-segmentation/6-roi.png" alt="Input image">
</div>

**Hough transform:** Hough transform is a technique used to detect straight lines in an image, even in noisy environments. Edge detection pre-processing is recommended. This method requires a threshold: it keeps track of the intersection between curves of every point in the image. If the number of intersections is above the threshold, then it declares it as a line with the parameters of the intersection point.
<div align="center">
	<img src="{{ site.baseurl }}/images/lane-segmentation/7-hough_transform.png" alt="Input image">
</div>

**Left and right lane boundaries:** Since several lines are detected from the Hough transform stage, the leftmost and rightmost lines are chosen as the lane boundaries based on their x-coordinates.
<div align="center">
	<img src="{{ site.baseurl }}/images/lane-segmentation/8-lanes.png" alt="Input image">
</div>

**Lane area filling:** A polygon is formed from the left and right lane boundaries. It is colored to represent the lane area.
<div align="center">
	<img src="{{ site.baseurl }}/images/lane-segmentation/9-fill-lane.png" alt="Input image">
</div>

### Results

The classical method results in a mean IoU of 0.101 on the KITTI training set. This shows poor performance of this image processing pipeline. This can be explained because the classical method relies on manually defined rules, heuristics and assumptions such as thresholds for edge detection, Region of Interest coordinates to detect lanes. The pipeline does not have a semantic understanding of the classes present in an image. Thus, it lacks flexibility. It works best in perfect conditions where the assumptions used are met but struggles with variations, such as faint or obscured road markings. It is also more sensitive to noise, leading to lower IoU.

Furthermore, this classical method does not have the ability to learn or adjust its prediction based on performance metrics. It relies on static algorithms that lack the capacity to adapt to a wide range of road conditions. Thus, it is not robust for real driving conditions. 

These limitations motivate the use of deep learning models, which can learn complex, semantic features from data and thus, adapt to diverse and challenging road scenarios, offering significantly improved performance and robustness.


## References

[1] V7Labs, “Image segmentation: Deep learning vs traditional [guide].” [Online]. Available:
https://www.v7labs.com/blog/image-segmentation-guide

[2] “The KITTI Vision Benchmark Suite — cvlibs.net,” https://www.cvlibs.net/datasets/kitti/eval semseg.
php?benchmark=semantics2015.

[3] W. Chen, W. Wang, K. Wang, Z. Li, H. Li, and S. Liu, “Lane departure warning systems and lane line detection methods based on image processing and semantic segmentation: A review,” Journal of Traffic and Transportation Engineering (English Edition), vol. 7, no. 6, pp. 748–774, Dec. 2020, doi: https://doi.org/10.1016/j.jtte.2020.10.002.

[4] N. Lakhani, R. Karande, and V. Ramakrishnan, “LANE DETECTION USING IMAGE PROCESSING IN PYTHON.” Available: https://www.irjet.net/archives/V9/i4/IRJET-V9I4148.pdf.
‌
‌



