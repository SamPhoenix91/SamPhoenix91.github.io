---
title: Creating a Locally Controlled Thermostat
date: 2022-12-07 12:00:00 +/-0
categories: [Smart Home, Heating]
tags: [smart home, thermostat, heating, climate, local, homeassistant]     # TAG names should always be lowercase
---

# Introduction

Despite having 200,000 Home Assistant users (as of today), opting for local, cloud-free control of their devices; There seems to be no smart home thermostats that offer completely local control.

I can understand why these are not in the mainstream – Without a cloud component, accessing a generic thermostat outside the network would be a pain for users.

From my research the general consensus on the forums is to go with Ecobee, as while they do use the cloud, thanks to HomeKit (And Home Assistant’s hacky HomeKit integration), are able to be controlled locally.

Smart Thermostats are expensive though, and with Ecobee for \$200 a pop, plus not being available to get them in the UK without using an international shipping company - Adding further price; Paying that much for something you are going to ignore 80% of the features of didn’t cut it for me.

There are a handful of Zigbee and Z-Wave thermostats out there, but no one seems to be rallying around them and I cannot find a reliable source for a cheap, easy to use thermostat for Home Assistant.

# The Components

I had three requirements for the components,

Microcontroller  
In the prototype I opted for D1 Minis (ESP8266) and ESP32s as I had a few lying about, but the final design is for [D1 Mini ESP-32](https://www.amazon.co.uk/AZDelivery-ESP-WROOM-32-Bluetooth-Development-Compatible/dp/B08BTLYSTM/ref=sr_1_1?crid=33QD36GQ4JGJX&keywords=amazon+D1+Mini+esp-32&qid=1670450734&sprefix=amazon+d1+mini+esp-32%2Caps%2C85&sr=8-1) for their small size, power and ability to use Bluetooth for tracking as well.

### Readout

Looking around, [SSD1306 OLED Screens](https://www.amazon.co.uk/gp/product/B074N9VLZX/ref=ppx_yo_dt_b_asin_title_o00_s01?ie=UTF8&psc=1) seem to be the go to for ESP projects and they seemed fairly easy to use to simply display some text and symbols on the screen.

### Dial

I opted for Rotary Encoders as the dial method of choice. The ones I chose have a pushbutton, which will be wired up for easy on/off or mode switcher control of the thermostat

### Temperature Sensor

The prototype used DHT11’s as I stupidly ordered the wrong ones, by the final version is designed for [DHT22](https://www.amazon.co.uk/AOICRIE-Digital-Temperature-Humidity-Monitor/dp/B08XTC8CQX/ref=sr_1_13?crid=RFFBZO1OITJV&keywords=DHT22&qid=1670451128&sprefix=dht22%2Caps%2C100&sr=8-13) for measurements of 1 decimal point with an accuracy of ±0.5.

# The Software
