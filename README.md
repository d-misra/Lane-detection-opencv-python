## Lane detection using OpenCV

Python implementation of lane detection using image processing algorithms in OpenCV.

![final-result](https://github.com/d-misra/Lane-detection-opencv/blob/master/Lanes.png)

### Requirements
- opencv (cv2)
- matplotlib
- numpy

### Dataset
This project demonstrates the image processing pipeline using a single image. The lanes are marked by a solid white lines (on the right) and alternating short line segments with dots (on the left).

![test-image](https://github.com/d-misra/Lane-detection-opencv/blob/master/Test.png)

The test image was taken from a road dataset video used in this Kaggle [project](https://www.kaggle.com/dpamgautam/video-file-for-lane-detection-project).  

### Image processing pipeline

1. Loading of test image
2. Detect edges (gray scaling, Gaussian smoothing, Canny edge detection)
3. Identify region of interest
4. Apply Hough transforms
5. Post-processing of lane lines (averaging, extrapolating)
6. (Optional) Apply to video streams
