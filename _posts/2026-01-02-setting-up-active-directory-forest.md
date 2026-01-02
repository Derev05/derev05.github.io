---
title: Setting up an Active Directory Forest in my homelab
date: 2026-01-02 19:00:22 +0800
categories: [cybersecurity, homelab, proxmox, windows, active-directory]
tags: [cybersecurity, homelab, proxmox, windows, active-directory]
---

Introduction
---
In this post, I will be going over how I set up an Active Directory forest of Windows virtual machines (VMs) in my homelab.
This AD forest will serve as the testing grounds for testing out exploits and CVE proof-of-concept code, as well as setting up blue team tools
such as Wazuh.

I will be setting up a flat AD forest, where the domain controller and client machines will be on the same subnet (172.16.23.1/24).
This will allow me to leverage as many attacks as possible without the need for port-forwarding.

Getting the Windows Server ISO Files
---
For this network, I will be using Windows Server 2022 for the domain controller and Windows Server 2019 for the client machines.
I initially wanted to use Windows 10 Enterprise for the client machines, but with Microsoft's push towards Windows 11, I was unable
to download the ISO, so Windows Server 2019 will have to do.

Using Windows Server 2019 also gives me the opportunity to experiment with the Potato exploits for local privilege escalation on Windows.
_(SigmaPotato/GodPotato/PrintSpoofer/SharpEfsPotato)_

Here's the download links for Windows Server 2022 and Windows Server 2019:

[Windows Server 2022](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022)

[Windows Server 2019](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)

Staging the Network
---
Before I set up the Windows VMs, I will recap on how I've set up the VLAN network on Proxmox and pfSense.

The AD forest will be hosted on VLAN 80 with 172.16.23.1/24 as the static IPv4 configuration.

Here is how I've set up the VLAN interface on pfSense:
![ad-lab-interface](/assets/img/setting-up-pfsense/ad-lab-interface-assign.png)

Here is how I've set up the firewall rules on the VLAN interface:
![ad-lab-firewall-setup](/assets/img/setting-up-pfsense/ad-lab-firewall-setup.png)

A short recap on the firewall rules:
* The RFC1918 alias defines all private network subnets as defined by RFC1918.
* The "Allow traffic to the Internet rule" allows VMs and devices on the VLAN subnet to send traffic to any address not in RFC1918.
* The "Allow traffic to default gateway" will whitelist the VLAN gateway address (172.16.23.1) from the above rule. 

There are a few settings I will also be implementing on the VLAN interface in pfSense as well.
In particular, I want the domain controller to run a DNS server and a DHCP server, since this will be a small setup.

pfSense offers the capability for my AD lab VLAN interface to host a DHCP server on pfsense, which I disabled.

![disable-dhcp](/assets/img/setting-up-active-directory-forest/ad-lab-dhcp-disable.png)

The DNS server will act as a resolver for the local domain of the Active Directory forest.
The DHCP server will allow the domain controller to auto assign IP addresses to the client machines I will be setting up later.

Setting up the Domain Controller (Windows Server 2022)
---
### VM Setup
These are the settings I used for the initial setup of the Windows VM.

![dc-vm](/assets/img/setting-up-active-directory-forest/dc-vm-setup-1.png)

![dc-vm](/assets/img/setting-up-active-directory-forest/dc-vm-setup-2.png)

![dc-vm](/assets/img/setting-up-active-directory-forest/dc-vm-setup-3.png)

![dc-vm](/assets/img/setting-up-active-directory-forest/dc-vm-setup-4.png)

![dc-vm](/assets/img/setting-up-active-directory-forest/dc-vm-setup-5.png)

![dc-vm](/assets/img/setting-up-active-directory-forest/dc-vm-setup-6.png)

![dc-vm](/assets/img/setting-up-active-directory-forest/dc-vm-setup-7.png)

### Installing Windows Server 2022
After setting up the VM, I booted up the VM and started the installation process.
Windows will ask for the language and timezone before starting and I went with the default settings.

After that, Windows will ask for which OS to install. I went with `Windows Server 2022 Standard Evaluation (Desktop Experience)`.

