## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


In this project, your goal is to write a software pipeline to identify the lane boundaries in a video, but the main output or product we want you to create is a detailed writeup of the project.  Check out the [writeup template](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) for this project and use it as a starting point for creating your own writeup.  

Creating a great writeup:
---
A great writeup should include the rubric points as well as your description of how you addressed each point.  You should include a detailed description of the code used in each step (with line-number references and code snippets where necessary), and links to other supporting documents or external references.  You should include images in your writeup to demonstrate how your code works with examples.  

All that said, please be concise!  We're not looking for you to write a book here, just a brief description of how you passed each rubric point, and references to the relevant code :). 

You're not required to use markdown for your writeup.  If you use another method please just submit a pdf of your writeup.

The Project
---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

The images for camera calibration are stored in the folder called `camera_cal`.  The images in `test_images` are for testing your pipeline on single frames.  If you want to extract more test images from the videos, you can simply use an image writing method like `cv2.imwrite()`, i.e., you can read the video in frame by frame as usual, and for frames you want to save for later you can write to an image file.  

To help the reviewer examine your work, please save examples of the output from each stage of your pipeline in the folder called `ouput_images`, and include a description in your writeup for the project of what each image shows.    The video called `project_video.mp4` is the video your pipeline should work well on.  

The `challenge_video.mp4` video is an extra (and optional) challenge for you if you want to test your pipeline under somewhat trickier conditions.  The `harder_challenge.mp4` video is another optional challenge and is brutal!

If you're feeling ambitious (again, totally optional though), don't stop there!  We encourage you to go out and take video of your own, calibrate your camera and show us how you would implement this project from scratch!

------------------

## Writeup Advanced Lane Finding

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

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./output_images/undistorted2.png "Road Transformed"
[image3]: ./output_images/threshold.png "Binary Example"
[image4]: ./output_images/perspective1.png "Warp Example"
[image5]: ./output_images/perspective2.png "Warp Example"
[image6]: ./output_images/finding.png "Fit Visual"
[image7]: ./output_images/output.png "Output"
[video1]: ./output_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./CarND-Advanced-Lane-Lines-master.ipynb". (Result of notebook is shown in HTML file: "./CarND-Advanced-Lane-Lines-master.html").  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Undistortion][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Undistortion][image2]
#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of gradient thresholds from 2 different channels of color spaces to generate a binary image (thresholding steps in the sixth cell of notebook). I have chosen **gradient on V channel of HSV space** to detect easily white line on dark background. This was better than R channel in RGB space. I have chosen **gradient on S channel of HLS space** to detect especialy yellow line on clear background. This was better than S channel in HSV space. I prefer to use gradient rather a color threshold binary for S channel, because it take account detection with a large shadow like under the bridge in the challenge video. Here's an example of my output for this step.

![Gradient][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, in the 9th code cell of the IPython notebook.  The `perspective_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
lane_width = 600
border = (image_size[1] - lane_width)//2
src = np.float32(
				[[200,image_size[0]],
				[1120,image_size[0]],
				[723,470],
				[563,470]])
dst = np.float32(
				[[border, image_size[0]],
				[border + lane_width, image_size[0]],
				[border + lane_width, 0],
				[border, 0]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 340, 720      | 
| 1120, 720     | 340, 720      |
| 723, 470      | 940, 0        |
| 563, 470      | 940, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Warped][image4]

![Warped][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for that is between the 13rd and 19th code cell of the notebook. I did 2 kinds of detection:
 * First one is applied for the first image or when the last detection has detect less than 2000 pixels (when the last detection are biased). For this kind of detection, I use firstly a gaussian mask `focus_window = 2*gaussian(1280, std=300)` to priviledge center pixels and filter out border line or aside vehicules (Code cell 13 and 14). **Then I use a convolution technic** with gaussian window: `window = gaussian(151, std=20)` to detect the optimal position of window (See code cell 15). I use a gaussian window rather a square window because it centers more accurately. Then i select pixels inside windows with a window width of 50 pixels.
 * Second one is applied for all other cases. I use the precedent polynomial fit to determine a **search area** with a margin of 60 pixels.
 
Finally **I smooth** the fit with the 10 last detections with a memory rate of 0.9 and i correlate left side result and right side with a rate of 70% / 30%.

Here is example of 2 kinds of detection:

![line detection][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did these in the 20th code cell (curvature) and 21th code cell (center position) of my notebook. I use the formula with polynomial fit and i mean left side and right side results for curvature. I transform pixel measure in meter measure by assuming that width lane in real world is 3.7 m and lane depht selection correspond to 22m (dashed line is 3m and gap is 9m)

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 22th code cell of notebook in the function `drawing_pipeline()`.  Here is an example of my result on a test image:

![lane finding][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here project was very intersting. But result of curvature is certainly not very precise, because of incertainity of all the transformations. I have tried during this project to think what the human beeing does during driving a car for detecting lane lines.

Threshold binary image transformation has to be certainly improved. To detect lines with shadow and some different background colors, and to filter the outliers are complex. To obtain something good with challenge videos was hard.

For line detection, i like the convolution technic. I tried to obtain something accurate only with this technic. I tried to use a mask which fits the 2 lines in a same time. But challenge video show us that the lane have some different widths. So i finally use one detection by line. And what is the more robust and resistant to outlier is to use the simple area search according to the last detections.

In order to go further in my project, I have to:
* Improve the threshold binary transformation to maximize lane dectection, and minimize the outlier detection.
* Improve smooth filter, in order to maximize accuracy and to be able to detect very curvy lane.
* Maybe have an automatical depht of search area according to the quality of threshold binary transformation. That means the area used for perspective warping, or the area used for lane detection after warping, will be adaptative.
 
     

