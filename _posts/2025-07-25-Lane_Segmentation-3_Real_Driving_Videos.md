---
layout: post
title: Part 3 Running my Lane Segmentation Model on Real Driving Videos
category: A_Study_on_the_Trade-offs_Between_Performance,_Robustness_and_Efficiency_of_Computer_Vision_Systems
---

## Backstory

Now that I have a DeepLabV3+ model with a ResNet backbone, fine-tuned on the KITTI Road dataset, it is time to see how well it performs on real-world driving videos.

My previously trained model achieved a mean IoU of 0.87 on test data. It showed promising results. The segmented lanes were clean and consistent. Clear boundaries were detected between different classes present in an image. However, this testing was done using the test set of the same dataset on which the model was trained.

Now, we want to test the generalization capabilities of the model, i.e., analyze how the model performs with unseen driving videos, under different conditions and from different sources. This is important as it helps to evaluate the robustness and practical usability of the model in real-time lane segmentation use cases.
## Setting up 

### Scenarios covered
<br> To simulate a broad range of scenarios encountered while driving in real-life, I defined five distinct conditions:
1. urban 
2. rural
3. rain
4. night
5. snow

Videos corresponding to these conditions were sourced from YouTube and trimmed down to a duration of approximately two minutes per scenario.
### Preprocessing

Each video is read frame by frame using OpenCV. Each frame is preprocessed to match the format expected by the KITTI-trained model. The frames resized to 256x256, normalized and converted into input tensors.  

### Inference pipeline?

The trained parameters of the DeepLabV3+ model are stored on a Google Cloud Storage bucket. They are retrieved and the model is loaded. 

The preprocessed frames are passed through the model to generate segmentation masks. The NVIDIA RTX 4090 GPU was used for inference. The masks are converted into RGB colour space and overlaid on the original frame for visualization. 

Finally, the annotated frames are written to an output video using OpenCV, which allows for easy playback and comparison across different driving conditions.

### Results

Urban scenario

<div style="display: flex; justify-content: center; gap: 20px; flex-wrap: wrap;">
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/JfdFtId3ftE?autoplay=1&loop=1&playlist=JfdFtId3ftE&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  </div>
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/ptUX-Qoy_4g?autoplay=1&loop=1&playlist=ptUX-Qoy_4g&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  </div>
</div>


Rural scenario 

<div style="display: flex; justify-content: center; gap: 20px; flex-wrap: wrap;">
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/KK8SbAfiqBg?autoplay=1&loop=1&playlist=KK8SbAfiqBg&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  </div>
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/JabNbPih0m8?autoplay=1&loop=1&playlist=JabNbPih0m8&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  </div>
</div>


Rain scenario 

<div style="display: flex; justify-content: center; gap: 20px; flex-wrap: wrap;">
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/WM2NDPIasQY?autoplay=1&loop=1&playlist=WM2NDPIasQY&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  </div>
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/9kIS59TgEes?autoplay=1&loop=1&playlist=9kIS59TgEes&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  </div>
</div>


Night scenario

<div style="display: flex; justify-content: center; gap: 20px; flex-wrap: wrap;">
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/UA5Udtf7668?autoplay=1&loop=1&playlist=UA5Udtf7668&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  </div>
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/EbRHXdINw4g?autoplay=1&loop=1&playlist=EbRHXdINw4g&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  </div>
</div>


Snow scenario 

<div style="display: flex; justify-content: center; gap: 20px; flex-wrap: wrap;">
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/3bZ3VLXlB3o?autoplay=1&loop=1&playlist=3bZ3VLXlB3o&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  </div>
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/GnsajjIjw1Y?autoplay=1&loop=1&playlist=GnsajjIjw1Y&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  </div>
</div>

## Analysis of model performance
* Where it worked and where it failed?
* FPS metric
* Mask flickering
* Others

**Lane Segmentation**

In most conditions, i.e., urban, rural and rain, the lane area and sidewalk are correctly segmented. However, the model struggles in correctly capturing the lane area with night and snowy conditions. In the snow condition, the true lane area and snowy sidewalk are not distinguished; they are both classified as lane. 

**Semantic Misclassification**

The key observation made from the segmentation results is that on real driving videos, vehicles are segmented correctly but are consistently misclassified as "person" (coloured red and not blue). This occurs even though the "vehicle" class was represented in the original training data. In fact, the "person" class was rare compared to the "vehicle" class in the training set. It must be noted that the "vehicle" class was correctly classified on the KITTI test set. Thus, this might be a classic example of domain shift.

KITTI dataset was collected from roads in Karlsruhe, Germany. It inherently contains biases in terms of weather conditions, road layout, lighting and camera perspective. On the other hand, YouTube videos exhibit different camera angles, lighting, weather conditions and image quality. Therefore, the visual appearance of objects has changed, even if the semantic meaning (class) is unchanged. This visual difference in training and inference domain is commonly known as visual domain shift. It causes the model to fail to generalize and make semantic misclassifications. 

**Inference Speed**

The key metric used in performance evaluation is the video frame rate, often referred to as FPS (frames per second). The FPS of a video is the number of images (frames) captured or displayed per second. 

To evaluate the model, we use two main properties: 
1. End-to-end FPS: 
This is the speed of the total processing pipeline, which includes reading input video stream, preprocessing, model inference, postprocessing and writing output video stream. To calculate it, the time taken from the beginning of reading the first frame of the video stream until the last frame is written to the output video stream is recorded and divided by the total number of frames. This metric is important in real-time systems where output latency must be minimized. 

2.  Mean inference time:  
This is the inference speed of the model, i.e., the average time taken for the model to generate a segmentation mask. This metric enables comparison between models. 

The FPS of the original videos is 30. The end-to-end FPS is on average 51.8 while the mean inference time is 0.01 s. This means that the model generates a segmentation for each frame in just 1 ms and the full processing pipeline runs faster than the original video frame rate on the GPU used.

This shows that the DeepLabV3+ model is highly performant on the NVIDIA RTX 4090 GPU, motivating its usage for real-time applications. 
