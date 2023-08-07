---
title:       "Utilities on my home server"
subtitle:    ""
description: "Useful utilities on my home server"
date:        2023-08-07
author:      "Marco"
image:       ""
categories:  ["Tech"]
---

## Introduction
Last post I talked about the hardware, network setup of my home server with Proxmox. 
In this post, I will talk about the utilities I have added and implemented on my home server, which I find handy.

## Utilities

### TailScale
TailScale is a VPN service that allows you to create a mesh network for all connected device under your account.
Once installed and connected on the client, you can access all other connected devices with a private IP address and a
magic private hostname. Also pfSense has a plugin for TailScale, so you can install it on pfSense VM and then access
all other devices in the private home-network. I find it very easy to use, it works with one-click and no other 
configurations are needed.

### Remote Desktop
When I did my server setup with Proxmox, my plan was to have an VM with more RAM and CPU assigned, so it behaves like
a main VM to setup heavy apps on. Also, I want to have a Desktop UI on that VM, so I don't need to type in command-line
all the time. 

In the end, I installed Ubuntu 23.04 with Desktop, it works great on Proxmox, but it comes with a problem: How do I access
the Desktop UI other than through Proxmox web UI? Proxmox web UI has some limitations, the quality is not great, you cannot
resize the window, and it is not very responsive, also, as it is using noVNC behind the scene, you cannot copy and paste, which
for my usage, is a big problem.

So I decided to setup a remote desktop connection to my VM. I have tried a few options, and finally settled with [xrdp](https://xrdp.org/).
I followed this guide [here](https://www.digitalocean.com/community/tutorials/how-to-enable-remote-desktop-protocol-using-xrdp-on-ubuntu-22-04) 
to setup xrdp on my Ubuntu VM. Then, all I need is to forward the port 3389 to my VM, and I can access the remote desktop.

### Terraform
Create VM one-by-one in Proxmox UI is tedious and error-pruning. I want to have a way to create VMs with IaC, so I can
add VMs easily with a few lines of code and easier control. I have used Terraform before in my work, so I decided to use
Terraform to create VMs on Proxmox. 

bpg is a great Terraform provider for Proxmox, it is easy to use and has a lot of features. It is well documented with examples
so I didn't have many issues setting it up. At the time of implementing, these are some major issues I have faced
- Creating VMs in parallel was not working smoothly for me, often bpg will break with a disk error, I have to create VMs with parallelism=1 in Terraform
- QEMU agent is not working great for me, when enabled, it will cause `terraform refresh` to hang and eventually timeout, so I have to disable it

With Terraform, I can now manage VMs much more easier, instead of going through the Proxmox UI.

### K3s-Ansible
I want to have a Kubernetes cluster on my home server, so I can deploy apps on it. K3s is a lightweight Kubernetes distribution
that is easy to setup and use. However, it will still require some manual setup on each VM to get it connected and working.
Therefore, I want to see if it is possible to setup the K3s cluster with IaC.

I found this [repo](https://github.com/techno-tim/k3s-ansible) that has a working example of setting up K3s cluster with Ansible.
I got it working with very few changes needed and it is working great. I can now setup a K3s cluster with a few lines of code.

## What's next
I am still exploring more utilities to add to my home server, and I will keep updating this post with new findings.
I am planning to try out the following:

- Grafana for monitoring
- Rancher for Kubernetes management
- Portainer for Docker management
- Home Assistant for home automation

