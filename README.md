# Advanced Lane Finding Project**

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

[image1]: ./output_images/undistort_output.png "Undistorting Image"
[image2]: ./output_images/road.jpg "Road Transformed"
[image3]: ./output_images/binary.jpg "Binary Example"
[image4]: ./output_images/perspective.jpg "Warp Example"
[image5]: ./output_images/lane_lines.jpg "Lane Lines Visualization"
[image6]: ./output_images/output.jpg "Final Output"

A pipeline was produced to process the images in the video, which wlil be explained in the following steps. 

## Camera Calibration

The camera calibration was performed using the chessboard images in the camera calibration file. Using the cv2 function, the "object points",which will be the (x, y, z) coordinates of the chessboard corners in the world are detected. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

## Pipeline (single images)

### 1. Undistortion

The first step of the pipeline is to undistort incoming images. The image below is an example of the undistorted image. 
![alt text][image2]

### 2. Perspective transform

The image was subsequently passed through a perspective transform to obtain the birds eye view of the road for lane detection. This is done by identifying a polygon in the raw image to be mapped to a rectangle in the transformed image. To simplify the process, an image in which the road was straight was used to, as the transform would be validated when a rectangle is seen in the final image. 

I chose the hardcode the source and destination points. The exact parameters may be found in Code Block 14 of the project file. 

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. 

![alt text][image4]

### 3. Color Transforms and selection of Color Space
The next step in the pipeline is to generate a binary image based on the different thresholds across different colorspaces, to increase the contrast between lane lines and ground. Several colorspaces were explored and the experimentation may be found in the Project_working.ipynb code. Examples of these color spaces include using the Sobel Operator, RGB, HLS, YUV and grayscale. 

The final lane-line detection combination was used by merging the L-channel of the HLS and the Y-channel of the YUV color space. An either-or condition was used to enhance the robustness of the detection method, although this potentially introduces false positives as well, which will be discussed in the last section. 

The final color transformation is done via the function lanes_bw


![alt text][image3]

### 4. Lane Lines Identification 

The sliding window convolution method was used to detect the lane pixels for the left and right lane lines. This is performed in the get_lane_lines function, which returns the pixel coordinates of the left and right lane lines in the form of a binary image

The result is then passed into a convert_to_xy function, which converts this binary image into an array of points for polynomial fitting. The polynomial fitting is performed to a 2nd order polynomial using the np.polyfit function. This can be found in the get_rad_and_curv function. 

![alt text][image5]

### 5. Computing the radius of curvature and position relative to the center

The radius of curvature was computed in pixels, then converted to meters using a conversion factor of meters per pixel. This can be found in the get_rad_and_curv function. The radius of curvature of the left and right lane lines are computed, then averaged to obtain the radius of curvature at the center of the lane. 

Assuming the camera is mounted in the center of the car, we would assume the center of the car to be positioned at the center of the image. However, after obtaining the position of the left and right lane lines, we are also able to obtain the position of the center of the lane. As such, the number of pixels between the center of the image and the lane center. By multiplying this number by the conversion factor, we are able to obtain the off-center distance of the car. 


### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

## Pipeline (video)

### Video Submission

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 

There were several difficulties faced while working on this project. 

1. Hardcoding of the perspective transform. The perspective transform was hardcoded, meaning it is highly contingent on a fixed camera angle. As such, any slight shift in the camera angle could affect the perspective transform. To improve the robustness of the algorithm, the perspective transform should be calibrated on a per video basis. Also, calibration is best performed driving on a straight road, as it is difficult to obtain a trapezium on a curved road for the perspective transform. 

2. Obtaining an optimal threshold for identifying the lane lines. The grey roads tended to affect the detection of lanes. In addition, yellow lane lines were particularly difficult to detect. As such, the HSV colorspace was used to detect the yellow lines, and YUV for white lines. Superimposing the results would produce a binary image that could detect both yellow and white lines. 

3. Lane line detection: the pipeline would tend to crash should it not be able to detect lane lines on a particular side of the road. As such, the thresholds were loosened in order to allow for the detection of lane lines. 
