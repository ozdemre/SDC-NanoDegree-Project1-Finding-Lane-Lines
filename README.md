# **Finding Lane Lines on the Road** 

## Udacity Self Driving Car Nanodegree Project - 1 Report


---

**Finding Lane Lines on the Road**

The goal of this project is:
* Making a pipeline that finds lane lines on both images and video feed


[//]: # (Image References)

[image1]: ./test_images/solidWhiteCurve.jpg "Before"
[image2]: ./test_images_output/solidWhiteCurve.jpg "Before"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

In this project, first image processing pipeline developed and applied into series of example road images. Then pipeline is further improved for processing video feed. Here are the steps of pipeline and some modifications that are made. Here is an example for using the pipeline on one of the example images.

#### Before
![alt text][image1]

#### After
![alt text][image2]



Pipeline is consist of 6 steps:
1. Images are converted into gray scale by using `grayscale()` function
2. Gaussian Smoothing is applied to the image by using `gaussian_blur()` function
3. Canny Edge Detection is applied for finding the edges by using `canny()` function
4. Four sided polygon is used for masking the regions of the image which are not needed. `region_of_interest()` function is used for this step.
5. Hough transformation is applied for extracting points.In Hough space, "x vs. y" lines can be represented as a points in "m vs. b". The Hough Transform is a conversion from image space to Hough space. 
   Therefore, characterization of a line in image space will be a single point at the position (m, b) in Hough space. Then these points can be used to identify the straight lines on the image by using cv2.line method. 
   For video feed processing additional changes are made in the draw_line function where edge detection conditions are taken into account. This all process is done by using `hough_lines()` and `draw_lines()` functions.
6. Finally detected lanes are merged with original image by using `weighted_img()` function.

All the pipeline is tested with example images firs and hen put into `pipeline()` function for video feed processing. 
For parameters like, Gaussian blur kernel_size, Canny Edge Detection threshold and Hough Transform parameters I mainly stick with values that I used during class quizzes.

#### 1. Additional Changes on the draw_line() function

Even though pipeline worked well on images some additional change should be made for dotted line cases. For that I used sign changes on slopes for differentiating left (negative) and right lanes (positive). 
Then these points are used for calculating average slope and midpoint on each lane. With those informations left and right lines are plotted from bottom of the image up to region of interest. 

Lastly on additional measurement must be taken for processing video feed. 
In some of the frames of the video edges are also detected which cause producing NaN x1,x2,y1 and y2 values during Hough Transformation. This cause calculating NaN slope which cannot be converted into int().
To avoid this simply an if condition is added for skipping those cases.Here is the modified `draw_lines()` function.

```python
def draw_lines(img, lines, color=[255, 0, 0], thickness=2):
    """
    NOTE: this is the function you might want to use as a starting point once you want to 
    average/extrapolate the line segments you detect to map out the full
    extent of the lane (going from the result shown in raw-lines-example.mp4
    to that shown in P1_example.mp4).  
    
    Think about things like separating line segments by their 
    slope ((y2-y1)/(x2-x1)) to decide which segments are part of the left
    line vs. the right line.  Then, you can average the position of each of 
    the lines and extrapolate to the top and bottom of the lane.
    
    This function draws `lines` with `color` and `thickness`.    
    Lines are drawn on the image inplace (mutates the image).
    If you want to make the lines semi-transparent, think about combining
    this function with the weighted_img() function below
    """
    # picture with dimensions: (540, 960, 3)
    left_slope = []
    right_slope = []
    x1_leftvalues = []
    x2_leftvalues = []
    y1_leftvalues = []
    y2_leftvalues = []
    x1_rightvalues = []
    x2_rightvalues = []
    y1_rightvalues = []
    y2_rightvalues = []
    
    for line in lines:
        for x1,y1,x2,y2 in line:
            
            if math.isnan(x1) or math.isnan(x2) or math.isnan(y1) or math.isnan(y2):
                #print('edge condition detected, skipping')
                break             
            
            else:
                slope = ((y2-y1)/(x2-x1))
                #print('valid points are detected')
                if slope <= 0: # represents left lane, roughly -0.76
                    left_slope.append(slope)
                    x1_leftvalues.append(x1)
                    x2_leftvalues.append(x2)
                    y1_leftvalues.append(y1)
                    y2_leftvalues.append(y2)

                    #cv2.line(img, (x1, y1), (x2, y2), color, thickness) #not useful for dotted lines
                    #print('left line')

                else: # represents right lane, roughly 0.56
                    right_slope.append(slope)
                    x1_rightvalues.append(x1)
                    x2_rightvalues.append(x2)
                    y1_rightvalues.append(y1)
                    y2_rightvalues.append(y2)

                    #cv2.line(img, (x1, y1), (x2, y2), color, thickness) #not useful for dotted lines
                    #print('right lane')
                
                
    left_ave_slope = np.average(left_slope)
    right_ave_slope = np.average(right_slope)
    
    if math.isnan(left_ave_slope) or math.isnan(right_ave_slope):
        #print('nan slope detected')
        return
    
    x1_left_ave = np.average(x1_leftvalues)
    x2_left_ave = np.average(x2_leftvalues)
    y1_left_ave = np.average(y1_leftvalues)
    y2_left_ave = np.average(y2_leftvalues)
    x_left_ave = (x1_left_ave + x2_left_ave)/2
    y_left_ave = (y1_left_ave + y2_left_ave)/2

    x1_right_ave = np.average(x1_rightvalues)
    x2_right_ave = np.average(x2_rightvalues)
    y1_right_ave = np.average(y1_rightvalues)
    y2_right_ave = np.average(y2_rightvalues)
    x_right_ave = (x1_right_ave + x2_right_ave)/2
    y_right_ave = (y1_right_ave + y2_right_ave)/2    

    x1_left_n = int(x_left_ave + ((540-y_left_ave)/left_ave_slope))
    x2_left_n = int(x_left_ave - ((y_left_ave-350)/left_ave_slope))
    cv2.line(img, (x1_left_n, 540), (x2_left_n, 350), color=[0, 0, 255], thickness=10)
    
    x1_right_n = int(x_right_ave + ((540-y_right_ave)/right_ave_slope))
    x2_right_n = int(x_right_ave - ((y_right_ave-350)/right_ave_slope))
    cv2.line(img, (x1_right_n, 540), (x2_right_n, 350), color=[0, 0, 255], thickness=10)  
```



 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


Some potential shortcomings would be:

1. Cases where there is an another car in front of the camera.
2. Curvy roads and different light-weather conditions. 

As this is a very primitive way of detecting road lines it would not work in real life conditions. When I tested the pipeline with challenge video, pipeline start to come short even with minor differences.



### 3. Suggest possible improvements to your pipeline

One possible improvement would be to using some sort of polynomial fit for lane lines as it can cover curvy roads.

Another potential improvement could be using HSV color spectrum for processing images, which would be more robust on different light conditions.
