# Object Detection and Image Processing for Visual Content Creation

Emma Liu

Department of Computer Science

University of Wisconsin Madison 

liu763@wisc.edu


## Initial Project Proposal

[project proposal](https://yliuemma.github.io/CS639FALL20website/CS_639_Project_Proposal.pdf)

## Midterm Progress Report

[Midterm Progress report](https://yliuemma.github.io/CS639FALL20website/COMP_SCI_639_Midterm_Progress_Report.pdf)

## Motivation

Visual content creation has become wildly applied in many industries. 
However, Manual image processing of recurring objects of choice from a massive amount of digital media
can be very tedious for the visual content creation process. Automated object detection and image
processing can free visual content creators from doing repetitive tasks, such as blurring out the same
object that occurs in a video, and allow them to focus more on the creative process.


![chart1](https://s3.amazonaws.com/thumbnails.venngage.com/template/b93f660f-5460-40f6-9952-ff7d150dac21.png)
![chart2](https://venngage-wordpress.s3.amazonaws.com/uploads/2020/03/Visual-Content-Marketing-Statistics-4.png)

Instead of manually adjusting each object in the image, I would like to automate this process and allow content creators to focus more on the creative process through object detection and image processing.

I want to make it simple, fast, and more accessible for beginners in visual content creation or for those who do not have enough budget.

## Current State-of-Art and My Approach

### Youtube Studio Face blur and Custom Blur:

For face blur: Youtube Studio breaks up videos into frames and detecting faces on each frame individually. Once it has detected the faces in each frame of the video, it will start matching face detections within a single scene of the video, relying on both the visual characteristics of the face as well as the face’s motion.

For Custom Blur: after the user dragging a certain region over an object, it will automatically blur the object as it moves.

### Photoshop object selection tool:

draw a rectangular region or a lasso around the object.

Object Selection tool automatically selects the object inside the defined region. 

### Smartphone's portrait mode:

For iPhone: it uses the phone's dual cameras and Apple's software to mimic the quality you would get from a DSLR camera. One lens capturing the actual image, the other capturing data to create a nine-layer depth map. 

For Pixel 2: it utilizes pixel splitting to create a depth map and machine learning helps to identify the subject and create a mask. 

Most of the smartphone's portrait mode approaches involve instance segmentation or extra hardware to extract data and achieve object detection. Instance segmentation like Mask R CNN and depth maps usually have high demand on hardware, especially GPU, because they need to split up the image and read through the image multiple times. The ideal approach for beginning visual content creators should be fast, simple, and less expensive.

### My Approach -- YOLO(You Only Look Once) with OpenCV libraries:

YOLO was designed by Joseph Redmon, Santosh Divvala, Ross Girshick, and Ali Farhadi. It splits the input image into a set of grid cells with each grid cell has associated vector that would tell if an object exists in the grid cell, the class of the object(i.e car or person) and the predicted bounding box for that object.

![yoloprocess](https://miro.medium.com/max/1152/1*m8p5lhWdFDdapEFa2zUtIA.jpeg)

Although YOLO is less accurate than Mask R CNN and other existing methods with instance segmentation, it is way faster and way less demanding than Mask R CNN because it only reads through the image once, so I believe YOLO is the ideal algorithm for approaching the problems.

I use OpenCV for similar reason: it's fast, simple and effecient. OpenCV can achieve around 30 fps in real time processing compared to 4-5 fps from Matlab, and it does not use much RAM for real time applications, so even if the user is on a low budget and using a computer that does not have a good GPU. 


## My Implementation and Results

### My Implementation

### Results


## Discussion and References

### Discussion
Obviously I learned about YOLO and OpenCV, and I integrated image processing methods that I learned from CS639. 

I also learned the pros and cons of my approach:
  Pros: It's definitely fast and simple to use. I shared it with my friends who are studying communication arts and they were happy to use it on their finals. One of them is using my program to blur out non-participants in a video recording of a BLM protest that happened on State Street this year.
  Cons: However, it is also essentially less accurate since it splits images into grids instead of instance segmentation. The most you can get from it are the bounding boxes, but you can't really do it on a pixel level with YOLO alone.

In the future I am planning to implement this approach to be both fast and more accurate than just bounding boxes. I am considering applying instance segmentation inside bounding box regions instead of doing it on the whole picture, so you only get to read the bounding boxes multiple times. I hope this instance segmentation at smaller scale would speed up the process and keep the accuracy. I will refer to similar approaches, such as YolAct or Poly-YOLO.


### References

Bolya, Daniel, et al. "Yolact: Real-time instance segmentation." Proceedings of the IEEE international conference on computer vision. 2019.

Redmon, Joseph, et al. "You only look once: Unified, real-time object detection." Proceedings of the IEEE conference on computer vision and pattern recognition. 2016.

Khoja, Nadya. “14 Visual Content Marketing Statistics to Know for 2020 [Infographic].” Venngage, 11 Mar. 2020, venngage.com/blog/visual-content-marketing-statistics/.

