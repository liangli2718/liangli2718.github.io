---
layout: post
title: WEMOS D1 Mini Lite (ESP8285) First Impression
categories: [blog ]
tags: [Arduino, ESP8266]
description: Small, stackable, yet powerful.
---

# Intro
Recently I got some cheap Arduino compatible modules from [aliexpress](https://www.aliexpress.com/item/WEMOS-D1-mini-lite-V1-0-0-WIFI-Internet-of-Things-development-board-based-ESP8285-1MB/32795857574.html). The main attraction is its size and its ability to stack also pico-sized extension modules. The second part is inherited from the original Arduino board, however, the originals are about 5-8 times larger than the WEMOS solution.

# The Boards
So this is what I've got:

## WEMOS D1 Mini Lite
![D1](/img/2017-11-23/d1-lite.jpg)

This is the core board with ESP8285 chip on it. The ESP8285 is newer, more-integrated version of ESP8266 and it has an on-chip 1MB flash. This eliminates the need of having a discrete Flash chip on the PCB meaning a cleaner layout and less BOM. There is another version (WEMOS D1 mini) which is equipped with a ESP8266 and a discrete Flash memory.

The D1 Mini Lite pin out is exactly same as the D1 Mini, so (high probably) all extension boards that work with D1 Mini with work with D1 Mini Lite

## Battery Shield
![battery_mgr](/img/2017-11-23/battery-mgr.jpg)

This board is an extension board that can be stacked on the D1 Mini Lite (or D1 Mini). It features a TP5410 power management chip which allows charging a standard 3.7V lithium-ion battery from its USB port. It also generates 5V output for the D1 Mini Lite and other stacked boards by its boost circuit.
This shield makes it easy to add a rechargeable battery to a project.

## Temperature & Humidity Sensor
![t_h_sensor](/img/2017-11-23/sht-30.jpg)

This extension board has a SHT-30 sensor chip on it which can measure the temperature and humidity with typical accuracies of ±0.3°C and ±3%RH. Nothing special, but just another stackable solution.

## OLED Shield
![oled](/img/2017-11-23/oled.jpg)

This shield provide a monochromatic 0.66 inch OLED screen with 2 extra push buttons on each side. The 0.66" areas includes 64x48 pixels.
The OLED is controlled by SSD1306 driver IC and connected via I2C to the core board.

# Final Look and Size Comparison
>TODO: add photo

The above is the final look of the stack. From bottom to top are the battery shield, D1 Mini Lite core board, SHT-30 sensor and the OLED display. I didn't use the pluggable pin header because they occupy too much vertical space between boards which cancels out the benefit of this tiny stackable solution (also because I simply have too many ESP82xx board... so I don't need to make this one detachable that I can reuse them later.).

In the end, it becomes a full fledged sensor node in a cute cubic form factor. Just look at the image and you know how small it is compared to the original Arduino board or even the ESP8266 core board.

What is still to be added is a small lithium-ion battery on the bottom and a 3D printed housing which I will do in later post.

# Demo Application
To test the whole assembly, I used two applications. One is the stock example from [SparkFun OLED breakout](https://github.com/sparkfun/Micro_OLED_Breakout/tree/V_1.0/Libraries/Arduino/examples/MicroOLED_Cube).

>TODO: add gif

The other one is to test both the SHT-30 sensor and the OLED which is from the example of [WEMOS SHT3X library](https://github.com/wemos/WEMOS_SHT3x_Arduino_Library/tree/master/examples/SHT30_OLED_test).

I modified the code a bit to make it work with SparkFun lib.

>TODO: add gif

# Notes
When uploading program to the D1 Mini Lite, I noticed that I need to press down the reset button at pretty precise time just before the IDE shows "Uploading..." message otherwise the communication will fail.

To ease my fingers, I turned to use the other method to get the ESP8285 into program mode which is shorting D3 (marking on the board, or ESP8285 GPIO0) to ground.

Let's stop here, I will write more on this cute cube when I have more experience in using it in actual projects.

# References
- [ESP8285 news](https://hackaday.com/2016/06/21/espressif-releases-esp8266-killer/)
- [WEMOS wiki site](https://wiki.wemos.cc/start)
- [WEMOS github](https://github.com/wemos?tab=repositories)
- [Sparkfun OLED breakout github](https://github.com/sparkfun/Micro_OLED_Breakout/tree/V_1.0)




