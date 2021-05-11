---
layout: default
title: "RFM69 Temperature Sensor: Experiment 1"
---

# Introduction

For a while now, I've been using the NRF24L0 as the RF module in my wireless temperature sensors. However, like many others, I'm having issues getting a reliable connection throughout my home. Therefore, I've been working on using the RFM69 module to see if it would be a viable alternative.

This is the first in a series of experiments in using the RFM69 module.

# Hardware

As already mentioned, I'm using the RFM69 module. To be more specific, I'm using the 433MHz version, as that's what I had lying around. 

I'm actually using two different RFM69 modules, a generic version for the sensor node, and the Adafruit RFM69 Bonnet as the "gateway". The reason for using the Bonnet for the gateway, is that it allows me to connect it to a Raspberry Pi (Zero W in this case), and have the gateway send data about the sensor data directly to my HomeAssitant server via MQTT.

For the sensor node IC, I'm using the STM32L010F4. Being part of STMs L range, it's designed for low power consumption, which will be useful here. Also, given the current IC shortage (May 2021), I happen to have a number of these already, which means I'm able to experiment without having to wait months for new ICs to be available!

For the actual temperature sensing, I'm using the MCP9808 I2C sensor. I've used this is all my temperature sensing nodes as it's easy to setup, and works really well. I'm currently using the Adafruit MCP9808 module board, as I happen to have one around, it is easier to play with as it breaks out all the required pins, and contains all of the required extra parts such as debouncing caps and pull-up resistors.

The schematic for this is pretty simple, as can be seen below.


For flashing and debugging, I'm using the Segger Edu Mini rather than an ST-Link V2. This allows me to use Segger's OZone debugging software, which I've used a number of times now and find it to be much better than debugging using the STM32CubeIDE. I'm in the process of learning GDB, so maybe I won't need that in the future, but for now I quite like the setup.

For this initial stage of the setup and experimentation, I'm using wire-wrap to connect everything together. I'm quite new to wire-wrap, but so far I do like it over breadboard or veroboard. At some point I will design a PCB that I'll get made and then solder it up (I do enjoy soldering, especially with small SMD parts, so I look forward to making this getting hold of the PCB in the near future).

Below you can see my wire-wrap. It's messy, but it works.

![Wire-Wrap Top](images/wire-wrap-top.jpg)
![Wire-Wrap Bottom](images/wire-wrap-bottom.jpg)

# Software

## Sensor Node

For the RFM69 module, I'm using a C port of Felix Rusu's RFM69 Library ([https://github.com/LowPowerLab/RFM69](https://github.com/LowPowerLab/RFM69)). This is a very good library, but it uses C++ as it's designed to work with Arduino boards. Luckily, there's very little C++ magic going on here, so it was relatively easy to get the base RFM69 module, along with the ATC extension ported over to C.

The ATC is useful because it will automatically adjust the transmission power based on the RSSI and if transmissions get missed.

This is a work in progress, but you can find the code over at the Github repository [here](https://github.com/sdaglish/rfm69_temperature_sensor_node)

I'm currently not using any encryption, though that will be added at a later date. 

I've also only added bits as I need them, so it's currently not a full port.

At this point, pretty much everything is defaulted to what it is in the Arduino Library, though I do intend to play with it in the near future to change it to my needs.

## Gateway

For the gateway, I'm using the Adafruit RFM69 Bonnet, connected to a Raspberry Pi Zero W.

On this, I'm running a simple Python script (actually, it's Python 2, as that was what I found [https://pypi.org/project/rpi-rfm69/](https://pypi.org/project/rpi-rfm69/)), I then modified this to use Paho MQTT client to send the received data to my Home Assistant setup using MQTT (the MQTT broker is on the same server as Home Assistant).

I'm currently sending: temperature (in degrees C), the RSSI value, and the transmit power setting. These are then being show on Home Assistant as shown below.

![Home Assistant Screenshot](images/home-assistant-screenshot.jpg)


# Results

I've tried an number of placing both the gateway and sensor node in various places around the house (including placing the sensor node in the detached brick garage), and I'm able to get a good strong signal from everywhere. This is much better than the NRF24L01 setup, where there were places in the house where I struggled to get a reliable signal from.

At the moment, when waiting between transmissions, no power-saving features have been added. As a result, the power draw is pretty high at 1.88mA. This is to be expected. In this setup, the node would last about 4 days and 15 hours, which isn't great.


It's interesting looking at the graphs from HA as you can see when I turned on my PC in the morning to start working. The gateway is near my office PC, so there's clearly some interference going on, which isn't all that surprising.

# Todo Next

One thing that I have noticed is that there seem to be quite a few packets being missed, causing the signal power to be increase, despite the RSSI being pretty low. Therefore, before working on the low-power modes, I'm going to look in reducing the amount of packets being lost.

I've also noticed that the RSSI is higher than the target RSSI, but the signal power level isn't 0. I'm going to look into that and see what's going on there.

I'm also slowly starting to get the hardware designed, but I'm going to put off designing the hardware as I'm sure I'll need to change the hardware when I look into low-power modes. So for now, I'll just keep to the wire-wrap board and adjust that as needed.
