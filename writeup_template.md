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

[image1]: ./camera_cal/calibration2.jpg "Original Image with radial distortion"
[image2]: ./output_images/calibration2.jpg "Undistorted Image"
[image3]: ./test_images/straight_lines2.jpg "Straight Lane"
[image4]: ./undistorted/straight_lines2.jpg "Undistorted Image"
[image5]: ./s_gradient/straight_lines2.jpg "Applied Sobel X -> S (HLS) and V (HSV) color space re-map"
[image6]: ./b_warped/straight_lines2.jpg "2D bird-eye view"
[image7]: ./polyfit/straight_lines2.jpg "Fit 2nd order polynomial to interpolate lanes"
[image8]: ./warped/straight_lines2.jpg "Project and overlay lane detection contour, estimated lane curvature and vehicle offset to mid road"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

![alt text][image1]
![alt text][image2]
1. Use the OpenCV functions findChessboardCorners() and drawChessboardCorners() to automatically find and draw corners for chessboard pattern
2. Use chessboard images to obtain image points and object points using 
3. Use the OpenCV functions cv2.calibrateCamera() and cv2.undistort() to compute the calibration and undistortion.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

![alt text][image3]
![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
  
Sobel X matrix is used together with Saturation channel from HLS and Value channel from HSV color spaces

![alt text][image5]

    # Convert to HLS color space and separate the S channel
    hls = cv2.cvtColor(img, cv2.COLOR_RGB2HLS)
    s_channel = hls[:,:,2]
    
    # Convert to HSV color space and separate the V channel
    hsv = cv2.cvtColor(img, cv2.COLOR_RGB2HSV)
    v_channel = hsv[:,:,2]
    
    # Sobel x
    sobelx = cv2.Sobel(gray, cv2.CV_64F, 1, 0) # Take the derivative in x
    abs_sobelx = np.absolute(sobelx) # Absolute x derivative to accentuate lines away from horizontal
    scaled_sobel = np.uint8(255*abs_sobelx/np.max(abs_sobelx))
    
    # Threshold color channel S
    thresh_min = 50
    thresh_max = 255
    sxbinary = np.zeros_like(scaled_sobel)
    sxbinary[(scaled_sobel >= thresh_min) & (scaled_sobel <= thresh_max)] = 1

    # Threshold color channel V
    v_thresh_min = 200
    v_thresh_max = 255
    v_binary = np.zeros_like(v_channel)
    v_binary[(v_channel >= v_thresh_min) & (v_channel <= v_thresh_max)] = 1

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

i) Define 4 corners of region of interest at original image as sROI
ii) Define target 4 corners assuming vertical since the test image is of a straight road
iii) Transform to 2D bird-eye using getPerspectiveTransform and warpPerspective

![alt text][image6]

    sROI = [[210,720],[1100,720],[720,470],[565,470]]
    dROI = [[320,720],[960,720],[960,0],[320,0]]
 
    src = np.float32([sROI[0], sROI[1], sROI[2], sROI[3]])
    dst = np.float32([dROI[0], dROI[1], dROI[2], dROI[3]])
    M = cv2.getPerspectiveTransform(src, dst)
    warped = cv2.warpPerspective(img, M, img_size,flags=cv2.INTER_LINEAR)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?


![alt text][image7]
Interpolation of lane by Sliding Windows

i) find_lane_pixels() - Plotting a histogram of where the binary activations occur across the image
ii) Define a window that vertically span the vertical dimension and horizontally span and envelope the peak with histogram plot on both left and right sides
iii) All binary image points inside the window are used to interpolate a curve fitting by regression
iv) search_around_poly() - take the polynomial functions to determine which activated pixels fall into the green shaded areas


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

    left_curverad = (0.5 / np.abs(left_fit[0]))*(1+(2*left_fit[0]*y_eval*ym_per_pix+left_fit[1])**2)**1.5
    right_curverad = (0.5 / np.abs(right_fit[0]))*(1+(2*right_fit[0]*y_eval*ym_per_pix+right_fit[1])**2)**1.5
    
    the curvature with smaller value is chosen
    
    offset = ((left_fit[-1]+right_fit[-1])/2 - img.shape[1]/2) * xm_per_pix

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.



![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_videos_output/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
