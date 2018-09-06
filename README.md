
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

[image1]: ./output_images/UndistortedChessBoard.jpg "Undistorted"
[image2]: ./output_images/distortedChessBoard.jpg "Distorted"
[image3]: ./output_images/SaturationColorChannel.jpg "Saturation Channel"
[image4]: ./output_images/RedColorChannel.jpg "Red Channel"
[image5]: ./output_images/perspectiveTransform.jpg "Perspective Transform"
[image6]: ./output_images/polyfitlines.jpg "Polynomial Picture"
[image7]: ./output_images/finalResult.jpg "Result"
[image8]: ./output_images/calibratedImage.jpg "Calibrated Image"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading this glorious work of art aren't you?

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the jupyter notebook located in "Project4/AdvancedLaneFinding.ipynb"   

I start by generating "objectPoints", which are the assumed (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each chessboard calibration Image.  Thus, `objectPoints` is just a replicated array of coordinates, and will be appended with a copy of it every time all the chessboard corners are detected in a test image.  `corners`, which represent each intersection of four squares on the chessboard, will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objectsPoints` and `corners` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result, in the first image, from the second image:

![alt text][image1]
![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image8]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color  thresholds to generate a binary image for the left and right lane respectively shown below. The code can be found in cell 10 of 'AdvancedLaneFinding.ipynb'.

### Left Lane Saturation Image
![alt text][image3]
### Right Lane Red Channel Image
![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspectiveTransform()`, appears in cells 6 and 7 of AdvancedLaneFinding.ipynb.  The `perspectiveTransform()` function takes as input an image (`img`). I chose to hardcode the source and destination points in the following manner:

```python
src = np.float32([[15, 720],[560, 450],[720, 450],[1280, 720]])
dst = np.float32([[0, 720],[0, 0],[1280, 0],[1280, 720]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 15, 720      | 0, 720        |
| 560, 450      | 0, 0      |
| 720, 450     | 1280, 0      |
| 1280, 720      | 1280, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial

In cell 5 of 'AdvancedLaneFinding.ipynb' using find_lane_pixels(), the pixels were identified using a sliding windows approach. First, a histogram of the lower half of the image was constructed to determine the starting windows locations for the
left and right cell. The next window location is then changed based on the mean location of the pixels within the preceding window. The pixels locations are then used to produce the second-order polynomial coefficients using the np.polyfit() function.
These lines are then plotted on top of the image as seen below.

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.


In cell 14 of 'AdvancedLaneFinding.ipynb', I measure both the curvature of the lane and position of the vehicle with respect to center. The latter is calculated by taking the left and right lane coordinates at the
bottom of the image and averaging them, then subtracting them from half the width of the image. This left the measurement in pixel units which was then changed to meters by multiplying the number of pixels by the spatial
conversion factor. The radius of curvature was calculated using similar converions from pixelspace to meters and then using the formula R=[1+(y′(x))^2]*32.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this final pipeline in the process_image() function in cell 16 of AdvancedLaneFinding.ipynb.

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./final_output_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The principle problem faced in the implementation of the project involved the binary images produced through the thresholding function. I experimented with various values, tested different functions that relied on the average color values of the images, but all the combinations had parts of the video that they struggled with. In the end, this is the pipelines major barrier to robustness. In order to solve the problem
an approach to take would be using the sobel operator rather than color channels or a combination of both to detect the pixels where the lane markings exist. This would allow the pipeline to be applied to a variety of environments.
