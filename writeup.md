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

[image1]: ./output_images/cam_cal.png "Undistorted"
[image2]: ./output_images/test2_undist.png "Road Transformed"
[thresh]: ./output_images/test2_thresh.png "Binary Example"
[warped]: ./output_images/test2_warped.png "Warp Example"
[fitted]: ./output_images/test2_fitted.png "Fit Visual"
[final]: ./output_images/test2_final.png "Output"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./camera_calibration.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The camera calibration matrix and distortion coefficients were used with the OpenCV function `undistort` to remove distortion from highway driving images. An example is shown below:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color thresholds to generate a binary image. I used B channel of LAB color space for detecting yellow line and L channel of LUV color space for detecting white line. With some threshold values I successfully extracted both lines by combining both binary outputs from those channels. 
The code for this step is contained in the sixth code cell of the [IPython notebook](./Pipeline%20Image.ipynb#Thresholding).
Here's an example of my output for this step. The image is thresholded image after perspective transformation is applied to the undistorted image.

![alt text][thresh]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this step is contained in the fifth code cell of the [IPython notebook](./Pipeline%20Image.ipynb#Perspective-Transformation).
I chose the hardcode the source and destination points in the following manner:

```python
offset = 10 # offset for src points
src = np.float32(
    [(0, img_size[1]-offset),
    (490, 480),
    (820, 480),
    (img_size[0],img_size[1]-offset)])
dst = np.float32(
    [(0,img_size[1]),
    (0, 0),
    (img_size[0], 0),
    (img_size[0],img_size[1])])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 0, 710        | 0, 720        | 
| 490, 480      | 0, 0          |
| 820, 480      | 1280, 0       |
| 1280, 710     | 1280, 720     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
Here's an example of my output for this step. 

![alt text][warped]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I created a histogram with bottom half of warped image. Using left and right peaks, I was able to figure out starting points of left and right lines. Then I created sliding windows with following hyperparameters:

* Number of sliding windows: 9
* Width of the windows +/- margin: 100
* Minimum number of pixels found to recenter window: 50

These sliding windows helped me to fit two 2nd order polynomials using numpy's polyfit funtion.

Below is the image where you can see the sliding windows used for fitting the polynomial and left and right lines. You can see codes for this part in the [IPython notebook](./Pipeline%20Image.ipynb#Identify-lane-line-pixels-and-fit-via-polynomials).

![alt text][fitted]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in 2nd last cell of the [IPython notebook](./Pipeline%20Image.ipynb#Calculate-Radius-of-Curvature-and-Offset). For calculating radius of curvature I used the formula found [here](http://www.intmath.com/applications-differentiation/8-radius-curvature.php). As for the offset, I substracted the midpoint of the lane from the midpoint of the original image and change into meter multiplying by `3.7/700`.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in last cell of the [IPython notebook](./Pipeline%20Image.ipynb#Final-image-with-identified-the-lane-boundaries). Here is an example of my result on a test image:

![alt text][final]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4).

And this is the [link to the challenge video result](./challenge_video_output.mp4).

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Well, I faced a lot of difficulties like sudden distortion of the fitted polynomials may be due to lack of sufficient pixel points or due to outliers or due to the shadow of the overhead bridge in challenge video. I was fun solving these challenges although it was very difficult to work with the very curvy roads presented in harder challenge video. So, yeah overall fairly good detection but it will fail in the curvy roads and may be when there is sudden light changes and such. I think we can implement more robust deep learning techniques for solving those issues but I have not tried them out myself.
