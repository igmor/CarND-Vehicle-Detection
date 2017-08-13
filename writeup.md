##Writeup Template ###

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
[image1]: ./output_images/car.jpg
[image34]: ./output_images/notcar.jpg
[image2]: ./output_images/HOG_example_0.jpg
[image3]: ./output_images/HOG_example_1.jpg
[image4]: ./output_images/HOG_example_2.jpg
[image5]: ./output_images/sliding_windows.jpg
[image6]: ./output_images/final_frame0.jpg
[image7]: ./output_images/heatmap0.jpg
[image8]: ./output_images/final_frame1.jpg
[image9]: ./output_images/heatmap1.jpg
[image10]: ./output_images/final_frame2.jpg
[image11]: ./output_images/heatmap2.jpg
[image12]: ./output_images/final_frame3.jpg
[image13]: ./output_images/heatmap3.jpg
[image14]: ./output_images/final_frame4.jpg
[image15]: ./output_images/heatmap4.jpg
[image16]: ./output_images/final_frame5.jpg
[image17]: ./output_images/heatmap5.jpg
[image18]: ./output_images/final_frame6.jpg
[image19]: ./output_images/heatmap6.jpg
[image20]: ./output_images/labels6.jpg
[image21]: ./output_images/test_result1.jpg
[image22]: ./output_images/test_result2.jpg
[image23]: ./output_images/test_result3.jpg
[image24]: ./output_images/test_result4.jpg

[video1]: ./test_videos_output/project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code cell of the IPython notebook `P2.ipynb`, functions `get_hog_features` and `extract_features`
that builds feature vector for linear SVM classifier.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

Vehicle class
--

![alt_text][image1]

Not vehicle class
--

![alt text][image34]


I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YUV` color space and HOG parameters of `orientations=18`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)` for three YUV color
planes:

Y-plane
--

![alt text][image2]


U-plane
--

![alt text][image3]


V-plane
--

![alt text][image4]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and settled on colorspace 'YUV', number of orientation buckets = 18 and number of spatial histogram bins
32 as ones reliably giving a high performance in a sense of having low number of false positives and reliable car shape detection througout the video.
I also got some idea for increasing number orientation bins from presentation slides and a talk linked to a project. 

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using standard scaler that normalized feature vectors by subtracting a mean and deviding by vetor 
variance. This is a standard preprocessing technique to ensure good performance of a classifer.


### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search random window positions within the range of y position [300, 600] as it roughly represents lower bottom of a frame where all the action with cars  should be happenning. Top part of a screen is not interesting and also slows down the detection phase. Sliding window sizes was set to (64,64), (96, 128), (96, 96) and (128, 128) with 50% overlap in x and y direction. You can see them all on this frame: 


Sliding windows
--

![alt text][image5]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YUV 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image21]
![alt text][image22]
![alt text][image23]
![alt text][image24]


---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./test_videos_output/project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video and kept lst 10 frames of such detections.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. To further reduce the noise from false positives I intersected every bounding box with a region of interest representing part of the road in front of our car. A non empty intersection results identifies a true positive with a higher probability.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

Frame0
--

![alt text][image6]
![alt text][image7]

Frame1
--

![alt text][image8]
![alt text][image9]

Frame2
--

![alt text][image10]
![alt text][image11]

Frame3
--

![alt text][image12]
![alt text][image13]

Frame4
--

![alt text][image14]
![alt text][image15]

Frame5
--

![alt text][image16]
![alt text][image17]

Frame6
--

![alt text][image18]
![alt text][image19]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:

Labels
--

![alt text][image20]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image18]



---

### Discussion

There are several weak points in my pipeline. It seems like dataset I used for training linear SVM classifier is slightly biased against white cars so having more of them in a training set would likely help to improve reliability of white vechicle detectin. Unfortunately I could not find a reliable readily available data set with white cars. The one from udacity doesn't have a color attribute in a descriptor and when I tried to use it as a whole in fact worsen the preformance of the final classifier. 
Another technique I tried to use and fail was using CNN deep NN classifier and then using weighted combinatinon of two classifers to get the final label, that also turned rather to be unsuccessful for reasons I am still investigating. I got too many false positives from NN in this case. I believe I would need much more data point for training to make CNN shine in this particular case.

