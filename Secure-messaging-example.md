In order to build arm-examples using TrustZone, the following parameters should be given to *ament* in order to specify the path to the specific build of *Optee-OS* and *Optee-Client*.

## Reference board setup
You can find information on how to setup an HiKey 960 board on the arm community page:
https://community.arm.com/dev-platforms/w/docs/309/hikey-96-boards

## Trigger a build

```
optee_os_export=<Path to OPTEE-OS>/out/arm-plat-hikey/export-ta_arm64
optee_client_export=<Path to OPTEE-Client>/out/export

src/ament/ament_tools/scripts/ament.py build --force-cmake-configure \
        --cmake-args \
        -DCMAKE_TOOLCHAIN_FILE=`pwd`/aarch64_toolchainfile.cmake \
        -DTHIRDPARTY=ON \
        -DOPTEE_OS_EXPORT=$optee_os_export \
        -DOPTEE_CLIENT_EXPORT=$optee_client_export
```

Produced binaries and libraries will go under the following folders in ROS2:
- *./ros2/install/lib/aes_publisher_certify* and *./ros2/install/lib/aes_subscriber_certify* for the binary applications
- *./ros2/install/lib* for all the necessary libraries if ROS2 was Dynamically cross-compiled.
- *./ros2/install/lib/optee_ta_security* for the Trusted application.

## Setup the filesystem

All the binaries and libraries produced by ROS2 should be put in the filesystem of the target.
The Trusted Application should be stored into the following specific location: */lib/optee_armtz/*

The libraries and binaries produced by *Optee-Client* (*optee-client/out/export*) should be put in the filesystem.

## Launch the application

On the target, once Ubuntu has been boot up, setup *LD_LIBRARY_PATH* to point all directories containing the previously copied ROS2 and Optee-Client libraries.
```
export LD_LIBRARY_PATH="<Optee-Client>/out/export/lib:<ROS2>/install/lib"
```
Binary *tee-supplicant* must be started. It can be found into: *<Optee-Client>/out/export/bin*. Make sure you have the rights to execute.
```
<Optee-Client>/out/export/bin/tee-supplicant &
```
Start the applications by calling the corresponding binary:
```
<ROS2>/install/lib/aes_subscriber_certify
<ROS2>/install/lib/aes_publisher_certify
```

You should get the following result:
![](https://user-images.githubusercontent.com/38316392/38670613-2d382020-3e41-11e8-828b-0070c1547b78.png)