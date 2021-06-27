---
layout: default
title: "RFM69 Temperature Sensor Node: Experiment 2 - Packet Loss"
---

# Introduction

In the last experiment, I discussed that I was getting quite a large amount of packets being lost. In this experiment, I'm going to look into this further and see if I can get to the bottom of it, and hopefully reduce the amount of packets being lost.
	
For this, I'm going to update the sensing node so that it sends the amount of packets that were lost on each attempt. The gateway node will then print out this information so that we can get some idea of the amount of packets that are actually being lost (the first column is the packets lost in the previous transmission attempt).

# Packet Loss Experiments

For this, I'm sending the number of packets lost (technically, I'm actually sending the number of retries), the power level, and the RSSI value. I'm doing all of the experiments over 24 hours, as the RF noise levels will change throughout the day, and this will give a good understanding of where things are. It will also give a better amount of data for the packet loss to be averaged over, increasing the chance of the value being statistically significant.

The data was sent from the sensor node once a second, again to give more data for checking.

## Default settings

For this, I'm leaving everything to the default from the RFM69 Arduino Library. I'm expecting this to be the worst result, but it will be interesting to see.

Some interesting things to note:
 * You can really tell when my computer is on. At the moment, the sensor node is placed under my computer monitor for easy testing. Every-time I switch on my PC, there's a clear spike in the packet loss, an increase in the TX power level, and a reduction in the RSSI.
 
![Default library settings](/assets/images/2021/06/13/default-setup-packet-loss-screenshot.jpg)

As can be seen, this is pretty bad, with about 50% of all packets being lost.

## Whitening On

