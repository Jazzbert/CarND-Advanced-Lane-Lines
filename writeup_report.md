
# **Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image/Link References)

[image1]: ./writeup_images/undistorted_board.png "Undistorted"
[image2]: ./writeup_images/undistorted_road.png "Road Transformed"
[image3]: ./writeup_images/binary.png "Binary Example"
[image4]: ./writeup_images/warped.png "Warp Example"
[image5]: ./writeup_images/sliding_boxes.png "Sliding Boxes"
[image6]: ./writeup_images/found_lines.png "Found Lines Method"
[image7]: ./writeup_images/painted_lane.png "Painted Lane"
[radius_eq]: ./writeup_images/radiuseq.png "Radius Equation"
[eq3]: ./writeup_images/eq3.png "Final conversion"
[video1]: ./project_video.mp4 "Video"

[notebook]: (https://github.com/Jazzbert/CarND-Advanced-Lane-Lines/blob/master/CarND-AdvancedLaneFinding.ipynb)

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!  All code referenced in this write-up is located in the IPython notebook located in ["./CarND-AdvancedLaneFinding.ipynb"][notebook].

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the "Step 1" Section code cells of the [IPython notebook][notebook].  The function `cal_undistort()` implements the undistortion on the image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

There were some images where corners could not all be found because one of the cross points was just off image.  This was handled by dropping those particular images.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Here's the result of undistorting one of the test images:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Highlighting the lines is in "Step 3" of the [notebook][notebook].  The function `highlight()` ultimately creates the highlighted binary image.  This step is independent of Step 2 (Image warping).

To start, I defined a number of threshold functions to allow tweaking of different features of the image.  Not all of the functions were used in the submitted version; however, additional tweaking could be done for using these for the challenge videos and other conditions.

In the end, I used a combination of color thresholds on Hue and Saturation to bring out the yellow and a combination of Sobel-lines x, y, magnitude, and direction which seemed to highlight the white well.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in Step 2 of the [notebook][notebook].  It includes two functions called `warp()` and `unwarp()`, which which create warped/unwarped output for binary or color images.

I used one of the straight-line images to find four points that could represent a rectangle in top-down view manually using photo editing software (Pinta).  I drew two horizontal lines near and far and the road and identified the related points:

This resulted in the following source and destination points:

| Location      | Point         |
|:-------------:|:-------------:|
| Top-left      | 609, 442      |
| Top-right     | 670, 442      |
| Bottom-left   | 281, 668      |
| Bottom-right  | 1022, 668     |

I mapped this to four points offset from an *inverted* image size (e.g. portrait view) so that it is easier to see the shape.

The result is here:

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To identify the lane-line positions, Step 4 and Step 5 in the [notebook][notebook] create a second order polynomial fit line using sliding boxes and new fits of previously found lines.

##### Sliding boxes
Splitting the image vertically in half, I used the histogram function to show where there the most y-values with respect to 'x'.  Using that as a starting center, and starting at the bottom, the code looks for all points just in the bottom portion of the image, sets a center, then keeps stepping up through the image.  I used nine steps just as the instruction/quizzes used.

Here's a sample output of the boxes, along with colorization for the points used to create the next center, and the ultimate line developed:

![alt text][image5]

##### Starting the Line() Class
Now that I have the basic coefficients for a simple curve that can be found on the first image of a video, these can used to create a `Line()` class and establish an object for each line in the image/video pipeline.

This is much like sliding boxes, except the window is only 1 pixel tall and the starting center is from the equation of the line.  As noted in the last section below, this has some downsides.  

Step 5 in the [notebook][notebook] implements the new class and within that class, the `set_new_line()` method recalculates the new line based on the updated warped-binary image.  

While the code to find the line is in Step 5, the code to demonstrate the result is in Step 6 of the [notebook][notebook], where I started to generate code for the video pipeline as well.  Here's example of lines found from the video using previous fit:

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The curvature is calculated in Step 5 of the [notebook][notebook], within the `Line()` class in a method named `set_curve_radius()`.  

The curve was calculated using curvature formula for 2-order polynomial line:
![alt text][radius_eq]

Instead of measuring the curve from the very bottom of the image, I did bring it up to 10% from the bottom just to smooth out any fitting errors at the edge.

The trickier part was converting to feet.  Since the values are all in pixel space, I determined a foot:pixel conversion using one of the straight-line images, measuring 12 feet across for x and 3 feet in the length of a white line for y.

Using the following conversion to account for 2nd-order calculation, here's the conversion formulas used (although C value not really needed):
![alt text][eq3] where subscript 'p' designates values in the "pixel" space and subscript 'f' designates values in the "feet" space.  k<sub>x</sub> and k<sub>y</sub> are the conversion factors for pixels to feet.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for "painting" the lane onto the road is in Step 7 of the [notebook][notebook].  Here's a sample output:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The moment we've all been waiting for...TADA!!!  Here's a [link to my video result](./output_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Really the biggest challenge for me was finding a good way to highlight the lanes.  This is likely also that could cause failures easily.  For example when I run the same pipeline on the challenge images it clearly has a problems with the black sealant line, sharp turns, unlevel terrain, and periods where the line disappears.

One other key problem I noticed with this is that it takes a lot of time to process and neither on my 1GPU AWS instance nor my fairly beefy desktop was it real-time.

Some general thoughts on next steps:
* Add smoothing to make the line less jittery
* Not try to look so far ahead, getting more accurate information closer to the car.
* Periodically use windows to make sure the line didn't get distracted say by some road repair mark.
* I think we're getting to this but use multiple techniques with some confidence level to get the best possible highlighted picture given road color/conditions.
* Consider 3rd order polynomial for really curvy spaces
* The ideas continue...

This was a fun exercise.  Thanks for reviewing!
