---
title: 'Walkthrough on Network Enumeration with Nmap: HTB Academy'
date: 2025-09-01
permalink: /posts/2025/01/htb-walkthrough-1/
tags:
  - hackthebox
  - nmap
---

Hi, I'm Marc Almeda, and today I'm doing a walkthrough on the HTB Academy module `Network Enumeration with Nmap`. This module touches on how to use Nmap, a free and open source utility for network discovery and security auditing, for host and network enumeration and bypassing security measures such as intrustion detection systems (IDS) and intrusion prevention systems (IPS).

What is enumeration and why is it important?
=======
Picture this. You've started a HTB box, but don't know where to start looking. Some information is given to you at the start but where do you find more information on what you're dealing with, what options are open to you and what approach you can use to handle the box?

This is where enumeration comes in.

Enumeration, in the case of ethical hacking, is the active information gathering phase where we establish connections with the target system through scans and gather information on it during the scanning process. The information we gather during this stage is critical because it helps us define what we're dealing with, what strategies we can use and what approach we can take to gain access to the target system.

The information we want to look in the enumeration phase are usually the following:
* Functions and/or resources that allow us to interact with the target system and/or provide addtional information (*service version info, interesting HTTP directories on a web server*)
* Information that provides us with even more important information to access our target (*e.g. a robots.txt file that shows us an /admin/ link that leads to an admin portal login on a web application*)

So how does Nmap help in the enumeration phase?
-----
Nmap is one of the tools we'll be using for enumeration and information gathering. It is designed to scan networks and identify which hosts are available using raw packets, services and applications, including name and version, where possible. It can also do OS version detection and determine if packet filters, firewalls, or intrusion detection systems (IDS) are configured as needed.

In particular, we are looking to do the following with Nmap in this module:
* Host discovery (mapping out which hosts are available to explore)
* Port scanning (determining which ports are on a host)
* Service enumeration and detection (finding out which services run on a port, and what version of the service is present)
* OS detection (finding out which operating system runs on the host, which can open up some exploits)
* Scriptable interaction with the target services using Nmap Scripting Engine (Nmap comes with some scripts that can be used to help find more information on the target)

Host Discovery
======
