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

[image1]: ./camera_cal/calibration1.jpg "distorted"
[image2]: ./camera_cal/calibration1_undistorted.jpg "Undistorted"
[image3]: ./test1_undistorted.jpg "Road Transformed"
[image4]: ./test1_threshold.jpg "Binary Example"
[image5]: ./test1_warped.jpg "Warp Example"
[image6]: ./test1_line_detection.jpg "Fit Visual"
[image7]: ./test1_final.jpg "Output"
[video8]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the Jupyter Notebook in the second cell.  First, I prepared the object points that had 3 coordinates of the chessboard.  Then I looped through all the images within the calibration directory (all images are of chessboards because it is easier to calibrate the camera.  Each image is read in and converted to gray scale.  Then for each of the images, I found the corners using the cv2.findChessboardCorners.  After the corners are calculated, I appended the object points to obj_points and corners to img_points.  After doing this for all the images, the calibrateCamera from cv2 is used to find the calibration matrix.  The image below is the output of the undistort function in cv2 using the matrix output and the dist output. 


![alt text][image1]
![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a couple of different color transforms and gradient transforms in order to create the thresholded binary image.  This code can be seen in cells 5-9.  The first threshold performed is the sobel threshold.  This can be done in the x or y direction.  In the threshol_pipline function, I get the threshold for both the x and the y direction separetly.  The next one is the direction threshold.  I basically get the abosolute in the x and y direction and then take the arectan of them.  The next threshold would be the magnitude threshold.  I do the same thing in the direction function except I square the x and y sobels and then take the square root of that.  The last one I look at the saturation channel of the hls image.  Then I threshold this channel.  In the final cell, I run all these functions on the undistorted image and then combine these thresholds.  The image below is the output image that I get when running the pipeline.

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The next part of the pipeline is to get the perspective transform.  This is performed in the warp_image function in cell 10.  For this I needed to choose sorce point (points on the original image) and destination points (points that I wanted the sorce points to bet transformed to).  For this I basically eyeballed it.  The following is what I chose:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 285, 720      | 250, 720      | 
| 590, 460      | 250, 0        |
| 720, 460      | 1060, 0       |
| 1125, 720     | 1060, 720     |

Once these points are chosen, I needed to used the getPerspectiveTransform function in the cv2 package to get the transform matrix and inverse transform matrix.  Once these matrices are found, I found the warped image using warpPerspective.  The following is the output image of this function.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then next part of the pipeline is to identify the lines and then fit a polynomial.  First I took the warped image that we previouly found and made a histogram off of the activated pixels along the x axis.  This can be seen in cell 12.  The line detection function that the histogram is call is in cell 14.  I then split the image in half to separate the left lin from the right line.  I am using a sliding window to detect lines.  I split it up into 10 windows.  Each window slides along and finds the good left and right indices.  Once this is done, polyfit is used to fit a polynomial to these points for the left and right lines.    If specified, the image will be returned along with the fitted points.

Cell 13 is then used to fit these lines without using a sliding window.  It uses the previously found polynomial to project into the distance.  This allows for faster comput time.  As you can see below, this is the output image for the calculated polynomials without using the sliding window.  This function basically takes in the previous lines and projects them into the future.  From frame to frame, the lines don't dramatically change so projecting the polynomial into the future is a lot less cost efficiant than running the sliding window everytime.  So we check to see if the line fit input is none.  If it is, then we run the sliding window.  If it is not, then we run throw the projection algorithm.  We get the non zero pixel first, then we will project based off the previous fitts into the next frame using polyfit. 

My output from projecting lane lines:

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The next part of the pipeline (cell 14) is to get the offset from the center of the lane and the curvature.  first, we find the polynomials of the left and right lane using polyfit.  I also chose to use 30/720 and 3.7/800 to convert from pixels to meters.  Then I calculate the curvature of each lane.  Once each of the curvatures are found in meters, I then add the left curvature and right curvature and devide by 2 to get the center curvature of the lane.  Then, I find how off the car is from the center of the lane.  I get the width of the image and devide by 2 to get the center of the car.  center fo the lane by taking the first point of the  x left and x right of the polynomals for the lane lines and add them up and devide by 2.  Now we have the cars center and the lanes center.  We then subtract these to find how much the car is off from the center of the lane.  

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The output of the curvature and the offset of the car with respect to the center of the lane can be viewed below:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My solution performs really well for small curvatures.  The problem with my solution is when we get to sharper turns and less of a shoulder.  You can view my video of the challenge [here](./harder_challenge_video_output.mp4).  As you can see in the challenge video, there are a lot more turns so it tends to fall apart.  I also think it falls apart due to the lighting changes.  This could be over come by adjusting the parameters in the thresholding part.  I think that the one of the main problems in the thresholding part is the saturation channel.  I think that this part needs to be tweeked quite a bit.  Other than that, this technique that I used performs really well in highway driving. For country roads and bad lighting, this will cause issues.


