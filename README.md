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

### Frame Extraction and Processing:  	
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
       
### White and Yellow Image Masking
A color mask isolates a specific color in an image. You can apply a color mask to a clip to correct a particular color, exclude that color from corrections in the rest of the Image, or both. The frame is read first and filtered with a Gaussian filter. The frame is masked with yellow and white so that the lane lines can be seen perfectly.

    %% White and Yellow Image Masking
    %% Initializing the loop to take frames one by one
    while hasFrame(VideoFile)
        %% ----------------Reading each frame from Video File------------------
        frame = readFrame(VideoFile);
        figure('Name','Original Image'), imshow(frame);
        frame = imgaussfilt3(frame);
        figure('Name','Filtered Image'), imshow(frame);



        %% Masking the image for White and Yellow Color
        %% ------------Define Thresholds for masking Yellow Color--------------
        %% --------------------Define thresholds for 'Hue'---------------------
        channel1MinY = 130;
        channel1MaxY = 255;
        %% ----------------Define thresholds for 'Saturation'------------------
        channel2MinY = 130;
        channel2MaxY = 255;
        %% -------------------Define thresholds for 'Value'--------------------
        channel3MinY = 0;
        channel3MaxY = 130;
        %% ---------Create mask based on chosen histogram thresholds-----------
        Yellow=((frame(:,:,1)>=channel1MinY)|(frame(:,:,1)<=channel1MaxY))& ...
            (frame(:,:,2)>=channel2MinY)&(frame(:,:,2)<=channel2MaxY)&...
            (frame(:,:,3)>=channel3MinY)&(frame(:,:,3)<=channel3MaxY);
         figure('Name','Yellow Mask'), imshow(Yellow);
         %% ------------Define Thresholds for masking White Color---------------
         %% --------------------Define thresholds for 'Hue'---------------------
        channel1MinW = 200;
        channel1MaxW = 255;
        %% ----------------Define thresholds for 'Saturation'------------------
        channel2MinW = 200;
        channel2MaxW = 255;
        %% -------------------Define thresholds for 'Value'--------------------
        channel3MinW = 200;
        channel3MaxW = 255;
        %% ---------Create mask based on chosen histogram thresholds-----------
        White=((frame(:,:,1)>=channel1MinW)|(frame(:,:,1)<=channel1MaxW))&...
            (frame(:,:,2)>=channel2MinW)&(frame(:,:,2)<=channel2MaxW)& ...
            (frame(:,:,3)>=channel3MinW)&(frame(:,:,3)<=channel3MaxW);
        figure('Name','White Mask'), imshow(White);


### Edge detection
This section extracts edges from the masked Image and neglects closed edges with smaller areas. Edge detection is an image processing technique used to identify points in a digital image with discontinuities. are called the edges (or borders) of the picture.

        %% Detecting edges in the image using Canny edge function
        frameW = edge(White, 'canny', 0.2);
        frameY = edge(Yellow, 'canny', 0.2);
        %% Neglecting closed edges in smaller areas
        frameY = bwareaopen(frameY,15);
        frameW = bwareaopen(frameW,15);
        figure('Name','Detecting Edges of Yellow mask'), imshow(frameY);
        figure('Name','Detecting Edges of White mask'), imshow(frameW);

