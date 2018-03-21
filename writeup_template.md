## Writeup

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: http://i.imgur.com/7t82pz1.png
[image2]: http://i.imgur.com/ZaBNYpt.png
[image8]: http://i.imgur.com/L39Du0F.png
[image4]: http://i.imgur.com/OEa9B4R.png
[image5]: http://i.imgur.com/dqVhzl6.png
[image6]: http://i.imgur.com/kBif7R6.png
[image7]: http://i.imgur.com/7zPC5GA.png
[video1]: ./project_out.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the [In1 to 8] code cells of the IPython notebook.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here are examples using the `YUV` color space and HOG parameters of `orientations=11`, `pixels_per_cell=(16, 16)` and `cells_per_block=(2, 2)`:

![alt text][image2]
![alt text][image8]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and finally concluded on YUV as it is a color space typically used as part of a color image pipeline. It encodes a color image or video taking human perception into account, allowing reduced bandwidth for chrominance components, thereby typically enabling transmission errors or compression artifacts to be more efficiently masked by the human perception than using a "direct" RGB-representation. It was easier for me to manipulate.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

color_space = 'YUV'
hog_channel = 'ALL'
orient = 11
cell_per_block = 2
pix_per_cell = 16

I trained a linear SVM in [In 11]. I first  extracted the hog features in [In 9] and then I put them in a dataset in  [In 10]. Then I split the dataset to training and testing batches. Then I trained and then tested the LinearSVC in [In 11]. It was pretty accurate (SVC accuracy =  0.9828)

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

It is all done in [In 14]. I search with squares. And I define a top and bottom y tresholds on the image to not search on outside. But I search all x axis. The square's width is defined by "nxsteps = (nxblocks - nblocks_per_window) // cells_per_step" in Line 41 in [In 14]. And I scan through all the area iteratively.
Where I start from searching and the scale of window is predefined with manually selected values in [In 18] and  [In 27].
I tried not to make them overlap too much because that would increase the computing time. And I didn't make them too separate because otherwise I could miss some useful cars. Also scale helped me to scan different sized car features.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YUV 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  To optimize it I played around with sliding window parameters as explained on previous question. Here is an example image:

![alt text][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_out.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

All done in [In 20 to 26]
I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here is a frame and its corresponding heatmap:

![alt text][image5]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from the frame:
![alt text][image6]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image7]



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Well this technique is pretty strong but false detections make things a bit harder. We might not always classify a car right at the first sight, also we need to keep tracking it to be sure it actually is a car. Also I'm pretty sure this tecnique couldn't really tell a car image in real life from a real car. It is so bounded to 2d world from its nature. For my pipeline, a better designed sliding window approach would make it more flexible.
