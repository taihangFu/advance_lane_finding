
# Advanced-Lane-Finding

[//]: # (Image References)

[corners]: ./corners.PNG
[distortion_correction]: ./distortion_correction.PNG
[straight_lines1]:straight_lines1.jpg
[undistorted_lane]: ./undistorted_lane.PNG
[binary]: ./binary.PNG
[warped]: ./warped.PNG
[binary_warped]: ./binary_warped.PNG
[polyfit]: ./polyfit.PNG
[lane_boundary_with_data]: ./lane_boundary_with_data.PNG

## Goals

The goals/steps of this project are the following:

- Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
- Apply a distortion correction to raw images.
- Use color transforms, gradients, etc., to create a thresholded binary image.
- Apply a perspective transform to rectify binary image ("birds-eye view").
- Detect lane pixels and fit to find the lane boundary.
- Determine the curvature of the lane and vehicle position on center.
- Warp the detected lane boundaries back onto the original image.
- Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


### Camera calibration

#### Computed the camera matrix and distortion coefficients

I first located the corner of the chessboard using `cv2.findChessboardCorners` and the corner is stored in an array ` imgpoints'.

![corners]

Where the `objpoints` stored the object points and the object points will always be the same as the 'z' coordinate since the chessboard is flat.

I then calculate the **camera calibration** and the **distortion coefficient** using the output objpoints and imgpoints by the 'cv2.calibrateCamera'. 

After I obtained the required parameters, the **camera calibration** and the **distortion coefficient**, I used 'cv2.undistort' the following test image to apply distortion correction.


![distortion_correction]


### Pipeline (single images)

#### An example of a distortion-corrected image
The following figure shows the result of the application of camera calibration to the test image:

![undistorted_lane]

#### Create a thresholded binary image

I have explored several combination of color threshold and gradient with differnt range of threshold values such as Lightness and Saturation from HLS color model and Sobel Operator.

From my experiments, I found Saturation is best suit for lane since it has more contracts on the lane line, I think it is due to the high color intensity of the lane line as this is best for human to quickly recognise the lane line.

After the color transformation had been done, I implemented and added diffent combinations such as Sobel X and Sobel Y with both magnitude and gradients.

However, I only retained the combination of Sobel X and Sobel Y, and Saturation since including others such as magnitude and Lightness will just making the image noiser. The following is an example of binary image obtained with that combination:

![binary]


#### Process of perspective transform and  an example of a transformed image

The following image is an example of straight line lane:

![straight_lines1]


The follwing image is the destination points for the transformation where to get a clear picture of the street and the follwing parameter are the points and offset to map the bird view lane:
**offset = 200**


|Source|Destination|
|-----:|----------:|
|(585, 455)|(offset,0)|
|(705, 455)|(image_size_X - offset, 0)|
|(1130, 720)|(image_size_X - offset, image_size_y)|
|(190, 720)|(offset, image_size_Y)|

Transformation matrix was calculated by `cv2.getPerspectiveTransform`, and inverse transformation matrix was used to map the points back to the original image, and the following is the result of a bird-eye view perspective transformation:

![warped]


The following set of images are the bird-eye view perspective transformation of binary warped lanes

![binary_warped]

#### Identified lane-line pixels and fit their positions with a polynomial

I first calculates the histogram on the X axis so as to the peaks on the image, and collect the points that are > 0 contained on those windows. 

After that, `np.polyfit`is implemented to find the lines.

The following picture shows the points found on each window, and a yellow line polyfit  

![polyfit]

#### Calculated the radius of curvature of the lane and the position of the vehicle on the center

Follolwing the tutorial:
http://www.intmath.com/applications-differentiation/8-radius-curvature.php


The polyfit is calculated by the following

```
((1 + (2*fit[0]*yRange*ym_per_pix + fit[1])**2)**1.5) / np.absolute(2*fit[0])
```

where `fit` is the the array containing the polynomial, `yRange` is the max Y value and `ym_per_pix` is the meter per pixel value.

Assuming the camera is installed on the center of the front of the vehicle:

1. The center of the lane is calculated and the middle point is found by evaluating the left and right polynomials of the largest polynomial of Y.

2. Calculate the center of the vehicle, and change the center of the image from pixel to meter.

3. The distance between the lane center and the vehicle center indicates whether the vehicle is on the left or the right.


#### An example image plotted back down onto the road such that the lane area is identified clearly
The generated point is mapped to the image space by using the inverse transformation matrix generated from the perspective transformation as mentioned above, in order to display the image on the line in the linear polynomial calculation on Y coordinate space.

![lane_boundary_with_data]

### Pipeline ( Final video output)
**project_video_output.mp4**

### Some Reflection

The main difficulty I found is to combine different color transform and gradient to locate the lane line while at the same time to reduce **noises** as much at possible. 

Although the current combination above works fine on the project video, it is very difficult to keep the pipeline being robust in the shadow and at the same time detect the Yellow Lane on the white ground.

An example would be my pipeline will fail on the challenge video, especially in the frame that appears some more lines(or something looks like a line) on the lane the vehicles driving on.

In the future I will do more research and explore more on color and gradient trasnform to make lane deterction more robust under some conditions such as bad weather or car accident in front or broken lane...