![dc-windows](/assets/img/setting-up-active-directory-forest/dc-windows-setup-1.png)

After accepting the terms and conditions, I decided to go for the custom installation as shown below:

![dc-windows](/assets/img/setting-up-active-directory-forest/dc-windows-setup-2.png)

After a few more steps, Windows will go through the installation process and reboot.

After rebooting, Windows will prompt to set a local administrator password before login.
This password can be saved in a password manager or the 'Notes' field of the VM.

![set-admin-pwd](/assets/img/setting-up-active-directory-forest/set-admin-pwd.png)

### Configuring the Network Interface
Earlier in the post, I disabled the DHCP server on the AD Lab VLAN in order for the domain controller to act as the DHCP server.
However, this means that the domain controller is not automatically configured and I will have to set it up manually.

So, I will be changing the IP and DNS server addresses as shown below:

![dc-network](/assets/img/setting-up-active-directory-forest/dc-network-setup.png)

Changing the IP address here will subject this VM to the firewall rules set in my AD Lab VLAN instead of my LAN.

Changing the DNS server addresses here does the following:
* When a DNS query is sent, the DNS server (which will be set up later) running on the domain controller will be the first to answer.
* If the DNS server doesn't know the answer, the query is then forwarded to the default gateway and pfSense will resolve it.

After saving the changes, I renamed the server to DC01 and restarted the VM.

### Saving a Snapshot of the VM
Before converting this VM into a domain controller, I will save a snapshot of it pre-domain controller.
This will let me roll back to a pre-domain install should the need arise.

![dc-snapshot](/assets/img/setting-up-active-directory-forest/dc-snapshot.png)

### Installing and Configuring Active Directory Domain Services
Now that the server is properly configured and a pre-domain snapshot saved, it's time to convert this VM into a domain controller.
To do that, I will need to install Active Directory Domain Services (ADDS) and the DNS server.
The DNS server is required by ADDS and also lets me resolve the domain controller by DNS name.

I opened up the Add Roles and Features Wizard and selected ADDS and the DNS server as shown below:

![dc-ad-forest-dns-setup](/assets/img/setting-up-active-directory-forest/dc-ad-forest-dns-setup-1.png)

The installation process starts after confirming the server roles and features to install.
After installing ADDS and the DNS server, the wizard will prompt for some post-configuration steps to promote the server to a domain controller.

![dc-ad-forest-dns-setup](/assets/img/setting-up-active-directory-forest/dc-ad-forest-dns-setup-2.png)

This will open the ADDS configuration wizard and I chose the following options:

![ad-ds-setup](/assets/img/setting-up-active-directory-forest/ad-ds-setup-1.png)

![ad-ds-setup](/assets/img/setting-up-active-directory-forest/ad-ds-setup-2.png)

![ad-ds-setup](/assets/img/setting-up-active-directory-forest/ad-ds-setup-3.png)

![ad-ds-setup](/assets/img/setting-up-active-directory-forest/ad-ds-setup-4.png)

![ad-ds-setup](/assets/img/setting-up-active-directory-forest/ad-ds-setup-5.png)

After reviewing and confirming the options, the wizard will do a prerequisites check before allowing the installation of AD DS.
Since the server passed all the checks successfully, I can install ADDS.

![ad-ds-setup](/assets/img/setting-up-active-directory-forest/ad-ds-setup-6.png)

The server will automatically reboot after the installation process.

### Troubleshooting DNS name resolution for Linux OpenVPN clients
Previously, I set up OpenVPN for remote access to my internal network of VMs mainly for my Kali Linux VM to act as an attacker.
However, I ran into DNS issues while trying to verify the DNS server on the domain controller.

As it turns out, `resolv.conf` will not be updated automatically by OpenVPN in Linux without some packages and tweaks. 

I fixed this by configuring both my OpenVPN server and the client file on my Kali.

On my OpenVPN server, I pushed the following options in Advanced Configuration:

```
push "dhcp-option DNS 172.16.23.10";
push "dhcp-option DOMAIN twoamad.lab";
```

