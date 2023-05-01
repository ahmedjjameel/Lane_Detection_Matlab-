## Autonomous Vehicles Lane Detection using Image Processing Techniques

### Introduction
It is well known that lane recognition on freeways is an essential part of any successful autonomous driving system. An autonomous car consists of an extensive sensor system and several control modules. The most critical step to robust autonomous driving is to recognize and understand the surroundings. However, it is not enough to identify obstacles and understand the geometry around a vehicle. Camera-based Lane detection can be an essential step towards environmental awareness. It enables the car to be correctly positioned within the lane, which is crucial for every exit and the back lane - planning decision. Therefore, camera-based accurate lane detection with real-time edge detection is vital for autonomous driving and avoiding traffic accidents.
In this project, MATLAB serves as an image processing tool for the detection of lanes on the road and the following techniques are used for lane detection: Color Mask, Canny/Sobel Edge Detection, Region of Interest Selection, and Hough Transform Line Detection.

### Summary
1.	Load image or video 
2.	Frame conversion:  The data set has a video format converted into frames for processing.
3.	Image Edge Recognition Algorithm: Canny/Sobel Edge Recognition
4.	Segmentation of the area of interest. After the edge detection by the canny edge detection algorithm, we can see that the edge obtained contains the required lane line edges and other unnecessary lanes and the edges of the surrounding fences. This method can increase the speed and accuracy of the system. 
5.	Lane recognition Algorithm: Hough Transform Snapshot of the lane recognition output with the Hough transform method.
6.	Plot everything on the Image and write it into the output video.

1.	Frame Extraction and Processing
The first step is to import the video file, initialize the variables, and import from the .mat file in code.

    close all; clear; clc;
    %% -----------------------Importing the Video File-------------------------
    VideoFile = VideoReader('videos\project_video.mp4');
    %% -----------------Loading Region of Interest Variables-------------------
    load('roi_variables', 'c', 'r');
    %% ---------------Defining Variables for saving Video File-----------------
    Output_Video=VideoWriter('videos\project_video_output', 'MPEG-4');
    Output_Video.FrameRate= 25;
    open(Output_Video);
