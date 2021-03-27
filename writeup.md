# **Finding Lane Lines on the Road** 

## 1. My pipeline

The pipeline is `find_lane_lines` function.

### Step 1: Convert image to grayscale

- Using `cv2.cvtColor()` function.

### Step 2: Apply Gaussian smoothing

- Using `cv2.GaussianBlur()` function. 
- I chose a `kernel_size=3`. I didn't tune `kernel_size`.

### Step 3: Apply Canny edge detection to find edges

- Using `cv.Canny(img, low_threshold, high_threshold)`. 
- I use `low_threshold=50` and `high_threshold=150`. I didn't spend too much time tuning these 2 parameters.

### Step 4: Filter region of interest

- Using `cv2.fillPoly()` to remain pixels within a polygon. 
- I use a polygon with 4 vertices and set them to `(40,imshape[0]), (imshape[1]/2-40,imshape[0]/2+60), (imshape[1]/2+60,imshape[0]/2+60), (imshape[1], imshape[0])`. If most edges outside ego lane can be removed, it will be convenient later.

### Step 5: Apply Hough transform to find line segments by connecting edges

- Using `cv2.HoughLinesP(masked_edges, rho, theta, threshold, np.array([]), min_line_length, max_line_gap)`. 
- I set `min_line_length=30, max_line_gap=15`.

### Step 6: Merge line segments whose slopes and interacts are close

- Implemented as `merge_lane_lines()` function.

#### Step 6.1 Remove line segments with too small or too large slopes

- I select the slope threshold as `0.45 ~ 0.85 `. This is tuned mainly with `challenge.mp4`.

#### Step 6.2 Cluster line segments using slopes and interacts

- Line segments whose slope difference is smaller than `0.1` and intercept difference is smaller than `50` are put in same cluster.

#### Step 6.3 Create one new line segment for each cluster

- The new slope and interact are the mean of slopes and interacts within each cluster.
- Here I record the sum of y-axis distances of line segments in each cluster. This is used as a confidence of the new created line segment. 

### Step 7: Select left and right lane line segments

- Implemented as `select_two_sides()` function.
- Separate all line segments into two parts by looking if the slope is greater than 0.
- For each part (side of ego lane), line segment with highest confidence will be chosen as the final lane line.

### Step 8: Extrapolate left and right lane line segments

- Extend left and right line segments to a `[y_min, y_max]` range (which I set to  `imshape[0]/2+60, imshape[0]`).

### Step 9: Put extrapolated lane lines on original image

- Using the provided `weighted_img()` function.


## 2. Potential shortcomings with this pipeline

- First potential shortcoming is that this pipeline can not deal with curve road very well because right now, Hough transform only detects straight line segments.

- Second shortcoming is that, when another edges (such as shadows caused by sunlight and trees) appears on ego lane, the lane line segments could be buried in those edges and probably not detected by Hough transform.
- Third shortcoming is the slope range used in Step 6.1. I do think that `0.45 ~ 0.85` is strict. Maybe the upper bound is not necessary. For example, when ego vehicle is not running on the center of lane, the lane line may become vertical in image.
- Fourth shortcoming is the fixed region of interest. When there comes a **slope**, or when there comes a sharp curve, or when ego car is not running on center of the lane, the fixed ROI would not include enough information or would include information that would bother this pipeline.

## 3. Possible improvements to this pipeline

- First possible improvement would be to also use semantic segmentation result so that we can get lane mark pixels more robustly.

- Second potential improvement could be to search for methods of detecting curves such as sliding window method to cluster left and right white pixels and fit curves on each side.

