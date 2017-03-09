#**Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of:
1. Convert to grayscale
2. Gaussian smoothing (kernelsize: 5)
3. Canny edge detection (low: 60, high: 180)
4. Mask to a region of interest (a trapezoidal shape based on the image size)
5. Hough transform
6. Draw lines

In order to draw a single line on the left and right lanes, I modified the draw_lines() by:
1. Performing a linear regression (using polyfit) on each segment
2. Exclude "shallow slopes", between -0.4 > m > 0.4
3. Split slopes into 2 groups, positive (right side) and negative (left side)
4. Take the weighted average m and b values, weighted on the length of each hough segment.
5. Draw the weighted average m and b line in a thicker color
6. Crop drawn area to the ROI (this makes drawing more flexible given image size)

To explain why I took this approach, at first I wanted to do a spline fit, but looking at the challenge video it was clear that the Hough output was noisy enough that I could not be sure that all points would be valid. I also wanted to take at least some consideration for the possibility that the lanes would not conveniently along the left and right bottom corners.

So I figured as a first attempt I would attempt to just average the m and b values from the segments. This was somewhat noisy, so I added the weighting. 

All this got built into the draw function rather than a separate pipeline step, which is how it should be organized. But what was suggested by the comments.


###2. Identify potential shortcomings with your current pipeline

I fixed some shortcomings I saw in attempting to get the challenge image to work. Those were to allow for different-sized videos/images, to allow for curving lanes, to allow for yellow lines, and to discount short horizontal segments that were not part of lanes.

Remaining issues:

My lanes are always linear fit. This works well on a highway when the car is already in-lane but would not work well on other roads.

I am not sure how my approach would work if the car changed lanes. I believe it would be very confused during the transition.

I did not address the most difficult frames of the "challenge", which had two elements: first there is a change of surface (an overpass) which the pipeline mistakes for a lane line at one point, and secondly during that overpass the pipeline does not detect the yellow line because the concrete surface does not have enough edge. Relaxing the canny thresholds caused a lot of falsing. However, during almost the entire video the lines are drawn, so perhaps this could be handled by projecting the last known lane positions based on the heading.

###3. Suggest possible improvements to your pipeline

The clearest improvement would be to draw the lines as curves, I did not think much about how to do that in general, but it would be fairly easy to do a spline fit for most the images in this exercise.

The other improvement would be to work on the image processing step so that it can better detect edges in low contrasts. Probably by trying to improve contrast specifically of yellow markings on lighter brown/gray surfaces.