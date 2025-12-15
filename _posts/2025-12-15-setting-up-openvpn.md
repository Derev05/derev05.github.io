---
title: Setting up OpenVPN for remote access to my homelab
date: 2025-12-15 17:00:22 +0800
categories: [cybersecurity, homelab, proxmox]
tags: [cybersecurity, homelab, proxmox, openvpn]
---

Introduction
---
In this post, I will be going over how I set up remote access to my internal network using OpenVPN.
Configuring OpenVPN took me a while with a mix of experimentation, reading documentation and looking at tutorials.

The main reason for setting up an OpenVPN server was for me to set up remote access for my Kali virtual machine that I host on my home PC.
This would allow me to set a static VPN IP that will serve as the attacker's IP in my cyber range, which will make learning blue team tools much easier.

Using the Configuration Wizard
---
pfSense offers a configuration wizard for setting up OpenVPN, which was useful as I did not know where to start.

### Creating the Certificate Authority (CA)
The first step is to create a Certificate Authority (CA).
From my research, this will be a trusted third party that signs the OpenVPN server certificate and any user/client certificates.

![openvpn-wizard](/assets/img/setting-up-openvpn/openvpn-wizard-1.png)

### Creating the Server Certificate
The next step is to create the server certificate that the OpenVPN server will use, which will be signed by the CA I created earlier.

![openvpn-wizard](/assets/img/setting-up-openvpn/openvpn-wizard-2.png)

### Configuring the Server
The next step is to create and configure the OpenVPN server itself. This is how I configured the server.

![openvpn-wizard](/assets/img/setting-up-openvpn/openvpn-wizard-3.png)

![openvpn-wizard](/assets/img/setting-up-openvpn/openvpn-wizard-4.png)

> I use 172.16.22.0/24 as my tunnel network and checked "Redirect IPv4 Gateway" as I will be setting up a route for each VLAN I create. This is up to your preference.
{: .prompt-info }

![openvpn-wizard](/assets/img/setting-up-openvpn/openvpn-wizard-5.png)

![openvpn-wizard](/assets/img/setting-up-openvpn/openvpn-wizard-6.png)

> I use 172.16.22.1 as the DNS server for the VPN and created a custom hostname. This is up to your preference.
{: .prompt-info }

![openvpn-wizard](/assets/img/setting-up-openvpn/openvpn-wizard-7.png)

> For now, I have set up routes to both my LAN and AD lab VLAN subnets in the custom settings. MTU and other settings can also be changed here.
{: .prompt-info }

![openvpn-wizard](/assets/img/setting-up-openvpn/openvpn-wizard-8.png)

Creating a VPN User Profile
---
After setting up the CA, OpenVPN server and the server certificate, I needed to create a user that will serve as my authentication credentials to the OpenVPN server.

> The password set for the user here will be used for OpenVPN client authentication.
{: .prompt-info }

![openvpn-user](/assets/img/setting-up-openvpn/create-vpn-user-1.png)

After successfully creating the user, I needed to create a user certificate for the user, which will be signed by the CA I created.

![openvpn-user](/assets/img/setting-up-openvpn/create-vpn-user-2.png)

Exporting the Client
---
After setting up the user and the user certificate, I needed to export the client for testing the setup.
This requires the OpenVPN Client Export package to be installed, which can be found by searching in Package Manager -> Available Packages.

> pfSense requires Internet access for the Package Manager to fetch and install packages.
{: .prompt-info } 

![openvpn-client-export](/assets/img/setting-up-openvpn/install-client-export.png)

After installing the package, the new feature will appear in VPN -> OpenVPN.
This was how I configured the export settings.

![openvpn-client-export](/assets/img/setting-up-openvpn/openvpn-client-export-1.png)

The user client can be downloaded here and will come in the form of a .ovpn file.

![openvpn-client-export](/assets/img/setting-up-openvpn/openvpn-client-export-2.png)

### Some things to take note of before testing
The export settings I use here will use pfSense's WAN IP address for creating the ovpn file. 
If this IP address changes for any reason, any previously created VPN client exports will not work properly, as I have learnt recently.

To fix this temporarily for testing, I opened the VPN client export in a text editor and updated the IP address to the newly assigned pfSense WAN IP address.

After that, I created a static DHCP reservation for the pfSense VM, so that it always occupies the same IP address regardless of router reboots.
This ensures that I would not have to reconfigure previously created VPN client exports for testing.

Testing the VPN Setup
---
After downloading the user client file, I used my Kali VM to test the OpenVPN connection.

![openvpn-test](/assets/img/setting-up-openvpn/openvpn-test-1.png)

Then, I checked if the routes I pushed on the VPN server were properly configured with `ip route show`.

![openvpn-test](/assets/img/setting-up-openvpn/openvpn-test-2.png)

> Here I used a Windows Server 2019 VM for testing, which required me to disable Windows Firewall on the VM. Use a Linux distro VM for less headache.
{: .prompt-info } 

Then, I set up a VM on the AD lab subnet (172.16.23.0/24) with the IP address 172.16.23.10.
I did a ping sanity check and an Nmap scan to confirm the route was working as intended.

![openvpn-test](/assets/img/setting-up-openvpn/openvpn-test-3.png)

![openvpn-test](/assets/img/setting-up-openvpn/openvpn-test-4.png)

Conclusion
---
This concludes how I set up remote access to my internal network using OpenVPN.
This was a good learning experience as I had to deviate from guides that suggested to set up Kali in the same network interface.
I had to experiment a bit to see what worked and why. It took me a full day of experimenting and testing to arrive at this setup.
The next post will go into how I set up an Active Directory forest, which will serve as the testing grounds for my tools and exploits.

Marc out.
