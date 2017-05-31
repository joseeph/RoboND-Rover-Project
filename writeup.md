## Project: Search and Sample Return
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* [Download the simulator and take data in "Training Mode"](https://github.com/joseeph/RoboND-Rover-Project/tree/master/recorded_IMGs)
* [Test out the functions in the Jupyter Notebook provided](https://github.com/joseeph/RoboND-Rover-Project/blob/master/code/Rover_Project_Test_Notebook.ipynb)
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  [Include the video you produce as part of your submission.](https://github.com/joseeph/RoboND-Rover-Project/blob/master/output/test_mapping.mp4)

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the [`perception.py`](https://github.com/joseeph/RoboND-Rover-Project/blob/master/code/perception.py) script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./recorded_IMGs/screenshot1.png
[image2]: ./recorded_IMGs/screenshot2.png
[image3]: ./recorded_IMGs/screenshot3.png
## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

Within the color_thresh() I've done some modification, I've added a input string `sample_type=''`, and a if else condition to read the input, so every time I pass in there strings: 'rock' 'obstacles' 'road' would return the threshed img I need. 

```
def color_thresh(img, sample_type='',rgb_thresh=(160, 160, 160)):
    # Create an array of zeros same xy size as img, but single channel
    color_select = np.zeros_like(img[:,:,0])
    # Require that each pixel be above all three threshold values in RGB
    # above_thresh will now contain a boolean array with "True"
    # where threshold was met
    above_thresh = (img[:,:,0] > rgb_thresh[0]) \
                & (img[:,:,1] > rgb_thresh[1]) \
                & (img[:,:,2] > rgb_thresh[2])
            
             # Index the array of zeros with the boolean array and set to 1
    if(sample_type == 'road')and(sample_type == ''):
        color_select[above_thresh] = 1
        return color_select
    elif(sample_type == 'obstacles'):
        color_select[above_thresh] = 1
        color_select = np.logical_not(color_select)
        return color_select 
    elif(sample_type =='rock'):
        rock_thresh=(img[:,:,0] > 100) & (img[:,:,1] > 100) & (img[:,:,2] < 75)
        color_select[rock_thresh] = 1
        return color_select
    else:
        color_select[above_thresh] = 1
    # Return the binary image
    return color_select
```
![alt text][image1]
![alt text][image2]
![alt text][image3]


#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create [video output](https://github.com/joseeph/RoboND-Rover-Project/blob/master/output/test_mapping.mp4) of your result. 


### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.
```
def perception_step(Rover):
 

   
    # Perform perception steps to update Rover()
    # TODO: 
    # NOTE: camera image is coming to you in Rover.img
    # 1) Define source and destination points for perspective transform
    img = Rover.img
    dst_size = 5 
    bottom_offset = 6
    source = np.float32([[14, 140], [301 ,140],[200, 96], [118, 96]])
    destination = np.float32([[img.shape[1]/2 - dst_size, img.shape[0] - bottom_offset],
                          [img.shape[1]/2 + dst_size, img.shape[0] - bottom_offset],
                          [img.shape[1]/2 + dst_size, img.shape[0] - 2*dst_size - bottom_offset], 
                          [img.shape[1]/2 - dst_size, img.shape[0] - 2*dst_size - bottom_offset],
                          ])
    # 2) Apply perspective transform
    img_warped = perspect_transform(Rover.img, source, destination)
    # 3) Apply color threshold to identify navigable terrain/obstacles/rock samples
    terrain_warped_threshed = color_thresh(img_warped,'road')
    obstacles_warped_threshed = color_thresh(img_warped,'obstacles')
    rock_warped_threshed = color_thresh(img_warped,'rock')

    # 4) Update Rover.vision_image (this will be displayed on left side of screen)
            # Example: Rover.vision_image[:,:,0] = obstacle color-thresholded binary image
            #          Rover.vision_image[:,:,1] = rock_sample color-thresholded binary image
            #          Rover.vision_image[:,:,2] = navigable terrain color-thresholded binary image
    Rover.vision_image[:,:,0] = obstacles_warped_threshed 
    Rover.vision_image[:,:,1] = rock_warped_threshed
    Rover.vision_image[:,:,2] = terrain_warped_threshed



    # 5) Convert map image pixel values to rover-centric coords
    navigable_xpix, navigable_ypix = rover_coords( terrain_warped_threshed )
    obstacle_xpix, obstacle_ypix = rover_coords(obstacles_warped_threshed )
    rock_xpix, rock_ypix = rover_coords(rock_warped_threshed)
        
    # 6) Convert rover-centric pixel values to world coordinates
    navigable_x_world, navigable_y_world = pix_to_world(navigable_xpix, 
                                                            navigable_ypix, 
                                                            Rover.pos[0], 
                                                            Rover.pos[1], 
                                                            Rover.yaw, 
                                                            world_size=Rover.worldmap.shape[0], 
                                                            scale=10)
    obstacle_x_world, obstacle_y_world = pix_to_world(obstacle_xpix, 
                                                          obstacle_ypix, 
                                                          Rover.pos[0], 
                                                          Rover.pos[1], 
                                                          Rover.yaw, 
                                                          world_size=Rover.worldmap.shape[0], 
                                                          scale=10)
    rock_x_world, rock_y_world = pix_to_world(rock_xpix, 
                                                  rock_ypix, 
                                                  Rover.pos[0], 
                                                  Rover.pos[1], 
                                                  Rover.yaw, 
                                                  world_size=Rover.worldmap.shape[0],
                                                  scale=10)

        # 7) Update Rover worldmap (to be displayed on right side of screen)
            # Example: Rover.worldmap[obstacle_y_world, obstacle_x_world, 0] += 1
            #          Rover.worldmap[rock_y_world, rock_x_world, 1] += 1
            #          Rover.worldmap[navigable_y_world, navigable_x_world, 2] += 1
    Rover.worldmap[obstacle_y_world, obstacle_x_world, 0] += 1
    Rover.worldmap[rock_y_world, rock_x_world, 1] += 1
    Rover.worldmap[navigable_y_world, navigable_x_world, 2] += 1

        # 8) Convert rover-centric pixel positions to polar coordinates
        # Update Rover pixel distances and angles
            # Rover.nav_dists = rover_centric_pixel_distances
            # Rover.nav_angles = rover_centric_angles

    rover_centric_pixel_distances, rover_centric_angles = to_polar_coords(navigable_xpix, navigable_ypix)
    Rover.nav_dists = rover_centric_pixel_distances
    Rover.nav_angles = rover_centric_angles
        
     
        
     
        
        
    return Rover
```

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

I've spent most of my time to understand the rotation and coordinate issue, understanding how the change from robotic centric direction to world direction, I think the direction and coordinate system would be the most important bit for building  autonomous robot, beside with computer vision bit.





