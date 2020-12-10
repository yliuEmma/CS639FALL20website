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

![faceblur](https://yliuemma.github.io/CS639FALL20website/faceblur.gif)

For Custom Blur: after the user dragging a certain region over an object, it will automatically blur the object as it moves.

![customblur](https://yliuemma.github.io/CS639FALL20website/customblur.gif)

### Photoshop object selection tool:

The user would draw a rectangular region or a lasso around the object, and object Selection tool automatically selects the object inside the defined region. The tool works better on well-defined objects than on regions without contrast. So the user still needs to process multiple objects through manual labor and it does not work well in areas with low contrast. It also does not work on videos.

![obtool](https://yliuemma.github.io/CS639FALL20website/Photoshop-Object-selection-Hero-new.gif)

### Smartphone's portrait mode:

For iPhone: it uses the phone's dual cameras and Apple's software to mimic the quality you would get from a DSLR camera. One lens capturing the actual image, the other capturing data to create a nine-layer depth map. 

![iPhone](https://yliuemma.github.io/CS639FALL20website/ios14-iphone-11pro-camera-portrait-mode.jpg)

For Pixel 2: it utilizes pixel splitting to create a depth map and machine learning helps to identify the subject and create a mask. 

![pixel2](https://yliuemma.github.io/CS639FALL20website/orig-and-mask-comp-s.jpg) ![pixel22](https://yliuemma.github.io/CS639FALL20website/0132_20170801_154453_513-mask-only.jpg)

Most of the smartphone's portrait mode approaches involve instance segmentation or extra hardware to extract data and achieve object detection. Instance segmentation like Mask R CNN and depth maps usually have high demand on hardware, especially GPU, because they need to split up the image and read through the image multiple times. There are other approaches such as Fast R-CNN, Faster R-CNN which uses window slides over the image making it requires thousands of predictions on a single image. The ideal approach for beginning visual content creators should be fast, simple, and less expensive.

### My Approach -- YOLO(You Only Look Once) with OpenCV libraries:

YOLO was designed by Joseph Redmon, Santosh Divvala, Ross Girshick, and Ali Farhadi. It splits the input image into a set of grid cells with each grid cell has associated vector that would tell if an object exists in the grid cell, the class of the object(i.e car or person) and the predicted bounding box for that object. A single convolutional network simultaneously predicts multi-ple bounding boxes and class probabilities for those boxes.

![yoloprocess](https://miro.medium.com/max/1152/1*m8p5lhWdFDdapEFa2zUtIA.jpeg)

Although YOLO is less accurate than Mask R CNN and other existing methods with instance segmentation, it is way faster and way less demanding than Mask R CNN because it only reads through the image once, so I believe YOLO is the ideal algorithm for approaching the problems.

I use OpenCV for similar reason: it's fast, simple and effecient. OpenCV can achieve around 30 fps in real time processing compared to 4-5 fps from Matlab, and it does not use much RAM for real time applications, so even if the user is on a low budget and using a computer that does not have a good GPU. 


## My Implementation and Results

### My Implementation

I applied YOLOv3's neuro network configuration along with pre-trained model weights from COCO dataset. 

For images, since I am using OpenCV libraries, the images need to prepared as a blob (4D numpy array containing image source, scale factor, size, and the option swapBR=True) before it is fed to the neuro network.

```markdown

`blob = cv2.dnn.blobFromImage(image, 1 / 255.0, (416, 416), swapRB=True, crop=False)`

```
For videos, I need to read each frame as a blob before it is fed to te neuro network.

```markdown
`cap = cv2.VideoCapture(video_file)

_, image = cap.read()

h, w = image.shape[:2]

blob = cv2.dnn.blobFromImage(image, 1 / 255.0, (416, 416), swapRB=True, crop=False)`

```

For all images/video frames I use 416x416 as their sizes to keep the consistency.

The confidence threshold that I specified is 0.5. Any object detected with confidence lower than 0.5 will be discarded by the neuro network as it process the image/video. I have tried 0.3 and 0.7 as confidence as well, but the results can be very odd. When confidence = 0.3, many objects would be mistakenly detected as something else. A dog that moves in a video can be detected as an elephant or a horse for a considerable amount of frames. When confidence = 0.7, objects that can be easily detected in 0.5 will be ignored by the neuro network. In a street photo, the people in the far distance would get ignored. So confidence = 0.5 is a good middle ground without producing significant errors.

The YOLO uses Non-Maximal Suppression to eliminate extra bounding boxes that do not have highest confidence value and have a high IoU (Intersection over Union) value(which can be explained by the image below). I use 0.5 as IoU threshold to eliminate boxes with IoU lower than this value. Additionally, I also use 0.5 as a score threshold to eliminate extra boxes that contains confidence score lower than 0.5.

(image place holder)

```markdown
`idxs = cv2.dnn.NMSBoxes(boxes, confidences, SCORE_THRESH, IOU_THRESH)`
```

For images, the user would need to enter the path name of the image that they want to process, the type of object, and desired processing( m for mark out with a labeled bounding box, b for blur, s for sticker). If there is an object detected with a label that matches user's desired object, it will apply user's desired way of processing. If there is no object detected that matches the user's desired object, the program would do nothing with the image. 

For blurring objects in images and videos, I extract the coordinates of 4 corners of the bounding box and use them as ROI(Region of Interest) to create masks for blurring process.

```markdown
`roi_corners = np.array([[(x, y), (x + w, y), (x + w, y + h), (x, y + h)]], dtype=np.int32)`
```

I prepare a blur on the whole image before adding up the masks. The blur I used is GaussianBlur with kernel size of 43,43 and a sigmaX of 30:

```markdown
`blurred = cv2.GaussianBlur(image, (43, 43), 30)`
```
I used bitwise_and to add up the original image, the blurred image, and the masks created by the bounding boxes.

For adding stickers on the objects, since I would like to integrate the sticker image's alpha channel to make the sticker smoothly overlay on the original image, I require the user to use png files because they have alpha channels. I resize the sticker image according to the bounding box of the object with interpolation = cv2.INTER_ARE:

```markdown
`sticker = cv2.resize(sticker, (w,h), interpolation = cv2.INTER_AREA)`
```
However, the original image that is fed into the neuro network might not be png, so it may need an extra alpha channel for the area covered by bounding boxes. 

```markdown
`alpha_sticker = sticker[:, :, 3] / 255.0

alpha_image = 1.0 - alpha_sticker`
```

Because the sticker image is applied as a hard sticker, weighted blending is not necessary. 


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

“Portrait Mode on the Pixel 2 and Pixel 2 XL Smartphones.” Google AI Blog, 17 Oct. 2017, ai.googleblog.com/2017/10/portrait-mode-on-pixel-2-and-pixel-2-xl.html. 

Khoja, Nadya. “14 Visual Content Marketing Statistics to Know for 2020 [Infographic].” Venngage, 11 Mar. 2020, venngage.com/blog/visual-content-marketing-statistics/.

Make Quick Selections in Photoshop, helpx.adobe.com/photoshop/using/making-quick-selections.html. 

Redmon, Joseph, et al. "You only look once: Unified, real-time object detection." Proceedings of the IEEE conference on computer vision and pattern recognition. 2016.

“Use Portrait Mode on Your IPhone.” Apple Support, 22 Oct. 2020, support.apple.com/en-us/HT208118. 


