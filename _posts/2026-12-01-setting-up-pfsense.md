---
title: Setting up pfSense in my homelab
date: 2026-12-01 14:00:22 +0800
categories: [cybersecurity, homelab, proxmox]
tags: [cybersecurity, homelab, proxmox, pfsense]
---

Introduction
----
In this post, I will be going over how I set up my internal network with firewall rules using pfSense.
Configuring both of them took me a few days, which consisted of reading tutorials, documentation and testing.

The main tutorial I have been using is 0xBen's guide on converting a laptop into a cybersecurity lab which you can find here:
[Installing Proxmox on a Laptop and Building a Cybersecurity Lab](https://benheater.com/proxmox-cybersecurity-lab/)

This was why I wanted to set up pfSense:
* I needed clear network segmentation between my internal network of DVVMs (Damn Vulnerable Virtual Machines) and my home LAN.
* Hosting DVVMs on the home LAN is a recipe for disaster.
* For now I only have an Active Directory lab, but I need to factor in future plans that will require more VLAN subnets.
* I thought this would be a good way to get hands-on experience with network segmentation and firewall rules.
  
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

> Confirm the settings here but don't start the VM yet.
{: .prompt-warning }

I added **vmbr1** as a virtual NIC (Network Interface Controller) to the pfSense VM, which will allow me to configure it after installing pfSense.

![Add NIC](/assets/img/setting-up-pfsense/add-nic-to-pfsense-vm.png)
_Select the pfSense VM, go to the Hardware tab and press "Add Network Device"_

![Add vmbr1](/assets/img/setting-up-pfsense/add-vmbr1-to-pfsense-vm.png)

Installing pfSense
----
After configuring the VM settings, I started up the pfSense VM and used these settings for installation:

![pfsense-install-1](/assets/img/setting-up-pfsense/pfsense-install-1.png)

![pfsense-install-2](/assets/img/setting-up-pfsense/pfsense-install-2.png)

![pfsense-install-3](/assets/img/setting-up-pfsense/pfsense-install-3.png)

![pfsense-install-4](/assets/img/setting-up-pfsense/pfsense-install-4.png)

![pfsense-install-5](/assets/img/setting-up-pfsense/pfsense-install-5.png)

![pfsense-install-6](/assets/img/setting-up-pfsense/pfsense-install-6.png)

Then after the installation finished, I chose to reboot the machine:

![pfsense-install-reboot](/assets/img/setting-up-pfsense/pfsense-install-reboot.png)

Assigning and Configuring WAN, LAN and AD VLAN 
----
After rebooting, pfSense will ask if VLANs need to be setup first for interface configuration. This is where the network setup with the OVS switches and ports come in handy for setting up VLANs.

![pfsense-interface-setup-1](/assets/img/setting-up-pfsense/pfsense-interface-setup-1.png)
_I entered Y here to start setting up the VLAN_

![pfsense-interface-setup-2](/assets/img/setting-up-pfsense/pfsense-interface-setup-2.png)
_vtnet1(vmbr1) will be the parent interface for the Active Directory VLAN on VLAN 80_

![pfsense-interface-setup-3](/assets/img/setting-up-pfsense/pfsense-interface-setup-3.png)
_vtnet0(vmbr0) will be the WAN interface_

![pfsense-interface-setup-4](/assets/img/setting-up-pfsense/pfsense-interface-setup-4.png)
_vtnet1(vmbr1) will be the LAN interface_

![pfsense-interface-setup-5](/assets/img/setting-up-pfsense/pfsense-interface-setup-5.png)
_vtnet1.80(vmbr1_80) is the Active Directory VLAN which will be under vtnet1(vmbr1)_

![pfsense-interface-setup-final](/assets/img/setting-up-pfsense/pfsense-interface-setup-final.png)

> pfSense will assign itself an IP address on the WAN interface (vtnet0/vmbr0) based on vmbr0's MAC address. You can give it a static DHCP reservation so it always takes the same IP address in your router's settings.
{: .prompt-info }

After this, I went to configure the LAN interface's IP address with the folllowing:

![pfSense LAN IP setup](/assets/img/setting-up-pfsense/pfsense-lan-ip-setup.png)
_Enter 2 to set interfaces, then enter 2 to configure the LAN._

`Configure IPv4 address LAN interface via DHCP? (y/n)`

Enter `N`

`Enter the new LAN IPv4 address. Press <ENTER> for none:`

I used 172.16.0.1 as the new LAN address, but you can change this to fit your own setup.

`Enter the new LAN IPv4 subnet bit count (1 to 32)`

I used 24 as the subnet bit count, which results in the LAN using the network range of 172.16.0.1/24, but you can change this to fit your own setup.

`For a WAN, enter the new LAN IPv4 upstream gateway address.`
`For a LAN, press <ENTER> for none:`

This is specific to the WAN(**vtnet0**) and since I'm configuring the LAN, I just press ENTER.

`Configure IPv6 address LAN interface via DHCP6? (y/n)`

Enter `N`.

`Enter the new LAN IPv6 address. Press <ENTER> for none:`

Press ENTER, since we're not configuring IPv6 on this LAN.

`Do you want to enable the DHCP server on LAN? (y/n)`

Enter `Y`

pfSense will then prompt for the start and end addresses of the IPv4 client range.

I put the start address as 172.16.0.10 and the end address as 172.16.0.244, but you can change this to fit your own setup.

`Do you want to revert to HTTP as the webConfigurator protocol? (y/n)`

pfSense's web UI is on HTTPS by default.This can be changed later in the web UI settings later, but for now I entered `N` to remain on HTTPS.

pfSense Web UI First-Time Setup
----
To continue configuring the three network interfaces, I needed to access the pfSense VM's web UI.

By default, this web UI is blocked off by a firewall which I disabled by doing the following:
* Press 8 to open a shell on the pfSense VM
* Type the following to disable the pfSense firewall
```
pfctl -d
```
