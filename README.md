# abb_libegm

[![Travis CI Status](https://travis-ci.org/ros-industrial/abb_libegm.svg?branch=master)](https://travis-ci.org/ros-industrial/abb_libegm)

[![support level: vendor](https://img.shields.io/badge/support%20level-vendor-brightgreen.png)](http://rosindustrial.org/news/2016/10/7/better-supporting-a-growing-ros-industrial-software-platform)

## Important Note

RobotWare `6.07` introduced major changes in the EGM communication protocol, and this library has not been updated to support those changes yet.

**Avoid using this library with RobotWare 6.07 at the moment.**

## Overview

A C++ library for interfacing with ABB robot controllers supporting *Externally Guided Motion* (EGM). See the *Application manual - Controller software IRC5* for a detailed description of what EGM is and how to use it.

* See [abb_librws](https://github.com/ros-industrial/abb_librws) for a companion library that interfaces with *Robot Web Services* (RWS).
* See [StateMachine Add-In](https://robotapps.robotstudio.com/#/viewApp/7fa7065f-457f-47ce-98d7-c04882e703ee) for an optional *RobotWare Add-In* that can be useful when configuring an ABB robot controller for use with this library.

### Sketch

The following is a conceptual sketch of how this EGM library can be viewed, in relation to an ABB robot controller as well as the RWS companion library mentioned above. The optional *StateMachine Add-In* is related to the robot controller's RAPID program and system configuration.

![EGM sketch](docs/images/egm_sketch.png)

### Requirements

* RobotWare versions between `6.0` and `6.06.01` (higher versions are to be considered incompatible at the moment).
* A license for the RobotWare option *Externally Guided Motion* (`689-1`).

### Dependencies

* [Boost C++ Libraries](https://www.boost.org)
* [Google Protocol Buffers](https://developers.google.com/protocol-buffers)

### Limitations

This library is intended to be used with the UDP variant of EGM, and it supports the following EGM modes:

* Joint Mode
* Pose Mode

### Recommendations

* This library has been verified to work with RobotWare `6.06.01`. Other versions are expected to work, but this cannot be guaranteed at the moment.
* It is a good idea to perform RobotStudio simulations before working with a real robot.
* It is prudent to familiarize oneself with general safety regulations (e.g. described in ABB manuals).
* Consider cyber security aspects, before connecting robot controllers to networks.

## Usage Hints

This is a generic library which can be used together with any RAPID program which is using the RAPID `EGMRunJoint` and/or `EGMRunPose` instructions, and system configurations. The library's primary classes are:

* [EGMServer](include/abb_libegm/egm_server.h): Sets up and manages asynchronous UDP communication loops. During an EGM communication session, the robot controller requests new references, at the rate specified with RAPID `EGMAct` instructions. When an `EGMServer` instance receives an EGM message from the robot controller the message is passed on to an EGM interface instance (see below). The interface is expected to generate the reply message, containing the new references, which the server then sends back to the robot controller.
* [AbstractEGMInterface](include/abb_libegm/egm_server.h): An abstract interface, which specifies how the `EGMServer` class interacts with EGM interfaces. Can be inherited from to implement custom EGM interfaces.
* [EGMBaseInterface](include/abb_libegm/egm_base_interface.h): Inherits from `AbstractEGMInterface`, encapsulates an `EGMServer` instance, and implements a basic EGM interface. Can be configured to use demo references, which are intended for testing that EGM communication channels works.
* [EGMControllerInterface](include/abb_libegm/egm_controller_interface.h): Inherits from *EGMBaseInterface* and implements an EGM interface variant for execution inside an external control loop that needs to be implemented by a user. Provides interaction methods, which can be used inside external control loops to affect EGM communication sessions.
* [EGMTrajectoryInterface](include/abb_libegm/egm_trajectory_interface.h): Inherits from *EGMBaseInterface* and implements an EGM interface variant for execution of a queue of trajectories. Provides interaction methods, which can be called by a user to for example add trajectories to the queue and to stop/resume the trajectory execution.

The optional *StateMachine Add-In* can be used in combination with any of the classes above.

### [Optional] StateMachine Add-In

The purpose of the RobotWare Add-In is to *ease the setup* of ABB robot controllers. It is made for both *real physical controllers* and *virtual controllers* simulated in RobotStudio. If the Add-In is selected during a RobotWare system installation, then the Add-In will load several RAPID modules and system configurations based on the system specifications (e.g. number of robots and present options).

Retrieve the Add-In via RobotStudio by:

1. Go to the *Add-Ins* tab in RobotStudio.
2. Search for *StateMachine Add-In* in the *RobotApps* window.
3. Select the Add-In and retrieve the Add-In by pressing the *Add* button.
4. Verify that the Add-In was added to the list *Installed Packages*.
5. The Add-In should appear as an option during the installation of a RobotWare system.

See the Add-In's [user manual](https://robotapps.blob.core.windows.net/appreferences/docs/2093c0e8-d469-4188-bdd2-ca42e27cba5cUserManual.pdf) for more details, as well as install instructions for RobotWare systems.

#### Notes

If the EGM option is selected during system installation, then the Add-In will load RAPID code for using *EGMRunJoint* and *EGMRunPose* RAPID instructions. System configurations will also be loaded, and it is important to update the robot controller's EGM configurations. Especially the *Remote Address* and *Remote Port Number* configurations, under the *Transmission Protocol* topic, are vital to edit so that the robot controller will send EGM messages to the correct external destination. The configurations can be found in RobotStudio -> Controller tab -> Configuration -> Communication -> Transmission Protocol -> Edit each *ROB_X* instances. There will be one instance for each robot in the system.

The RWS companion library contain a class specifically designed to interact with the Add-In. For example, to control the RAPID program by starting/stopping EGM communication sessions.

## Acknowledgements

The **core development** has been supported by the European Union's Horizon 2020 project [SYMBIO-TIC](http://www.symbio-tic.eu/).
The SYMBIO-TIC project has received funding from the European Union's Horizon 2020 research and innovation programme under grant agreement no. 637107.

<img src="docs/images/symbio_tic_logo.png" width="250">

The **open-source process** has been supported by the European Union's Horizon 2020 project [ROSIN](http://rosin-project.eu/).
The ROSIN project has received funding from the European Union's Horizon 2020 research and innovation programme under grant agreement no. 732287.

<img src="docs/images/rosin_logo.png" width="250">

The opinions expressed reflects only the author's view and reflects in no way the European Commission's opinions.
The European Commission is not responsible for any use that may be made of the contained information.

### Special Thanks

Special thanks to [gavanderhoorn](https://github.com/gavanderhoorn) for guidance with open-source practices and ROS-Industrial conventions.
