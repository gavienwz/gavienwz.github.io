Ex---
layout: post
title:  "Embedded Linux"
date:   2025-05-05 16:21:46 +0800
categories: Embedded
---
## About
Embedded Linux is a lightweight, customizable version of the Linux operating system designed to run on embedded systems—devices built for specific, often resource-constrained tasks. It uses the Linux kernel but strips away unnecessary components, allowing developers to tailor the OS to their hardware and application needs.

Unlike general-purpose Linux distributions (like Ubuntu or Fedora), Embedded Linux is usually built from scratch or semi-automated build systems like OpenWrt, Buildroot, or Yocto.

## Basic Components of Embedded Linux
Bootloader: A Bootloader starts the system and loads the kernel. It can directly initialise the hardware, then loading the application firmware.
Kernel: Manages hardware and system resources.
Root Filesystem: Contains basic tools (BusyBox, shell, config files). System of files
Init System: Starts services and applications (Simple init or systemd).

# Bootloader
A bootloader is a small piece of code that runs immediately after the CPU resets/on power-up. It will initialise basic hardware (RAM, Clocks, Flash, Network stack), and load and start the operating system.

Almost all modern CPUs and MCUs have an internal boot ROM (Read Only Memory) that acts as a First-Stage bootloader. Depending on the application and necessity, a Second-Stage Bootloader (Eg. U-Boot) is required to initialise other hardware peripherals to load the application firmware. Examples of other hardware that are initialised are: RAM, External Flash, SD/eMMC, Ethernet/ Other network peripherals, for various uses.

The next common step taken after initialisation of these hardware peripherals is: Loading of Device Tree (DTB), Linux Kernel, Root Filesystem, while setting up environment variables and boot arguments to be passed to the kernel. Control is then handed over to the Linux Kernel.

# Device Tree (DTB)
A device tree/devicetree is a data structure that describe the hardware components of the embedded system. This is so that the kernel, after parsing the devicetree, can use and manage these components.

The device tree, as it name implies, is a tree that consists of nodes, while each node can contain properties and child nodes. Device trees have both a binary format for Operating Systems (OS) to use, and a text format for developer editing and management. Common device trees can be found in the Linux Kernel Source Tree (https://github.com/torvalds/linux/tree/master/arch/arm/boot/dts).

Nodes represent the components that make up the system, whether it is the CPU, memory, bus, etc. The basic properties of every node are: Compatible string, Register Address and Size, Status.

```i2c1: i2c@40005400 {
    compatible = "fsl,imx6ul-i2c";
    reg = <0x40005400 0x1000>;
    clock-frequency = <100000>;
    status = "okay";

    temperature-sensor@48 {
        compatible = "ti,tmp102";
        reg = <0x48>;
    };
};```

The code chunk above showcases an example I2C node (Not actual code). The I2C bus is implemented with FreeScale's IMX6UL I2C, with its corresponding registers being at 0x40005400 with size 0x1000 (of presumably the CPU), with a clock frequency of 100KHz. The I2C is enabled with `status = "okay"`.
The I2C node has a child node, a temperature sensor, located on the I2C bus. The sensor is a Texas Instruments, TMP102 sensor, with the register address of 0x48.

To determine what kind of properties a node should and can have. We look at the device tree bindings (https://github.com/torvalds/linux/blob/master/Documentation/devicetree/bindings/).

Device Tree bindings are .yaml files that are formal documentations on how to implement the node and its properties. Looking at i2c-imx.yaml which is the device tree binding documenting "fsl,imx6ul-i2c" we can look at the properties of the node. 

The "compatible" property shows the required string that the node would need if it were use this IMX I2C bus. While 'fsl,imx6ul-i2c' is listed, other compatible strings are listed as well, showing that each of these chips use the same i2c bus (although there can be different implementations).

The "required" property shows the properties that the node in the device tree MUST have for the node to be valid, else the Device Tree Compiler (DTC) will throw an error.

Lastly, device tree bindings usually will include an example of the node.

While Device tree bindings documents nodes, Device Tree Source (DTS) describes embedded systems. Common file types are .dts and .dtsi. DTS files with the included DTSI files are then compiled using the DTC to output DTB files. Which are the binary formatted files which the kernel parses to further initialise hardware.

