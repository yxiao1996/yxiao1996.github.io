# How to build a simple circle detector?

In some image processing or computer vision tasks, you wish you could recognize objects in images by their color. This tutorial build a simple circle detector, the detector detect the most possible circle in a static scene. After successfully detect, the node crop the input images according to the detection result.

Our detector takes series of images from a static viewpoint as input, and detect the most possible circle in the image through image processing and machine learning.

Apart from ROS, in this tutorial, I'm going to use the following tools:
* OpenCV (for image processing)
* scikit-learn (for machien learning)

### list of  