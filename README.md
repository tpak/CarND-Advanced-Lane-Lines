## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/undistorted.jpg "Undistorted"
[image2]: ./output_images/chessboards.png "Chessboards"
[image3]: ./output_images/combined.jpg "Combined"
[image4]: ./output_images/combined-hist.jpg "Combined Histogram"
[image5]: ./output_images/warped.jpg "Warped"
[image6]: ./output_images/sliding-window1.jpg "Sliding1"
[image7]: ./output_images/sliding-window2.jpg "Sliding2"
[image8]: ./output_images/lanes.jpg "Lanes"

[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. This writeup summarizes what I did for this project.  

All of my work is found in the Jupyter notebook project.ipynb found on github
[https://github.com/tpak/CarND-Advanced-Lane-Lines/blob/master/project.ipynb](https://github.com/tpak/CarND-Advanced-Lane-Lines/blob/master/project.ipynb)

I'll point the reader to specific sections in that notebook as a reference.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the cells 2 and 3 of my Jupyter notebook.  

Taking code from the lessons I started by preparing object points from a series of provided chessboard test images and using the openCV library to find the chessboard corners. 

I then used the output from that -- `objpoints` and `imgpoints` -- to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result. The before and after results can be seen below and in cell 3.

![alt text][image2]

I saved the calbration results in a pickle file for later re-use, although in this project I did not re-use them. 


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Once we have the camera calibration data in hand we can use the same technique on a real image taken with the same camera. You can see an example of a real image here as well as in cell 4.

![alt text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I spent a fair amount of time fine tuning various colora transforms and gradients using the techniques found in the class section. My experiments can be found in cells 6-24. I tried various combinations of Sobel thresholding using openCV and variations on color thresholding with openCV and bitmasking as well. I eventually settled on a combination of using the S channel from HSV color space combined with Sobel X transforms merged together. This combination normally gave a very good clean histogram that could be used to find the lane lines. 

Before and after of a gradient and color thresholded image

![alt text][image3]

Histogram of the gradient and color thresholded image

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The above images are actually perspective transformed. That is, we have takend them and turned a region of them into a birds eye view. An example of that is below. This is in cell 6 and 7. Here we have taken only a portion of the image that extends from about the front of the hood to 1/2 up the image in an attempt to isolate just the likely palce we will find the lane lines.  

![alt text][image5]

The code for my perspective transform in cell 7 calls a function called `perspective_shift(image)`, which appears cell 6.  The `perspective_shift()` function takes as inputs an image (`img`). In cell 6 I experimented extensively with calculating the src and destination corodinates. I   In the end I chose the hardcode the source and destination points in the following manner as I found them in one of the original lessons:

```python
    src_corners = np.float32([[225,720],[590,450],[690,450],[1025,720]])
    dest_corners = np.float32([[275,720],[275,0],[1025,0],[1025,725]])    
```

The proof that this works is that eventually the lane lines will be drawn successfully back onto this area. I had code that did this at one point but accidentally deleted it. 

One technique I was having luck with was to mask this area more aggressively as you can see in the comemented out code. The challenge I had was that later when I applied colora gradients and thresholding there was an abrupt edge at the masked area. The images appeared to work better except the histogram was thrown off. I suspect if I spen more time breaking the technique down I could get it to work much better. I definitely spent a lot of time fiddling with this and it would be interesting to see how this eventually gets solved.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Cell #25 and # asd are where I ook the polynomial fir functions from the lessons and applied them to finding the pixels. The first pass function `sliding_window_fitpoly()` slices the image up into left and right halves and `windows` which it then searches for the lane lines. you can see an illustarion of the bounding boxes and lines that it fit below and in cell 26. 

![alt text][image6]

Once we had a "lock" on lane lines we could search quicker using `sliding_window_fitnewpoly()` in cell 27. This just uses the previously found lines as a starting point and searches within a margin for them. Cell 28 shows the illustrated output as well as right here. 

![alt text][image7]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Cell 29 demonstrates calculating the curvature and the center position or offset from center of the vehicle. In order to calculate the curve iplement a aprabolic coefficient equation and then convert it into real world measurements using a pixels to meters calculation.

Cell 30 shows the outputs of that. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 31 `draw_lane_lines()` and in cell 33 `draw_telemetry()` I added curve information and centering information to the display. (I also left video frame numbers in wich I was using for debugging purposes)  Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

My video pipeline can be found in cell 36 `process_frame()`. it processes each frame it is handed and attempts to track the progress and accuracy compared to previous frames. I did end up using my area of interest cropping function here and that settled down the jitters considerably. There is still some occasional wobbliness but nothing that would crash a car. It always recovers within a second or so. My smoothing routine and calculation could use additional work to make these go away. Frame 37 executes the pipeline on the provided video and the video is in frame 38.

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

1. I ceratinly faced jitter issues and geometry issues in my code. I occasionally switched around how I was thinking about polygons and that caused all kind of confusion. Keeping everything working in one uniform way is critical.
2. I also had a challenging time getting my Line class to track things and average them correctly. This was just due to the fact that i am not coding in Pything day to day so needed to slow down and work through it carefully. 

