# Robocup 2021 World

## Description:

This repository contains the necessary packages and resources to run a simulated Gazebo World with the robot Tiago.

You have two methods to work with it:

## LXD Container

* Firt of all, you need to have installed LXD. You can do it with ``sudo snap install lxd``.

* Now, you have to download [the release](https://github.com/fgonzalezr1998/robocup2021_world/releases/tag/1.0). This is a *tar.gz* file.

* Import the image from the downloaded file:

  ```
  lxc image import 02df40d1d65eeb373173801aa94d171324970f021f23c6f31a38df7cb36341c8.tar.gz --alias rc2021world
  ```

* Launch the lxd container from the imported image:

  ```
  lxc launch rc2021world robocup2021world
  ```

Now, you should have the container running. Check it typing ``lxc list``

### Change IP address

We need to change the container IP address because it should take part of the same subnet of the host.

* Create a new bridge interface:

  ```
  lxc network create lxdbr1
  ```

* Change the subnet of this new bridge:

  ```
  lxc network edit lxdbr1
  ```

  Locate the next line: ``ipv4.address: 10.72.119.1/24`` and change it by ``ipv4.address: 192.168.1.240/24``

  > NOTE: It is assumed that the host subnet is of type 192.168.1.X... If it is not of this way, put an IP that take part of your subnet. For example, if you PC have the IP 192.168.0.33, you must put 192.168.0.240/24

* Now, we are going to attach our container to this new bridged interface and change the IP of the container:

  ```
  lxc stop robocup2021world
  lxc network attach lxdbr1 robocup2021world eth0 eth0
  lxc config device set robocup2021world eth0 ipv4.address 192.168.1.241
  ```

* Also, is important to run the next command. Otherwise, ROS1 bridges will not can be launched:

  ```
  lxc config set robocup2021world security.nesting=true
  ```

* At this moment, we can start our container:

  ```
  lxc start robocup2021world
  ```

* And enter to it:

  ```
  lxc exec robocup2021world -- su --login ubuntu
  ```

  > NOTE: I recommend to create an alias for this command

Now, you can check that you have a connection with the host from the container and vice versa

* The last step is **VERY IMPORTANT**. You must to add the container IP address to your */etc/hosts* file **in the host machine**. So type ``sudo nano /etc/hosts`` and write in a new line exactly this:

  ```
  192.168.1.241 robocup2021world
  ```

### Run the Simulation

From the *robocup2021world* container, run the next command:

```
roslaunch tiago_sim_robocup2021 tiago_sim_robocup2021.launch
```

It should run the GUI gazebo interface with the world. To run the simulation you have to click the start button on Gazebo.

---

#### From ROS1

If everything was fine, in your host machine you should can see all topics. Try to publish in one of them from ROS1. For example:

```
rostopic pub -r 3 /mobile_base_controller/cmd_vel geometry_ms/Twist "linear:
  x: 0.0
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.2"
```

The robot should move!
