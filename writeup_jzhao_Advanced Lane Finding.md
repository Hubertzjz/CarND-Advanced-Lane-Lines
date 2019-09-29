# **Advanced Lane Finding**

## Writeup - J. Zhao

### 2019/09/29

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
* Build video processing pipeline.

All the codes are built in the IPython notebook located in "./Advanced lane finding.ipynb".

[//]: # (Image References)

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./test_images/test4.jpg "Road Transformed"
[image3]: ./output_images/binary.png "Binary"
[image4]: ./output_images/warped.png "Warp"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/result.png "Output"
[video1]: ./output_project_video.mp4 "Video"

---

### Reflection

### Camera calibration using chessboard images

The code for this step is contained in the first code cell of the notebook.

It's started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the real world. Assuming the chessboard is fixed on the (x, y) plane at z=0. `objpoints` will be appended with a copy of `objp` if all chessboard corners in a test image are successfully detected. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

Then, `objpoints` and `imgpoints` are used to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Distortion correction of raw images

`cv2.undistort()` function located in 2nd code cell is applied with camera matrix and distortion coefficients to one of the test images to demonstrate this step:
![alt text][image2]

#### 2. Create thresholded binary image

A combination of saturation and gradient thresholds is used to generate a binary image (thresholding steps in 3rd code cell). Here's the output of the test image for this step.

![alt text][image3]

#### 3. Apply perspective transform

The code for my perspective transform includes a function called `perspective()`, which appears in 4th code cell.  The `perspective()` function takes as inputs the `binary image`. The source (`src`) and destination (`dst`) points are chosen as follow:

```python
src = np.float32([[180,img_size[0]],
                  [570,462],
                  [710,462],
                  [1100,img_size[0]]])
dst = np.float32([[200,img_size[0]],
                  [200,0],
                  [img_size[1]-200,0],
                  [img_size[1]-200,img_size[0]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 180, 720      | 200, 720        | 
| 570, 462      | 200, 0      |
| 710, 462     | 1080, 0      |
| 1100, 720      | 1080, 720        |

It is verified that the perspective transform was working as expected that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Detect lane pixels and polyniminal fit

This part includes two options, first is to find pixels from scratch with `find_lane_pixels()` appeared in the 5th code cell using sliding windows method, second is to search around polynominal fit from last fit lane line with a given margin using `search_around_poly()` in 6th code cell. The 2nd order polynomial is done like this:

![alt text][image5]

#### 5. Determine the curvature of the lane and vehicle position

The curvature of the lane and vehicle position is achieved by `find_curvature()` in the 7th code cell.

#### 6. Warp the detected lane boundaries back onto the original image and Output visual display of the lane boundaries

This step is implemented in the function `warp_back()` and `display_lane()` in 8th and 9th code cell respectively.  Here is the result of the test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Build video processing pipeline

The video pipeline is built in the 10th code cell, basiclly same with image pipeline but different in three parts:
1. Video processing (read and write by `cv2.VideoCapture()` & `cv2.VideoWriter()`).
2. Lane pixels detecting method `find_lane_pixels()` and `search_around_poly()` are chosen by frame number.
3. Data smoothing with between last `N_smooth` detected lane lines and vehicle position. 

Here's a [link to my video result](./output_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline is lack of sanity check, which is the reason why the pipeline does not work quite well with challenge.mp4, for further project, the sanity check and a more robust perspective transform may be an great improvement for the pipeline.
