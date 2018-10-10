## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./output_images/warped_straight_lines.jpg "Transformed Straight Lines"
[image3]: ./output_images/All_Thresholds_Combined.jpg "Binary with all thresholds combined"
[image4]: ./output_images/result_test3.jpg "Result"
[image5]: ./output_images/Color_Threshold_H.jpg "Color Threshold H"
[image6]: ./output_images/Color_Threshold_L.jpg "Color Threshold L"
[image7]: ./output_images/Color_Threshold_S.jpg "Color Threshold S"
[image8]: ./output_images/Gradient_Direction_Threshold.jpg "Gradient Direction Threshold"
[image9]: ./output_images/Gradient_Magnitude_Threshold.jpg "Gradient Magnitude Threshold"
[video1]: ./output_video/project_video.mp4 "Video"


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

In the section "Camera Calibration" the images of the calibration target from the "camera_cal" folder are being processed to obtain the calibration parameters of the camera.
There are two functions defined in the section: `cal_get_calib_params()` and `cal_undistort()`.

##### cal_get_calib_params()
This function goes through all the calibration images and applies the `cv2.findChessboardCorners()` function to extract the chessboard corners, which are being added to the `img_points` array. These image points are being paired with predefined object points. These pairs are used to compute the camera parameters.
This function is only executed once before the image processing pipeline is executed.

##### cal_undistort()
This function takes an image and the camera parameters as inputs and then undistorts the given image.
This function is executed for every image, that is being processed by the pipeline.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The result of the `cal_undistort()` function can be seen in the following image:
![alt text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In order to obtain a binary image, that contains the lane markings, I implemented several functions:

##### Color Space Thresholding
The function `cst_threshold()` takes an RGB image as input and converts it to the HLS color space by using `cv2.cvtColor()`. The function accepts a `cannel` parameter, that allows you to specify, which channel is supposed to be thresholded. The actual threshold is the third parameter, which is provided as a tuple (minimum and maximum value).
The function returns a binary image, where all pixels, that are laying between the minimum and maximum threshold value, are set to 1.

The results of the function can be seen in the following images:
![alt text][image5]
![alt text][image6]
![alt text][image7]

##### Gradient Thresholding
In this section two methods are implemented, `grad_mag_thresh()` and `grad_dir_thresh()`

`grad_mag_thresh()` computes the gradient in x and y direction of a given image and then combines these values into the overall gradient magnitude. The gradient values are then thresholded (the threshold is a function parameter) to produce a binary image.
The following image gives an example for the result:
![alt_text][image9]

`grad_dir_thresh()` also computes the gradient of a given image in x and y direction. It then computes the gradient direction using the `np.arctan2()` function. The direction values are then thresholded by the threshold parameter and a binary image is produced.
![alt_text][image8]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

In the section "Perspective Transform" the function `perspective_transform()` is implemented. Within the function, the source and target points for the transform are defined, which were determined by using the straight line images, that were provided in the "test_images" folder. I chose the following values for these points:

```python
src = np.float32([[200,720],
                  [560,470],
                  [720,470],
                  [1105,720]])
dst = np.float32([[400,720],
                  [400,0],
                  [880,0],
                  [880,720]])
```

The result of the transform can be seen in the following picture. 

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image2]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for identifiying lane pixels can be found in the section "Find the Lines". It contains three functions.

##### find_init_line_pos()
The `find_init_line_pos()` function is used to find the initial position of the lane markings in a transformed binary image. It calculates a histogram along the x axis of the image. The values for the x positions describe how many pixels in the respective image column are set to 1. The initial positions are then determined by the two histogram maxima left and right of the image center.

##### find_lines_sliding_win()
The `find_lines_sliding_win()` function uses the sliding window approach presented in the lectures, in order to find the lane pixels in a warped image without any prior knowledge about the line positions.

##### find_line_along_prev_line()
This function's purpose is to find line pixels in a warped image along given lines. The function searches for pixels within a defined margin to the left and the right of the given lines. It is used when the line postion is roughly known from previous video frames. This makes the line finding more robust and also speeds up the process.


The code for fitting a polynomial through the found pixel postions is located in the section "Fit Polynomial". It consists of the function `fit_polynomial()` which receives pixel positions of the right and left line and uses the `np.polyfit()` function to fit a second order polynomial through the positions.
The lines are returned as two `Line` objects. The `Line` class is defined in the "Line and LineHist classes" section of the notebook.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

##### Calculating the lane radius
As mentioned above, in the "Line and LineHist classes" section a `Line` class is defined. This class holds the three polynomial parameters of a line and provides functions to calculate the radius of the line in pixel units or in meters. The function to return the radius in meters is called `radius_m()`. The radius is calculated at the bottom if the image by default.

##### Calculating the vehicle position with respect to the lane center
The function `calc_dist_to_lane_center()` can be found at the end of the section "Helpers". It calculates the center between the left and the right line and based on that calculates the offset to the image center. The return value is in meters. We assume here, that the mounting position of the camera has no lateral offset.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Plotting back the detected lane onto the original image is done in the section "Process Image". After fitting the polynomials through the lines, a green area is drawn between the left and the right line in the warped image. For that the `draw_lane_area()` function from the "Helpers" section is used. The lane area is then unwarped by applying the inverse warping function. Finally the area is overlayed onto the original image.
In addition to the lane area the selected line pixels are marked in the original image as well. The following image shows the final output of the image processing pipeline.

![alt text][image4]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline performed reasonably well on the project video. However I had some trouble with areas, where there were shadows on the road surface and where the road surface changed. I think detecting the lane lines by color thresholding is not really realiable, especially in changing lighting conditions.
By applying a strict ROI masking, the pipeline is also not able to follow curves with a very small radius.

In order to make my pipeline more robust on the video, I implemented a line history, so that the lane estimation is based not only on the lines of the current frame, but also on the lines of the previous 10 to 15 ones. I implemented a sanity check, that checks the detected lines in each frame. If the check is passed, the lines are added to the line history and discarded otherwise. The sanity check checks for the lines to have a similar curvature, the same curvature direction (sign of the curvature) and the distance between the lines.

I think a better feature for detecting the lines is the gradient. Although it's very noisy, the gradient was quite reliable. An additional feature could be "double gradients". Each line has a certain width and therefore a gradient on its left and right edge. Maybe it would be possible to detect those gradient pairs.