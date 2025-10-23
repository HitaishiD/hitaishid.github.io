---
layout: post
title: Part 4 Calculating Driver Score
category: Real-time_Driver_Scoring_with_Lane_Segmentation
---

## From Segmentation Mask to Driver Score

Using the segmentation mask obtained as output from the segmentation model, we can extract the vehicle position as follows: 
1. We use the last row of pixel of each image and extract the leftmost and rightmost x-coordinate of the area corresponding to the 'lane' class. 
2. We find the road center by averaging these two values. 
3. We estimate the vehicle center point to be at the center of the image. 
4. We compare the vehicles center point with the road boundaries: 

road center point < vehicle center point < road right boundary → vehicle is in lane

<div align="center">
  <img src="{{ site.baseurl }}/images/lane-segmentation/vehicle_position.png" alt="vehicle position extraction">
</div>


### Demo

<div style="display: flex; justify-content: center; gap: 20px; flex-wrap: wrap;">
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/3P5XXG6ks5SQ?autoplay=1&loop=1&playlist=3P5XXG6ks5SQ&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  </div>
  <div style="flex: 1 1 45%; aspect-ratio: 16 / 9;">
    <iframe src="https://www.youtube.com/embed/U-eZoy9vz1Q?autoplay=1&loop=1&playlist=U-eZoy9vz1Q&mute=1&controls=0&modestbranding=1&rel=0"
            style="width: 100%; height: 100%;"
            frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Using the in-lane and out-of-lane data extracted, we calculate the driver score using the equation below: 

$$Driver Score Behavior = \frac{Time out of lane}{Total time of journey} * 100$$


## Deployment
This system can be deployed using a dashcam to capture real-time video, which feeds into a Jetson device for making predictions based on our trained model.

## Areas of Improvement
There are a few key areas where the system can be enhanced:

* Lane Segmentation: Currently, the lane area is treated as a single entity. Segmenting it into two separate lanes would improve detection accuracy.

* Score Calculation: Including weights in the score calculation could make predictions more reliable and reflective of real-world conditions, for example, when the speed of the vehicle is high, the time taken for an overtake is lower and the score must be improved to reflect this increased risk.
