# Social Distance Detector (using yolo)

## **Goal:** <br/>
given a video, check if there are people which violate the social distancing policy.

## **Notes:** <br/>
* this projects implements a small variation of the work of this: /// Details are given below.
* the goal of this project is to use exist implementation, and do some non-lernable metnod. 

## **Solution's general structure:**
For every frame in the video:
1) Classify objects which are people (using yolo)
2) Calculate the distance between the center points of each pair of people, and mark the pairs that violate the distance.



![results example](../master/images/Figure1.png)
