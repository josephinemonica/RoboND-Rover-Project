## Project: Search and Sample Return
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)
[output]: ./output/output.png 

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
The image processing for obstacles detection is similar with the navigable terrain, but with a modification of inverting the image processed by the `color_thresh` function:
```python
#obstacle
    obstacle=color_thresh(processed_img)
    obstacle=abs(1-obstacle)#invert 1 to 0, 0 to 1 (because it is obstacle)
```
For the rock samples detection, I use created `yellow_thresh` function, instead of the default `color_thresh` function. It will look for yellow color, which is defined to be between lower_yellow and upper_yellow.
```python
def yellow_thresh(img):
    ###### TODO:
    # Create an empty array the same size in x and y as the image 
    # but just a single channel
    color_select = np.zeros_like(img[:,:,0])
    # Apply the thresholds for RGB and assign 1's 
    # where threshold was exceeded
    # Return the single-channel binary image
    #BGR
    lower_yellow = np.array([115,100,0])
    upper_yellow = np.array([255,210,35])
    isyellow=(img[:,:,0] >= lower_yellow[0]) & (img[:,:,0] <= upper_yellow[0] )\
        &( img[:,:,1] >= lower_yellow[1]) & (img[:,:,1] <= upper_yellow[1]) \
    & (img[:,:,2] >= lower_yellow[2]) & (img[:,:,2] <= upper_yellow[2])

    color_select[isyellow]=1
    
    return color_select
```


#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
First, apply perspective transform to the image to convert the image from rover's camera perspective to the top view perspective:
```python
 processed_img=perspect_transform(img,source,destination)
 ```
 Next, apply color threshold, which will result to binary pictures, to identify navigable terrain, obstacles, and rock samples:
 ```python
  #navigable terrain
    navigable=color_thresh(processed_img)
    #obstacle
    obstacle=color_thresh(processed_img)
    obstacle=abs(1-obstacle)#invert 1 to 0, 0 to 1 (because it is obstacle)
    #rock samples
    rock=yellow_thresh(processed_img)
```
Then, convert the image pixels of the binary pic to the rover-centric coordinate using `rover_coords` function. 

And then, convert them into world coordinate, because we want to match the data with the world map. This is done using `pix_to_world` function, which takes the pose of the rover as input and use them for means of translation and rotation.

Now we have the positions (in absolute world coordinate) of navigable terrain/ obstacles/ rock samples of our image. We then use them to update different colors for each of them in the world map. Note: in case where the pixel is identified both as obstacle and navigable, we choose to trust the navigable; This is done through th efollowing code:
```python
#trust navigable
    nav_pix=data.worldmap[:,:,2]>0
    data.worldmap[nav_pix,0]=0
```

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.
* `perception_step` is mostly filled as in the jupyter notebook. Some additions include: converting the rover-centric pixel positions to polar coordinates; This is to get and update the distances and angles of the navigable terrain. These angles are used in the `decision_step` to determine the steering angle.
* `decision_step` : I haven't made any modification yet.


#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

The program completes all the requirements. It can cover >40% of map with >60% fidelity and detects >=1 rock sample. Below is an output of a sample run at:
Screen resolution: 1024x768; Graphics quality: Good; FPS output to terminal~ 16-17
![alt text][output]

#### Future Work
* Make the rover to also pick up the samples and return to the original position
* Improve fidelity: setting threshold for roll and pitch angles that will result to valid image for perspective transform.
* Improve decision tree. Currently the rover might stuck, at times.
