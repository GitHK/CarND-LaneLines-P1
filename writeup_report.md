# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[polygon_car_near_line]: ./img_references/polygon_car_near_line.jpg  "The car's influence is cout out becouse it is outside the green polygon"
[line_bent]: ./img_references/line_bent.jpg  "Note how the line is bent slightly at the bottom"

[solid_white_ok_1]: ./img_references/solid_white_ok_1.jpg ""
[solid_white_ok_2]: ./img_references/solid_white_ok_2.jpg ""
[solid_yellow_ok_1]: ./img_references/solid_yellow_ok_1.jpg ""
[solid_yellow_ok_2]: ./img_references/solid_yellow_ok_2.jpg ""
[solid_yellow_ok_3]: ./img_references/solid_yellow_ok_3.jpg ""

[solid_yellow_not_ok]: ./img_references/solid_yellow_not_ok.jpg "Lines go straight and do not follow the curvature of the lane lines"
[solid_yellow_not_that_good]: ./img_references/solid_yellow_not_that_good.jpg ""
[la_solid_yellow_ok_1]: ./img_references/la_solid_yellow_ok_1.jpg ""
[la_solid_yellow_ok_2]: ./img_references/la_solid_yellow_ok_2.jpg ""


[la_solid_white_ok]: ./img_references/la_solid_white_ok.jpg ""
[la_solid_white_almost_ok]: ./img_references/la_solid_white_almost_ok.jpg ""

[la_solid_yellow_ok]: ./img_references/la_solid_yellow_ok.jpg ""
[la_solid_yellow_almost_ok]: ./img_references/la_solid_yellow_almost_ok.jpg ""



---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

---

The **pipeline** will be described step by step illustrating all of its components and 
the ideas behind my choices:
 
1. **Color selection** the first operation is a color selection using OpenCV with an HSV
 color interval. In two separate operations a range of yellow and white colors are selected.
 Their masks are merged together to form a unique overlapping `color selection mask`.

1. **Grayscale** the image is converted to gray. To reduce noise in the edge detection 
 phase the brightness is reduced by half, this is an experimental value (the matrix was
 multiplied by 0.5). This technique proved useful especially in the challenge video,
 where due to the black to gray color transitions, in the asphalt's surface, lines
 were detected.

1. **Image composition** and **Gaussian blur** the `color selection mask` is added to
 the `grayscale`, the result is filtered with gaussian blur for further noise reduction.
 
1. **Line detection** is done inside certain areas of the image delimited by `polygons`,
 where the provided `hough_lines` unction is used. For each polygon an angle range
 is provided, which filters lines not parallel or almost parallel to the lane lines.
 Multiple polygons are supported at once. You need to use  an angle range in order to
 correctly detect curves in lane lines, otherwise these will not be ignored.

1. **Line drawing** there are 3 ways to draw lines on top of the image called:
 `CANNY`, `LINEAR_INTERPOLATION` and `LINE_AVERAGE`:
 
    - `CANNY` is present for debugging reasons
    - `LINEAR_INTERPOLATION` is best with straight lines (first two videos)  
    - `LINE_AVERAGE` does good in all situations, and is the only one which provides
    consistent results with the challenge video

1. **Output** an image with *continuous lane lines* drawn on top is returned.

---

The `draw_lines` drawing modes work as described below. All lines are filtered
 by the angle interval provided with the polygons.

1. `CANNY` draws lines as they are detected by the canny edge detection 
 function.
 
1. `LINEAR_INTERPOLATION` will interpolate and find the average 
 line between all lines and draw it from the `upper` to the `lower` part of 
 the polygon.
 
1. `LINE_AVERAGE` will weight each line by its importance (line length in pixels),
 will extract an average line between all points at a given `y` coordinate.
 Missing data points are replaced by drawing straight lines (in the middle of the 
 interval) and interpolating (at both ends of the interval). The result is a polyline 
 from the `upper` to the `lower` part of the polygon (which is drawn point by point).

---

Some images to show off how it works:

`LINEAR_INTERPOLATION` **works best** with straight lines as seen in the images blow.

![alt text][solid_white_ok_1]

![alt text][solid_white_ok_2]

![alt text][solid_yellow_ok_1]

![alt text][solid_yellow_ok_2]

![alt text][solid_yellow_ok_3]


`LINEAR_INTERPOLATION` **fails** with curves.

![alt text][solid_yellow_not_ok]


`LINE_AVERAGE` **works in all situation** and is good at drawing lines which follow
the lane lines.
 
![alt text][la_solid_yellow_ok_1]

![alt text][la_solid_yellow_ok_2]

![alt text][solid_yellow_not_that_good]

![alt text][la_solid_white_ok]

![alt text][la_solid_white_almost_ok]

![alt text][la_solid_yellow_ok]

![alt text][la_solid_yellow_almost_ok]


### 2. Identify potential shortcomings with your current pipeline

The final version fo the pipeline with the `LINEAR_INTERPOLATION` drawing mode, works
 best in absence of **noise**. With the word **noise** I define: lines in the image which
 are detected incorrectly detected as lane lines.

Some examples are: cars passing by in nearby lanes and color changes in asphalt.

---

The result is interpolated with a second degree polynomial, thus the resulting drawn
  lane line is **bend gently** in some cases. If the degree of the polynomial is raised,
  in noisy images the effect is more obvious.

![alt text][line_bent]

In the image above the right line is slightly bent on the bottom and at the top of 
 the image.

With better noise reduction the degree of the interpolating polynomial can be raised
 resulting in a better lane line, which follows the road. 

---

Cars passing in nearby lanes are filtered out by using polygons with hand picked shapes,
as seen in the image below.

![alt text][polygon_car_near_line]

If the car were to be even closer to the lane line, the bent will be visible (as described
 above).
 
I have no solution on how to solve this problem given the current lane detection pipeline.

---

All the parameters in the pipeline are hand tuned to work with the examples provided. For
 different data sample, more tuning may be required. The solution may not result to be
 flexible enough.

---

The system is designed to work for roads under the following assumption: `lane lines are
 going to be present on the left and right side of the road`.

I haven't tested the system without any lane line on the ground. I suppose that either
 something meaningless will be drawn or the program will crash.


### 3. Suggest possible improvements to your pipeline

It could help to remember the last lane line computed or the trend of the last
 computed lane line. Thus a better average can be computed. This new average will be less
 influenced by **noise**.

The result should be a line which has almost no sudden oscillations frame after frame.
 
---

The system works only on roads with lane lines drawn on the left and right side.
 Crossways and interchanges were not taken into consideration.

A possible improvement may be to create an adaptive system, capable of recognizing
 these situations and apply different types of lane detection/drawing algorithms.

