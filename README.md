# Jackal-Jetson-Xavier-NX-SDK

## Overview

The **Jackal-Jetson-Xavier-NX-SDK** repository provides a comprehensive software development kit (SDK) for integrating the Jackal robot with the NVIDIA Jetson Xavier NX hardware. This SDK allows users to leverage the Jetson Xavier NX's computational power to enhance the Jackal platform for advanced robotics applications.

This SDK contains the following preinstalled software:
- Ubuntu 20.04
- Nvidia Drivers
- ROS2 Humble
- Jackal Packages
- On-Boot TwistMux and Odometry Nodes

Simply Get the image into an SD card following the steps below and connect your Jetson Xavier NX onto your Jackal Robot.

We are assuming you are running Micro-ROS on the Jackal Motors Drivers PCB, if you have not done this step, please visit the section **Updating Firmware** in this website [Micro-ROS-Jackal](https://www.clearpathrobotics.com/assets/guides/foxy/jackal/JackalInstallRobotSoftware.html).

## Getting Started

### Prerequisites

Before getting started, make sure you have the following:

- **NVIDIA Jetson Xavier NX**: Ensure you have the Jetson Xavier NX development board.
- **Jackal Robot with Micro-ROS**: Access to the Jackal robot is required.
- **32GB+ SD Card**: A microSD card with at least 32GB of space.
- **Linux Machine**: A Linux-based system (Ubuntu recommended) for building the image with at least 40 free GB.
- **Required Tools**: 
  - `sfdisk` and `dd` utilities (for partitioning and flashing)
  - `unzip` (for extracting files)
  
To install these utilities on Ubuntu, you can run:
```bash
sudo apt install fdisk unzip
```

- **Optional**
   - `growpart` (for expanding the partitions, described better in the Installation section)

To install these utility on Ubuntu, you can run:
   ```bash
   sudo apt-get install cloud-guest-utils
```

### Installation

1. **Clone the Repository**

   ```bash
   git clone https://github.com/PaoloReyes/Jackal-Jetson-Xavier-NX-SDK.git
   cd Jackal-Jetson-Xavier-NX-SDK
   ```

2. **Generate Partitions Zip File**

   ```bash
   cat image_parts/zippart_* > image.zip
   ```

3. **Unzip the Partitions Binaries**

   ```bash
   unzip image.zip
   ```

4. **Prepare the Target Device** 

   **Important:** Replace `/dev/mmcblk0` with the correct device path (use lsblk to find your device):

   **Warning:** Double-check your target device to avoid overwriting important data.
   
   ```bash
   sudo sgdisk -l=backup_table.gpt /dev/mmcblk0
   ```

5. **Flash the Partitions**

   **Important:** Adjust the device names (/dev/sdb1, /dev/sdb2, etc.) according to your system.

   ```bash
   sudo dd if=imgs/part1.img of=/dev/mmcblk0p1 bs=4M status=progress
   sudo dd if=imgs/part2.img of=/dev/mmcblk0p2 bs=4M status=progress
   sudo dd if=imgs/part3.img of=/dev/mmcblk0p3 bs=4M status=progress
   ...
   sudo dd if=imgs/part22.img of=/dev/mmcblk0p22 bs=4M status=progress
   ```

6. **(Optional) Resizing the main partition**

   If your SD is bigger than 32GB, it's highly recommended to expand the main partition, so you can have more storage space.

   ```bash
   sudo growpart /dev/mmcblk0 1
   sudo resize2fs /dev/mmcblk0p1
   ```

   **Important:** Replace `/dev/mmcblk0` with the correct device path (use lsblk to find your device), as well the `/dev/mmcblk0p1` with the correct partition path (this will also be shown by lsblk).

  
7. **Connect and Boot your Jetson Xavier NX with the SD**

## Usage

Once you have successfully flashed the partitions on the Jetson Xavier NX, it’s highly recommended to connect the device to a monitor to proceed with the initial configuration process.

By default, the pre-installed image contains a user account with the following credentials:


- **Default User:** tomatoxplorers

- **Password:** Riku254

### TeleOperation

By default your Jetson will execute a service on boot called launch_teleop_jackal.service that automatically will wait for a serial connection with the robot on the port ```/dev/ttyACM0``` and launch our ROS2 node for listening a PS4 controller and move the robot in slow and fast mode (More details on teleop controls later).

If you need to change the robot port or the user executing the Micro-ROS connection, please edit the On-Boot .sh file in ```~/.scripts/launch_teleop_jackal```

### Setting up PS4 controller

The process of pairing the PS4 controller with the Jackal's computer is the following:
* Ensure the PS4 controller is fully charged before starting
* Start the bluetoothctl utility with `sudo bluetoothctl`
* Run `agent on`
* Put the PS4 controller into pairing mode by holding down the "share" and "PS" buttons until the light flashes rapidly. Once the light is flashing rapidly, the following steps must be done before the flashing stops.
* Run `scan on`
* Wait until a device called "wireless controller" appears in the list of detected devices. Then run `scan off`.
* Copy the MAC address of the wireless controller. It should be something like "70:20:84:5E:88:B5".
* Run `trust 70:20:84:5E:88:B5`, but put in the MAC address of your controller
* Run `connect 70:20:84:5E:88:B5`, but put in the MAC address of your controller
* At this point, the LED on the controller should have turned solid blue
* Close bluetoothctl with `exit`

From now your controller should auto pair each time you press the "PS" button.

### Default Controller Buttons

Once you've paired the PS4 controller to the Jetson Xavier NX and made sure that Micro-Ros is connected to the drivers, you can start moving the Jackal robot from the controller.

The speed control will be determined thanks to the left joystick of the controller (in the image below it can be seen as the K element of the diagram).

![image](assets/ps4_controller_front.jpg)

By design, the program has a security lock so it doesn't move with only the joystick, so you'll need to press R1 or L1 buttons (The D and A elements on the image below).

L1 button is for a lower speed, while R1 is used for a higher speed. You'll need to press either R1 or L1 button, and move the Left Stick at the same time for moving.

![image2](assets/ps4_controller_side.jpg)

## GraphView

![GraphView](assets/graph.jpg)

There is a problem to visualize `rqt_graph` since the nodes are launched at boot and the graph will only show nodes created by the user and not root. If you want to view the graph please run:

```bash
systemctl stop launch_teleop_jackal.service
ros2 launch jackal_control_humble jackal_control.launch.py
```

## Contact

For questions or support, please open an issue on the [GitHub repository](https://github.com/PaoloReyes/Jackal-Jetson-Xavier-NX-SDK/issues) or contact the project maintainers at 
- [paolo.alfonso.reyes@gmail.com](mailto:paolo.alfonso.reyes@gmail.com)

-  [ricardo.navgom@gmail.com](mailto:ricardo.navgom@gmail.com)
---
