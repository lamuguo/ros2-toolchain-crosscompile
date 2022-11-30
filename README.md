# ROS2 toolchain cross-compiler

This is a very minimal pass at a workflow that can use a custom toolchain to create a ROS2 build.

It cannot handle dependencies that are outside the source tree, so there is only a limited subset of ROS2 that this can build.

As of right now, it is specifically tuned to using a linaro aarch64 cross compiler, but to make it generic should not take much modification.

# Main Process

## Setup

You need to have Docker and `vcs` installed on your dev computer already

1. Download the linaro cross compiler (6.5)
    * https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz
    * extract it fully into this repo and change the directory's name to `gcc-linaro-armhf/`
1. Download the linaro sysroot (6.5)
    *  https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/sysroot-glibc-linaro-2.25-2019.12-arm-linux-gnueabihf.tar.xz
    *  extract it fully into this repo and change the directory's name to `sysroot-linaro7.5`
1. Get the ROS2 sources

```
vcs import src < minimal_cpp_ros2_master.repos
# we do not have log4cxx dependency and don't actually need it
touch src/ros2/rcl_logging/rcl_logging_log4cxx/COLCON_IGNORE
```

## Running the build
Rebuild the docker image, and start the container.
```
cd docker
docker build -t ros2_cross_compiler .
cd ..
docker run -it -v $(pwd):/ws/ros2build ros2_cross_compiler
```

Now run the build within the container

```
colcon build --mixin aarch64-linux --packages-up-to demo_nodes_cpp
```

## Deploying the build

Exit the build container. The `install/` directory is your ROS2 build, cp the whole folder to the target machine:

```
scp -r install <username>@<host>:~/install
```

## Try the build
Go to the target machine, open a terminal and run:
```
export LD_LIBRARY_PATH=~/install/lib
~/install/lib/demo_nodes_cpp/talker
```

Open another terminal in the target machine:
```
export LD_LIBRARY_PATH=~/install/lib
~/install/lib/demo_nodes_cpp/listener
```

Now, you should see that the listener is receiving messages from the talker.

# Tips / Advice

## Rebuilding the Docker image

If you need to change the docker image that we use for building, it's easy to rebuild

```
cd docker
docker build . -t candleends/ros2_cross_compiler
```
