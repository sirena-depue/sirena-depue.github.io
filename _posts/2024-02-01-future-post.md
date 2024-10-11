---
title: 'Distracted Driver Monitor'
date: December 20th, 2021
permalink: /posts/2012/08/blog-post-3/
tags:
  - CV
---


Overview:
======
The purpose of this project is to create a self-contained system that monitors and alerts the driver when distracted. There are several categories within ‘distracted driving’ that we considered. Using OpenCV and DLib to detect a face and extract facial features from picamera footage, we can detect whether the driver is drowsy, is facing away from the road, or is looking away from the road. In addition, a capacitive touch sensor is used to detect whether the driver’s hands are on the steering wheel. An OBDII scanner is used to log the speed of the vehicle in real-time. To avoid unnecessary alarms, the alarms are paused when the car is safely stopped. Additionally, there are time thresholds that must be exceeded before an alarm goes off. The triggering of the alarms are also reflected in a basic piTFT display. The following report outlines the implementation and testing of each feature and suggested future improvements. 

Interface and Alarm System:
======  

Interface:
------

Using pygame, we created a basic interface to display on the piTFT. We split the screen into five sections. One of these included a small section at the top of the screen to show/update the speed of the car. The remainder was separated into four quadrants and each quadrant was devoted to one of the four main features (drowsiness, facing away, eye tracking, and wheel detection). Each quadrant is initialized as black, and if distracted driving is detected, the appropriate quadrant is filled with red and a statement printed to the screen. At the bottom of each quadrant, there is also a counter, which records the number of instances for each distracted driving event. Finally, a quit button was implemented to allow the user to cleanly exit the program, using one of the physical piTFT buttons. An overview of the entire system is show below:

<div style="text-align:center">
  <img src="/images/dd_monitor.png" alt="Image" width="500" height="400" />
</div>

Alarm System:  
------

For each distracted feature, a different threshold is set that must be met before an alarm is triggered. This threshold is determined by a counter variable that is incremented by 1 for each successive frame the distracted feature is detected. Once this counter variable reaches a predefined threshold, a word indicating the distracted feature is printed and written to a file. For example, if the driver was looking away, we use f.write to write “EYE_OFF_ROAD” to the file. The last word from this file is extracted, and the corresponding alarm is triggered. We used ___ to audibly sound an alarm through speakers we connected to the audio jack. Once the distracted instance is detected, the audible alarm is sounded once, and then the counter is reset. 

Monitor Features:
======  

Drowsiness:
------
Where a face can still be detected when a person angles their head away from the camera, eye tracking is also implemented to check whether their eyes are focused on the road. In implementing the drowsiness detection, the recognition and extraction of facial features was already initialized. A function was added to return the coordinates of the outer corner (x1), inner corner (x2), bottom (y1), and top (y2) of the left eye. This is used to calculate the eye aspect ratio (EAR) formula, which is the ratio of height between the upper and lower eyelids to the width of the eye. Since humans cannot move their eyes independently of each other, we only considered the left eye to reduce image processing. The function is called and the returned values are used to crop each frame of the video. Below is an image of the drowsiness detection:
<div style="text-align:center">
  <img src="/images/drowsy.png" alt="Image" width="500" height="400" />
</div>


Looking Away:
------
Another feature we implemented was checking if the driver was looking away from the road for an extended period of time. From the eye coordinates established in the previous section, now the pupil needs to be delineated from the whites of the eyes. To achieve this, each frame is prepared using the function cv2.bitwise_not to invert the image and cv2.cvtColor to convert it to grayscale. Then, we use thresholding to segment the image into the foreground and background (pupil and whites of eye) via some threshold value, T. Where the lighting conditions vary, a constant threshold value will not predictably segment the image. Instead, we use adaptive thresholding via cv2.adaptiveThreshold, which automatically finds the optimal threshold value T(x,y) from the mean of the blockSizexblockSize neighborhood of (x,y) - C. Now we can find the contours of the pupil in each frame using cv2.findContours(), which as its name would suggest, extracts the contours from an image. We find the center coordinates of the pupil via cv2.moments(), and compare this value to the center of the left eye. If |$x_{center of eye}$ - $x_{center of pupil}$| > 8, then the person is looking away. The direction can be distinguished by considering the signs, but this was not necessary for this project. An image are shown below with a circle drawn on the detected pupil and with the thresholding applied. Although not perfect, the adaptive thresholding is able to find the pupil when looking left and right, enough so that the contours can be found. 

<div style="text-align:center">
  <img src="/images/eye_tracking.png" alt="Image" width="500" height="400" />
</div>

Facing Away:
------

An extension of this feature is to consider when the driver is facing away from the camera. When this occurs, dlib can no longer detect a face and facial features cannot be computed. If the numpy array returned is empty, we know that a face is not detected, and we assume the driver is facing away from the road and is therefore distracted. When this occurs, an alarm is sounded. One additional feature is in the case of the driver facing away while a drowsiness or looking away alarm is still triggered. If this occurs, the other alarms are disabled because it can no longer detect your face and determine if the driver is indeed drowsy or looking away.

Steering Wheel Detection:
------

A feature to detect whether the driver has their hands on the wheel is also implemented. For this, we first considered using an analog force sensor. After some testing, it was determined that the sensors were not sensitive enough for this application - a voltage difference was detected only when squeezed. Although this could be resolved via a voltage divider, we decided to try a simpler alternative: capacitive touch sensors. They detect a change in capacitance when someone touches the sensor, therefore only outputting a low or high state. Additionally, they provide flexibility in that they can be attached to conductive surfaces (aluminum tape) via alligator clips. This allowed us to increase the surface area for each sensor and cover a greater area of the steering wheel. Realistically, this would require a steering wheel cover with many sensors since a person may hold the wheel in any position. For the purposes of this project, we will only use two sensors for each hand at the recommended 9 and 3 o'clock positions. If at least one of the surfaces is touched (i.e. high), then it is assumed that the person has at least one hand on the wheel. However, if none of the surfaces are touched (i.e. low), then it is assumed the person has neither hand on the wheel and an alarm is sounded. This is achieved by checking the values of each pin within a polling loop and triggering an alarm when all values are low. 


Speed Detection Via OBDII:
------

Another feature we implemented was using real-time car speed data to determine whether to sound alarms. The data collection was achieved through an OBDII scanner, which is a port found in all cars sold in the USA after 1996. It can be used to access live data from the car, including speed. Using this, if the speed was greater than 0 and the driver was distracted, the system would issue an alarm. However, if the car was not moving (parked, stopped in traffic) and the driver was distracted, the system would not issue an alarm. This is to prevent the driver from being alerted when the car is safely stopped. 



Future Improvements:
======  

One potential extension of the OBDII would be to consider the direction of the car's travel. If the driver was reverse or parallel parking, this would require the person to look behind them, thus triggering an unnecessary alarm. Either we could use the OBDII to determine when the car is in reverse and disable the alarms, or there could be a physical button for the driver to momentarily disable the alarms (for 2-3 minutes), while the person parks. 

Where all of these detectors (except steering wheel) require a camera, the performance is expected to markedly decrease in the night-time. This presents an issue, since drivers are more likely to be drowsy at night, and despite 60% less traffic on the roads, more than 40% of all fatal car accidents occur at night. Since it is considered a distraction for a driver to have cabin lights on in the car, we would have to implement a night vision camera. 