This pushes both my domain controller IP address and domain name as DHCP options in the OpenVPN server configuration.
This can also be achieved by checking `DNS Server enable` in Advanced Client Settings and inputting 172.16.23.10 as one of the DNS servers.

On the client file in my Kali, I added the following lines:

```
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
```

This allows the client file to run scripts and calls the script `update-resolv-conf` to update resolv.conf to update the changes pushed to the OpenVPN server.
However, the `resolvconf` package must be installed in order for the script to run.

```
sudo apt install resolvconf
```

With that, the DNS server running on the domain controller is pushed into `resolv.conf`, which allows my Kali to run DNS queries on the AD lab.

### Testing AD DS and the DNS server
After fixing the DNS server issues, I did a comparison between both pre-domain and post-domain setups on my Kali VM.

Here is what nmap, nslookup and NetExec found on the pre-domain setup:

![pre-domain-scans](/assets/img/setting-up-active-directory-forest/pre-domain-scans.png)

Here is what nmap, nslookup and NetExec found on the post-domain setup:

![post-domain-scans](/assets/img/setting-up-active-directory-forest/post-domain-scans.png)

This confirms the installation of the DNS server and AD DS, as my domain name is shown in NetExec and can be queried in nslookup.
There are also more ports open such as DNS(53) and Kerberos(88).

### DNS Forwarders
The DNS server on the domain controller will act as the main DNS query resolver for machines within the domain.
However, it cannot resolve DNS queries for machines outside of the domain.

In order for the DNS server to do this, I will need to assign the default gateway (172.16.23.1) as a DNS forwarder for pfSense to resolve these queries.

![dns-forward-setup](/assets/img/setting-up-active-directory-forest/dns-forward-setup-1.png)

After opening the DNS manager, I went to assign the default gateway as a DNS forwarder as shown below:

![dns-forward-setup](/assets/img/setting-up-active-directory-forest/dns-forward-setup-2.png)

![dns-forward-setup](/assets/img/setting-up-active-directory-forest/dns-forward-setup-3.png)

This makes it possible for machines within the AD domain to resolve DNS queries outside of the domain, such as Google.

![external-dns-query](/assets/img/setting-up-active-directory-forest/external-dns-query.png)


### Setting up and Configuring Active Directory Certificate Services
The next step is to set up Active Directory Certificate Services.
This will enable LDAPS on port 636 on the domain controller and will let me explore the ESC attacks with Certipy.

The installation process is the same as earlier when I installed AD DS and the DNS server.

![adcs-setup](/assets/img/setting-up-active-directory-forest/adcs-setup-1.png)

I will only install the Certificate Authority (CA) for this domain.

![adcs-setup](/assets/img/setting-up-active-directory-forest/adcs-setup-2.png)

After the installation finishes, I will need to set up and configure the CA for this domain.

![adcs-setup](/assets/img/setting-up-active-directory-forest/adcs-setup-3.png)

The AD CS wizard will need to specify credentials to configure the CA. For this, I will be using the Administrator as this is the only user.

![adcs-setup](/assets/img/setting-up-active-directory-forest/adcs-setup-4.png)

I will be setting up an enterprise CA using the default options.

![adcs-setup](/assets/img/setting-up-active-directory-forest/adcs-setup-5.png)

![adcs-setup](/assets/img/setting-up-active-directory-forest/adcs-setup-6.png)

![adcs-setup](/assets/img/setting-up-active-directory-forest/adcs-setup-7.png)

![adcs-setup](/assets/img/setting-up-active-directory-forest/adcs-setup-8.png)

![adcs-setup](/assets/img/setting-up-active-directory-forest/adcs-setup-9.png)

![adcs-setup](/assets/img/setting-up-active-directory-forest/adcs-setup-10.png)

After confirming and configuring the CA, I used NetExec's ADCS module with the administrator's credentials to verify the AD CS deployment.

![adcs-confirm](/assets/img/setting-up-active-directory-forest/nxc-adcs-confirm.png)

