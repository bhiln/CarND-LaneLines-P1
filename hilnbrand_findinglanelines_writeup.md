# **Finding Lane Lines on the Road** 

## Brian Hilnbrand, 4/17/2018

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:

* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/edges_solidWhiteRight.jpg "White Line Matching"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of seven steps, as follows. First, I converted the images to grayscale using grayscale(). Then I ran a Gaussian blur algorithm on the gray image with a kernel size of 3 with the helper function gaussian_blur(). After that, I detected edges using the Canny edge detection helper function with a low threshold of 50 and a high threshold of 150. Next, I determined where the vertices of the trapezoidal mask should be with get_vertices(). The vertices were determined as follows:

* (0, height of image in pixels)
* (19/40ths of the width of image in pixels, 320)
* (21/40ths of the width of the image in pixels, 320)
* (width of image in pixels, height of image in pixels)

Using these vertices, I masked out the rest of the image using region_of_interest(), setting all pixels outside of this trapezoid to black. This trapezoid encompassed just the area of the image where the drivable lane is generally going to reside.

Then, I generated the individual Hough lines using the hough_lines() helper function. In order to draw a single line on the left and right lanes, I modified the draw_lines() function by first filtering all of the lines into left and right lane line groups by the lines' slope. If its negative and below -0.15, it'll be on the right lane line. Oppositely, if it's positive and above 0.15, it'll belong to the left lane line. This is because the lines tend to converge to a center point in the image as the road goes to the horizon. Therefore a line starting on the bottom left side of the screen will converge towards the center, having a positive slope. Vice versa, a line starting at the botton right side of the screen will converge towards the center, having a negative slope. I've decided to filter out lines with slope of -0.15 to 0.15 because these are lines that are near 0 slope, and obviously not a member of the lane line. These might me horiontal road paint. After filtering the lines, I run a line best fit algorithm on all of the start/end line points for one lane line, generating one line of best fit for that lane line. Once I have the polynomial, I can deduce the X coordinate for any Y coordinate. Using this method, I calculate the (X,Y) coordinate for the bottom of the screen (Y = imshape[0]) and the (X,Y) coordinate along the top of the region of interest (Y = 320). Once the start and end coordinates are calculated, lines are drawn on a black image, as defined by the start and end points.

Finally, I combine the original image and the lane lines image using weighted_img().

This pipeline results in the following image: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


* One potential shortcoming would be what would happen when the road is curved. Using a 1 degree polynomial (a straight line) will not work to create a best fit for a curved lane line. However, using a 2nd or 3rd degree polynomial best fit is not ideal for a straight line. 
* Slightly horizontal lines that are in the region of interest (outside slope of -0.15 -> 0.15) will be classified as part of the left of right lane line. These lines will need to be properly filtered.


### 3. Suggest possible improvements to your pipeline

* Filter lines based on slope standard deviation or outliers to keep just the actual lane lines.
* De-noise the original image to remove Gaussian noise
* Use 1st, 2nd, or 3rd order polynomial best fit for lane lines, determined by shape of lane lines
* Automatic low and high thresholds for canny algorithm, possibly using OTSU thresholding.
