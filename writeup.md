
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

<!-- [image1]: ./examples/undistort_output.png "Undistorted" -->
[image1]: ./examples/chessboard.png "Found Chessboard"
[image2]: ./examples/before_und.png "Before usdistorte"
[image3]: ./examples/after_und.png "After usdistorte"
[image4]: ./examples/und_testingImages.png "Testing image before and after usdistorte"
[image5]: ./examples/filters.png "After usdistorte"
[image6]: ./examples/warp.png "Warp Images"
[image7]: ./examples/hist.png "Histogram"
[image8]: ./examples/FindLane.png "Find Lane"
[image9]: ./examples/search.png "Search region"
[image10]: ./examples/sign.png "Sign image"
[image11]: ./examples/unwarp_lane.png "Unwarp lane image"
[image12]: ./examples/result.png "Fusing result"
<!-- [image3]: ./examples/binary_combo_example.jpg "Binary Example" -->
<!-- [image4]: ./examples/warped_straight_lines.jpg "Warp Example" -->
<!-- [image5]: ./examples/color_fit_lines.jpg "Fit Visual" -->
<!-- [image6]: ./examples/example_output.jpg "Output" -->
[video1]: ./output_images/project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

##### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

##### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in  "./src/CameraCalibration.py" it was a class for camera calibration, and the processing result was show in the title of Camera Calibration in the IPython notebook which located in "./MainProject.ipynb" 

In camera calibration, I using the `cv2.findChessboardConners` and `cv2.calibrateCamera` function to compute the camera calibration and distortion coefficients. Then using `cv2.undistort` to usdistorte all of images, images for calibration, and obtained this results :

The result of `cv2.findChessboardConners` :

![alt text][image1]

In this part it have some images can not find the chessboard, I think the chessboard is so close to image border that it can't find the conner well.

Camera calibration set before usdistorte :

![alt text][image2]

Camera calibration set after usdistorte :

![alt text][image3]

### Pipeline (single images)

##### 1. Provide an example of a distortion-corrected image.

In here, I using the coefficients which computed by `cv2.calibrateCamera` function and `cv2.undistort` to usdistorte all of testing images, then obtained this results :

Testing set before and after usdistorte :

![alt text][image4]

##### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Here I also build the class named `gradFilters` and `colorFilter` in "./src/Filters.py" to get binary filters, include `x gradint`, `y gradint`, `magnitude gradint`, `direction`, and `S channel from HLS color space` .

The result of all filters :

![alt text][image5]

##### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform, I compute the transfrom matrix (`M`) by source (`src_p`) and destination (`dst_p`) points for "warp" the image. Meanwhile, I also compute the transfrom matrix (`inv_M`) for "unwarp" the images. The code of this two part are titled by "warp" and "unwarp" in IPython notebook.

I chose the hardcode the source and destination points in the following manner:

```python
src_p = np.float32(
    [[535, 490], 
    [770, 490], 
    [1100, 719], 
    [190, 719]])

dst_p = np.float32(
    [[320, 0], 
    [1000, 0], 
    [1000, 720 - 1], 
    [320, 720 - 1]])
```

The result after "warp" : 

![alt text][image6]

##### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I design the class named "`Lane`", and the code of this class is titled by "class : Lane" in the IPython notebook. To identified lane-line pixels and fit their positions, I following the step which I learned in the class to achieve this, and I build the two function to processing it, named `find_lane_pixels` and `fit_polynomial`. However, find lanes via sliding window need to use a lot of computation source, so I build the `search_around_poly` function to find the line in query image when the lane was found.

The following steps when lane wasn't found :

* Caculate the histogram to find the left and right line.
* Using sliding window to find the lines.
* Using `np.polyfit` to find the second order polynomial to fit the lines

The following steps when lane was found :

* Setting the search region around the polynomial and count the point in the region
* Using `np.polyfit` to find the "new" second order polynomial to fit the lines

The result of each step :

* Histogram

![alt text][image7]

* Sliding windows and polynomial

![alt text][image8]

* Search region 

![alt text][image9]

##### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in function `measure_curvature_real` in class "`Lane`". To acheive this, I following the documents which I learned in the class and I set the ratio of meters to pixel in x and y dimension. The ratio I chose to the hardcode are in the following manner:

```python
ym_per_pix = 30/720
xm_per_pix = 3.7/650 
```

##### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the IPython notebook which titled by "unwarp". For the good result, I construct the `sign_img`, a white image, to let me know where the pixel is I need when I recover the image from "wrap" image, as shown in the below.

![alt text][image10]

Following this sign image can easy know which piexl I need, and I recover the warp the image and fusing it into original image via sign image, when the pixel value of sign image is 1 position was I need to fuse image to original one, otherwise 0 is no need.

* Unwarp image

![alt text][image11]

* Fusing image

![alt text][image12]

---

### Pipeline (video)

##### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

First problem I faced was which filters I need and how to fuse it. To find the best solution, I try many combination and finally I get it, and my combination is 
```python
(((xFilter == 1) & (yFilter ==1))==1) | (colorFilter == 1)
```
I think this was the best result now, but when I run in video I found a lot of issue on it. Following the recommendation, I build the Lane class, but I didn't know when is detected or not, so I try to find the different between polynomial coefficients, if the different is too large it should be fail case, luckly it seems right direction. But I face the another problem soon, when the lane detection is fail, I will using the prior points I save in the list to draw the lane, temporarily. However, I wasn't stable, the main problem of this was I save too many prior points, it let result of draw line can't fit the lane. Finally, I set the iteration parameter as 10, when the size of list of the prior points are over 10, it will upadte it and remove oldest data. Alought I do those work, it seen not really robust, because the white lane was have large gap so it can't find it easily, so I try to reduce the parameter in minpixel, to make it do easily, and now it seem well.

I think I still have some problem can make it more robust, but I have no idea to acheive it. I think the key problem of this project was how to filter the lanes in the picture, despite we already have many good filter but the threshold of those filter was fixed, but the influence of illumination was too serious. So, I think if I can develop the algorithm that can decide the threshold in different illumination situation, and figure out the best combination of all filters.

