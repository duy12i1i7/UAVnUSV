## Description

XTDrone is a UAV simulation platform based on PX4, ROS and Gazebo. XTDrone supports mulitrotors (including quadrotors and hexarotors), fixed wings, VTOLs (including quadplanes, tailsitters and tiltrotors) and other unmanned systems (such as UGVs, USVs and robotic arms). It's convenient to deploy the algorithm to real UAVs after testing and debugging on the simulation platform.
## Installation
### Installation of Dependencies
- In the process of using apt to install (including the installation of ROS), if there is a dependency problem that is difficult to solve, you can use aptitude install (if there is no aptitude, use sudo apt install aptitude to install). For example, sudo aptitude install ros-kinetic-desktop, it will recommend dependency solutions in turn. During the instruction execution process, if you think the logic is feasible, press Y, and if it is not feasible, press N. Of course, this tool is not omnipotent. If it can't solve the dependency problem, you still need to analyze and solve it yourself.
- Sometimes an error will be reported in the process of apt, indicating that "Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?" At this time, follow the prompts.
```
sudo apt install curl -y
sudo apt install -y ninja-build exiftool ninja-build protobuf-compiler libeigen3-dev genromfs xmlstarlet libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev python3-pip gawk

pip3 install packaging numpy empy toml pyyaml jinja2 pyargparse

sudo apt install python2
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py # Fetch get-pip.py for python 2.7 
python2 get-pip.py

export PATH="$HOME/.local/bin:$PATH"

pip2 install pandas jinja2 pyserial cerberus pyulog==0.7.0 numpy toml pyquaternion empy pyyaml

python3 -m pip install --upgrade pip
pip uninstall em
pip install empy==3.3.4
```
- Note that the cmake version should be between 3.10 and 3.15, too high or too low will not work.
- If the following error is reported, you can update setuptools and pip first.
```
Collecting pandas
  Using cached https://files.pythonhosted.org/packages/64/f1/8fdbd74edfc31625d597717be8c155c6226fc72a7c954c52583ab81a8614/pandas-1.1.2.tar.gz
    Complete output from command python setup.py egg_info:
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-qtvsjq8t/pandas/setup.py", line 349
        f"{extension}-source file '{sourcefile}' not found.\n"
                                                             ^
    SyntaxError: invalid syntax
    
    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-qtvsjq8t/pandas/
```

```
pip install --upgrade setuptools
# If there is no error, you do not need to enter these two command.
python -m pip install --upgrade pip
```
- Note: Do not use 'apt-get autoremove' easily in the whole environment configuration, see details in Careful with apt-get autoremove.
- Note: PX4 and ROS can not use Anaconda. If it has been installed before, you must comment out the relevant code in .bashrc.

