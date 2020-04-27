# Lane detection using OpenCV

Python implementation of lane detection using image processing algorithms in OpenCV.

![final-result](https://github.com/d-misra/Lane-detection-opencv/blob/master/Hough_lines_avg.png)

## Requirements

- opencv (cv2)
- matplotlib
- numpy

## Dataset

This project demonstrates lane detection using a single image from a road dataset. The lanes are marked by a solid white line (on the right) and alternating short line segments with dots (on the left).

![test-image](https://github.com/d-misra/Lane-detection-opencv/blob/master/Test_image.png)

The test image was taken from a road dataset video used in this Kaggle [project](https://www.kaggle.com/dpamgautam/video-file-for-lane-detection-project).  

## Image processing pipeline

1. Loading of test image
2. Detect edges (gray scaling, Gaussian smoothing, Canny edge detection)
3. Identify region of interest
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
