# Manual GCC Build, Flash, and Debug Workflow for STM32F401RE
A command-line guide for manually building an ELF file, flashing an STM32 target, and debugging (without an IDE) using GNU ARM Embedded toolchain. The workflow creates toolchain awareness that transfers directly to Zephyr development.
=============================

## References
This repository is a fork of metabr's template                              (https://github.com/metabr/stm32-nucleo-f401re-basic-template.git)
The following video helps putting the pieces together                       (https://www.youtube.com/watch?v=-p26X8lTAvo)

## Step 1: Toolchain
- VSC and its extensions as shown in the video
    - C/C++
    - CMake
    - Cortex Debug
    - Makefile Tools
- xPack OpenOCD and GNU ARM Embedded toolchain
- MSYS2 and its packages (e.g., make, python3)
    - Add the installed gcc-arm-none-eabi /bin folder to the MSYS2 path     (export PATH="$PATH:yourOwnPath")

## Step 2: makefile
- The libopencm3-examples repository used in the video is not useful to this guide, and is the reason why this repository exists. However, that repository contains a wide variety of examples spanning different MCU vendors and families, and is worth considering when looking for a template for a different device.
- With MSYS2, sit on the root directory of this repository (the project to be debugged) and type the command "make". This will generate all relevant files to flash our MCU.
    - .elf
    - .bin
    - .hex

## Step 3: Debug via VSC extension
- VSC Run and Debug > create a launch.json file > Cortex Debug > Add Configuration > Cortex Debug: OpenOCD > Delete the existing configuration
- Change the path of the .elf
- Modify the configFiles parameters
    "configFiles": [
    "interface/stlink.cfg",
    "target/stm32f4x.cfg"
    ]
- Follow the Youtube video from minute 24:00 to set the required settings.json. These paths are relative to the installation of both arm-none-eabi-gdb and OpenOCD in your computer.

## Step 4: Upload the binary (.elf)!
Once all the steps are followed correctly, and the directory paths point to the right pc locations, we can press "Start Debugging (F5)" in VSC. This will upload the .elf and allow us granular control, debugging line by line and seeing our internal LED blink repeteadly.

If you would like to modify the code and re-build, simply open MSYS2, sit on the project directory, and use the command "make clean all". This will build the new .elf file!