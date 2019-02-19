[image1]: https://github.com/FreedomChal/Path-Planning/blob/master/PathPlanningModelVisualization.png "Model Visualization"
[image2]: https://github.com/FreedomChal/Path-Planning/blob/master/CarChangingLanes.png "Lane Change"




# Path Planning
Classwork for the Udacity Self-Driving Car Nanodegree

---

This code is an implementation of the [Udacity Path Planning Project](https://github.com/udacity/CarND-Path-Planning-Project). Additional information on the dependencies and build instructions can be found at the project's repository.

## Project Details

This project implements a model to control a car in the [Term 3 Simulator](https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2), adjusting speed and changing lanes to drive around the track safely and efficiently. The model is capable of navigating through traffic while obeying traffic laws, avoiding collisions, and driving smoothly without excessive acceleration or jerk.

To run the project after the environment is set up and all the dependencies installed (see the [Project Repo](https://github.com/udacity/CarND-Path-Planning-Project)), navigate to the master directory, then run 

`cmake . && make`

Then

`./path_planning`

The program should output `Listening to port 4567`. Start the simulator, and the car should start driving.

---

## The Model

### The Path

This model plans its path using 50 points ahead. I used the point generator shown in the [Project Q&A](https://classroom.udacity.com/nanodegrees/nd013/parts/6047fe34-d93c-4f50-8336-b70ef10cb4b2/modules/27800789-bc8e-4adc-afe0-ec781e82ceae/lessons/23add5c6-7004-47ad-b169-49a5d7b1c1cb/concepts/3bdfeb8c-8dd6-49a7-9d08-beff6703792d). The point generation pipeline is as follows:

* First, if there are less than two points remaining from the last iteration (points the car did not hit on the previous iteration are returned to the Model by the Simulator), the car is used as the starting reference.

* Otherwise, if there are still several points remaining from the last iteration, the furthest of the previous points is used as a reference.

* Next, three 30-meter-spaced points are generated in front of the reference point ("Waypoints" are given which correspond to the center of lanes at various locations, and these are used to keep the points in the center of the lane).

* The coordinate system is then shifted to be in the car's perspective (car is at (0, 0) and oriented at 0 degrees) to make the math easier.

* A spline is the generated (I used [this](https://kluge.in-chemnitz.de/opensource/spline) spline library) which goes through the reference point and the previously generated 30m spaced points.

* Then, all the points from the previous iteration are used, and whichever ones were passed by the car are replaced with new ones from the spline, so that there are always 50 points. Points are also spaced at an appropriate distance so the car will go at the desired velocity.

This point generation method works quite well; it generates a smooth path and is especially effective while changing lanes because the sparsely-spaced points given to the spline result in a clean, reasonably fast lane change.

### State Machine

The behavior of the model can be summarized as follows:

![alt text][image1]

There are 4 basic states: staying in the lane, preparing for lane change, lane change left, and lane change right.

* Staying in the lane is the default state; the model will automatically be in this state if it is not in any other states.

* If the car detects another car in front of it, it will change to the "preparing for lane change" state. In this state, the car will slow down if it gets close to another car, and brake hard if it gets too close. Also, the car will always be looking for a possibility to change lanes, which it judges using a loss function that will be discussed later. The car will not exit the lane change state until it changes lanes.

* If the car decides to change lanes, it will enter the lane change left or lane change right state. The car will set its "lane value" to 0 (left lane), 1 (middle lane), or 2 (right lane). This means that new reference points will be generated in the new lane, and the spline will make a path that changes into the other lane. Once the lane value is changed, the car will go back to staying in its lane. A lane change is shown below.

![alt text][image2]

#### Loss Function

The loss function is what the model uses to decide which lane to change to. Each lane is given a starting loss equal to the bias against that lane, or the "antibias". Each lane then has its loss value increased by a number proportional to the number of cars in that lane and how far they are away from the car (cars behind the car are penalized less by being considered farther away). Cars 75 meters away or more are ignored, cars 74 meters away or less increase the loss for the lane they are in by 10, and cars less than 18 meters away increase the lane loss by an additional 100, which prevents the car from ever turning into that lane. Also, cars that are less than 25 meters away and going more than 8 mph faster than the car will increase the loss by 100 to prevent the car from being hit while changing lanes.

A lane is considered "safe" if the loss for it is less than 35. If there are two possible lane changes, the model will choose the one with a lower loss. However, if the two lane change options have the same loss, the model will turn left.

Also, the model will not change lanes if the target lane does not have a lower loss value than the current lane

##### Antibias

The "Antibias" value is the bias against a certain lane. The default values are 0 for the middle lane, 3 for the left lane, and 5 for the right lane. Additionally, when the car changes lanes, the antibias for the lane the car was previously in will go up by 1500 to prevent the car from becoming "indecisive", where it keeps changing between the same lanes because they are both safe, but not actually very good options. This antibias will then decrease until it gets below 25, at which point it will reset to its default value. Also, when the car changes lanes from the left or right lane, the antibias for the opposite lane will increase by 500 to prevent the model from making two consecutive lane changes, which can cause excessive acceleration and/or jerk.

## Weaknesses/Improvements

One of the main weaknesses of the model I encountered was its indecisiveness while in traffic. When the model was presented with two safe but not very good lane options, it would not be able to decide between the two, and would constantly change between them, often actually staying on the lane line. To overcome this bad behavior, I added the antibias, among other things. These changes significantly decreased the model's indecisiveness, but the model still can be uncertain while in traffic. This uncertainty could potentially be overcome by making changes to the loss function so that the model has a better intuition on when lane changes should be done.

Another problematic feature of the model occurs during lane changes. While the model is in the "prepare for lane change" state, the model cannot exit said state without changing lanes. As a result of this feature, the car will often make unnecessary lane changes upon exiting traffic. This problem could potentially be fixed by making the model exit the prepare for lane change state when the loss for its lane drops below a certain value.

One other weakness is that when presented with two different lane change options, the model will sometimes change into the lane which has more traffic ahead if there are more cars behind the car in the other lane, even if both lane changes are perfectly safe. The easiest way to fix this might simply be fine-tuning the loss function.
