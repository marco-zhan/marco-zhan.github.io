---
title:       "Home Server Setup with Proxmox"
subtitle:    ""
description: "Setting up my home server with proxmox"
date:        2023-08-02
author:      "Marco"
image:       ""
categories:  ["Tech"]
---

## Introduction
I have been wanting to setup a home server for a while, and finally got a chance to do it. 
I have been using a Raspberry Pi 4 stack as my home server for a while, but it is not powerful 
enough to run some of the services I want to run, also, my initial purpose to setup a home server was
to explore Kubernetes, scalability becomes an issue as I would like to run multiple nodes. So, instead of
buying another few Pis and setup a bigger cluster, I decided to get a proper server with some virtualization.
I have a Desktop PC under my table with Windows 10 installed, but I hardly use it, so I decided to wipe everything
and turn it into a home server.

I have been using Proxmox at work for a while, and I am quite familiar with it, so I decided to use it as my virtualization 
tool.  Proxmox is a Debian based virtualization tool, it is free and open source, and it is quite easy to use. 
There are other options like VMware, but I am not familiar with it, and I don't want to spend too much time on learning.

## Hardware
I have a Desktop PC with the following specs:
- CPU: Intel i9
- RAM: 64GB
- SSD: 2TB
- Motherboard: ASUS Motherboard with 2 network cards
- GPU: NVIDIA GeForce RTX 2080 Ti
- Power Supply: 750W

The most important thing here is the CPU and RAM, as I want to run multiple VMs and Kubernetes.

## Setting up Proxmox
I am not going to go through the installation process, as it is quite straightforward. You can download the ISO
from [here](https://www.proxmox.com/en/downloads/category/iso-images-pve), and follow the installation guide [here](https://pve.proxmox.com/wiki/Installation).

## Network setup
For security practice, and also to practice my network skills. I decided to setup a separate network for my home server and hide
Proxmox server node behind. After some research, I decided to use [pfSense](https://www.pfsense.org/) as my router and
the single point of entry to my home server network. pfSense is a free and open source firewall and router. The ISO can be downloaded
from [here](https://www.pfsense.org/download/). It takes a while form me to setup the network but eventually I got it working.
The below diagram shows the network setup.

![Network setup](/home-server-network-setup.png)

Something to note here:
- My Desktop PC has two network cards on the Motherboard, one is connected to the room AP, the other one is connected to the switch.
Even though the switch is only used for connecting to my Pi 4s, for some reason the enp2s0 interface is not activated unless
I plug a network cable connecting it to the switch
- pfSense VM is connected to both network interfaces, and it is acting as the gateway of the home server network
- All other VMs are connected to the home server network interface only
- DHCP is enabled on the pfSense VM, and it is serving IP addresses to all the VMs in the home server network
- After pfSense gives an IP to the proxmox server, I setup a static IP for it, so that I can remember it easily
- Proxmox server and all the VMs are connected to the home server network and therefore cannot be connected from outside
- pfSense VM will also give IP address to connected devices from the switch using DHCP
- pfSense also has a plugin for tailScale, which I can install on the pfSense VM so I can access my home server anywhere
in the world

## What's next
Now that I have the network setup, I can start to setup the VMs and Kubernetes. I will write another post about how I setup
