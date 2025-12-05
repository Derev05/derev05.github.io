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

For this setup, I'll be following 0xBen's guide and using OVS (Open vSwitch) bridges instead of the default Linux bridges.
I've tried it with Linux bridges and I feel that OVS bridges make it easier to manage the VLAN setups from the pfSense VM.

Before I started, I created a backup of the default network interface setup with the following command:

```
cp /etc/network/interfaces /etc/network/interfaces.bak
```

This backup proved essential as I ended up locking myself out of the web UI a few times while learning how to set it up.
I got back web UI access by doing the following:

```
cp /etc/network/interfaces.bak /etc/network/interfaces
ifreload -a
```

This is the network setup I went with:

![Network Setup](/assets/img/setting-up-pfsense/proxmox_network_setup.png)

**vmbr0**: This is the OVS bridge that will act as the production switch for my home LAN, which uses the only network device(*shown here as OVS Port*) on my server. vmbr0 is set up as a Linux bridge by default, but I converted it into an OVS bridge.

![vmbr0](/assets/img/setting-up-pfsense/vmbr0_ovs_bridge.png)

**vmbr0_mgmt**: This OVS port will serve as the main access to my Proxmox server's web UI and subsequently the pfSense VM's web UI after installation.

![vmbr0_mgmt](/assets/img/setting-up-pfsense/vmbr0_web_mgmt_ovs_switch.png)

**vmbr1**: This OVS bridge will be used as the pfSense internal switch for LAN and VLAN support, which I will use to set up the internal network.

![vmbr1](/assets/img/setting-up-pfsense/vmbr1_ovs_bridge.png)

**vmbr1_80**: This will be used for my Active Directory lab on VLAN 80.

![vmbr1_80](/assets/img/setting-up-pfsense/vmbr1_vlan_80.png)

Creating and Configuring the pfSense VM
----
pfSense has a Community Edition ISO image that you can download here:
[pfSense download](https://www.pfsense.org/download/)

> Netgate requires users to create an account and provide personal information in order to download the pfSense CE ISO images. If you aren't very keen on doing that, or find it a hassle like I did, you can download it from here instead:
[pfSense alt download](https://atxfiles.netgate.com/mirror/downloads/)
{: .prompt-info }

After downloading the pfSense CE ISO image, I created the pfSense VM with the following settings:

![VM Setup 1](/assets/img/setting-up-pfsense/vm-setup-1.png)

![VM Setup 2](/assets/img/setting-up-pfsense/vm-setup-2.png)

![VM Setup 3](/assets/img/setting-up-pfsense/vm-setup-3.png)

![VM Setup 4](/assets/img/setting-up-pfsense/vm-setup-4.png)

![VM Setup 5](/assets/img/setting-up-pfsense/vm-setup-5.png)

![VM Setup 6](/assets/img/setting-up-pfsense/vm-setup-6.png)

![VM Setup 7](/assets/img/setting-up-pfsense/vm-setup-7.png)

![VM Final Setup](/assets/img/setting-up-pfsense/vm-final-setup.png)

> Don't start the VM yet.
{: .prompt-warning }

I added **vmbr1** as a virtual NIC (Network Interface Controller) to the pfSense VM, which will allow me to configure it after installing pfSense.
