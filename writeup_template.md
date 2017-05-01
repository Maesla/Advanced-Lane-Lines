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


[image1]: ./output_images/undistorted_image.png "Undistorted"
[original]: ./output_images/test5.jpg "Original"
[image2]: ./output_images/undistorted_road.png "Road Transformed"
[thresholded]: ./output_images/thresholded_images.png "Thresholded Image"
[transformed]: ./output_images/transformed.png "Transformed Image"
[window]: ./output_images/window_lane_detection.png "Window Image"
[polynomial]: ./output_images/polynomial_detection.png "Polynomial Image"
[final]: ./output_images/final_result.png "Final Image"

[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in [advance\_lane\_lines.ipynb](./advance_lane_lines.ipynb)

In this block, I use the images located in the calibration folder (camera_cal). With this images, I use the function **cv2.findChessboardCorners** to find the corners of the chessboard.

For calibrating the camera, I use **cv2.calibrateCamera**

The result can be found at the end of cell #3

![alt text][image1]

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)
To demostrate the pipeline, I have prepared a code block named as Image Playground (cell #6 and #7), at  [advance\_lane\_lines.ipynb](./advance_lane_lines.ipynb)

![alt text][original]

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In cell #6 and #7 I have tested several thershold methods, but isolated.
The final implementation I use can be found at cell #8, named by **Test full Preprocess**

I use 3 masks.

I use 2 color channels. I use red channel from RGB color space and saturation channel from HSL color space.
I also use a sobel x gradient, applied to saturation channel.

Each mask as its own threshold values. Finally, I combine the free masks as:

Masked <- ((Red Mask) AND (Saturation MASK)) OR (Sobel Gradient)

The next image shows the result:

- Red channel is mapped to the red mask
- Green channel is mapped to the sobel gradient mask
- Blue channel is mapped to the saturation mask
![alt text][thresholded]

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
I transform the image by two steps.

In the first step, I calculate the Transformation Matrix and its inverse. To do this, I define 4 points in the original image and their equivalents in the transformed image.

I use these points:
This resulted in the following source and destination points:

| Source        | Destination   | Place |
|:-------------:|:-------------:| :-------------:|
| 197, 715      | 300, 715        | Bottom Left |
| 1136, 715      | 1000, 715      | Bottom Right |
| 594, 447     | 300, 0      | Top Left |
| 691, 447      | 1000, 0        | Top Right |

With these points, I get the matrix with **cv2.getPerspectiveTransform** and I transform the image with **cv2.warpPerspective**

The code can be found at cell #6 **perspective_transform_full_process** and **perspective_transform**.


![alt text][transformed]

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

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

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
I have implemented the both methods suggested at Lesson "Advanced Lane Finding".
##### Finding by windows
The first method is by window method. In this method, you get the 2 anchor points by an histogram.
From this point, the algorithm calculates windows from this anchor points and find pixels in that window. With this pixels, it calculates a new line point.

With all these points found, we can calculate the polynomial with **np.polyfit**

![alt text][window]

This method can be found at cell #9, named by **Sliding Window**
##### Finding by Polynomial
This method use a precalculated polynomial, with the previous method for instance.
With the polynomial calculated, you can find a new polynomial searching pixels in the proximity of the previous line

![alt text][polynomial]
This method can be found at cell #10, named by **Polynomial**






Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The curvature is based on Lesson **Advanced Lane Finding**, point 35, **Measuring curvature**, and in [this article](./http://www.intmath.com/applications-differentiation/8-radius-curvature.php)

This method can be found at cell #11, named by **Curvature**
In this block I calculate the curvature in pixels and in meters. I calculate also the vehicle bias from the lane center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This method can be found at cell #12, named by **Draw In image**
In this block, I draw a polygon with the calculated lines. 
Then, I transform this polygon with the inverse matrix transformation and I merge it with the original image.
Then, I write at the image the curvature and the bias

![alt text][final]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).
To provide the video, I have merged all the algorithm and techniques in two classes, LaneDetector and Line. LaneDetector has two Lines, left line and right line.

Line class is at cell #13 and LaneDetector is at cell #14

Here's a [link to my video result](./out_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline is basically the pipeline explained at Lesson **Advanced Lane Finding**


1. Calibration
2. Apply calibration to the image.
3. Preprocess image (thresholding)
4. Transform perspective
5. Lane Detection  

In my opinion, there are two tricky parts.

The image processing could be very tricky. It is very trial and error. A threshold combination could work very well for an image, but it could fail in some situations. The finding algorithms are applied to this image, so if this image is wrong, the algorithm is going to fail.

The second problem is the false positive. When you find the right and left lane, the curvature is ok and they are parallel, but maybe you have detected something else as line, not the true line.

I improve this problem, I think it is very important to check the result, with historical data or even better, with 2 or 3 algorithm totally different, based on very different approach, and verified the result is similar
