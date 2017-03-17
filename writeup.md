## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./images/car_not_car.png
[image2]: ./images/HOG_example.png
[image3]: ./images/normalized_features.png
[image4]: ./images/classifier_test.png
[image5]: ./images/heatmap.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the third code cell of the IPython notebook `P5.ipynb`.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.


Here is an example using the `YCrCb` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image2]



#### 2. Explain how you settled on your final choice of HOG parameters.

Actually I copied and pasted the code from course material, then I changed the color space to YCrCb which gave higher test accuracy than RGB. The result 0.9885 is quite high so I didn't play with the parameters.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I used a combination of HOG features, histograms of color(number of hist bins = 16) and spatial binning(spatial_size = (16, 16)). I normalized these features to train a linear SVM.

HOG features are useful for the shape of the car, histogram of color is helpful for the color of the car, spatial binning is useful for both shape and color of the car. As above, I did not spend much time tweaking these parameters.


### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

My code is based on the code from "Hog Sub-sampling Window Search", which requires computing hog features only once for the whole image. I used two window scales. The first one is 1.8 which I searched with from y=400 to 656. The second one is 1.2. It is a small window only needed for distant cars. So I set a range of y=400 to 500, x=240 to 1040 because when a car is at the rightmost or leftmost of x axis is likely to be near my car.  


#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]

Note that I didn't use the exact same window size shown in the image for the video. Please refer to the question above.

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.

Here's an example of heatmap for the test images above. I used a simple threshold of 2.
![alt text][image5]

However, there are still many false positives in the video and the detected rectangle is wobbly. So I tested recording all the detected boxes and only show them when they were detected for 15 frames in a row.


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

This project is very interesting and a little bit easier than the previous one. My major issue is that there are too many false positives as you can see from the blue rectangles in my video. This is solved by observing 15 consecutive frames. However, it may fail when the speed difference between that car and my car is too high. Also, it might take a little while for some cars to be detected when they first appear such as the black car on the right in the last few frames of the video.

Perhaps I can make it more robust by add more training data such as the Udacity data and tweaking hyperparameters. Maybe we can calculate the trajectory of each car detected and use that probability for detecting in the next frame. 
