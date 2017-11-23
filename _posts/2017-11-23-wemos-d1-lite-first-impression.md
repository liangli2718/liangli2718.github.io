---
layout: post
title: WEMOS D1 Lite (ESP8285) First Impression
categories: [blog ]
tags: [Arduino, ESP8266]
description: Small, stackable, yet powerful.
---

# Intro
Recently I got some cheap Arduino compatible modules from [aliexpress](https://www.aliexpress.com/item/WEMOS-D1-mini-Lite-V1-0-0-WIFI-Internet-of-Things-development-board-based-ESP8285-1MB/32795857574.html?spm=a2g0s.9042311.0.0.WzDIGy). The main attraction is its size and its ability to stack also pico-sized extension modules. The second part is inherited from the original Arduino board, however, the originals are about 5-8 times larger than the WEMOS solution.

# The Boards
So this is what I've got:

## WEMOS D1 Lite
![D1](img/2017-11-23/d1-lite.jpg)

This is the core board with ESP8285 chip on it. The ESP8285 is newer, more-integrated version of ESP8266 and it has an on-chip 1MB flash. This eliminates the need of having a discrete Flash chip on the PCB meaning a cleaner layout and less BOM. There is another version (WEMOS D1 mini) which is equipped with a ESP8266 and a discrete Flash memory.

## Battery Shield
![battery_mgr](img/2017-11-23/battery-mgr.jpg)

This board is an extension board that can be stacked on the D1 Lite (or D1 Mini). It features a TP5410 power management chip which allows charging a standard 3.7V lithium-ion battery from its USB port. It also generates 5V output for the D1 Lite and other stacked boards by its boost circuit.
This shield makes it easy to add a rechargeable battery to a project.

## Temperature & Humidity Sensor
![t_h_sensor](img/2017-11-23/sht-30.jpg)

This extension board has a SHT-30 sensor chip on it which can measure the temperature and humidity with typical accuracies of ±0.3°C and ±3%RH. Nothing special, but just another stackable solution.

## OLED Shield
![oled](img/2017-11-23/oled.jpg)

This shield provide a monochromatic 0.66 inch OLED screen with 2 extra push buttons on each side. The 0.66" areas includes 64x48 pixels.
The OLED is controlled by SSD1306 driver IC and connected via I2C to the core board.

# Size Comparison
