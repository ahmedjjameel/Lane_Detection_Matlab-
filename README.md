## Autonomous Vehicles Lane Detection using Image Processing Techniques

![project_video](https://user-images.githubusercontent.com/81799459/235457658-bd6a6179-5367-4351-8ce1-e691ff798d55.gif) | ![project_video_output](https://user-images.githubusercontent.com/81799459/235457672-c48b9d4b-d8dd-4866-95df-3288734f2010.gif)
:-------------------------:|:-------------------------:

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

### 1. Frame Extraction and Processing:  	
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
       
### 2. White and Yellow Image Masking
A color mask isolates a specific color in an image. You can apply a color mask to a clip to correct a particular color, exclude that color from corrections in the rest of the Image, or both. The frame is read first and filtered with a Gaussian filter. The frame is masked with yellow and white so that the lane lines can be seen perfectly.

    %% White and Yellow Image Masking
    %% Initializing the loop to take frames one by one
    while hasFrame(VideoFile)
        %% ----------------Reading each frame from Video File------------------
        frame = readFrame(VideoFile);
        figure('Name','Original Image'), imshow(frame);
        frame = imgaussfilt3(frame);
        figure('Name','Filtered Image'), imshow(frame);
![Fig111](https://user-images.githubusercontent.com/81799459/235461650-8cea76f4-cc3b-4cd2-a307-98c85121c936.gif)    |    ![Fig222](https://user-images.githubusercontent.com/81799459/235461659-bf566855-f73b-40dd-96c6-12ba32a4befa.gif)  
:-------------------------:|:-------------------------:
  

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

![Fig3](https://user-images.githubusercontent.com/81799459/235507501-453b92d0-ce20-42ea-a63d-001f4aec38a3.gif) | ![Fig4](https://user-images.githubusercontent.com/81799459/235507509-5bb15eb3-7667-4452-bd44-460275072019.gif)
:-------------------------:|:-------------------------:

### 3. Edge detection
This section extracts edges from the masked Image and neglects closed edges with smaller areas. Edge detection is an image processing technique used to identify points in a digital image with discontinuities. are called the edges (or borders) of the picture.

        %% Detecting edges in the image using Canny edge function
        frameW = edge(White, 'canny', 0.2);
        frameY = edge(Yellow, 'canny', 0.2);
        %% Neglecting closed edges in smaller areas
        frameY = bwareaopen(frameY,15);
        frameW = bwareaopen(frameW,15);
        figure('Name','Detecting Edges of Yellow mask'), imshow(frameY);
        figure('Name','Detecting Edges of White mask'), imshow(frameW);


![Fig5](https://user-images.githubusercontent.com/81799459/235508040-462feb0c-0a50-4029-809a-8117cef1d460.gif)  |  ![Fig6](https://user-images.githubusercontent.com/81799459/235508057-aa240d38-79cd-4e8c-a8fc-40cdbb8b32fa.gif)
:-------------------------:|:-------------------------:

###	4. Extraction of the region of interest
As indicated in the pipeline for project implementation, the region of interest is extracted with the function 'Roipoly' and selection of frame points. An area of interest (ROI) is part of an image that you want to filter or edit somehow. You can display an ROI as a binary mask image: In the mask image, the pixels belonging to the ROI are set to 1, and the pixels outside the ROI are set to 0.

    %% Extraction of the region of interest ROI
    %% -------Extracting Region of Interest from Yellow Edge Frame---------
    roiY = roipoly(frameY, r, c);
    [R , C] = size(roiY);
    for i = 1:R
        for j = 1:C
            if roiY(i,j) == 1
                frame_roiY(i,j) = frameY(i,j);
            else
                frame_roiY(i,j) = 0;
            end
        end
    end
    figure('Name','Filtering ROI from Yellow mask'), imshow(frame_roiY);
    %% -------Extracting Region of Interest from White Edge Frame----------
    roiW = roipoly(frameW, r, c);
    [R , C] = size(roiW);
    for i = 1:R
        for j = 1:C
            if roiW(i,j) == 1
                frame_roiW(i,j) = frameW(i,j);
            else
                frame_roiW(i,j) = 0;
            end
        end
    end
    figure('Name','Filtering ROI from White mask'), imshow(frame_roiW);  

![Fig7](https://user-images.githubusercontent.com/81799459/235508449-f4926026-c2d7-44fa-a1bb-eb5f6c89de63.gif)  |  ![Fig8](https://user-images.githubusercontent.com/81799459/235508456-c267a1ed-d221-48c7-8e9a-b9106a17a31d.gif)
:-------------------------:|:-------------------------:

###	5. Hough Transform
In this section, I have used the Hough function to get the Hough transform of the binary edge-detected Image that gives us the Hough values. Then I have the Hough diagram. It is a technique that can be used to isolate features of a particular shape within an image since the desired features must be parametrically specified in some way.

    %% Applying Hough Tansform to get straight lines from Image
    %% --------Applying Hough Transform to White and Yellow Frames---------
    [H_Y,theta_Y,rho_Y] = hough(frame_roiY);
    [H_W,theta_W,rho_W] = hough(frame_roiW);
    %% ------Extracting Hough Peaks from Hough Transform of frames---------
    P_Y = houghpeaks(H_Y,2,'threshold',2);
    P_W = houghpeaks(H_W,2,'threshold',2);
    %% --------Plotting Hough Transform and detecting Hough Peaks----------
    figure('Name','Hough Peaks for White Line')
    imshow(imadjust(rescale(H_W)),[],'XData',theta_W,'YData',rho_W,'InitialMagnification','fit');
    xlabel('\theta (degrees)')
    ylabel('\rho')
    axis on
    axis normal
    hold on
    colormap(gca,hot)
    
    x = theta_W(P_W(:,2));
    y = rho_W(P_W(:,1));
    plot(x,y,'s','color','blue');
    hold off
    
    figure('Name','Hough Peaks for Yellow Line')
    imshow(imadjust(rescale(H_Y)),[],'XData',theta_Y,'YData',rho_Y,'InitialMagnification','fit');
    xlabel('\theta (degrees)')
    ylabel('\rho')
    axis on
    axis normal
    hold on
    colormap(gca,hot)
    
    x = theta_W(P_Y(:,2));
    y = rho_W(P_Y(:,1));
    plot(x,y,'s','color','blue');
    hold off
    %% -------------Extracting Lines from Detected Hough Peaks-------------
    lines_Y = houghlines(frame_roiY,theta_Y,rho_Y,P_Y,'FillGap',3000,'MinLength',20);
    figure('Name','Hough Lines found in image'), imshow(frame), hold on

    max_len = 0;
    for k = 1:length(lines_Y)
        xy = [lines_Y(k).point1; lines_Y(k).point2];
        plot(xy(:,1),xy(:,2),'LineWidth',2,'Color','green');
        
        % Plot beginnings and ends of lines
        plot(xy(1,1),xy(1,2),'x','LineWidth',2,'Color','yellow');
        plot(xy(2,1),xy(2,2),'x','LineWidth',2,'Color','red');
    end

    lines_W = houghlines(frame_roiW,theta_W,rho_W,P_W,'FillGap',3000,'MinLength',20);
    max_len = 0;

    for k = 1:2
        xy = [lines_W(k).point1; lines_W(k).point2];
        plot(xy(:,1),xy(:,2),'LineWidth',2,'Color','green');
        
        % Plot beginnings and ends of lines
        plot(xy(1,1),xy(1,2),'x','LineWidth',2,'Color','yellow');
        plot(xy(2,1),xy(2,2),'x','LineWidth',2,'Color','red');
    end
    hold off

![Fig9](https://user-images.githubusercontent.com/81799459/235509439-f6e5d2ad-b6aa-4ec6-a22e-6ebc8dc88381.gif)  |  ![Fig10](https://user-images.githubusercontent.com/81799459/235509445-9d45af83-d1ea-4158-90a9-7011fb7b28fd.gif)
:-------------------------:|:-------------------------:

### Canny's edge detection algorithm in connection with Hough transform
Canny's edge detection algorithm has the advantage of a higher computational speed. Still, it can overlook some apparent details of the crossing edge because isotropic Gaussian kernels are used. In this section, the lines into Hough lines are extrapolated into the filtered main Image.

    %% Canny's edge detection algorithm in connection with Hough transform
    %% Plotting best fitting line after Extrapolation
    %% ---------------Extract start and end points of lines----------------
    leftp1 = [lines_Y(1).point1; lines_Y(1).point2];
    leftp2 = [lines_Y(2).point1; lines_Y(2).point2];
    
    rightp1 = [lines_W(1).point1; lines_W(1).point2];
    rightp2 = [lines_W(2).point1; lines_W(2).point2];
    
    if leftp1(1,1) < leftp2(1,1)
        left_plot(1,:) = leftp1(1,:);
    else
        left_plot(1,:) = leftp2(1,:);
    end
    
    if leftp1(2,2) < leftp2(2,2)
        left_plot(2,:) = leftp1(2,:);
    else
        left_plot(2,:) = leftp2(2,:);
    end
    
    if rightp1(1,2) < rightp2(1,2)
        right_plot(1,:) = rightp1(1,:);
    else
        right_plot(1,:) = rightp2(1,:);
    end
    
    if rightp1(2,1) > rightp2(2,2)
        right_plot(2,:) = rightp1(2,:);
    else
        right_plot(2,:) = rightp2(2,:);
    end

![Fig11](https://user-images.githubusercontent.com/81799459/235509275-9ec2e59a-2625-4e5d-ac3e-bcdb2a1909b3.gif)  |  ![Fig12](https://user-images.githubusercontent.com/81799459/235509287-7b8789f3-dd75-4df7-a0d9-7ab45c763100.gif)
:-------------------------:|:-------------------------:

    %% --------------Calculate slope of left and right lines---------------
    slopeL = (left_plot(2,2)-left_plot(1,2))/(left_plot(2,1)-left_plot(1,1));
    slopeR = (right_plot(2,2)-right_plot(1,2))/(right_plot(2,1)-right_plot(1,1));
    %% ----Make equations of left and right lines to extrapolate them------
    xLeftY = 1; % x is on the left edge
    yLeftY = slopeL * (xLeftY - left_plot(1,1)) + left_plot(1,2);
    xRightY = 550; % x is on the reight edge.
    yRightY = slopeL * (xRightY - left_plot(2,1)) + left_plot(2,2);
    
    xLeftW = 750; % x is on the left edge
    yLeftW = slopeR * (xLeftW - right_plot(1,1)) + right_plot(1,2);
    xRightW = 1300; % x is on the reight edge.
    yRightW = slopeR * (xRightW - right_plot(2,1)) + right_plot(2,2);
    %% ----Making a transparent Trapezoid between 4 points of 2 lines------
    points = [xLeftY yLeftY; xRightY yRightY ;xLeftW yLeftW; xRightW yRightW ];
    number = [1 2 3 4];
    %% -----------------------Turn Prediction------------------------------
    Yellow_dir = cross([left_plot(1,1), left_plot(1,2), 1], [left_plot(2,1), left_plot(2,2), 1]);
    Yellow_dir = Yellow_dir ./ sqrt(Yellow_dir(1)^2 + Yellow_dir(2)^2);
    theta_y = atan2(Yellow_dir(2), Yellow_dir(1));
    rho_y = Yellow_dir(3);
    yellow_line = [cos(theta_y), sin(theta_y), rho_y];
    %% ------------Finding vanishing point using cross poduct--------------
    white_dir = cross([right_plot(1,1),right_plot(1,2),1], [right_plot(2,1),right_plot(2,2),1]);
    white_dir = white_dir ./ (sqrt(white_dir(1)^2 + white_dir(2)^2));
    theta_w = atan2(white_dir(2),white_dir(1));
    rho_w = white_dir(3);
    white_line = [cos(theta_w), sin(theta_w), rho_w];
    
    line1 = [0, 1, -left_plot(2,1)];
    point_on_w_lane = cross(line1,white_line);
    point_on_w_lane = point_on_w_lane ./ point_on_w_lane(3);
    line2 = [0, 1, -left_plot(2,2)];
    point_on_w_lane_2 = cross(line2,white_line);
    point_on_w_lane_2 = point_on_w_lane_2 ./ point_on_w_lane_2(3);
 
    vanishing_point = cross(yellow_line, white_line);
    vanishing_point = vanishing_point ./ vanishing_point(3);
    vanishing_ratio = vanishing_point(1) / size(frame, 2);
    
    if vanishing_ratio > 0.47 && vanishing_ratio < 0.49
        direction = 'Turn Left';
    elseif vanishing_ratio >= 0.49 && vanishing_ratio <= 0.51
        direction = 'Go Straight';
    else
        direction = 'Turn Right';
    end

### 6. Output Video
After applying various algorithms, we will obtain results after plotting everything on the Image and writing it into video.

        %% Plotting Everything on the image
        %% -Plot the extrapolated lines, Trapezoid and direction on each frame-
        figure('Name','Final Output'), imshow(frame);
    


![Fig13](https://user-images.githubusercontent.com/81799459/235509891-01a736e4-99a2-4a0d-90b5-c3eaa8b2ffda.gif)
:-------------------------:|:-------------------------:
    
    
        hold on
        plot([xLeftY, xRightY], [yLeftY, yRightY], 'LineWidth',8,'Color','red');
        plot([xLeftW, xRightW], [yLeftW, yRightW], 'LineWidth',8,'Color','red');
        text(650, 65, direction,'horizontalAlignment', 'center', 'Color','red','FontSize',20)
        patch('Faces', number, 'Vertices', points, 'FaceColor','green','Edgecolor','green','FaceAlpha',0.4)
        hold off
        %% ----------------Save each frame to the Output File------------------
        writeVideo(Output_Video,getframe);
        end
        %% -------------------Closing Save Video File Variable---------------------
        close(Output_Video)



![project_video_output](https://user-images.githubusercontent.com/81799459/235457737-78597d7a-8ecd-44fe-bc6b-9ca323eea9a7.gif)

### References
[1]

[2]

[3]