### Installation of ROS
- Ubuntu 20.04 with Noetic is recommended because it's the most stable
- Following this link: [ROS Noetic](http://wiki.ros.org/noetic/Installation/Ubuntu) in order to install ROS into your system.
- Note: select `desktop` installation mode by using `sudo apt install ros-noetic-desktop-full`
To create a new workspace, we recommend you to use catkin-tools  to manage the workspace, and then except for PX4 simulation environment startup, the rest ROS related projects are managed under this workspace.
```
mkdir -p ~/catkin_ws/src
mkdir -p ~/catkin_ws/scripts
cd ~/catkin_ws && cd ~/catkin_ws/src && catkin_init_workspace
cd .. && catkin_make
```
### Installation of Gazebo
Gazebo includes Gazebo itself and ROS plugins, which need to be installed separately. First uninstall the previous Gazebo.
```
cd
sudo apt-get remove gazebo* -y
sudo apt-get remove libgazebo* -y 
sudo apt-get remove ros-noetic-gazebo* -y
```
You must to install the latest gazebo9, do not install gazebo11.
```
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
wget https://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
sudo apt-get update -y
sudo apt-get install gazebo9 -y
sudo apt-get install libgazebo9-dev -y
```
We modified the ROS plug-in of Gazebo. Therefore, source code compilation is required. Please install dependencies first:
```
sudo apt-get install -y ros-noetic-moveit-msgs ros-noetic-object-recognition-msgs ros-noetic-octomap-msgs ros-noetic-camera-info-manager ros-noetic-control-toolbox ros-noetic-polled-camera
```
Then compile (if other dependencies are missing during compilation, install them in the same way as above) . Since the files of XTDrone need to be used, XTDrone Source Code Download needs to be completed first.
```
cd
git clone https://github.com/robin-shaun/XTDrone.git
cd XTDrone
git submodule update --init --recursive

cd ~/catkin_ws/src/
cp -r ~/XTDrone/sitl_config/gazebo_ros_pkgs ./
cd .. && catkin_make
```
Gazebo has many open source model files, which can be downloaded from github. 
Unzip the attachment and put it under ~/.gazebo. At this time, you can see many models in ~/.gazebo/models/. If you run the Gazebo simulation without doing this step, many models may be missing and will be downloaded automatically. 
```
sudo rm -rf ~/.gazebo/models/
mkdir -p ~/.gazebo/models/
git clone --recurse-submodules https://github.com/osrf/gazebo_models.git  ~/.gazebo/models/
```
### Installation of MAVROS
```
cd
rm install_geographiclib_datasets.sh
sudo apt install ros-noetic-mavros ros-noetic-mavros-extras -y
wget https://raw.githubusercontent.com/mavlink/mavros/master/mavros/scripts/install_geographiclib_datasets.sh
sudo chmod a+x ./install_geographiclib_datasets.sh
sudo ./install_geographiclib_datasets.sh
```
### PX4 Configuration
The recommended configuration is given below
```
cd
git clone https://github.com/PX4/PX4-Autopilot.git
mv PX4-Autopilot PX4_Firmware
cd PX4_Firmware
git checkout -b xtdrone/dev v1.11.0-beta1
git submodule update --init --recursive
make px4_sitl_default gazebo
```
If the compiler prompts an error as follows:
```
/home/XXX/Firmware/Tools/sitl_gazebo/src/gazebo_usv_dynamics_plugin.cpp:285:15: error: ‘xformV’ was not declared in this scope
```
You can change it to `ignition::math::Matrix4<double> xformV(vq);`
Then save and recompile.
```
make px4_sitl_default gazebo
```
Afterthat, Gazebo will start at this time. Terminate it by using `Ctrl+C`

Therefore, Run
```
cd ~/PX4_Firmware
source ~/catkin_ws/devel/setup.bash    
source Tools/setup_gazebo.bash $(pwd) $(pwd)/build/px4_sitl_default
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)/Tools/sitl_gazebo
```
At this point, the basic configuration of PX4 and ROS has been completed, MAVROS and SITL communicate successfully. If it is false, it is usually because the path in the .bashrc is incorrect, please check carefully.
### XTDrone Source Code Download
Because the order in which Gazebo looks for models is to search in `.gazebo/models/` first, and then search in other paths, so when copying models to PX4 SITL, pay attention to whether there is a file with the same name under `.gazebo/models/` (such as  `stereo_camera`) , if any, either delete the file with the same name, or replace the file with the same name.

Because the  `gazebo_ gimbal_ controller_ plugin.cpp` in `PX4/Firmware/Tools/sitl_gazebo` has been modified (the source code can't control the pan tilt of multiple UAVs), so the project needs to be compiled again.
```
cd ~/XTDrone
cp sensing/gimbal/gazebo_gimbal_controller_plugin.cpp ~/PX4_Firmware/Tools/sitl_gazebo/src/
cp sitl_config/init.d-posix/rcS ~/PX4_Firmware/ROMFS/px4fmu_common/init.d-posix/
cp sitl_config/worlds/* ~/PX4_Firmware/Tools/sitl_gazebo/worlds/
cp -r sitl_config/models/* ~/PX4_Firmware/Tools/sitl_gazebo/models/ 
cp -r sitl_config/launch/* ~/PX4_Firmware/launch/
cd ~/.gazebo/models/
rm -r stereo_camera/ 3d_lidar/ 3d_gpu_lidar/ hokuyo_lidar/
```

```
cd ~/PX4_Firmware
make px4_sitl_default gazebo
```

### UAVs and USVs Cooperation
#### Building Ocean environment with USV
```
cd
sudo apt install python-is-python3 -y
cp -r ~/XTDrone/sitl_config/usv/* ~/catkin_ws/src/
cd catkin_ws
catkin_make
```
#### Start Simulation
If the simulation speed is quite slow, you can reduce the number of the UAVs.
```
roslaunch px4 sandisland.launch
```
#### Control the USV
The movement of the UAV can be controlled via the following topics:
```
/wamv/thrusters/left_thrust_angle
/wamv/thrusters/left_thrust_cmd
/wamv/thrusters/right_thrust_angle
/wamv/thrusters/right_thrust_cmd
```
For example, the left and right propellers of the USV release the same thrust:
```
rostopic pub -r 1  /wamv/thrusters/left_thrust_cmd std_msgs/Float32 "data: 1.0"
```
```
rostopic pub -r 1  /wamv/thrusters/right_thrust_cmd std_msgs/Float32 "data: 1.0"
```
The control of the UAV is the same as usual. The coordination of these two types unmanned vehicle is depended on your needs.


https://github.com/user-attachments/assets/c1f77f09-6ae9-4ccc-9719-f09899500ef6


At this point, the preliminary simulation of UAV and USV is completed!