I was looking into packet loss online, and came up with the following forum post about a similar issue [here](https://lowpowerlab.com/forum/moteino/is-packet-loss-common/30/). The suggested improvements involved changing REG_PACKETCONFIG1 from RF_PACKET1_DCFREE_OFF to RF_PACKET1_DCFREE_WHITENING.

As can be seen, things seem to be a little bit worse, and a little bit more variable.

![Whitening On](/assets/images/2021/06/13/Whitening-on-Screenshot_2021-05-21 Overview - Home Assistant.png)

## Changing Lowest RSSI Level to -70

Since I've added the ATC implementation from the Arduino library, I'm able to set a minimum RSSI level, and the power should be automatically adjusted to try and keep above the level. As with the Arduino library examples, I've initially set this value to -80, as this is generally seen as a good low value. However, increasing the base level should help reduce the amount of signal loss. I've now changed the base RSSI value from -80 to -70, and this has had a marked improvement, with the packet loss closer to 5%

![Base RSSI changed to -70](/assets/images/2021/06/13/RSSI-min-70-Screenshot_2021-05-22 Overview - Home Assistant.png)

## Changing Bit Rate

With the RFM69 library, it's possible to change the bit-rate of the transmitted signal. It makes sense that a slower bit-rate would reduce the number of missed packets, though there would be a cost for that, which I'll discuss in a bit.

Using values from the MySensor RFM69 library, I tried a high, low, and medium bit-rate to see how this affects packet loss. By using the MySensor values, I could be more sure that they are correct, rather than having to try and figure them out myself.

### High Bit Rate

From the MySensor library, the setup parameters are:

 * RFM69_xxx_BR250_FD250 	FSK/GFSK/OOK 	250000 	250000 	111_16_0 	Whitening

As can be seen below, the packet loss rate went up a lot! Therefore, from a pure packet loss point of view, the high bit rate is a bad idea.

![High Bit Rate](/assets/images/2021/06/13/rfm69_xxx_br250_fd250-Screenshot_2021-05-26 Overview - Home Assistant.png)

### Low Bit Rate

From the MySensor library, the setup parameters are:

 * RFM69_xxx_BR2_4_FD4_8 	FSK/GFSK/OOK 	2400 	4800 	111_24_4 	Whitening 
 
Wow. Reducing the bit-rate to the lowest values I could (from the MySensor library), I was able to reduce the average packet loss down to between 3% and 4%. This is much better than anything I've been able to get close to with other settings. However, as I'll discuss in a bit, I'll not be choosing these settings, because it does have a knock-on effect on the power consumption.

![Low Bit Rate](/assets/images/2021/06/13/br2_4_fd4_8_Screenshot_2021-05-27 Overview - Home Assistant.png)

### Back in the middle

From the MySensor library, the setup parameters for the middle bit-rate (which is similar to the Arduino library defaults) are:

 * RFM69_xxx_BR55_5_FD50 	FSK/GFSK/OOK 	55555 	50000 	111_16_2 	Whitening 
 
As can be seen below, the packet losses are somewhere between the high and low bit rate losses. However, they are better that what I was getting from the default Arduino library settings.

![Middle Bit Rate](/assets/images/2021/06/13/rfm69_xxx-b955_5_fd50-Screenshot 2021-06-09 at 08-28-23 Overview - Home Assistant.png)

### Power Consumption

Although I won't go into detail here, using the low bit rate isn't the best choice for low power consumption. That's because the low bit rate means a high transmission and receive time. The lower the bit-rate, the longer it takes to send the signal, and to wait for the ACK reply from the gateway. 

The transmission phase is the highest power consuming phase, with the receiving phase being second. As a result, you want to reduce the amount of time in these phases as much as possible, with the middle bit-rate probably being the compromise between the two. In a future post, I'll look into this in more detail, and show the potential power consumption for various bit-rates.

## Improving location and orientation

One thing that I've noticed a few times with this RF module, as well as with the NRF24 RF module, is that placement of both the sensor and gateway are very important. Therefore, I tried moving both the sensor, and then the gateway to see if this would make any obvious change.

### Sensor node

First I moved the sensor node. Specifically, I moved it away for my work bench, and away from any obvious electronic devices, though still within the same room.

As can be seen below, this had a smallish improvement on the packet loss, taking it down to about 18% after 24 hours. It looks like it would have improved further if I had left it longer, though this might not actually be the case.

![Moving Sensor Node](/assets/images/2021/06/13/Better-location-1-Screenshot 2021-06-11 at 07-54-22 Overview - Home Assistant.png)

### Gateway

Next, I moved the gateway sensor. Again, I moved it away from any other electronic devices that might be causing interference. I also straightened up the antenna (which at this point is just a simple wire of correct length).

As can be seen below, this makes a huge difference in packet loss, taking to down to about 4%. This make a much bigger difference than I thought it would. In fact, it seems to make more difference that all of the previous changes combined, making me think that the initial position one of the reasons for the initially poor packet loss.

![Moving Gateway Node](/assets/images/2021/06/13/better_gateway_position-Screenshot 2021-06-12 at 12-53-50 Overview - Home Assistant.png)

## Coiled antenna on gateway

The final change in this experiment is changing the antenna on the gateway node. For the previous tests, the gateway node had a simple wire of the correct length acting as the antenna. This time, I changed it to a pre-wound coil, which takes up less room, and might be better for a final product.

This did improve the packet loss, but not by a huge amount. However, since it does take up less space, it's probably worth doing.

![Changing Gateway Antenna](/assets/images/2021/06/13/coil-areal-on-gateway-Screenshot 2021-06-13 at 13-51-20 Overview - Home Assistant.png)

# Conclusion and Next Steps

So far, the biggest improvements in packet loss have come from changing the location of the gateway. I would improve the packet loss further by reducing the bit rate, but this would increase the power consumption by having the sensor node transmitting for longer than is needed. Using the results above, I'm able to improve the packet losses from about 50%, down to about 3.5%, potentially improving the overall power consumption of the sensor node.

The next steps in this series is to determine the best compromise between the bit-rate, packet loss, and power consumption. This one will be a bit more difficult as I'll have to start measuring the current draw, or at the very least, calculate the current draw based on information available in the data-sheet.

