## **Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature.

[//]: # (Image References)

[image1]: ./output_images/image_undist_chessboard.jpg "image_undist_chessboard"
[image2]: ./output_images/image_undist.jpg "image_undist"
[image3]: ./output_images/image_binary_edges.jpg "image_binary_edges"
[image4]: ./output_images/image_binary_warped.jpg "image_binary_warped"
[image5]: ./output_images/histogram.jpg "histogram"
[image6]: ./output_images/image_fit_lines.jpg "image_fit_lines"
[image7]: ./output_images/result.jpg "Result"
[video1]: ./project_video_output.mp4 "Video"

### [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./P4.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image with 'cv2.findChessboardCorners'.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. 

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. The coefficients are stored into the pickle file `wide_dist_pickle.p` for later useage. I applied this distortion correction to the chessboard image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used color thresholds to generate a binary image (thresholding steps within `findedges()` function).  Here's my output for this step. 

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which is used to calculate the transform matrix M and Minv with cv2.getPerspectiveTransform function. The `warp()` function takes the image size (img_size) as input, which will be used in the destination points. The source points and destination pointes are as follows:

```python
src = np.float32([[572,  465], 
                  [712,  465], 
                  [1100, 720], 
                  [200,  720]])
dst = np.float32([[offset, 0], 
                  [img_size[0]-offset, 0], 
                  [img_size[0]-offset, img_size[1]], 
                  [offset, img_size[1]]])
```

Here the offset is set to 200. This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 572, 465      | 200, 0        | 
| 712, 465      | 1080, 0       |
| 1100, 720     | 1080, 960     |
| 200, 720      | 200, 960      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto the image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I take a histgram along all the colums of the image. The result looks like this:

![alt text][image5]

With this histogram I am adding up the pixel values along each column in the image. In my thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. I can use that as a starting point for where to search for the lines. From that point, I use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame. The result looks like this:

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `calcurvature` function. Since the fit lines have already calculated on the image. I need to change the fit lines to the real world space with the convertion parameters `ym_per_pix` and `xm_per_pix`, and then calculate the curvature with the fomula regarding the bottom of image, where the `y_eval` is the maximum. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `visualizelines()`.  Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

As I was working on this project, I have met sevaral problems:

* Well, I am not familiar with the perspection transform, the offset provided by Udacity was adopted firstly. But I found that it didn't work well, as there is no need to take offset of 'dst' for the up and down side of image. Offset of the left and right side is enough in case of curvature. 

* The histogram of half the warped binary image may get nothing if the edges were not detected very well. This will not happen in the pipeline of image, but will in the video. So It is a little bit confusion as the pipeline can dispose the image well but failed on the video.  At last, I used the ffmpeg to extract the frame from the video and took a deeper insight of image, and got where the problem is.

* The last one is the image format with cv2's BGR and moviepy's RGB. Also with the help of ffmpeg, I finally found the problem. 

The future work will be the edges finding. Since the color space method perform not well on the white lines in case of the dark and shadowness in the challenge video. Sobel may be used too. Extract the not-well-recognized image from the video and took insight of it is really helpful. Augmentation of the image like contrasting may be helpful as a pre-processor as in the Bahavior Cloning Project.