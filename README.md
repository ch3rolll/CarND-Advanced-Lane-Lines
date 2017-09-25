# CarND-Advanced-Lane-Lines
---
[//]: # (Image References)

[image1]: ./screenshot_cameracali.png "Camera Calibration"
[image2]: ./ss_undist.png "Original vs Undistorted"
[image3]: ./pipeline.png "Combine approach lane marking detection Top Down view"
[image4]: ./drawback.png "Drivable ego lane"
[image5]: ./perspective.png "Source points for perpective transformation"
[video1]: ./project_output.mp4 "Video1"
[video2]: ./project_output_masked.mp4 "Video with mask"


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

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  
---
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

As leaned from the course, I started using cv2.calibrateCamera(objpoints, imgpoints, gray.shape[::-1], None, None). This function requires objpoints and imgpoints. So in this sense, using cv2.findChessboardCorners() to find imgpoints is a good way to go. By assigning the corners numbers as (9,6), the cal_undistort() works well.

To get a better visualization, I used cv2.drawChessboardCorners() to draw the corners on the images. Here is a screenshot from the ipython notebook.

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

A combine approach with color and gradient thresholds to generate a binary image and followed by a masking layer, which is pretty much like project one.  Here's an example of my output for this step on the left side.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. For having a better resolution, I picked one straight line scenario and visualized the anchor point in the image, as shown below:

![alt text][image5]

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 600,446]      | 260,100     | 
| 680, 446      | 1045,100      |
| 260, 680    | 260,700    |
| 1045, 680     | 1045,700       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image, as shown below on the right side:

![alt text][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this, still on the right side:

![alt text][image3]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I went throught the following steps:
* Step 1: Fit new polynomials to x,y in world space
    fit_cr = np.polyfit(ploty*ym_per_pix, (left_fitx+right_fitx)/2*xm_per_pix, 2)
    
* Step 2: Find the center pixel for ego lane center
    center = (left_fitx[0] + right_fitx[0])/2
* Step 3: Calculate the new radii of curvature
      center_curverad = ((1 + (2*fit_cr[0]*y_eval*ym_per_pix + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])
      
* Step 4: Draw the lane onto the warped blank image
    cv2.fillPoly(color_warp, np.int_([pts]), (0,255, 0))


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image4]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_output.mp4)

Afterwards, since the result is not that good when it drives in shadow area, then I created a mask and regenerated a video.

Here is a [link to my  masked video result](./project_output_masked.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I spent a lot of time on tuning the vertices for the perspective transformation and the mask area to have a better result on shadow area.

Perhaps I need to use the more image processing approaches to get a better result, plus having a sanity check will help a lot as well.

If it is driving through a broken line or botts dot, it will have a chance to fail. Or if there are some potholes on the road, it will give this pipeline a hard time.
