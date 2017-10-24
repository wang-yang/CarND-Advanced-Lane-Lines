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
[image2]: ./output_images/undistorted_road.png "Road Transformed"
[image3]: ./output_images/threshold.png "Threshold"
[image4]: ./output_images/perspective_transform.png "Perspective"
[image5]: ./output_images/lane_line.png "Lane line"
[image6]: ./output_images/marked.png "Marked"
[image7]: ./output_images/final.png "Final"

[video1]: ./project_video.mp4 "Video"

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell `#1` of the Jupyter notebook located in `./project.ipynb`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![Undistorted][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![Road Transformed][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used the color thresholds to generate a binary image (thresholding steps at code cell `#4` in `./project.ipynb`).  Here's an example of my output for this step.

![Threshold][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in code cell `#6` and `#7` in `./project.ipynb`.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src        = np.float32([[(200, 720), (570, 465), (730, 465), (1200, 720)]])
dst        = np.float32([[(320, 720), (320, 0), (980, 0), (980, 720)]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 200, 720      | 320, 720      |
| 570, 465      | 320, 0        |
| 730, 465      | 980, 0        |
| 1200, 720     | 980, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Perspective transform][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code cell I used to identified the lane-line pixels are `#8` in `./project.ipynb`.
I used sliding window algorithm to search the line: it divide the image into a series of windows, and find a series of valid indices. Then use these indices to fit into the left line and right line with `np.polyfit(lefty, leftx, 2)` and `np.polyfit(righty, rightx, 2)`

After that, `draw_fit()` function will use the fitted lines to generate a series polygon to get the final lane boundary.

![Lane Line][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code is in cell `#10` and `#11` in `./project.ipynb`.
The `compute_curvature_and_distance()` function first use the average curvature of the left line and right line to calculated the curvature of the current vehicle position. Then calculated the center of the car and the center of the lane, the difference of them is the distance.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code is in cell `#12` to `#15` in `./project.ipynb`.
The `mark_lane()` in `#12` use the inverse perspective matrix which we calculated before Minv to wrap the lane line back to the original image.
And the `draw_curvatures_and_distance()` will draw the curvature and distance on the output image.
Below is the result:

![Marked][image6]

![Final][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I found the camera calibration is the easiest and the binary thresholding is the hardest part. After many experiments, I found to detect white and yellow line separately works for me better. Also I have tried different color spaces, like S,V channels from HSV space and L from LUV space, and finally I choose L from LUV space.

The perspective transformation will fail because it use fixed points. So when the roads with different lane size, the pipeline will not properly handle the transformation. I think we could compute the point using ratio instead of hardcoded value to tackle with different lane size.
