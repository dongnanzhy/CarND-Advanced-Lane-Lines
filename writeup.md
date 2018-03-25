## Writeup

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

[image1]: ./output_images/chessboard_corners/calibration17.jpg "chessboard corners"
[image2]: ./camera_cal/calibration4.jpg "origin chessboard"
[image3]: ./output_images/camera_undistortion/calibration4.jpg "undistort chessboard"
[image4]: ./output_images/perspective_transform_label.png "perspective_transform_label"
[image5]: ./output_images/perspective_transform.png "perspective_transform"
[image6]: ./output_images/perspective_transform_binary.png "perspective_transform_binary"
[image7]: ./output_images/detect_lane_pixels/test2.jpg "detect lane pixels test2"
[image8]: ./output_images/detect_lane_pixels/test6.jpg "detect lane pixels test6"
[image9]: ./output_images/final_result/test1.jpg "final_result test2"
[image10]: ./output_images/final_result/test6.jpg "final_result test6"

### [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

---
### Repository Structure
1. `./writeup.md`: final report
2. `./camera_calibration.ipynb`: exploration on camera calibration and undistortion
3. `./binary_generation.ipynb`: exploration on color transformation, gradients and binary threshold
4. `./perspective_trans.ipynb`: exploration on perspective transformation
5. `./final_pipeline.ipynb`: final pipeline code
6. `./camera_cal/camera_calibration_mat.p`:  camera matrix and distortion coefficients
7. `./camera_cal/perspective_transformation_mat.p`: perspective transformation matrix
8. `./output_images`: output images/videos of each step
9. `./output_images/project_video.mp4`: output project videos

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### 1. Camera Calibration

#### Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the IPython notebook [here](./camera_calibration.ipynb).

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  Detected corners example can be seen below.

![chessboard corners][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

original image
![original chessboard][image2]

undistort image
![undistort chessboard][image3]

### 2. Color Transform and Gradients

The code for this step is contained in the IPython notebook located [here](./binary_generation.ipynb). I did not include too many images in the report cause I had most of the exploration in the IPython notebook directly.

The course provides us a vareity of tools for detecting lane lines in the images as follows: (1) gradient in x and y directions, (2) magnitute of gradient, (3) direction of gradient, (4) HSV and HLS color spaces.

I spent a lot of time playing around with various combinations of them, but my tunning did not beat the tunning I saw during the office hours. So I ended up using that. The combination of x, y sobel gradient on the third channel in HLS transformation turned out to be the most useful to cleanly detect the lane lines and colors.

### 3. Perspective Transform

The code for this step is contained in the IPython notebook located [here](./perspective_trans.ipynb).

I spent hours playing around with the source and destination points to get a reasonable mapping for the lanes. In the end, I borrowed some other students' ideas and applied the transformation. One of the key factors was that the src points extended across the y-axis of the image. the offset parameter which is shown in the code snippet below controls how much of the original image is included in the perspective transform. The lower the number the more streched out the final image. This means less of the original image is going to be included in the final image. This was a key parameter to exclude excessive features from going into the model. For instnace if offset was larger, we would be including cars and lanes in which the car is not driving in.

The result of labeled points can be seen below.

![perspective transform label][image4]

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 585, 455      | 200, 0        |
| 190, 720      | 200, 720      |
| 1130, 720     | 930, 720   |
| 705, 455      | 930, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. See images below for some examples:

![perspective transform][image5]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

With exploration in **camera calibration** above, I stored camera matrix and distortion coefficients, so that I can directly use them in the pipeline. Below is how I apply undistortion, which is part of my final pipeline [code](./final_pipeline.ipynb).
```python
# Undistortion
def undistort(img, mtx=mtx, dist=dist):
    return cv2.undistort(img, mtx, dist, None, mtx)
```

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

With exploration in **Color Transform and Gradients** above, I used a combination of x, y sobel gradient on the third channel in HLS transformation.  Code can be found in my final pipeline [code](./final_pipeline.ipynb).

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

With exploration in **Perspective Transform** above, I stored perspective transformation matrix, so that I can directly use them in the pipeline. Below is how I apply perpective transformation, code can be found in my final pipeline [code](./final_pipeline.ipynb).
```python
def perspective_transform(img, M=M):
    img_size = (img.shape[1], img.shape[0])
    warped = cv2.warpPerspective(img, M, img_size)
    return warped
```

Result of perspective transformation on gradient threshold binary image can be seen below.

![perspective transform binary][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For detecting lines I used the sliding window methodology introduced in the course lectures. A given image was sliced into 9 segments. A rectangular area was defined along each segment, for the left and right lanes. All the active pixels (non-zero) were identified inside each rectangle. A second order polynomial was fit into the point clouds.

Please see images below for some samples from this process:

![detect lane pixels 1][image7]

![detect lane pixels 2][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The polynomial fits were implemented on the image scale. I then used a conversion, to map points along each lane in the image into real world scale (unit meter). A polynomial was then fit to the real-wrold points, and the closest point to the vehicle was chosen to calculate the curvature. the curvature was calculated according to the formula introduced in [Perspective Transformation](http://www.intmath.com/applications-differentiation/8-radius-curvature.php) as suggested in the course lectures. The curvature was then added to the final image. Based on the results and video implementation
I think the curvature values are within acceptable range.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

All of the steps above was combined together into a single pipeline. Before feeding the images to the pipeline they were undistored and warped accordingly.

Images below show that the algorithm is able to identify the region of inetersted between the lane lines accurately. One problem is that I stored the test images into `BGR` instead of `RGB` for now, will figure it out and fix this.

![final result test 1][image9]

![final result test 2][image10]
---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video.mp4)

---

### Discussion

#### Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

1. The pipeline initially failed when another vehile started to approach from the right. I had to tune the perspective transformation to make sure it does not cover too much of the original image. We are mainly concerned with the area in front of the vehicle.  I spent alot of time on perspective transformation and gradients. I wish there was a better and faster way of doing this.
2. This pipeline did not do well on more challenging videos. I also applied an outlier removal strategy to remove polynomial fits that were very different from their previous fits. This did not improve the result because polynomials have many degrees of freedom and I did not find any threshold that does well across the entire movie.
3. After the first submission, I got feedback from Udacity to improve on 2 following aspects.
	- One issue is with the logic for determining which direction the car has deviated (right or left from center). Video shows car actually in the left side of the lane for the entire video, but my output shows the car as being right of center. To fix this, I checked my logic at `Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position` code section of Jupyter Notebook, found I made a mistake on deciding vehicle positions. If estimated `line_middle` is larger than half of image width, it actually means vehicle is to the left of lane center. After fixing this, result seems to be pretty good.
	- Another issue is although positions of the lane lines have been accurately identified in almost every frame of the video, there are still several significant errors where the detected lane lines have deviated from the actual locations of the lane lines. To fix this, I first checked and found that these detections deviated significantly from those in recent frames, which means these errors may be easily fixed by checking recent continuous frames. In `Video Pipeline` code section of Jupyter Notebook, when the calculated curvature is significant different from recent few frames, I will instead not to update it. After fixing this, the output videos seem be be much better and smoother than before. In the future, I will continue tuning other color space or sobel parameters, as well as improve my curvature calculation functions to improve this.

