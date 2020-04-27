# Lane detection using OpenCV

Python implementation of lane detection using image processing algorithms in OpenCV.

![final-result](https://github.com/d-misra/Lane-detection-opencv/blob/master/Hough_lines_avg.png)

## Requirements

- OpenCV (cv2)
- matplotlib
- numpy

## Dataset

This project demonstrates lane detection using a single image from a road dataset. The lanes are marked by a solid white line (on the right) and alternating short line segments with dots (on the left).

![test-image](https://github.com/d-misra/Lane-detection-opencv/blob/master/Test_image.png)

The test image was taken from a road dataset video used in this Kaggle [project](https://www.kaggle.com/dpamgautam/video-file-for-lane-detection-project).  

## Image processing pipeline

1. Loading of test image
2. Detect edges (gray scaling, Gaussian smoothing, Canny edge detection)
3. Define region of interest
4. Apply Hough transforms
5. Post-processing of lane lines (averaging, extrapolating)
6. (Optional) Apply to video streams

## Canny edge detection

A multi-stage algorithm that detects a wide range of edges in an image was developed by John F. Canny in 1986. More details can be read from [wikipedia](https://en.wikipedia.org/wiki/Canny_edge_detector) and official OpenCV [tutorials](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_canny/py_canny.html). It is important to highlight 2 steps before using ```cv2.Canny()``` :

- **Noise reduction** - Edges in an image are the regions of changes in pixel intensity. Edge detections are mainly based on gradient computations, which is why results are highly sensitive to image noise. So, the first step is to apply a Gaussian blur on the image to smoothen rough edges.
```
cv2.GaussianBlur(image, (kernel_size, kernel_size), 0)
```  
A 5x5 Gaussian kernel was used here. The ```kernel_size``` can be tweaked with to find best values (should be positive and odd). Bigger kernels imply more blur in the image and also requires more time to process. Hence, smaller values are preferred if the effects are similar.

- **Grayscale conversion** - The edge detection algorithm works on grayscale images as we are only interested in intensity gradients. So the input image has to be converted to grayscale.
```
cv2.cvtColor(image, cv2.COLOR_RGB2GRAY)
```

- **Edge detection** - OpenCV function for canny edge detection takes 3 main arguments : input image and edge intensity gradient thresholds.
```
cv2.Canny(image, min_threshold, max_threshold)
```
The threshold values decide which edges to be kept and which ones to discard. Edges having intensity gradient more than ```max_threshold``` are kept while ones lower than ```min_threshold``` are rejected. Edges with values mid-way, are decided depending on their connectivity - if linked to an *edge-confirmed* pixel then they are considered as part of the edge, otherwise discarded. Threshold values are empirically determined (Canny suggested 2:1 or 3:1 ratio between max and min values). If ```max_threshold``` is very high, no edge will be found whereas if too low, high number of edges are detected. Here, ```min_threshold=50``` and ```min_threshold=150``` values were chosen. Canny detected edges can be seen in the image below. 

![canny](https://github.com/d-misra/Lane-detection-opencv/blob/master/Canny_edges.png)

## Region of interest

Not all edges in the image are useful to the task, which is to identify lanes on the road. As the edges corresponding to the sky, trees etc are irrelevant, they need to be removed. The *region of interest* should fully cover mainly only the lane lines. A simple polygon shape that could define this is a triangle with vertices roughly around the bottom left corner, the image center and near the bottom right corner of the image. So, a triangular polygon is defined as the ROI to be cropped out from the original image. All other portions in the image are excluded by applying a mask using ```cv2.fillPoly()``` and ```cv2.bitwise_and()```

![ROI](https://github.com/d-misra/Lane-detection-opencv/blob/master/ROI.png)

## Hough transforms for line detection

Hough transforms is a feature extraction technique to identify simple shapes such as circles, lines etc in an image. Read more about it from OpenCV [here](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_houghlines/py_houghlines.html) and in this [article](https://www.learnopencv.com/hough-transform-with-opencv-c-python/) by LearnOpenCV.

The lane detection problem requires to identify lines that intersects through all nearby edge pixels, from edges detected in the region of interest. A transformation of lines in Hough Space allows to solve for their intersections simply and then the intersection point can be transformed back into image space to meet this goal. OpenCV function on probabilistic hough transforms ```cv2.HoughLinesP``` for generating Hough lines from an image with edge pixels, is used with reasonable parameters.

```
lines = cv2.HoughLinesP(
    cropped_image,
    rho=2,              #Distance resolution of accumulator in pixels
    theta=np.pi / 180,  #Angle resolution of accumulator in radians
    threshold=100,      #Min. number of intersecting points to detect a line 
    lines=np.array([]), #Vector to return start and end points of the lines indicated by [x1, y1, x2, y2]
    minLineLength=40,   #Line segments shorter than this are rejected
    maxLineGap=25       #Max gap allowed between points on the same line
)
```
A set of lines is returned, which when super-imposed on the image looks like this: 

![hough](https://github.com/d-misra/Lane-detection-opencv/blob/master/Hough_lines_original.png)

## Average and extrapolation of lines

Hough lines generated indicate multiple lines on the same lane. So, these lines are averaged to represent a single line. Also, some lines are partially detected. The averaged lines are extrapolated to cover the full length of the lanes.

The averaging process is performed based on the slopes of the multiple lines, which are to be grouped together belonging to either the *left* lane or the *right* lane. In the image, the ```y``` co-ordinate is reversed, (as the origin is at the top left corner) thus having a higher value when ```y``` is lower in the image. By this convention, the left lane has a negative slope and the right one has a positive slope. All lines having positive slope are grouped together and averaged to get the right lane, and vice versa for the negative slopes to obtain the left lane. Final lanes detected can be seen below:

![final](https://github.com/d-misra/Lane-detection-opencv/blob/master/Hough_lines_avg.png)

## Conclusions

This image processing pipeline architecture can also be successfully applied to any video streams. It applies mostly to scenes having straight lane lines.

Few limitation of the process are:
- Line averaging considers only 2 lanes in the final steps, whereas there can be multiple lanes. Additionally, this process assumes one lane on each side, and would fail if both lanes were on one side  (i.e two left lanes or two right ones)
- If road is steep, the region of interest needs to be modified to detect horizon first (up-to which the lane lines should extend to)
- Curved line detection is beyond the scope of this project and involves more advanced topics such as perspective transforms and poly fitting lines. However, since lanes in the vicinity of cars usually appear straight (curvature farther in the distance), this method should still perform fine.

