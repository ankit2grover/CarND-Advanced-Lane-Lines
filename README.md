# Project 04 - Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

## README
The code I used for doing this project can be found in 'Main.ipynb' file.
Also, helpers folder contains classes that are performing various functionalities.
All the cell numbers I refer to in this document is for `'Main.ipynb'. The following sections go into further detail about the specific points described in the [Rubric](https://review.udacity.com/#!/rubrics/571/view).

```
Project 04 - Advanced Lane Detection

To run the program just run Main.ipynb on the Jupyter notebook and you can also see my results on the notebook.

All the output videos are in the output_video folder.

All the output images are in the output_images folder. 
This folder contains below mentioned images.

- Chessboard images of before and after camera calibration
- Threshold image (Color, Sobelx, Sobely, Magnitude and Gradient Directional).
- Warp Image (Showing Bird view of one of image frame).
- Unwarp image.

---
## Camera Calibration and Distortion Correction
The camera calibration and image undistortion code is contained in lines 18-39 in the function `calibrate` and `undistort` of the class Camera in the file helpers/calibrate.py. 
The calibration is performed using chessboard pattern images taken using the same camera as the project videos, such as the one shown below:

![alt text](https://github.com/ankit2grover/CarND-Advanced-Lane-Lines/blob/master/output_images/CameraOriginal.jpg)

The chessboard pattern calibration images (in the `cal_images` folder) contain 9 and 6 corners in the horizontal and vertical directions, respectively (as shown above). First, a list of "object points", which are the (x, y, z) coordinates of these  chessboard corners in the real-world in 3D space, is compiled. The chessboard is assumed to be in the plane z=0, with the top-left corner at the origin. There is assumed to be unit spacing between the corners. These coordinates are stored in the array `objp`.


```python
  # cal_images contains names of calibration image files
  for fname in cal_images:
      img = cv2.imread(fname)
      gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

      ret, corners = cv2.findChessboardCorners(gray, (nx, ny), None)
      if ret == True:
          objpoints.append(objp)
          imgpoints.append(corners)

  ret, cam_mtx, cam_dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, gray.shape[::-1],None,None)
```

For each calibration image, the image coordinates (x,y), of the chessboard corners are computed using the `cv2.findChessboardCorners` function. These are appended to the `imgpoints` array and the `objp` array is appended to the `objpoints` array.

The two accumulated lists, `imgpoints` and `objpoints` are then passed into `cv2.calibrateCamera` to obtain the camera calibration and distortion coefficients as shown in the above code block. The input image is then undistorted (later in the image processing pipeline on cell 13 in Main.ipynb) using camera class.

## Pipeline (single images)

### 1. Example of Distortion corrected image

The image from the camera is undistorted using the camera calibration matrix and distortion coefficients computed in the previous step. This is done using the `cv2.undistort` function as shown below:

`undist = cv2.undistort(image, cam_mtx, cam_dist, None, cam_mtx)`

An example of an image before and after the distortion correction procedure is shown below.

![alt text](https://github.com/ankit2grover/CarND-Advanced-Lane-Lines/blob/master/output_images/CameraUndistort.jpg)

### 2. Thresholding

Tried with multiple combinations of thresholding techniques and later on I found that best results were given if I use combination of sobelx, sobely, magnitude and directional gradient. It allows me to detect the lane lines correctly and filter out other objects.

All the code of thresholding is in helpers/threshold.py file and you can see combined threshold code in `create_combined_thresh` function at line 61.

Later on I tried with number of color channels like HSV and HLS and apply thresholds with previous combined results. After trying various options, I found out that better results are with S channel combination with Combined thresholds. Its code is in `create_combined_color_thresh` function at line 75.

Below is the image that shows the final result.

![alt text](https://github.com/ankit2grover/CarND-Advanced-Lane-Lines/blob/master/output_images/Threshold.png)

### 3. Perspective transform

Perspective transform code is in class PerspectiveTransform in the file helpers/helpers/warp.py file

The perspective transformation is computed using the function `create_combined_thresh` .

It converts our threshold image into bird eye view that help us to detect lane lines later on with sliding window algortihm.

A perspective transform maps the points in a threshold image to different, desired, image points with a new perspective. The perspective transform we are  most interested in is a bird’s-eye view transform that let’s us view a lane from above; this will be useful for calculating the lane curvature later on.

See below the warp as a sample after applying Perspective Transformation

Warp Image

![alt text](https://github.com/ankit2grover/CarND-Advanced-Lane-Lines/blob/master/output_images/Warp.png)

Unwarp the image.

![alt text](https://github.com/ankit2grover/CarND-Advanced-Lane-Lines/blob/master/output_images/Unwarp.png)


### 4. Lane Detection

The lane detection was primarily performed using two methods -- histogram method and sliding window (or Sliding Window Look Ahead Filter after confident results). The latter only works when we have some prior knowledge about the location of the lane lines.

After calculating lane fits, I use a sanity check which checks if the radius of curvature of the lanes have changed too much from the previous frame. If the sanity check fails, the frame is considered to be a "dropped frame" and the previously calculated lane curve is used. 

If sanity check is passed I take out the weighted average of coefficients and calculate new lane fits as per average coefficients.

See below all the methods in details.

#### (a) Histogram Method

This method code is in LaneSearch class in helpers/lane.py file.

The first step in this method is to compute the base points of the lanes. This is done in the `sliding_window` function in lines 41-44. 

The first step is to compute a histogram of the lower half of the thresholded image. The histogram corresponding to the thresholded, warped image in the previous section is shown below:

![alt text](https://github.com/ankit2grover/CarND-Advanced-Lane-Lines/blob/master/output_images/hist.png)


### (b). Sliding Window
The `sliding_window` function in lines 42-44 in the helpers/lane.py file is used to identify peaks. The indices thus obtained are further filtered to reject any below a certain minimum value as well as any peaks very close to the edges of the image.

After that Sliding Window algorithm is used to identify poly coefficients and this algorithm is very well explained in Lesson 33 in Project Advanced Lane Finding of Udacity.

You can see the complete code in `sliding_window` from lines 46-102. 

### (c). Sliding Window Look Ahead Filter.
After getting fist confident result of the first image for the subsequent image frames Sliding Window Look Ahead Filter algorithm is used. In it we use first image frame confident results and add margin on it and then look for lane fit lines with in that margin. It is very well explained in Lesson 33 in Project Advanced Lane Finding of Udacity.

You can see the complete code in function `sliding_window_look_ahead_filter` from lines 105-123. 

Then, left and right fits are computed and radius of curvature is computed in function `draw_lane_results` from lines 129-137.

### (d). Radius of Curvature

Radius of Curvature is used to perform to do sanity check and the image frames that are more than 50% of change in radius are dropped and previous best average results are used.Sanity check code is in function `_sanity_check` from lines 200-208

After sanity check lane coefficiennts are average weighted and final lane fits are drawn on the image using cv2.fillPoly with radius of curvature and offset of the lane fits from the center camera. (offset is computed in  function `calc_offset` from lines 191-198).

### (e).Average weighted of Line Coefficients and Fits

Line is used in helpers/line.py file. This class process lane fit results and sanity checked passed coefficients are averaged weighted here in the function `process` from lines 34-55).


## Summary and Result

The complete pipeline is defined in the `process_image` function in cell 13 of Main.ipynb file that performs all these steps and then draws the lanes as well as the radius and offset information on to the frame. The steps in the algorithm are:

**Distortion correction → Edge Detection → Perspective Transform → Lane Detection (using Histogram and Sliding Window) → Sanity Check (using Radius of Curvature) -> Average weight of lane fits**

See below the one of image frame after applying all these techniques.

![alt text](https://github.com/ankit2grover/CarND-Advanced-Lane-Lines/blob/master/output_images/video_output_result_images/Result2.jpg)

### Videos

**Project video output**

This video can be found at:  [project_video_output.mp4](./output/project_video_output.mp4)

**Challenge video output**

This video can be found at:  [challenge_video_output.mp4](./output/challenge_video_output.mp4)

**Harder Challenge video output**

This video can be found at:  [harder_challenge_video_output.mp4](./output/harder_challenge_video_output.mp4)


The pipeline was able to detect and track the lanes reliably in  the project video and it did fine for the challenge video. The main issue with the challenge video due to sharp curves and lack of contrast. The algorithm has to be improved and made more robust for that video.


### Improvements

1) Should improve on Threshold conversion and look to try more combinations to see if any more combination can give better results.

2) Should remove unnecessary objects from the image and may be use Hough Lines detection algorithm before Perspective Transformation, that might help in focussing only on lane lines.

3) Average weighted technique is not looking good in challenger videos, I should have checked if around 4 image frames have been dropped then restart again and look for new confident results.

4) Insted of Sliding window, may be Smart Sliding window technique should have been used that is mentioned in lesson 34.


