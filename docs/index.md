# Object Detection and Image Processing for Visual Content Creation

Emma Liu

Department of Computer Science

University of Wisconsin Madison 

liu763@wisc.edu

## Initial Project Proposal

[project proposal](https://yliuemma.github.io/CS639FALL20website/CS_639_Project_Proposal.pdf)

## Midterm Progress Report

[Midterm Progress report](https://yliuemma.github.io/CS639FALL20website/COMP_SCI_639_Midterm_Progress_Report.pdf)

## Final Presentation
[presentation](https://yliuemma.github.io/CS639FALL20website/CS639_Final.pptx)
In the future I am planning to implement this approach to be both fast and more accurate than just bounding boxes. I am considering applying instance segmentation inside bounding box regions instead of doing it on the whole picture, so you only get to read the bounding boxes multiple times. Taking the image that I use to mark/blur out people as an example, the time it took to record all people's bounding box is around 0.3 seconds, while feeding the image to PixelLib's Mask R Cnn model takes around 2.5 seconds(given that they all just run on CPU). The image's size is 515(height) x 775(width), and the total area of all bounding boxes(which can be easily obtained by adding up all extracted bouding box w x h) that contain detected person is around 5% of the total area of the image. If I apply Mask R Cnn or similar instance segmentation methods on those bounding boxes(detected by YOLO first) instead of applying Mask R Cnn on the full image, I assume the total time would be around 0.425, which is slower than YOLO, but still faster than Mask R Cnn and more accurate than just YOLO.
## Table of contents
1. [Motivation](#motivation)
2. [Current State-of-Art and My Approach](#current)
3. [My Implementation and Results](#implementation)
4. [Discussion](#discussion)
5. [References](#references)

## Motivation

Visual content creation has become widely applied in many industries. However, manual image processing of recurring objects of choice from a massive amount of digital media can be very tedious for the visual content creation process. Automated object detection and image processing can free visual content creators from doing repetitive tasks, such as blurring out the same object that occurs in a video, and allow them to focus more on the creative process.

![chart1](https://s3.amazonaws.com/thumbnails.venngage.com/template/b93f660f-5460-40f6-9952-ff7d150dac21.png)
![chart2](https://venngage-wordpress.s3.amazonaws.com/uploads/2020/03/Visual-Content-Marketing-Statistics-9.png)

Visual content creation can be very difficult to pick up for non-professionals. Many of my friends studying Communication Art were intimidated by the complexity of Adobe Photoshop or Adobe AfterEffects when they first learned how to edit pictures or videos for their projects. 
<a name="paragraph1"></a>
Instead of manually adjusting each object in the image, I would like to automate this process and allow content creators to focus more on the creative process through object detection and image processing.

I want to make it simple, fast, and more accessible for beginners in visual content creation or for those who do not have enough budget.

## Current State-of-Art and My Approach <a name="current"></a>

### Youtube Studio Face blur and Custom Blur:

For face blur: Youtube Studio breaks up videos into frames and detecting faces on each frame individually. Once it has detected the faces in each frame of the video, it will start matching face detections within a single scene of the video, relying on both the visual characteristics of the face as well as the face’s motion.

![faceblur](https://yliuemma.github.io/CS639FALL20website/faceblur.gif)

For Custom Blur: after the user dragging a certain region over an object, it will automatically blur the object as it moves.

![customblur](https://yliuemma.github.io/CS639FALL20website/customblur.gif)

### Adobe Photoshop object selection tool:

The user would draw a rectangular region or a lasso around the object, and object Selection tool automatically selects the object inside the defined region. The tool works better on well-defined objects than on regions without contrast. So the user still needs to process multiple objects through manual labor and it does not work well in areas with low contrast. It also does not work on videos.

![obtool](https://yliuemma.github.io/CS639FALL20website/ps_obtool.gif)

### Smartphone's portrait mode:

For iPhone: it uses the phone's dual cameras and Apple's software to mimic the quality you would get from a DSLR camera. One lens capturing the actual image, the other capturing data to create a nine-layer depth map. 

![iPhone](https://yliuemma.github.io/CS639FALL20website/ios14-iphone-11pro-camera-portrait-mode.jpg)

For Pixel 2: it utilizes pixel splitting <a name="paragraph1"></a>to create a depth map and machine learning helps to identify the subject and create a mask. 

![pixel2](https://yliuemma.github.io/CS639FALL20website/orig-and-mask-comp-s.jpg) ![pixel22](https://yliuemma.github.io/CS639FALL20website/0132_20170801_154453_513-mask-only.jpg)

Most of the smartphone's portrait mode approaches involve instance segmentation or extra hardware to extract data and achieve object detection. Instance segmentation like Mask R CNN and depth maps usually have high demand on hardware, especially GPU, because they need to split up the image and read through the image multiple times. There are other approaches such as Fast R-CNN, Faster R-CNN which uses window slides over the image making it requires thousands of predictions on a single image. The ideal approach for beginning visual content creators should be fast, simple, and less expensive.

### My Approach -- YOLO(You Only Look Once) with OpenCV libraries:

YOLO was designed by Joseph Redmon, Santosh Divvala, Ross Girshick, and Ali Farhadi. It splits the input image into a set of grid cells with each grid cell has associated vector that would tell if an object exists in the grid cell, the class of the object(i.e car or person) and the predicted bounding box for that object. A single convolutional network simultaneously predicts multi-ple bounding boxes and class probabilities for those boxes.

![yoloprocess](https://miro.medium.com/max/1152/1*m8p5lhWdFDdapEFa2zUtIA.jpeg)

Although YOLO is less accurate than Mask R CNN and other existing methods with instance segmentation, it is way faster and way less demanding than Mask R CNN because it only reads through the image once, so I believe YOLO is the ideal algorithm for approaching the problems.

I use OpenCV for similar reason: it's fast, simple and effecient. OpenCV can achieve around 30 fps in real time processing compared to 4-5 fps from Matlab, and it does not use much RAM for real time applications, so even if the user is on a low budget and using a computer that does not have a good GPU. 


## My Implementation and Results <a name="implementation"></a>

### My Implementation

I applied YOLOv3's neuro network configuration along with pre-trained model weights from [Joseph Chet Redmon's website](https://pjreddie.com/darknet/yolo/). Instead of building YOLO from scratch, I use OpenCV's Deep Neural Networks(DNN) in Python to load in the YOLO neuro network from the configuration file and the weights file. The readNetFromDarknet() function in DNN returns an artificial neural network model based on the YOLO configuration file and weights file.

```markdown
`cv2.dnn.readNetFromDarknet() `
```

For images, since I am using OpenCV libraries, the images need to prepared as a blob (4D numpy array containing image source, scale factor, size, and the option swapBR=True) before it is fed to the neuro network. The blob generated from the image is the actual input that I feed to the YOLO neuro network.

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

The bounding boxes are extracted by looping through each objects detected in each convolutional layer. The boxes with confidence score higher than my confidence threshold will be extracted and scaled relative to the size of the image. YOLO network itself returns the center coordinates of the bounding box followed by the boxes' width and height, so I calculated the bouding box's top left corner's coordinates from the datas produced by YOLO to make the later image processing easier, and store the calculated coordinates along with width and height in an array named boxes. I also stored each detected and not discarded result's confidence into an array named confidences and each label(type of object) into an array named class_id.

The box's top left corner's coordinates are calculated as below:

```markdown
` x = int(centerX - (width / 2))
  y = int(centerY - (height / 2))`
```

The YOLO uses Non-Maximal Suppression to eliminate extra bounding boxes that do not have highest confidence value and have a high IoU (Intersection over Union) value(which can be explained by the image below). I use 0.5 as IoU threshold to eliminate boxes with IoU lower than this value. Additionally, I also use 0.5 as a score threshold to eliminate extra boxes that contains confidence score lower than 0.5.

![iou](https://yliuemma.github.io/CS639FALL20website/iou.png)

OpenCV's dnn library contains a function that can perform non maximum suppression:

```markdown
`idxs = cv2.dnn.NMSBoxes(boxes, confidences, SCORE_THRESH, IOU_THRESH)`
```
The idx stores all boxes after non maximum suppression.

For images, the user would need to enter the path name of the image that they want to process, the type of object, and desired processing( m for mark out with a labeled bounding box, b for blur, s for sticker).  

I extract each bounding box coordinates(top left corner's coordinate), its width and height, and its label from idx. If there is an object detected with a label that matches user's desired object, it will apply user's desired way of processing. If there is no object detected that matches the user's desired object, the program would do nothing with the image.

For marking objects in images and videos, I use cv2.rectangle() function to draw out uniquely colored boxes (I use numpy's random.randint() to generate random colors for each object every time a user run this program) and use OpenCV's addWeighted() to adjust box's transparency with an alpha value of 0.6 on the final image.

For blurring objects in images and videos, I extract the coordinates of 4 corners of the bounding box and use them as ROI(Region of Interest) to create masks for blurring process.
(w and h are wdith and heights extracted from bounding boxes)

```markdown
`roi_corners = np.array([[(x, y), (x + w, y), (x + w, y + h), (x, y + h)]], dtype=np.int32)`
```

I prepare a blur on the whole image before adding up the masks. The blur I used is GaussianBlur with kernel size of 43,43 and a sigmaX of 30:

```markdown
`blurred = cv2.GaussianBlur(image, (43, 43), 30)`
```
I used bitwise_and to add up the original image, the blurred image, and the masks created by the bounding boxes.

For adding stickers on the objects, since I would like to integrate the sticker image's alpha channel to make the sticker smoothly overlay on the original image, I require the user to use png files because they have alpha channels. I resize the sticker image according to the bounding box of the object with interpolation = cv2.INTER_AREA:

```markdown
`sticker = cv2.resize(sticker, (w,h), interpolation = cv2.INTER_AREA)`
```
However, the original image that is fed into the neuro network might not be png, so it may need an extra alpha channel for the area covered by bounding boxes. 

```markdown
`alpha_sticker = sticker[:, :, 3] / 255.0

alpha_image = 1.0 - alpha_sticker`
```

Because the sticker image is applied as a hard sticker, weighted blending is not necessary. I use direct matrix addition to add the resized sticker on the areas where the bounding boxes are in an iteration to go through the original image's channels.

### Results

For images: 

I used the image from https://news.wisc.edu/surge-covid-19-testing-offered-at-nielsen-tennis-stadium/ as original image:
![ogimg](https://yliuemma.github.io/CS639FALL20website/UWTEST.jpg)

If I choose to mark out chairs in the image with bounding boxes, I would get this result:
![chairm](https://yliuemma.github.io/CS639FALL20website/UWTESTchairm_yolo3.jpg)

And if I choose to mark out people in the image with bounding boxes, I would get similar result:
![personm](https://yliuemma.github.io/CS639FALL20website/UWTESTperson_yolo3.jpg)

If I choose to blur the people in the image, I would get this result:
![personb](https://yliuemma.github.io/CS639FALL20website/UWTESTpersonb_yolo3.jpg)

If I choose to apply an image of kirby as sticker:
![kirby](https://yliuemma.github.io/CS639FALL20website/kirby2_sm.png)

and apply the sticker on the chairs, I would get this result:
![chairs](https://yliuemma.github.io/CS639FALL20website/UWTESTchairs_yolo3.jpg)

The process of applying kirby on chairs as stickers only takes around 0.32 seconds, and other process also produce similar time result:

![time](https://yliuemma.github.io/CS639FALL20website/timeTook.png)

I also tried similar process on other pictures:
![streetm](https://yliuemma.github.io/CS639FALL20website/streetpersonm_yolo3.jpg)
![streetb](https://yliuemma.github.io/CS639FALL20website/streetpersonb_yolo3.jpg)

For video:

I used a clip from: https://www.youtube.com/watch?v=Fi2WwRJDii0 as original video

![test1](https://yliuemma.github.io/CS639FALL20website/test-video.gif)

If I choose to apply mark out people in the video, I would get this result:
![personm](https://yliuemma.github.io/CS639FALL20website/test-videooutpersonm.gif)

If I choose to apply blur on the people in the video, I would get this result:
![personb](https://yliuemma.github.io/CS639FALL20website/test-videooutpersonb.gif)

With just using CPU, the average processing time for each frame is around 0.2 sec, about 5 fps in this implementation. Usually Mask R CNN can achieve 5 fps when it's running on GPU.

I have also tried to detect objects in images and process them with PixelLib library that can load in Mask R Cnn model. The average processing time for an image running on the CPU is around 2.5 second.

![maskRCNN](https://yliuemma.github.io/CS639FALL20website/maskRCNNUWTEST.jpg)

## Discussion
Obviously I learned about YOLO and OpenCV, and I integrated image processing methods that I learned from CS639. 

I also learned the pros and cons of my approach:

  Pros: It's definitely fast and simple to use. I shared it with my friends who are studying communication arts and they were happy to use it on their finals. One of them is using my program to blur out non-participants in a video recording of a BLM protest that happened on State Street this year.
  
  Cons: However, it is also essentially less accurate since it splits images into grids instead of instance segmentation. The most you can get from it are the bounding boxes, but you can't really do it on a pixel level with YOLO alone.

In the future I am planning to implement this approach to be both fast and more accurate than just bounding boxes. I am considering applying instance segmentation inside bounding box regions instead of doing it on the whole picture, so you only get to read the bounding boxes multiple times. Taking the image that I use to mark/blur out people as an example, the time it took to record all people's bounding box is around 0.3 seconds, while feeding the image to PixelLib's Mask R Cnn model takes around 2.5 seconds(given that they all just run on CPU). The image's size is 515(height) x 775(width), and the total area of all bounding boxes(which can be easily obtained by adding up all extracted bouding box w x h) that contain detected person is around 5% of the total area of the image. If I apply Mask R Cnn or similar instance segmentation methods on those bounding boxes(detected by YOLO first) instead of applying Mask R Cnn on the full image, I assume the total time would be around 0.425, which is slower than YOLO, but still faster than Mask R Cnn and more accurate than just YOLO. 

I hope this instance segmentation at smaller scale would improve the accuracy. In the future, I will refer to similar approaches, such as [YolAct](https://github.com/dbolya/yolact) or [Poly-YOLO](https://github.com/gladcolor/poly_yolo).


## References

Bolya, Daniel, et al. "Yolact: Real-time instance segmentation." Proceedings of the IEEE international conference on computer vision. 2019.

“Portrait Mode on the Pixel 2 and Pixel 2 XL Smartphones.” Google AI Blog, 17 Oct. 2017, ai.googleblog.com/2017/10/portrait-mode-on-pixel-2-and-pixel-2-xl.html. 

Hurtik, Petr, et al. "Poly-YOLO: higher speed, more precise detection and instance segmentation for YOLOv3." arXiv preprint arXiv:2005.13243 (2020).

Khoja, Nadya. “14 Visual Content Marketing Statistics to Know for 2020 [Infographic].” Venngage, 11 Mar. 2020, venngage.com/blog/visual-content-marketing-statistics/.

Make Quick Selections in Photoshop, helpx.adobe.com/photoshop/using/making-quick-selections.html. 

Redmon, Joseph, et al. "You only look once: Unified, real-time object detection." Proceedings of the IEEE conference on computer vision and pattern recognition. 2016.

“Use Portrait Mode on Your IPhone.” Apple Support, 22 Oct. 2020, support.apple.com/en-us/HT208118. 


