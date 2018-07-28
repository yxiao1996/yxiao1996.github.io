# How to build a simple circle detector?

In some image processing or computer vision tasks, you wish you could recognize objects in images by their color and shape. This tutorial build a simple circle detector, the detector detect the most possible circle in a static scene. After successfully detect, the node crop the input images according to the detection result.

Our detector takes series of images from a static viewpoint as input, and detect the most possible circle in the image through image processing and machine learning. The core method behind the detector is [Hough Transform](https://en.wikipedia.org/wiki/Hough_transform)

Apart from ROS, in this tutorial, I'm going to use the following tools:
* OpenCV (for image processing)
* scikit-learn (for machien learning)

## walk through the principles

The detector's working pipeline consists of mainly the following components: 
* 1. Image preprocessing with color filtering
* 2. Perform Hough Transform on preprocessed images
* 3. Post processing raw images with detection result

#### Image Preprocessing

The Hough Transform module in OpenCV only accept greyscale image as input, so we have to prepare our raw image for it. The most naive method is to directly transform raw images into greyscale image. However, since our task is to find circle with specific color, we can first perform some kind of filtering to get rid of objects with wrong colors. 

#### Hough Transform

I used the Hough Transform module from OpenCV, which takes greyscale image as input and return the radius and center coordinate of the circle. Please refer to the [OpenCV official tutorial](https://docs.opencv.org/3.1.0/da/d53/tutorial_py_houghcircles.html) for more detail.

#### PostProcessing

After figure out where the circle is, we can start to crop the image stream.
