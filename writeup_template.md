## Writeup Template

### You can use this file as a template for your write-up if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./output_images/un_distorted_real.png "Undistorted"
[image2]: ./output_images/un_distorted_real_test.png "Road Transformed"
[image3]: ./output_images/binary_combo_real.png "Binary Example"
[image4]: ./output_images/warped_straight_lines_real2.png "Warp Example"
[image5]: ./output_images/warped_line_detected_box_method.png "Fit Visual"
[image6]: ./output_images/final_image.jpg "Output"
[video1]: ./output_images/project_video_output_final.mp4 "Video"
[Notebook Link]: ./examples/example.ipynb "Notebook Link"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### All the codes are written in Jupyter note book, found in [Notebook Link]
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your write-up as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion-corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in [Notebook Link] 

I start by preparing "object points (obj_points)", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `obj_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `img_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `obj_points` and `img_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the test image using the `cv2.undistort()` function. In the notebook, you can find the method  `remove_distortion` in the  second cell where I am performing these operations obtained below result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
Below I have provided an example of original image and distortion corrected image.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. In 3rd code cell in the notebook, you can find method  `gradient_color_transform`.  Below is the brief summary

  * Converted the image to gray and applied Sobel transformation, with a threshold (20, 100) to get the sobel_x image
  * Converted the image to HLS format, extracted the saturation part
  * Applied the threshold (170, 255) on the saturation part to get the s_bianry part
  * combined sobel_x and s_binary part to get the image
 Below is the sample image as a result of the transformation
 
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

In the notebook, 4th code cell, I have added function `transform(cur)` for applying the perspective transformation. I chose to hardcode the source and destination points as follows
```python

    src = np.float32([[200, img_size[1] - 1], 
                      [595, 450], 
                      [685, 450], 
                      [1100, img_size[1] - 1]])
    
    dst = np.float32([[350, img_size[1] - 1],
                      [350, 1],
                      [970, 1],
                      [970, img_size[1] - 1]])
```
Using `getPerspectiveTransform` function I got the perspective matrix, also calculated `Minv` inverse perspective transformation for later purpose. Transformed the image to warped using `cv2.warpPerspective` function.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Code for transformation can be found in the 5th cell of the notebook. I have followed the sliding window search approach to detect the lines. There are two functions I have used. 

In `find_lane_pixels(binary_warped)`  function, I am finding the line using a sliding window approach. Below is the brief summary of the algo
   * Find out the peak non-zero pixels using the histogram, for x coordinates
   * Divide the image into two halves for left and right lanes, ignore some portion of left inorder to avoid the misidentification of divider as the line
   * Divide the image into different boxes according to y coordinates
   * In each box, find the non-zero pixels in the box for corresponding left and right side and add them to the list
   * Find all the x and y coordinates for left and the right side plane from the list

Second function `fit_polynomial(binary_warped)` uses the above method to get the pixels and fit it into the line. Brief steps
   * Get the left and right x, y pixels from `find_lane_pixels(binary_warped)` method
   * Fit those in np.polyfill method to get the coefficients for the second-degree polynomial
   * Calculate the x coordinate using the coefficient and y (calculated using np.linespace method)

Below is the sample image
![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to the center.

Both the radius of curvature and position of the vehicle w.r.t center I have calculated within  `getCurvature_and_center_of_line` method in the 5th code(first method in the cell) cell of the notebook. 
   
   The radius of the center
   * First need to find the coefficients in the meter scale. For that, I have used the `np.pollyfill` method. Used below x and y value for meter conversion
     ```python
      ym_per_pix = 30 / 720  # meters per pixel in y dimension
      xm_per_pix = 3.7 / 700  # meters per pixel in x dimension
     ```
   *  Then using the equation given in the lesson, find the radius for both left and right plane. Then took their mean for the final value
   
   Center of vehicle w.r.t lines center 
   * Found the bottom left and bottom right values of x coordinate of the lines using y_max value to get the line bottom
   * Found the center of the car by taking the half of image width
   * Found the difference between the car center value and center of line bottoms 
   * if the difference is negative, the car is on the left side, otherwise on the right side
   
#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this is in the 6th code cell. ` transform_back_add_text` does the job. It plots the image back to the original image and annotates with color to indicate the area between the lanes. Also, this function add radius of center and center of vehicle w.r.t the center of the line 

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [video1]

---

### Discussion

#### 1. Briefly discuss any problems/issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Below are the problems which I found in my approach
   * Sliding window approach takes too much time for identification of the line since each time it is searching for the line from scratch
   * When the lines are not visible properly due to the sunlight or shades of a tree, some time the algo fails to detect the line properly
   * when there is a significant variation in the color gradient (as we can see the challenge video), the algo misidentifying the color difference in the road surface as the lane.
   * When the roads are too curvy, and other objects are exist in the roads (as in the harder video), algo misses the lanes, so the mapping 
   