### Setting up and Configuring the DHCP server
The next step is to set up the DHCP server on the domain controller.
This will allow the domain controller to auto assign IP addresses within the AD Lab VLAN subnet to any domain joined client machines.

The installation process is the same as earlier, so I will go through the post-configuration steps.

![dc-dhcp-setup](/assets/img/setting-up-active-directory-forest/dc-dhcp-setup-1.png)

![dc-dhcp-setup](/assets/img/setting-up-active-directory-forest/dc-dhcp-setup-2.png)

After the post-configuration steps, the DHCP server will need a new scope for distributing IP addresses to the domain joined machines on the VLAN.
I will set 172.16.23.3-254 as the scope's address range with the /24 subnet.

![dc-dhcp-setup](/assets/img/setting-up-active-directory-forest/dc-dhcp-setup-3.png)

![dc-dhcp-setup](/assets/img/setting-up-active-directory-forest/dc-dhcp-setup-4.png)

I won't be setting any DHCP exclusions on the subnet since I don't have a need to reserve static IP addresses on the VLAN yet.
For now, leases on this DHCP scope will be set to last for a year since I am emulating a stable corporate AD setup.
From my research, best practices on lease time vary depending on the network setup, with public networks using shorter lease times.

![dc-dhcp-setup](/assets/img/setting-up-active-directory-forest/dc-dhcp-setup-5.png)

![dc-dhcp-setup](/assets/img/setting-up-active-directory-forest/dc-dhcp-setup-6.png)

The wizard will then prompt to set up the DHCP options, which consist of the default gateway, domain name, DNS servers and WINS servers.
Since I haven't assigned the default gateway yet, this presents me a good opportunity to do so.

![dc-dhcp-setup](/assets/img/setting-up-active-directory-forest/dc-dhcp-setup-7.png)

Here, I use 172.16.23.1 as the default gateway, which points to pfSense.

![dc-dhcp-setup](/assets/img/setting-up-active-directory-forest/dc-dhcp-setup-8.png)

![dc-dhcp-setup](/assets/img/setting-up-active-directory-forest/dc-dhcp-setup-9.png)

The domain name and DNS servers were set up earlier as part of the DNS server setup and there are no WINS servers on the domain for now,
so I went with the default options.

Then, I activated the scope to start the DHCP service on the domain controller.

![dc-dhcp-setup](/assets/img/setting-up-active-directory-forest/dc-dhcp-setup-10.png)

In order for me to verify the DHCP server is installed properly, I will be setting up the client machines later.

### Creating Domain Administrator and 2 Normal Users
The next step is to create users for the domain.
In particular, I want to create a domain administrator and 2 normal users, which will serve as the test bed for Access Control Entry (ACE) exploits.

First, I will create a domain administrator.

![domain-admin](/assets/img/setting-up-active-directory-forest/domain-admin-setup-1.png)

![domain-admin](/assets/img/setting-up-active-directory-forest/domain-admin-setup-2.png)

![domain-admin](/assets/img/setting-up-active-directory-forest/domain-admin-setup-3.png)

Here, I chose the "Password never expires" option so that I can keep the passwords consistent across my experiments.

After creating the user, I added them to the Domain Admins group.
This makes the user a domain administrator and gives them admin privileges over the AD domain.
I verified the user creation by logging in as the user and checking which user groups they are in with PowerShell.

![domain-admin](/assets/img/setting-up-active-directory-forest/domain-admin-check.png)

Next, I will create two normal users. The process is mostly the same as creating the domain admin,except I will not be adding them to any groups. 

![user](/assets/img/setting-up-active-directory-forest/user-setup-1.png)

![user](/assets/img/setting-up-active-directory-forest/domain-admin-setup-3.png)

![user](/assets/img/setting-up-active-directory-forest/user-setup-2.png)

![user](/assets/img/setting-up-active-directory-forest/domain-admin-setup-3.png)

With that, the domain controller is finished and I will move on to the client machines.

Windows Server 2019 Clients
----
Next, I will be setting up the client machines that will be joining the domain I set up.
Here, I will be setting up a template VM for the client machines in Proxmox.
This is mainly to get the AD forest up and running with minimal effort, as the template can be cloned as many times as needed for setup.

