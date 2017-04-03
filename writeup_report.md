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

[image1]: ./output_images/undistort_image.png "Undistorted"
[image2]: ./output_images/original_threshold_image.png "Original"
[image3]: ./output_images/thresholded_binary_image.png "Thresholded Binary"
[image4]: ./output_images/image_with_source_points.png "Source Points"
[image5]: ./output_images/perspective_transformed_image.png "Perspective Transform"
[image6]: ./output_images/binary_lines.png "Binary Lines"
[image7]: ./output_images/final_result.png "Final Result"
[image8]: ./output_images/final_result_complete.png "Final Resul Step-by-Step"
[video1]: ./project_results_video.mp4 "Video"

## Rubric Points
#### Here I will consider the [rubric points](https://review.udacity.com/#!/rubrics/571/view) individually and describe how I addressed each point in my implementation. 
---

###Camera Calibration

The code for this step is contained in the 1st, 2nd and 3rd code cells of the IPython notebook located in "./P4.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####Threshold Binary Image

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps are contained in cells 4, 5, 6, 7 and 8 of the P4.ipynb IPython notebook).  

Here's an example of my output for this step.

![alt text][image3]

####Perspective Transform

The code for my perspective transform includes a function called `region_of_interest()`, which is located at the 10th cell of my notebook. That function strips out all the noise - sky, trees, etc - from the image, returnig just the desired portion where the perspective transformation will be applied. 

Another function called `perspective_transform()`, which is located at the 11th cell of my notebook is also used for the perspective transformation. That function takes the original image as well as the matrix and distortion points we found earlier with the camera calibration (note that a 4th variable is passed to this function to indicate if the image should be plotted or not). This function creates the following source and destination points:

```
#define offsets
xfd=54
yf=450
offset=205

#get source points
src = np.float32([
        (offset, img.shape[0]),
        (xcenter - xfd, yf),
        (xcenter + xfd, yf),
        (img.shape[1] - offset, img.shape[0])])

#get destination points
dst = np.float32([
        (offset,img.shape[1]),
        (offset,0),
        (img.shape[0] - offset, 0),
        (img.shape[0] - offset, img.shape[1])])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 205, 1280        | 
| 586, 450      | 205, 0      |
| 694, 450     | 515, 0      |
| 1075, 720      | 515, 1280        |

Example of image with source points detected.
![alt text][image4]

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

####Lane pixels and fit to find the lane boundary.

After applying calibration, thresholding and perspective transform, I had a binary image where the lane lines stand out clearly. To identify what pixels actually composed the lane lines, I use a histogram to verify what are the two most prominent peaks in the graph, which should indicate the x-position of the base of the lane lines. I then use a sliding window technique to search through the image starting from the x-position found by histogram to find and follow the lines up to the top of the frame. The code for the sliding window search is located at the 13th cell of the notebook, on a function called `find_lines_pixel_position()` that takes the warped image as an argument. That function is called by another function (cell 14th of the notebook, called `draw_binary_lines()`, which takes the warped image as an argument), which, with the results of the `find_lines_pixel_position()` function creates a binary image with the lane lines identified highlighted by fitting the lane lines with a 2nd order polynomial.

![alt text][image6]

####Curvature of the lane and vehicle position with respect to center.

The radius of curvature is calculated on the function called `find_curvature()` by converting the pixels of both the x and y coordinates to meters so it can be fit into the world space. This function is located on cell 15 of my notebook.

The position of the car related to the center of the image is calculated on cell 16 of my notebook, on a function called `draw_lanes_on_real_image()`. The postion is found by simply calculating the distance of the right/left lanes from the center of the picture. With that in hand I convert the pixel value into meters and fit it into the the world space.

####Warp the detected lane boundaries back onto the original image and output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.
The final result - lane area plotted back onto the real image with the curvature and position information - is also located on cell 16 of my notebook. The function `draw_lanes_on_real_image()` takes the polynomial and lines arrays (ploty, left_fitx, right_fitx) as well as the matrix and distortion information and project those lines into the original image - aside from also calculating the position of the car related to the center of the image, as described above.

Here is an example of my result on a test image:
![alt text][image7]

Here is an example of my result on a test image with all the steps taken to plot the lanes:

![alt text][image8]

---

###Pipeline (video)

The pipeline for the video output is located on cells 19 and 20 of my notebook. I verified that my pipeline performs reasonably well on the entire project video.

Here's a [link to my video result](./project_results_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Although the pipeline works reasonably well on the project video, it will fail on the challenge videos. This implies that my pipeline will not perform well where light is not sufficient (fog, clarity on camera lens, etc) as well as where the lane lines are not clearly identifiable (snow, dirt on the road, etc). I believe this could be resolved by searching for the lane lines within a margin from the previous point where the lines were found, instead of searching for the lane lines in each frame, as I'm doing on my pipeline. That would allow the pipeline to be more accurate on where the lanes are, even if the lane itself is not actually visible, as it would leverage the information from the previous found point and try to continue on that path, which is a much more reliable method than just guessing where the lines are.    

