## Writeup
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

[image1]: ./test_images/test3.jpg "Actual Image"
[image2]: ./output_images/Project_Video_Outputs/Undistorted_test1.jpg "Undistorted"
[image3]: ./output_images/Project_Video_Outputs/Thresholded_Binary_test1.jpg "Thresholded Binary Example"
[image4]: ./output_images/Project_Video_Outputs/Warped_image_test1.jpg "Warp Example"
[image5]: ./output_images/Project_Video_Outputs/LanesOnly_image_test1.jpg "Lanes Only Visual"
[image6]: ./output_images/Project_Video_Outputs/LaneDrawnImage_test1.jpg "Output Image"
[image7]: ./camera_cal/calibration2.jpg "Distorted Chessboard Image"
[image8]: ./output_images/ChessBoard_Undistorted_Outputs/chessboard_undist2.jpg "Undistorted Chessboard Image"

[video1]: ./test_videos_output/project_video_final.mp4 "Output Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  



### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./P2.ipynb"

Camera Matrix was calculated by finding corners of an IDEAL 9x6 chessboard which we call as "object points", which will be the (x, y, z) coordinates. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.

Secondly `cv2.findChessboardCorners()` function let us identify the `imgpoints` on the calibration images which is (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration Matrix and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image7]
![alt text][image8]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image1]
After application of Distortion Correction as per the coefficients found in previous step, we can Undistort image as:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps is present in "./P2.ipynb" with function `color_Sobel_thresholder()`(which appears in the 5th code cell of the IPython notebook).
Main components are:
* sobel x & Sobel y Threshlod
* Sobel Magnitude & Sobel Direction(For selecting Lane Angles only)
* h-channel and s-channel from HLS Color Space has been used for covering images with shades, etc
* r-channel from RGB Color Space has been used in addition to h and s channel to ignore patches on the road and highway border edges.

For Low light conditions, Sobel Lower Magnitude has been reduced based on Average Brightness Calculation for lower half of image. 
Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform_image()` (which appears in the 8th code cell of the IPython notebook).  The `transform_image()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:


```python
src = np.float32(
    [[(img_size[0] / 2) - 72, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```
Note: Following values were fine tuned even after above calculation.

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 568, 470      | 200, 0        | 
| 717, 470      | 200, 680      |
| 260, 680      | 1000, 0       |
| 1043, 680     | 1000, 680     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:
The code for my Lane Pixel Detection is included in 2 functions namely `locate_lines()` and `locate_lines_further()` (which appears in the 9th code cell of the IPython notebook)
* `Locate_Lines()` has been used to detect left and right lanes in inital frame/Reset frame due to continuous errors.This has used Sliding Window method to detect pieces of lane pixels.
Additionally , for extra curvy paths, number of windows have been reduced as Lanes cross over the left/right edges.
I have been keeping track of average slope values for last 10 frames to determine the more curvy lanes.

* `Locate_Lines_further()` has been used to carry forward the lane result detected by `Locate_Lines()`

In case of lane detection array is empty for any frame, we are using last detected values and ignore the current values.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for Radius Of Curvature is included in function `radius_curvature()` (which appears in the 11th code cell of the IPython notebook).
As the Lane equation has been a second order polynomial which looks like:
f(x)=Ax^2 +Bx+C (considering the less frequent changes in x wrt y).

Radius of Curvature  = ((1+ (Derivative of x wrt y)^2)^3/2)/(|Second order derivativeof x wrt y|)

Result can be found in image
![alt text][image6]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `draw_on_image()` (which appears in the 13th code cell of the IPython notebook). Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's the final video for the project ./test_videos_output/project_video_final.mp4
![alt text][video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The approach used to detect Lane has worked fine for project_video.mp4 and challenge_video.mp4. However, there are many shortcomings while detecting lanes in harder_challenge_video.mp4.
Challenges faced:
* Difficult to get the same thresholds working for all image conditions including low light, normal light, shades, road patches, highway boundary edges, etc. --> Need to detect the conditions beforehand and changing the thresholds accordingly might work.

* Curvy roads does have much variations in x direction--> Detection parameters must be different for curvy and non-curvy roads.

* Without using high confidence lane parameters, it is difficult to detect lane on a cury road quick enough.

* Current Detection algorithm needs higher ECU processing compared to previous Basic Lane Detection method.
