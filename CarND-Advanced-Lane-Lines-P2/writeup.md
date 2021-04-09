## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
![Lanes Image](./examples/example_output.jpg)
---

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
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: utils/grad_thresh_1.png "grad_thresh_1"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./utils/line_fitting.png "Fit Visual"
[image6]: ./utils/final.png "Output"
[image7]: ./utils/undistort_road.png "Output"
[image8]: ./utils/grad_thresh_2.png "grad_thresh_2"
[image9]: ./utils/hls.png "grad_thresh"
[image10]: ./utils/hsv.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
---
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.
 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Distortion-corrected image.

On the test images provided, we get a result of undistortion like:

![alt text][image7]

#### 2. Colour and Gradient thresholding

I used a combination of color and gradient thresholds to generate a binary image. Below are the available gradient thresholdings.

![alt text][image3]
![alt text][image8]

Although we had 4 different thresholding methods, After tons of testing only a combination of magnitude and sobel-x were chosen as most appropriate.

| Gradient method | Kernel Size  | Threshold   | 
|:-------------:|:-------------:| :-------------:| 
| sobel-x      |          3      | 50-150       | 
| magnitude    |          3      | 50-150     |

In term of colour thresholding, below were the available colour space conversions where thresholding was possible.

![alt text][image9]

Firstly we had the HLS space. After testing many combinations, the S and H chanels of this space were selected for thresholding as they had the most defination for yellow and white.

![alt text][image10]

The HSV space was not used.

| Space         | Chanel        | Threshold |
|:-------------:|:-------------:| :-------------:| 
| HLS           | S             | 100-200 |
| HLS           | H             |100-150|


#### 3. Perspective transform

The code for my perspective transform includes a function called `p_trans()`, which appears in the file `advaced_lane_line.ipynb`I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[w, h-10],[0, h-10],[546, 460],[732, 460]])
dst = np.float32([[w, h], [0, h], [0, 0],[w, 0]])
```
Where h and w is the height and width of the image.

This resulted in the following source and destination points:

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Lane-Line Pixels and Line fitting

The line fitting code is present in `advanced_lane_lines.ipynb` under the **Line Fitting Helpers** section. The two helper functions used were `find_lane_pixels` and `fit_polynomial`.

![alt text][image5]

I simply used the sliding window technique to find the histogram, and then found the (x, y) points of the respected lane lines. I then used the `cv2.fillPoly()` function to fit a line through these points.

#### 5. Radius of curvature

The ROC calculation is present in the `advanced_lane_lines.ipynb` under the **Radius of curvature Helpers** section. The helper function `roc_measure()` was used.

I simply calculates the left and right curve (by using `np.polyfit()` as suggested in the lecture content) in radians and averaged them.

#### 6. Pipeline

I implemented the pipeline in the `advanced_lane_lines.ipynb` file. 

The steps in the pipeline are as follows:

- undistot input image
- convert to grayscale
- gradient thresholding ([As discussed here](####-2.-Colour-and-Gradient-thresholding))
- colour thresholding ([As discussed here](####-2.-Colour-and-Gradient-thresholding))
- combine thresholds
- perspective warping on combined binary
- find lane lines
- fit polynomial line
- inverse perspective transform
- apply this as a weighted mask to input image.

![alt text][image6]

---

### Pipeline (video)

Here's a [link to my video result](./project_output.mp4)

---

### Discussion

- Due to heavy time constraints, I did not implement the more efficient search around the polynomial function for easy recalculation of lane-lines. My current method is not the most robust.

-  A possible improvement maybe to make the lane detection vehicle specific (there would be psedo lane lines created if no markings are detected).

- In a real world scenario, there may be gaps in the lane markings (or no marking at all) on the road. This makes a NN styled semantic networks more relevant than our current implementation.

- Smoothing of lane lines over time should also be implemented.
 
