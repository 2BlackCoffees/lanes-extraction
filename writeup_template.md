# **Finding Lane Lines on the Road** 

## Writeup Template

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/solidWhiteCurve.jpg
[image2]: ./test_images_output/solidWhiteRight.jpg
[image3]: ./test_images_output/solidYellowCurve.jpg
[image4]: ./test_images_output/solidYellowCurve2.jpg
[image5]: ./test_images_output/solidYellowLeft.jpg
[video1]: ./test_videos_output/challenge-with-debug.mp4
[video2]: ./test_videos_output/challenge.mp4
[video3]: ./test_videos_output/challenge-50-last.mp4

---

### Reflection

### 1. Description of my pipeline. 

My pipeline consisted of 5 steps. 
1. I converted the images to grayscale: This is a preprocess to better find edges in an image.
    * Once an image is converted to gray only one value defines the intensity of each pixel (instead of the 3 RGB)
    * This means find an edge consists simply in finding a big change in the intensity
2. I blured the image: This is another preprocess to remove noise  
    * noise could be identified as an outlier
    * edge detention might work with lesser results because of the noise

3. Canny edge detection   
    * This is where we transform the image in edges, for this we need to define 2 thresholds that will be used by the algorithm to extract edges in an image.
    * Having edges, the extraction of lines is much easier for the next stage. 

4. I extracted a region of interest
    * I defined my region of interest as follow:
        > x_offset = imshape[1] / 20
        
        > Left line: (0,imshape[0]), (imshape[1] / 2 - x_offset,  3 * imshape[0] / 5), 
        
        > Top line: (imshape[1] / 2 + x_offset,  3 * imshape[0] / 5), 
        
        > Right line: (imshape[1],imshape[0])]], dtype=np.int32)

5. I applied a Hough transformation with the following parameters and modified the draw function
    * rho = 1             # distance resolution in pixels of the Hough grid
    * theta = np.pi/180   # angular resolution in radians of the Hough grid
    * threshold = 1       # minimum number of votes (intersections in Hough grid cell)
    * min_line_len = 15   # minimum number of pixels making up a line
    * max_line_gap = 1    # maximum gap in pixels between connectable line segments

    The out put of the algorithm are many lines, out of these lines we extract 
    * m = (y2-y1)/(x2-x1): Watch out cases where x2 = x1 and 
    * b = y2 - x2 * m
    * Depending on the sign of m (positive or negative we can identify which line is being referenced)
    * As there are several lines returned, I compututed the mean of all m and all b
    * I also computed the highest Y that was returned by all values (thus considering the case where for some reason the line mark down would end)
  
    * All the above worked fine for static picture but was very unstable with videos: I handled this stability with the simple following idea:
        * I save the last 5 values for m and b for each drawn line (each picture of a movie)
        * I use the mean of these last 5 values
        * This improved drastically the stability of the output

    * I as well created a debug mode displaying the recognized lines by openCV & the region of interest: this was very helpful.     

![alt text][image1]
![alt text][image2]
![alt text][image3]
![alt text][image4]
![alt text][image5]

### 2. Identify potential shortcomings with your current pipeline
1. Traffic is a shortcoming because lines could be hardly visible.
2. In the night lines might not be visible very far.
3. Curves are not handled properly 
4. No good handling of lines partially visible
5. It is very slow

### 3. Suggest possible improvements to your pipeline
1. No idea yet regarding traffoc
2. Hough algorithm was extended to handle not only lines but as well curves. This could be an improvement.
3. Regarding the challenge I see the following problems that need to be addressed:
    1. There is a curve: I could handle this with 
        * removing outliers (on a picture) and 
        * increasing the latest saved values to compute the mean from 5 to 150
    2. The lines are hardly visible on the road
        * Again the mean could help here again
        * Removed outlier in the class computing the mean of m and b values associated with each picture
    3. The result is impressive see here with debug where I could understand the problem
    ![See path:][video1]
    >./test_videos_output/challenge-with-debug.mp4
    4. See here the whole movie with 150 last entries: This generates quite some inertir in case of change of curve
    ![See path:][video2]
    > ./test_videos_output/challenge.mp4
    4. See here the whole movie with 50 there the lines are not very stable: 
    ![See path:][video3]
    > ./test_videos_output/challenge-50-last.mp4
4. Regrading performance use Deep Learning with C++ instead
    