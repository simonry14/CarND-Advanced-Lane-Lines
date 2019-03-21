## Advanced Lane Finding Writeup 

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

[image1]: ./output_images/Undistorted_Calibration_Image.jpg "Undistorted"
[image2]: ./output_images/undistorted_images/undistorted_test1.jpg "Road Transformed"
[image3]: ./output_images/binary_combined_example.jpg "Binary Example"
[image4]: ./output_images/warped_images/test1.jpg "Warped Example"
[image5]: ./output_images/polynomial_images/test5.jpg "Fit Visual"
[image6]: ./output_images/warpedback_images/test1.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./Advanced Lane Finding Project.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  

An example of a distortion corrected image (calibration3.jpg) is shown below:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I applied this distortion correction to all the images in the test folder  using the `cv2.undistort()` function and the undistorted images were saved in the "output_images/undistorted_images" folder. The code for this operation is in the second code cell of the IPython notebook located in "./Advanced Lane Finding Project.ipynb".

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. I explored various color spaces, and I eneded up using a combination of the HLS, LAB and LUV Color Spaces. From the LAB Color space, I isolated the B channel which I used to identify Yellow Lane lines. From the LUV color Space I isolated the L channel which I used to identify the white lane lines. I then used a combination of the above together with gradient thresholds to produce a thresholded binary image. The code for this is in the 3rd code cell of the IPython notebook located in "./Advanced Lane Finding Project.ipynb". All binary images for the test images were saved in the "output_images/binary_images" folder

Here's an example of my output for this step. 

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I perform perspective transform in the 4th code cell of the IPython notebook located in "./Advanced Lane Finding Project.ipynb". In this cell I have a function `perspective_transform()` which takes in an image and returned a warped image. This function also requires source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

All warped images for the test images were saved in the "output_images/warped_images" folder.  Here's an example of my output for this step. 

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I identify lane line pixels in the 5th code cell of the IPython notebook located in "./Advanced Lane Finding Project.ipynb". I do this by taking a histogram of the bottom half of the image and finding the peak of the left and right halves of the histogram. These peaks are the starting point for the left and right lines. I then used the sliding window method with 9 windows and a margin of 100 pixels to average the position of the lane pixels moving upward in the image.

After finding all pixels belonging to each line through the sliding windows method, I fit a 2nd order polynomial to the lines using np.polyfit. Since the lines don't necessarily move alot from frame to frame, using the sliding window method on each frame is inefficient and as such in the next frame I instead search in a margin around the previous line position. The green shaded region in the image below shows the margin where I searched for the lines.

All polynomial fit images for the test images were saved in the "output_images/polynomial_images" folder. Here's an example of my output for this step.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the 6th code cell of the IPython notebook located in "./Advanced Lane Finding Project.ipynb". First a coversion from pixel space to meters was done. The radius of curvature for each of the lines (left and right) was calculated using the formula R = ((1+(2Ay + B )^2)^1.5) / |2A|. An average of the two radii was then calculated.

To calculate the position of the vehicle with respect to the center, I first calculated the lane center by getting the mid points of the left and right fits. I then got the difference betwwen this lane center and the the mid point of the image. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in 7th code cell of the IPython notebook located in "./Advanced Lane Finding Project.ipynb". The 'drawLine' function warps the rectified image back onto the original image with the identify the lane boundaries clearly visible. An inverse perspective matrix (Minv) was used to 'un warp' the image. The addTextToImage function is used to add the radius of curvature and vehicle position text onto the image.

Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](.output_video/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The biggest problem I faced was determining the thresholds that give the best bianry image. This involved a lot of trial and error until good enough thresholds were found.

The pipeline will likely fail on a winding road with very sharp corners.

The pipeline could be made more robust by applying a more robust technique of optimising the thresholds used to produce the binary image. Perhaps a machine learning technique utlising convolutional neural networks could come in handy.
It is also possible that the thresholds that work well on certain road conditions fail on another so getting thresholds that are  good in all road conditions.
  
