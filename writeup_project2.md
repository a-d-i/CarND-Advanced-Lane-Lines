## Writeup Project 2 - Advanced Lane Finding Project

### Goals of this project are as follows:

---
* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image0]: ./output_images/test4.jpg                  "Provided Image"
[image1]: ./output_images/undistorted_test4.jpg      "Undistorted"
[image2]: ./output_images/camera_calibration17.jpg   "Camera calibration"
[image3]: ./output_images/perspec_warp_test4.jpg     "Perspective Transform"
[image4]: ./output_images/lanes_test4.jpg            "Lanes"
[image5]: ./output_images/warp_back_test4.jpg        "Final"
[image6]: ./output_images/thresh_with_mask_test4.png "Threshold_with_mask"

[image7]: ./output_images/P2_imageout_4.png "P2.ipynb images 1-4"
[image8]: ./output_images/P2_imageout_8.png "P2.ipynb images 5-8"

[imageA]: ./output_images/straight_lines1.jpg              "straight line image"
[imageB]: ./output_images/perspec_warp_straight_lines1.jpg "Prespective straight line image"

[imageCameraB]: ./output_images/camera_calibration1.jpg          "Distorted chessboard"
[imageCameraC]: ./output_images/undist_calibration1.jpg   "Undistorted chessboard"

[video1]: ./output_images/project_video.mp4 "Video"
[videoimg1]: ./output_images/snapshot_video1.png "Video Snapshot1"
[videoimg2]: ./output_images/snapshot_video2.png "Video Snapshot2"
[videoimg3]: ./output_images/snapshot_video3.png "Video Snapshot3"
[videoimg4]: ./output_images/snapshot_video4.png "Video Snapshot4"
[videoimg5]: ./output_images/snapshot_video5.png "Video Snapshot5"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### rubric points covered below in the writeup.  

---

### Writeup / README

This document!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./P2.ipynb" cell 1 and 2. Cell 1 is the boilerplate code using provided chessboard images and comparing to a known chessboard image of 9 by 6 grid. Once the object and image points were determined, I checked all the provided images for dimension and found that image camera_cal/calibration7.jpg which is (721, 1281, 3) and all other images are (720, 1280, 3). As the project images and video are all 1280 by 720, I used a 1280 by 720 image size to determine the camera matrix and distortion parameter using `cv2.calibrateCamera()`.    

Image points are the identified x and y pixel position of the inner corners of the 9 by 6 chessboard. object points are known with z=0 (assuming the chessboard is in a flat surface in the x-y place at z=0). image points from all the given images are used to calibrate against the known object points.
    
Camera calibration, example image shown with identified imagepoints of a chessboard.
![alt text][image2]

After calibration using all images, example of undistorting a chessboard image.

Distorted image:
![alt text][imageCameraB]

Image after distortion removal (undistortion):
![alt text][imageCameraC]

### Pipeline (single images)

![alt text][image0]

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images shown above. The result of undistored image is shown below:

Given the distortion parameters and camera matrix from the calibration step (for a given size of image, in this case 1280 by 720), `cv2.undistort()` is used to generate the undistorted version of the image.

![alt text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (In file `P2.py`, function binary_pipeline, cell 6). I have used the following thresholding:

Gradient based thresholding (using sobel kernel size 9)
- x gradient magnitude
- Y gradient magnitude
- X-Y resultant gradient magnitude
- direction of gradient 

For color thresholding
First I convered the image from BGR to HLS color space. 
- s channel based threshold
- l channel based threshold
- h channel based threshold was taken out, needs more tweaking.

Logic combining these thresholds is this:
`((s_binary == 1) | ((gradx == 1) & (grady == 1) & (lightbinary == 1))) | ((mag_binary == 1) & (dir_binary == 1))`

image shown with the binary thresholding applied and the polygonal mask overlaid to extract only lane lines as much as possible. I have used a slight variation on the trapezoid, with a notch at the bottom to take out any unwanted pixes between
lanes. This also allows to get the histogram clean and towards truth.

![alt text][image6]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in `P2.ipynb` file, cell 7, function: `bird_eye_view_transform()`

The `bird_eye_view_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python

src = np.float32(
    [ [image_size[1]//2.3, image_size[0]//2 + 100],
      [image_size[1]//1.6, image_size[0]//2 + 100],
      [image_size[1], image_size[0]],
      [0, image_size[0]]
    ])  

dst = np.float32(
    [ [image_size[1]//4, 0],
      [image_size[1]//1.108,0],
      [image_size[1]//1.18, image_size[0]],
      [image_size[1]//4, image_size[0]]
    ])

```
                  
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 550, 460      | 320, 0        | 
| 800, 460      | 1155, 0       |
| 1280,720      | 1080, 720     |
| 195, 720      | 320, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][imageA]
![alt text][imageB]


Continuing with the example from the start of this writeup, the perspective view on the binary masked image is as:
![alt text][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used a global state to tack the left and right lane polynomials. Initially the polynomials are empty and hence
a search from scratch is done using the histogram based approach to indetify lane starting points and then a 
sliding window approach to track high pixel density going from bottom of the image to the top. 

If the pipeline misses to find a polynomial or in case of other errors, the pipelines sets the global polynomials
to empty and triggers a search for lanes from the scratch.

The code for the functionality is in cell 8 in the following functions:
`fit_polynomial()` is the entry point function which decide (based on global polynomials being set or not) to trigger
a lane search using sliding windows and historgram or to propopagate around the already known polynomial from the last
frame. In both cases the gloabl lane are updated with the new polynomials.

Example lanes from the test image.

![alt text][image4]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Mean radius of curvature for the two lanes (at the bottom point in the image) is in `P2.ipynb` function: `measure_curvature_real()`. Meters per pixel in x and y direction are used as in the lesson videos:

`ym_per_pix = 30/720 # meters per pixel in y dimension`
`xm_per_pix = 3.7/700 # meters per pixel in x dimension`

Vehicle position is determined in the function: `measure_vehicle_pos()` assuming the camera is in the center and using the known x points for the bottom points on the lane (maximum y value of 720 in this example)

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
Final output is generated as specified in the tips for the project. This is done in function : `unwarp()`:

Output of the test image is shown below.

![alt text][image5]

#### 7. Output pipeline from P2.ipynb for the eight image smaples looks like below

First 4 images
![alt text][image7]

Next 4
![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

Samples of video output snapshot are below:

![alt text][videoimg1]
![alt text][videoimg2]
![alt text][videoimg3]
![alt text][videoimg4]
![alt text][videoimg5]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Project video is working fine, but to be more robust I need to experiment more with the color spaces and increase the robustness of the binary image to shadow and dark grooves in the asphalt.

Frame to frame filtering for curvature and vehicle position change can be used to limit sudden jumps in lane position.

In the hard challenge video lanes are lost when vehicle makes sharp turns. `vertices` variable used to mask the image for lane pixels can be oriented towards the lane curvature to look for lanes in the correct position in the image when vehicle is making sharp turns






