# FlyGS
Following are the main OS/software/frameworks we use at FlyGS :
* [Ubuntu 20.04 (OS)](https://releases.ubuntu.com/20.04/)
* [ROS Noetic LTS (Middleware)](http://wiki.ros.org/noetic)
* [MAVROS](http://wiki.ros.org/mavros), [Mavlink](https://mavlink.io/en/), [PX4-Autopilot](https://docs.px4.io/master/en/) (UAV simulation packages)
* [NS3](https://www.nsnam.org/) - built with waf (Network simulator, this version will be used to standalone (without Gazebo, ROS interface) simulations)
* [NS3](https://gabrielcarvfer.github.io/NS3/) - built with CMake (Network simulator to interface with Gazebo via ROS)

## [A] Setting up PX4 SITL with ROS Noetic (Ubuntu 20.04) in a simulated environment in Gazebo.

### 1) Install ROS Noetic from the official [ROSWiki documentation](http://wiki.ros.org/noetic/Installation/Ubuntu)
Make sure you install the correct ROS Distribution corresponding to your Ubuntu version. 
| ROS version | Ubuntu version | 
| -------- | -------- | 
| Melodic  | 18.04    | 
| Noetic   | 20.04    | 

### 2) Install dependencies

* Execute the following commands to install dependencies : 
```
sudo apt-get update -y
sudo apt-get install git zip cmake build-essential protobuf-compiler libzmqpp-dev libeigen3-dev genromfs ninja-build exiftool astyle python3-empy python3-toml python3-dev python3-pip python3-catkin-tools python3-rosinstall-generator ros-noetic-geographic-msgs ros-noetic-gazebo-* -y
sudo apt-get install libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-bad gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-gl -y
sudo -H pip install --upgrade pip
sudo -H pip install pandas jinja2 pyserial pyyaml
sudo -H pip3 install pyulog
sudo usermod -a -G dialout $USER
sudo apt-get remove modemmanager -y

pip install \
	pandas \
	jinja2 \
	pyserial \
	cerberus \
	pyulog \
	numpy \
	toml \
	pyquaternion
```

### 3) Download MAVROS, MAVLink packages and build them
* After ROS installation is complete, set up and initialize a ROS workspace with any name (for e.g. `flyGS_ws`)
```
mkdir -p flyGS_ws/src
cd flyGS_ws
catkin init && wstool init src
```
* Add the .rosinstall files for MAVROS and MAVLink and build them (Note that the below steps are to be done in the root of the workspace)
```
rosinstall_generator --rosdistro noetic mavlink | tee /tmp/mavros.rosinstall
rosinstall_generator --upstream mavros | tee -a /tmp/mavros.rosinstall
wstool merge -t src /tmp/mavros.rosinstall
wstool update -t src -j4
rosdep install --from-paths src --ignore-src -y
sudo ./src/mavros/mavros/scripts/install_geographiclib_datasets.sh

```
* Build your workspace
```
catkin build 
```

### 4) Download the PX4 Firmware package and build it
```
cd ~/flyGS_ws/src
git clone -b v1.12.0 https://github.com/PX4/PX4-Autopilot.git --recursive
cd PX4-Autopilot
make px4_sitl_default gazebo 
```
* After successfully building it, PX4 will launch automatically and spawn the drone.
### 5) Add the Gazebo model and ROS package paths to `flyGS_ws/devel/setup.bash` file
* Add the following lines to the `setup.bash` file inside (at the end) `<path_to_ws>/devel` so Gazebo can find the sdf models and ROS can find the packages when you source your ROS workspace (flyGS_ws).
```
source ~/flyGS_ws/src/PX4-Autopilot/Tools/setup_gazebo.bash ~/flyGS_ws/src/PX4-Autopilot/ ~/flyGS_ws/src/PX4-Autopilot/build/px4_sitl_default
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/flyGS_ws/src/PX4-Autopilot
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/flyGS_ws/src/PX4-Autopilot/Tools/sitl_gazebo
```
* If you don't want the Gazebo paths to be printed in your terminal every time, comment out the print (`echo`) statements in the `flyGS_ws/src/PX4-Autopilot/Tools/setup_gazebo.bash` file.
```
#echo -e "GAZEBO_PLUGIN_PATH $GAZEBO_PLUGIN_PATH"
#echo -e "GAZEBO_MODEL_PATH $GAZEBO_MODEL_PATH"
#echo -e "LD_LIBRARY_PATH $LD_LIBRARY_PATH"
```
### 6) Testing the setup
* Source your workspace in the current working terminal
```
source ~/flyGS_ws/devel/setup.bash
```
* Launch PX4 and MAVROS nodes using the following command, you must see an iris drone in a Gazebo world environment.
```
roslaunch px4 mavros_posix_sitl.launch
```
* In a new terminal, run the following command
```
rostopic echo /mavros/state
```
If it shows the `connected` parameter to be `True` as shown below, then you may assume that your setup works fine. 
```
---
header: 
  seq: 29
  stamp: 
    secs: 29
    nsecs: 336000000
  frame_id: ''
connected: True
armed: False
guided: False
manual_input: True
mode: "AUTO.LOITER"
system_status: 3
---
```

### n) Tips for fast execution
* Create an alias in your `.bashrc` file with the name of your workspace to source your workspace from any terminal super fast. For e.g. if the name of your workspace is `flyGS_ws` (assumed to be there in the `~` directory), add the following line in your `.bashrc` file (present in the `~` folder).
```
alias flyGS_ws='source ~/flyGS_ws/devel/setup.bash'
```

## [B] Installing and setting up NS3 with cmake

### 0) Resources used
* NS3 official documentation : https://www.nsnam.org/wiki/Installation#Installation
* NS3 with cmake : https://gabrielcarvfer.github.io/NS3/installation

### 1) Installation of standard/normal and cmake versions of NS3
* Install the standard version of NS3 ; Install all the required dependencies from [this](https://www.nsnam.org/wiki/Installation#Ubuntu.2FDebian.2FMint) link, and then install NS3 from [this](https://www.nsnam.org/wiki/Installation#Installation) link in the following way: 
* Clone the cmake version of NS3 inside the `ns-3-allinone` folder
```
cd ~
git clone https://github.com/Gabrielcarvfer/NS3.git
```

* Build the cmake version of NS3
```
cd NS3
mkdir cmake-cache
cd cmake-cache
cmake -DCMAKE_BUILD_TYPE=DEBUG ..
make
```

## Content Description
* [mavad](https://github.com/Sarang-BITS/airborne_networks/tree/main/mavad) : Networking + Planning stack of the co-simulator codebase
* [pci](https://github.com/Sarang-BITS/airborne_networks/tree/main/pci) : (Planner - Control Interface) Interface between **mavad** and **MAVROS** to further connect to PX4-SITL in Gazebo


## Setup and demo
* To use this project, you need to install ROS, MAVROS, PX4-SITL and NS3 (using CMake). Note that this project has been tested on Ubuntu 18.04 OS and ROS Melodic distribution. All the instructions to install these dependencies are given in the [SETUP.md](https://github.com/Sarang-BITS/airborne_networks/blob/main/SETUP.md) file
* Clone this repo in your systems and copy [pci](https://github.com/Sarang-BITS/flyGS/tree/main/pci) and [planner_msgs](https://github.com/Sarang-BITS/flyGS/tree/main/planner_msgs) to `flyGS_ws/src` (ROS workspace), [mavad](https://github.com/Sarang-BITS/flyGS/tree/main/mavad) to `ns3-all-in-one/NS3`, `sitl8drones.launch` to `<path_to_ws>/src/PX4-Autopilot/launch`, and `pci8drones.launch` to `<path_to_ws>/src/pci/launch`
```bash
cd ~
git clone https://github.com/ChengYuanWu83/FlyGS.git
cp -r ~/flyGS/pci <path_to_ws>/src/
cp -r ~/flyGS/flightgoggles <path_to_ws>/src/
cp -r ~/flyGS/camera_orientation_plugin <path_to_ws>/src/
cp -r ~/flyGS/camera_orientation_plugin <path_to_ws>/src/
cp -r ~/flyGS/mavad <path_to_ns3>/
cp ~/flyGS/sitlSingleDrone.launch <path_to_ws>/src/PX4-Autopilot/launch/
cp ~/flyGS/iris.sdf.jinja <path_to_ws>/src/PX4-Autopilot/Tools/sitl_gazebo/models/
```
* Build your ROS workspace (flyGS_ws)
```bash
cd <path_to_ws>
catkin build
```
* Edit the `CMakeLists.txt` file of `ns3-all-in-one/NS3` to build `mavad`
```bash
cd <path_to_ns3>/
gedit CMakeLists.txt 
```
* Add the following contents in the CMakelists.txt file and enable the option to build using emulation support
```bash
# Build mavad scripts
add_subdirectory(mavad)

option(NS3_EMU "Build with emu support" ON)
set(NS3_EMU ON)
```
* Build `mavad` (source your ROS workspace before building so it can find `planner_msgs` which is a dependency for `mavad`). After the build is successful, you would find an executable `mavad_main` inside `<path_to_ns3-all-in-one>/NS3/build`.
```bash
source <path_to_ws>/devel/setup.bash
cd <path_to_ns3>//cmake-cache
make
```
* **Running the simulation demo**
    * Launch the the drones with PX4 autopilot and MAVROS in Gazebo (in terminal 1). You should see 1 unarmed, landed drone in the Gazebo simulator window
    ```bash
    source <path_to_ws>/devel/setup.bash
    roslaunch px4 sitlSingleDrone.launch
    ```
    * Launch the planner control interface for the drones (in terminal 2). You should see the 1 drone armed and hovering at a certain height in the Gazebo simulator window. 
    ```bash
    source <path_to_ws>/devel/setup.bash
    roslaunch pci single_drone.launch
    ```
    * Launch the network simulator and planner stack (in terminal 3). You should see the IP initialization and message communication logs in the terminal and the formation motion of drones in the Gazebo simulator window
    ```
    cd <path_to_ns3>//build
    ./mavad_main
    ```


## Acknowledgement
Our implementation is built upon following repositories:
[flyGS](https://github.com/Sarang-BITS/flyGS) for interface of ROS and NS3
[FlightGoogles](https://github.com/mit-aera/FlightGoggles) for interface of ROS and Unity

