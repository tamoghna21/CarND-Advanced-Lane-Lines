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

[image1]: ./camera_cal/undist.jpg "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output_images/binary_combo_test1.jpg "Binary Example"
[image4]: ./output_images/warped_test3.jpg "Warp Example"
[image5]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/example_output_test3.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second and third code cells(Section 1. Camera calibration) of the IPython notebook located in "./Advanced_Lane_Finding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I applied cv2.undistort() for distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds (method color_gradient_thresh in section 2)to generate a binary image.First I converted the undistorted image into HLS. Threshold has been applied on the s channel. Also, gradient calculated on l channel and threshold applied. Then pixels are selected if s channel threshold and l channel gradient threshold are satisfied.  Here's an example of my output for this step. 

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

First the Perspective transformation matrix M is calculated in section "4. Perspective calculation". I chose the image 'test_images/straight_lines2.jpg'. The image is first undistorted and threshold applied.I applied histogram at the bottom of the resulting image to find the two points corresponding to where left and right lane at the bottom ofthe image. I also applied histogram on a thin rectangle at the middle of the image to find 2 points corresponding to left and right lanes. These four points together were taken as 'src'. Then I chose the destination points and applied cv2.getPerspectiveTransform to calculate M.




```python
src = np.float32([[leftx_base,height-59],[rightx_base,height-59],[rightx_upper,height-259],[leftx_upper,height-259]])
dst = np.float32([[leftx_base,height-10],[rightx_base,height-10],[rightx_base,offset],[leftx_base,offset]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 294, 661      | 294, 710      | 
| 1002, 661     | 1002, 710     |
| 703, 461      | 1002, 50      |
| 579, 461      | 294, 50       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then lane-line pixels have been detected in find_lanes() in section 6.Histogram is calculated at the bottom half of the image to find left and right lines where the white pixels concentrate. Then sliding windows have been drawn from the bottom of the image to top and pixels within the windows have been detected as lane pixels.Then second order polynomials have been fitted for left and right line like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Radius of curvature have been calculated in the same find_lanes() function in section 6. I used the lane fit polynomial coefficients to calculate the radius of curvature of left and right lanes in meters.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in section 7 of the jupyter notebook. Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. I ran the pipeline on an example project video. Here is the result:

Here's a [link to my video result][video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline has 4 main parts. The first part is undistortion. The second part is taking the undistorted image and applying color and/or gradient threshold to detect lanes in the image. Depending on what threshold is being used, the performance varies widely. I have seen that, in the video, when the contrast of the lane line with the road is not much, this step fails causing lane detection fail.
Next part is Perspective transformation.
Next is Findng the lanes by polynomial fitting. This step totally depends on how good the detected pixels are in second step. Overall, the result on the video is good, but it has difficulty when there is not much contrast beween lane line color and road color.
