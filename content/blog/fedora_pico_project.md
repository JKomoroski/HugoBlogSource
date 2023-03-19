---
title: "Setting Up Pico Development in Fedora"
date: 2021-02-16T22:57:00-07:00
tags: [rust, embedded, raspberry-pi-pico]
draft: false
---
# Setting Up Pico Development in Fedora

Things to install:

```
dnf install gcc gcc-c++ cmake arm-none-eabi-*
```
Clone the following repositories:
```
git clone https://github.com/raspberrypi/pico-examples.git
git clone https://github.com/raspberrypi/pico-sdk.git
```

Export the following variable to point to the pico sdk location:
```
export PICO_SDK_PATH=/home/MY_USERNAME/path/to/pico-sdk
```

Make a new Development Directory:
```
mkdir my_pico_dir
cd my_pico_dir
```

Copy simple example from the examples and sdk repositories:
```
cp /home/MY_USERNAME/path/to/pico-examples/blink/blink.c .
cp "${PICO_SDK_PATH}/external/pico_sdk_import.cmake" .
```

Create a cmake file:
```
cat <<EOF > CMakeLists.txt
cmake_minimum_required(VERSION 3.13)

include(pico_sdk_import.cmake)

project(blink)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

pico_sdk_init()

add_executable(blink
    blink.c
)

target_link_libraries(blink pico_stdlib)

pico_add_extra_outputs(blink)

EOF
```

Make a build directory for cmake and initialize cmake:
```
mkdir build
cd build
cmake ..
```

Build the blink project:
```
make blink
```

Copy the blink uf2 output to the pico:
```
cp blink.uf2 /run/media/MY_USER_NAME/RPI-RP2
```
