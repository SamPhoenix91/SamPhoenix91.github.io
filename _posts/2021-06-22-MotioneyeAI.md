---
title: Improve Motioneye Motion Detection Using DeepStack AI and Home Assistant
date: 2021-06-22 12:00:00 +/-0
categories: [Smart Home, Cameras]
tags: [smart home, ai, cameras, cctv, motioneye, deepstack, homeassistant]     # TAG names should always be lowercase
img_path: /assets/posts/MotioneyeAI
---

# Introduction

I have been searching for a Linux alternative to the [Blue Iris AI motion events](https://www.youtube.com/watch?v=fwoonl5JKgo) for what seems like forever. Since Home Assistant added image processing, some people have been using it to create notification using Deepstack and NOOBS AI. I thought this was a step in the right direction into improving Motioneye motion recordings, so I thought I’d take the extra step and relay the results of a scan back into motioneye, to remove recordings that have not been triggered by a person or vehicle.

*Anything indented is code to be run in the console*

# Things you need:

Software

[Home Assistant](https://www.home-assistant.io/)

[HASS-Deepstack-object custom-component](https://github.com/robmarkcole/HASS-Deepstack-object)

[Mosquitto MQTT broker](https://mosquitto.org/)

[Motioneye](https://github.com/ccrisan/motioneye)

[Deepstack](https://deepstack.cc/)

[loknibor's Motioneye to MQTT](https://github.com/kloknibor/motioneye-to-mqtt/commits?author=kloknibor)

![Installing Deepstack as a docker container via portainer.](media/9a7a3b1c2b8184184879d311a33f989c.png)

Installing Deepstack as a docker container via portainer.

# Getting Started

## Home Assistant

If you havent already, make sure Home Assistant and Motioneye are setup. [You can install Home Assistant many ways](https://www.home-assistant.io/installation/). I personally have it installed as a LXC container on proxmox using [Whiskerz007 one-command install script](https://github.com/whiskerz007/proxmox_hassio_lxc).

## Motioneye

If you are running a supervised version of Home Assistant or HomeAssisantOS, Motioneye can be installed very easily as an addon in the supervisor tab. If you Home Assistant installed as a docker container, you can run Motioneye in a [different container alongside it.](https://github.com/ccrisan/motioneye/wiki/Install-In-Docker)

## Mosquitto MQTT

MQTT is how we are going to make sure Home Assistant has the right image to process, As I already have these cameras integrated into Home Assistant and they do not always sync up the image pre-view. HA supports MQTT cameras, so we will send two MQTT messages, an “on” payload for a binary sensor, to tell Home Assistant that the image has changed and the image itself. This can again be installed using the addons tab, or as a [docker container](https://hub.docker.com/_/eclipse-mosquitto).

## Deepstack

Deepstack is what we will use to process the images. It can be easily installed [as a docker container](https://www.deepstack.cc/install/docker), but is not available as a Home Assistant addon. To get around this, you can add the “Portainer” addon, and as Home Assistant Supervisor uses docker as a back-end, you can install it using portainer as a normal addon.

**I would recommend backing up at this point as portainer can break your Home Assistant install if you do not know what you’re doing**

Using [this link](https://www.deepstack.cc/install/docker) look for step 3, for the variables. You can use t[his portion of this video](https://youtu.be/fwoonl5JKgo?t=782) (up to 14:22) for direct instructions if you are unsure of how to convert a docker command into portainer.

**Make sure all your addons/docker containers are running and will restart on reboot.**

![Portainer volume](media/745b609a23c196c55a66aef10475d22c.jpeg)

Portainer volume

![](media/828a45648ab76004ef67f238476cc531.png)

Config File

![](media/49a41c91174d7835ab6e4d61db6d9bf9.png)

Successful Test

# Installing MQTT Script

Now we will get into installing the MQTT script for motioneye. We could just configure a simple script to send an MQTT message and get Home Assistant to use the image from the stream but it pulls the image every few seconds so may not have the image that triggered motion. To combat this, I have made a more advanced script that will send the image that triggered motion alongside the message to toggle the sensor.

In portainer, create a volume for motioneye that goes to /var/lib motioneye to somewhere on the host system. I have made it go to /motioneye on the host system (refer to the image on the left) and hit deploy.

It's good practice to always update and upgrade linux before doing anything to avoid errors.

```
    sudo apt update

    sudo apt upgrade
```

Using the host system (or in the terminal addon if running on Home Assistant) run the following commands

```
    cd /motioneye
```

We’re now in the motioneye folder, grab my code from github and unpack it.

```
    wget https://github.com/SamPhoenix91/MotioneyeMQTT/releases/download/0.1/MotioneyeMQTT.tar.gz

    tar -xf MotioneyeMQTT.tar.gz
```

You will now have two new folders: “OnMotionScipt” and “ResultsProcessScript” and one new file “config.yaml”

You can clean-up, by removing the MotioneyeMQTT.tar.gz

Go into the script and edit the configuration section of the script to include your MQTT broker details.

```
    sudo nano config.yaml
```

You can test the script and it will generate a log, If you do not see “Connection Successful” the script was unable to connect to your MQTT broker.

```
    OnMotionScript/OnMotion-MotioneyeMQTT/OnMotion-MotioneyeMQTT TEST TEST

    cat log1.txt
```

# Sending Commands on Motion

Open your motioneye instance and add your camera(s) if you haven’t already.  
Set up your motion detection, along with a snapshots and movies recording on motion as you wish (I recommend both an image and motion recording - with a limit around 2 mins of recording which I will explain later). You also want to air your motion detection on the more sensitive side, as the point is we will remove videos that we don’t need.

Scroll down to “notion Notification” and turn on “Run a command” and enter the following command (Replace “Front_Door” with the exact name of your camera - spaces turn into \_)

```
    python3 MotioneyeMQTT.py ON Front_Door
```

For the end command, do the same but replace “ON” with “OFF”

```
    python3 MotioneyeMQTT.py OFF Front_Door
```

Copy this to each of your cameras, replacing the camera name appropriately and Motioneye will now publish motion states over MQTT, you can check this by using a chrome addon like MQTTLens or software like MQTT Explorer

![Graphical user interface, application Description automatically generated](media/889a681b900683284391cf585dd055c8.png)

# Home Assistant Addon

On Home Assistant, either using the terminal like before, or via samba or any other way, put the [HASS-Deepstack-object](https://github.com/robmarkcole/HASS-Deepstack-object/releases) custom component in the custom-components folder in the Home Assistant main directory and restart Home Assistant.

After Home Assistant restarts, you want to edit the configuration.yaml file. We will be creating 3 types of entities for each camera; image processor, binary sensor, camera.

Here is a [pastebin](https://pastebin.com/D4z0iaFz) of what should be entered in configuration.yaml

**You need to make sure that the MQTT topics are named the same as the Motioneye camera (i.e. My Motioneye camera is called Front_Door so my MQTT camera must be configured for “motioncameras/Front_Door/picture”**

Check the yaml and restart Home Assistant.

When it restarts you should have three new entities.

![Graphical user interface, application Description automatically generated](media/b4f344aed7fb6b75fca85ff7a546137a.png)

![configuration.yaml](media/23a9bb40e071d2236b53d135a0780582.png)

configuration.yaml

# Running the Deepstack scan

The deepstack does not automatically run a scan on the camera images, which is the reason we needed the binary sensor. We need to set up an automation to call the scan service when a new motion image comes through.

Go to settings and automations, add a new automation.

Set the trigger to the binary sensor state and the action to the image processing service

![Graphical user interface, application Description automatically generated](media/cfe318df98e6c413a475500e32e6624e.png)

![Graphical user interface, text, application, chat or text message Description automatically generated](media/52b6dad833c9a3a2d4b846f5fd67bb94.png)

# Returning the Scan Back to Motioneye

Now that we have scanned the image, we need to get the information back so we can either keep or delete the recordings.

We will set up new automations that send an MQTT result back to a script that will delete files if there was no person or vehicle detected, freeing up space of mis-triggering motion events.

We just want a state change as the trigger, as we will do the logic when deciding what MQTT payload to send.

In actions, we will first wait for the binary sensor to turn off, so we know Motioneye has stopped recording the video. I will also add a timeout, so if something goes wrong, no payload will be sent and the files will not be deleted as a fail safe - we would still rather have videos with mis-triggered motion than remove an important video. The timeout should be ever so slightly longer than the video limit, so it does not fail to send a payload because of a long video.  
Next we will add a condition, one side will have a numeric state above 0, and the other with a state of 0, we will then use the MQTT publish service to publish “YES” and “NO” payloads to a topic “motioncameras/[camera]/result”, respectively.
