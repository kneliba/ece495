# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of several steps:

1)  **Colour Masking**
This is the first stage where I create two individual masks for white and yellow lanes. The masks are then combined using `cv2.bitwise_or` and applied to the image. Once applied it can be seen that quite a bit of the background gets picked up, which will have to be removed in ROI editing.
	
2)  **Grey Scale**
Following the suggested pipeline, the colour masked image is fed into the given `grayscale()` function

3) **Gaussian Blur**
The next step takes the grey image and applies a Gaussian blur, to diminish some of the granularity. The lowest kernel_size is 3 and several values were tested before deciding that a kernel_size of 5 would return acceptable results as the colour thresholding does a good job removing most high-frequency details 

4) **Edge Detection**
The following step takes the image into the provided `canny` function to outline most edges, which turns out well

5) **Region of Interest**
Selecting the region of interest is important since as mentioned earlier there is quite a significant amount of background that is picked up. A trapezoid shape is created by picking out how far into the road the lanes should be detected and estimating the pixel values at that point.

These values are edited a few times to ensure that no background features are included in the mask and finalized, and plugged into `region_of_interest`

6) **Hough Lines**
Canny edges trimmed from the region of interest are then fed into the `hough_lines` function with a few other parameters. 

	 - Minimum line length is set to 0 since our colour thresholding removed any extremities, the only edges fed into this algorithm are parts of actual lane lines now.
	 - Max line gap is set to 300, since there can be pretty long gaps between lanes 
	 - Threshold is set to 10, it being lower generates more lines to help with line fitting in the next part
	 - rho and theta are set to 1, and pi/180
	 
7) **Line Fitting**
The first part here is to edit the `hough_lines` function to only return the lines it generates from the edges. These lines then have to be sorted via the left and right side by inspecting each line segments slope, done in `split_L_R`. If the slope was negative, it was a right lane, and it it was positive, it was a left lane. 

These left and right lanes are then returned and put into a `fit_line` function. This function takes in all of the line segments and collects each point corresponding to the x and y axis. These points are then plugged into the RANSACRegressor's fit function, with an error bound of 20. The ransac function then predicts the x values at the edges of our ROI's given a y value. Collecting these values then creates a new line segment that is the best fit for the lane line. 

8) **Draw**
The lanes are then drawn back into the original image. The thickness of the default `draw_lines()` function is increased to 10 to match the example.

### 2. Identify potential shortcomings with your current pipeline
An immediate short coming is curved lane lines. Since the line fitting currently only works with start and end points of a line, any curve will be completely ignored and only a straight line will ever be fit. 

A second short coming would be a change in lighting or scenery. There is a big chance that the colour thresholding is overfit, and might not be able to accept all hues of white and yellow if the car drives in a shady region, i.e. in a forest. 

Lastly, if a car switches lanes, or is not driving in the complete centre of a road, then the region of interest will be wrong, and it will not be able to handle such a maneuver. 

### 3. Suggest possible improvements to your pipeline
A possible improvement would be to take more points into the line fitting function and try to find a polynomial line fit instead of a linear one. This can be done by creating smaller line segments from the hough line function to ensure the lane is not linearized too early. This would allow for more curved lanes later.

Secondly, with very curvy lanes or with sharp turns, the simplicity of sorting the left and right lane by their slope in the image would not work. A new method would be to split the image in half and detect which half of the image the lane precedes the most in.
