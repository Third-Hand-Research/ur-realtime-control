# Universal Robots realtime control examples
A collections of examples to control a Universal Robots arm in realtime by streaming joint values directly over ethernet.

## Why using realtime control?
Implementing realtime control of a robot arm can be a daunting task, we provide here a minimalistic implementation of realtime control, with the goal of being simple to understand without using the lower-level, more obscure typical implementations.

The general idea is as follow: what we mean by realtime control is that the robot controller is not in charge of the robot movements, it is not executing a preplanned trajectory with preset waypoints and joint or linear movements in a saved file. Here we want the robot to follow a dynamic pose, that will be generated on a separate machine (eg: your computer, a raspberry pi, etc) and constantly sent to the robot controller so that it can instantly follow the newest pose.

In effect, under realtime control, the robot is simply trying to catchup as fast as possible to the list of 6 joint values we keep sending to it. This allows to build interactive projects, for example reacting to a webcam stream and having it follow a dynamic target.

There are 2 major downsides under realtime control:

- **Safety**: as there is no more "preplanned" trajectory, the robot can move in much more unpredictable ways which is inherently more dangerous. Your own code will be responsible to generate the joint target values so any bug might lead to very fast and unexpected movements. It is a really good idea to use the speed scaling at very small percentage and enable the Most Safe mode to prevent such situations in case of bug during development of a project.
- **Cartesian** movements are *hard*: when programming cartesian movements (linear movements in space via XYZ coordinates) we relied on the internal robot controller to figure out the inverse kinematics for us and generate the sequence of joint values to go to precise locations. Under realtime control we become entirely responsible of that as well, so as a general rule of thumb, realtime control is fairly easy when you only will need to move in joint-space, so moving the arm only in joint space and not in cartesian space. If you absolutely need to do realtime cartesian work, this is where frameworks like ROS (https://www.ros.org/) come in handy, but this is a huge, *huge* increase in complexity and is out-of-scope for the Third Hand research project.

## How it works?

Under realtime control, the robot will keep calling our computer, and we should respond with a list of 6 joint values in the form of `[0,0,0,0,0,0]`, with each values the desired position for Axis 1 to Axis 6. Once received it will try to move each axis towards these new targets and call us again for updated values. It will do this very quickly, 125 times per second on a CB-serie robot, and 500 times on a e-series robot, so your program will have to generate these values and respond back with the proper formatting.

We provide 3 example implementations to control the robot in realtime: Processing, NodeJS and TouchDesigner. All use the same program on the robot, and serve as a starting point for you to modify them for your own project.

## Running the examples

- Download this repository on your machine
- in the Settings tab of the robot, Network settings, change the IP address to something like 192.168.0.10, then set your own machine's IP address to something like 192.168.0.11, your machine should be connected over Ethernet to the robot controller, you can test if you have connectivity by pinging the robot controller: `ping 192.168.0.10` in Terminal, if you see a response you are set.
- The *realtime.urp* file has to be loaded on a USB key and opened on the UR robot controller
- For Processing: on your machine, open `realtime.pde` and play it
- For NodeJS: in Terminal, navigate to the downloaded repository, go in `èxamples/nodejs` and launch it: `node realtime.js`
- For TouchDesigner: open the `realtime.toe` file in `examples/touchdesigner`
- before playing, make sure the Speed Scaler on the robot is set to a low value like 5% so we are on the safe side
- Now Play the program on the robot, you should have now realtime control of it
