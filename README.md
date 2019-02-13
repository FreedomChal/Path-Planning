# Path Planning
Classwork for the Udacity Self-Driving Car Nanodegree

---

This code is an implementation of the [Udacity Path Planning Project](https://github.com/udacity/CarND-Path-Planning-Project). Additional information on the dependencies and building instructions can be found at the project's repository.

## Project Details

This project implements a model which controls a car that drives around in the [Term 3 Simulator](https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2), adjusting speed and changing lanes to drive around the track safely and efficiently. It is capable of navigating through traffic while obeying traffic laws, avoiding collisions, and driving smoothly without excessive acceleration or jerking.

To run the project after the environment is set up and all the dependencies installed (see the [Project Repo](https://github.com/udacity/CarND-Path-Planning-Project)), navigate to the master directory, then run 

`cmake . && make`

Then

`./path_planning`

The program should output `Listening to port 4567`. Start the simulator, and the car should start driving.
