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

[ColorThreshold]: ./writeup_images/ColorThreshold.png
[RockMinThreshold]: ./writeup_images/MinRockThreshold.png
[RockMaxThreshold]: ./writeup_images/MaxRockThreshold.png 
[ImageProcessing]: ./writeup_images/ImageProcessing.png
[RobotView]: ./writeup_images/RobotView.png
[StuckRover]: ./writeup_images/StuckRover.png
[ExampleRock]: ./calibration_images/example_rock2.jpg
[GoodResult]: ./writeup_images/GoodResult.png

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

Obstacle and rock samples were identified using color thresholds and ranges.  

Differentiating obstacles from clear paths consisted of mapping the rgb values of each pixel to a binary array based on the grey-colored rgb threshold (160, 160, 160) shown below.  A color threshold function assigned a value of 0 to pixels below this grey threshold to indicate free path while a value of 1 was assigned to pixels above the grey threshold to indicate obstacles and unnavigable terrain.  The navigable terrain function used this color threshold function as-is, while a obstacle threshold hold function reversed the 0 and 1 assignments.

![Color Threshold][ColorThreshold]

Differentiating rocks from other materials in the environment was simplified by their unique yellow color relative to all other elements.  Instead of using an rgb color mode to assign rocks, the rock threshold function utilized the hsv color type to assign yellow hues of various saturation and value to account for various levels shading.  A color range of the min and max hsv values shown below was used to identify rocks in the enviroment.

![Rock Min Threshold][RockMinThreshold]
![Rock Max Threshold][RockMaxThreshold]
![Example Rock][ExampleRock]

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 

The process image function was implemented using previous execute quizzes starting with a perspective transform to convert the rover perspective into a two-dimensional birds eye view.  Color thresholds were utilized to assign parts of the terrain to navigable paths, obstacles, and rocks.  Pixel positions in the camera coordinate system were converted to the rover coordinate system, and then to the world coordinate system.  The assigned elements in the world coordinates were used to update the world map based on the location of rocks, navigable, and un-navigable terrain.  The world map rgb channels were updated, with red representing obstacles, green unnavigated terrain, and blue navigated terrain.  Rover space pixels were mapped to world space pixels by using through rotation based on the rover yaw angle.  The rover space was also scaled to match the world map and clipped to show the view.  The figures below exhibit the various stages of image processing from the raw camera view converted to the two-dimensional birds-eye, to the mapped image differentiating clear path from unnavigable terrain, and finally to the path vector calculated as the mean of the navigable pixel angles using polar coordinates.

![Stages of Image Processing][ImageProcessing]

Below is an example of the robot view showing the results of image processing with the raw image, the perspective transformed, and the updated world map.

![Robot View][RobotView]


### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

The perception_step() function, used to recognize features rover's camera and map onto the world view, was filled out using the process_image() function from the jupyter exercise as described above, with some key exceptions needed when moving to the actual autonmous driving interface.  In order to obtain higher fidelity between the expected world map and the calculated world map, the world map could only updated when the rover's roll and pitch were stable enough so that perspective transformation was within an acceptable overlap with the worldmap.  If the roll or pitch were too high, the rover's perspective transform would skew the assignment of pixels when overlayed with the worldmap.

Most of the stock decision_step() function, used to navigate the rover through terrain based on brake, acceleration, and steer values, was suitable for obtaining the said requirements with the exception of the steering angle implementation.  The steering was simply executed based on the mean polar coordinate angle of all navigable pixels relative to the rover's perspective. Also tried, was using the standard deviation of the angles to cause the rover to swing back and forth while driving in order to capture more of the terrain with the camera and prevent getting stuck. While the rover achieved the >40% mapped and >60% fidelity requirement using this strategy, the picture below shows how better navigation AI is necessary to prevent a pile of rocks from ruinning a billion dolar NASA project, when the rocks happen to be the mean of the navigable angles:

![Stuck Rover][StuckRover]

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

FPS: 30
Resolution: 1920 X 1080
Graphics Quality: Fantastic

High map fidelity of roughly 80% compared to the ground truth was achieved when mapping greater than 40% of the available terrain.  Only updating the map when the rover's perspective was stable enough via roll and pitch thresholds was necessary in providing good fidelity between the map created by the rover's pespective and the absolute ground truth.  As mentioned above, using the mean angle of navigable pixels led to passing results, but with the possibility of getting stuck when the direction of obstacles happened to be the mean angle for navigable terrain.  Adding additional functionality to use the mean of individual navigable paths instead of the mean of all navigable pixels would lead the rover to individual paths and help prevent the rover from getting stuck.  Converting already navigated paths to obstacles could also help direct the rover to new un-navigated areas instead of revisiting old paths.  Of course, some function, such as a time stamp, would be needed to allow the rover to go to old paths if these paths have not been fully explored.  Steering the rover along the edge of the navigable terrain could also lead the rover closer to yellow rocks for better collection.  Better strategies of collecting rocks were attempted by steering the rover based on the rock in the rover's perspective.  Oddly, some frames would not recognize the rock, causing the rover to redirect, causing a staggering approach to the rock that was not successful.  Navigating the rover based on the coordinates of nearby rocks from the worldview instead of from the rover's persepctive may be a better approach for rock collection by stabilizing the movement so that the rock does not have to recognized in every camera frame.


![Good Result][GoodResult]


