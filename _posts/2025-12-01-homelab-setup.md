---
title: Setting up a homelab
date: 2025-12-01 14:00:22 +0800
categories: [homelab, cybersecurity,proxmox]
tags: [homelab, cybersecurity, proxmox]     # TAG names should always be lowercase
---

Introduction
----
A few months ago, my brother said he would be letting go of this old PC below as it could not be upgraded to Windows 11:

![Brother's old PC](/assets/img/homelab-setup/promox_setup.jpg)

I was initially not very interested in taking it, as I was doing pre-course preparation for the OSCP with HackTheBox and job hunting at the same time.
My current PC could handle the workloads that Hackthebox asked for, so I thought I didn't really need it.

In the midst of all this, I was reading online forums and asking connections I made from networking events about potential projects I could do to boost my resume.
A homelab stood out as the most suggested project. It would give me a safe space to experiment with the various tools used in cybersecurity.

So, I decided to take the PC (*with my brother's consent of course*) and turn it into a homelab with a focus on cybersecurity.

Why build a homelab?
----
I figured out early on in my job hunt that I was lacking practical experience with the tools that are frequently used in cybersecurity, such as SIEM, EDR and firewalls.
So I pivoted to learning pentesting, as online resources for pentesting such as HackTheBox and TryHackMe were readily available.

As I learnt more and more about pentesting and in particular, Active Directory attacks, I started to wonder how these attacks would be flagged on the blue team's side.
I also wanted to understand how the tools I was using such as NetExec, Bloodhound and the Impacket suite worked under the hood as well.

What I've learnt so far
----
![Proxmox](/assets/img/homelab-setup/proxmox_flex.png)

After installing Proxmox and getting the server onto the proper subnet, I was greeted with this screen. 

I searched online for a bit and decided to experiment with implementing a pfSense VM hosting an OpenVPN server so I could create an internal network of AD machines that is segregated from my home network.

More on that in the next blog post.

The plan
----
Before I can start testing and tinkering around with tools, I must build the playground first.

This is how I'm planning to set up the network for now:
![Homelab Network Diagram](/assets/img/homelab-setup/plan.png)

I've already set most of this up and am left with setting up the AD client VMs to complete the AD setup.

Some future projects I will explore:
* Implement Wazuh and get it to ingest logs from the domain controller and the client machines, then doing a full compromise of the AD network with Kali and review the logs
* Do deep dives into the various Active Directory attacks (Kerberoasting, ACL/ACE abuse, DCSync) and the measures to prevent them
* Host my old school project website (*which is full of holes and is very vulnerable*) and experiment with WAF and honeypot solutions

Conclusion
----
Over the next few weeks, I'll be documenting how I set up the homelab and the various experiments that I will do.
In a way, I'll also be building a portfolio for this homelab and seeing where it takes.

Yes, I'm still looking for a job and I probably won't be done job hunting anytime soon.

Marc out.




