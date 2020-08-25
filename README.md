# Social Distance Detector (using yolo) :mask:

## **Goal** <br/>
Given a video, check if there are people which violate the social distancing policy.

## **Note** <br/>
The main part of this project is taken from [here](https://www.pyimagesearch.com/2020/06/01/opencv-social-distancing-detector/). This project is only intended to perform a variation on the original version in order to deal with a case that the original project did not handle.

## **Solution's general structure**
For every frame in the video:
1) Classify objects which are people (using yolo).
2) Calculate the distance between the center points of each pair of people, and mark the pairs that violate the distance limitation.

## We'll focus in phase 2 – How to calculate the distance between pair of people?

#### Idea 1: Real distance
Find in the scene an object which its real size is known, and then calculate real measurements by using some techniques (such as cross-ratio) in order to calculate the real distance between two people. A problem: It is not always possible to find such an object.

#### Idea 2: Distance in pixels
Calculate distance in pixels between every pair of people, and define some threshold that from it two people considered as close to each other. <br/>
It turns out that this approach will work well only for some cases, such as when the point view of the camera is from above the scene (see *Figure1*). 
| ![Figure1](../master/images/Figure1.png) | 
|:--:| 
| *Figure 1* |

However, in case the camera is placed in angle such that the people "are moving to the depths of the image" this idea as it is won't work. Let's demonstrate this by a specific example – Figure2 is a frame from a video showing Rothschild Boulevard in Tel Aviv, and we used idea2 on it. 
| ![Figure2](../master/images/Figure2.png) | 
|:--:| 
| *Figure 2* |

We can see that two mistakes were made in the context of distance limit:
1) The pair at the front of the scene were not classified as violating the distance limit, even though they are close to each other (in constrant the women and the baby-wagon were classified as close to each other as requierd).
2) The two people behind the woman with the baby-wagon were classified as violating the distance limit, even though they are far from each other in the real world.  <br/>

What went wrong? The answer is that these mistakes were derived from the depth dimension of the scene that reflected more in this video compared to the previous video. In more detail the 2 problems are: <br/>

1) There is no uniform distance threshold: In this frame, for two pairs of pixels that their distance in pixels is the same, the real distance they reflecting is not necessarily the same. This is because as we get closer to the infinity point in the image, the real distance between the objects becomes smaller. Hence the single defined threshold worked for objects that appeared in the middle of the image (the woman and the baby-wagon), and not for objects that appeared in the front of the image (the pair). In other words, the distance in pixels between two people is vary when the pair of people is in the front or in the background , and hence we can't define a single threshold  that is suitable for every pair of people in the frame.

2) Not every pair of people in the image are "comparable": It can be that the centers of two people are close to each other in the image, while in the real world one person is in the front and the other is in the background (that is, they are far from each other), and therefore it is redundant to compare the distance between them in first place (hereinafter: "non-comparable objects").

#### Idea 3: Distance in pixels adjusted by heights
The two problems above can be solved by using the heights in pixels of the people in the image. Note that the deeper a person is in the picture, the smaller his "height in pixels" (hereinafter:  height) becomes. In order to calculate the height of each person we use the rectangular frames that yolo provides as an output after it classify the objects as people. The calculation formula will be: 
<br/>
Height = the difference in the Y-axis between the bottom edge and the top edge of the rectangular frame. 
<br/>

In addition, we will use the fact that the scene is unchanging and therefore the people in the scene are walking in a defined path, and also use the prior knowledge that humans do not fly in the air and do not shrink or grow all of a sudden, so the height of a person who is in the depth of the image is indeed smaller than the height of a person who is in in the front of the image. 
From here we get that the height values of "comparable" people necessarily be similar, or in other words – the division of the height values will be close to the value 1. 

Illustration: in Figure 3, above each person we wrote his height. It can be seen the heights of people in the front of the image are around 240 pixels, in the middle of the image are around 50 pixels, and in the depth of the image are around 13 pixels.
| ![Figure3](../master/images/Figure3.png) | 
|:--:| 
| *Figure 3* |

<br/>
Now we can improve idea 2: 
First, for every pair of people in the image we'll check if they are comparable by calculating the division of their heights. If the given value is close to 1 it indicates that we can check if these people are close to each other, otherwise it is not relevant to check this. This step handles problem 2. <br/>
Second, in order to solve problem 1, we divide the scene into imaginary 3D rectangles (see Figure 4) such that in each rectangle there will be a different threshold for deciding whether two people are close to each other. In order to implement it practically, I defined several height's ranges and for each range decided on different distance threshold which is suitable for it. 

| ![Figure4](../master/images/Figure4.png) | 
|:--:| 
| *Figure 4* |

## **Final algorithm:**
For every frame in the video: <br/>
1) Classify object that are people (using yolo)
2) For every pair of people:
   - Calculate the heights of the people, and determine whether the two people are "comparable".
   - If they are comparable:
  Calcuate the distance in pixels between them, and determine whether the distance limitation has been violated according to the distance threshold which suitable to the heights of these pair.
  
## **Results** <br/>

| *Note that the algorithm was able to detect the violation in the front and the violation in the middle of the image, even though the distance in pixels between each pair is different.*            |           *Note that although the centers of the woman with the baby-wagon and the other woman are very close, no violation is shown, as required.*                          |
:-------------------------:|:-------------------------:
![](../master/images/result1.png)  |  ![](../master/images/result2.png)
| |   |

| | |
|-------------------------|-------------------------|
:---: | :---: | :---:
|![](../master/images/result1.png){width=200px}  |  ![](../master/images/result2.png){width=200px}|
|*Note that the algorithm was able to detect the violation in the front and the violation in the middle of the image, even though the distance in pixels between each pair is different.*  |    *Note that although the centers of the woman with the baby-wagon and the other woman are very close, no violation is shown, as required.*  |

| ![](../master/images/GIF.gif) | 
|:--:| 
| *GIF* |

**Disadvantages of idea 3:**
* Distance in pixels is inaccurate.
* We divide the scene into a finite-small number of rectangles such that in each rectangle there is a different distance threshold, and it's not accurate.
* We chose the distance thresholds based on trial and error, which is also inaccurate.
* This solution is very much adapted to this specific video, and if we want to use it for other video we will need to choose different thresholds etc.  We may even not be able to use this solution due to the different characteristics of each video.
* This solution didn't consider cases which affecting the heights, such as:
  * Children (whose heights are significantly different from that of adults).
  * The fact that a person can bend or lie on the floor.

**Advantages of idea 3:**
- Easy to implement. 
- Provides a reasonable solution

