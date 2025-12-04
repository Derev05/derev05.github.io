---
title: Setting up pfSense and OpenVPN on my homelab
date: 2026-12-01 14:00:22 +0800
categories: [cybersecurity, homelab, proxmox]
tags: [cybersecurity, homelab, proxmox, pfsense, openvpn]
---

Introduction
----
In this post, I will be going over how I set up my internal network with firewall rules using pfSense and OpenVPN.
Configuring both of them took me a few days, which consisted of reading tutorials, documentation and testing.

The main tutorial I have been using is 0xBen's guide on converting a laptop into a cybersecurity lab which you can find here:
[Installing Proxmox on a Laptop and Building a Cybersecurity Lab](https://benheater.com/proxmox-cybersecurity-lab/)

My main motivation for setting up pfSense and OpenVPN were the following:
* I needed clear network segmentation between my internal network of DVVMs (Damn Vulnerable Virtual Machines) and my home LAN.
* Hosting DVVMs on the home LAN is a recipe for disaster.
* I wanted a remote access point to the internal network using OpenVPN. 
* For now I only have an Active Directory lab, but I need to factor in future plans that will require more VLAN subnets.
* I thought this would be a good way to get hands-on experience with network segmentation, firewall rules and VPN routing.

Configuring the Network: OVS Switches and IntPorts
----
Before I can install pfSense, I will need to add an additional network interface into my Proxmox network config to house my LAN and VLAN subnets.
For now, I'm setting up only one VLAN subnet that will be used for my Active Directory Lab.