### VM Setup

Here are the settings I used for setting up the client VM:

![client-setup](/assets/img/setting-up-active-directory-forest/client-setup-1.png)

![client-setup](/assets/img/setting-up-active-directory-forest/client-setup-2.png)

![client-setup](/assets/img/setting-up-active-directory-forest/client-setup-3.png)

![client-setup](/assets/img/setting-up-active-directory-forest/client-setup-4.png)

![client-setup](/assets/img/setting-up-active-directory-forest/client-setup-5.png)

![client-setup](/assets/img/setting-up-active-directory-forest/client-setup-5.png)

![client-setup](/assets/img/setting-up-active-directory-forest/client-setup-6.png)

![client-setup](/assets/img/setting-up-active-directory-forest/client-setup-7.png)

### Installing Windows Server 2019

The installation process is the same as earlier when I installed Windows Server 2022 for the domain controller.

![client-windows-install](/assets/img/setting-up-active-directory-forest/client-windows-install-1.png)

![client-windows-install](/assets/img/setting-up-active-directory-forest/client-windows-install-2.png)

After the installation finishes, the system will prompt to set a password for the Administrator account on the VM.

### Creating the Template using Sysprep

After setting the password for the Administrator and logging into the server, I will be creating the template using sysprep.
From my research, sysprep will generalize the Windows installation by removing computer-specific info such as installed drivers and the computer SID.
This will allow me to create a Windows image template, which I can use to spin up multiple Windows clients depending on my needs.

I opened a PowerShell terminal as admin and entered the following to run sysprep:

```
C:\Windows\System32\Sysprep\sysprep.exe
```

Here, I check `Generalize` and choose System Out-of-Box Experience.

![sysprep](/assets/img/setting-up-active-directory-forest/sysprep.png)

From there, I can convert this VM into a template by right-clicking on it and selecting `Convert to Template`.

### Creating the Clients and Joining them to the Domain
Now that the template has been created, I can clone as many client machines as needed from the template depending on how many I need.
For this homelab, I will be setting up only 2 clients.

> For demo purposes, this is for one computer. Repeat this process for every additional computer you intend to deploy and join to the domain.
{: .prompt-info }

Cloning the template is done by right-clicking on the template and selecting `Clone`.

![client-clone](/assets/img/setting-up-active-directory-forest/client-clone-1.png)

Since I chose the System Out-of-Box Experience in the sysprep process, the Windows server installation process is skipped.
Instead, the setup starts after the installation is finished, where I have to set the password for the Administrator account.

![set-admin-pwd](/assets/img/setting-up-active-directory-forest/set-admin-pwd.png)

After setting the password and logging in, I will need to join the computer to the AD domain.

I did this by searching for `View advanced system settings`.

![domain-join](/assets/img/setting-up-active-directory-forest/domain-join-1.png)

From there, I can change the computer name and domain membership in System Properties.

![domain-join](/assets/img/setting-up-active-directory-forest/domain-join-2.png)

![domain-join](/assets/img/setting-up-active-directory-forest/domain-join-3.png)

Windows will then prompt to enter the credentials of an account with permission to join the domain.
For this, I used the domain admin credentials.

![domain-join](/assets/img/setting-up-active-directory-forest/domain-join-4.png)

![domain-join](/assets/img/setting-up-active-directory-forest/domain-join-success.png)

After that, the client computer is now joined to the domain and domain users can access the client.

![domain-join](/assets/img/setting-up-active-directory-forest/domain-join-check.png)

Conclusion
----
This concludes how I set up an Active Directory forest in my homelab.
Setting it up was a good learning experience as I learnt more about DNS, DHCP and Active Directory services.
I also learnt how these services show up on the scanning tools I learnt from HackTheBox and Offsec such as Nmap and NetExec.

The next post will be about how I set up Wazuh for security incident and event monitoring on my homelab, mainly to understand how to read
security logs and how the tools I learnt from HackTheBox and Offsec show up in security logs.

Marc out.
