---
layout: post
title: "Deployment Live Cloud Ready Deployment and Recovery Guide."
categories: DeploymentLive ipxe eval uide
---

# Deployment Live Cloud Ready Recovery and Installation and Evaluation Guide

`Updated Sept. 18th 2025`

Well it's been a bit of a roller coaster here at Deployment Live. 

I announced our product to much fanfare back at MMS 2025 at the Mall of America back in May. And then went into Development hell. 
Going through the Microsoft UEFI Certification process proved harder than I thought. Got the bits signed, continued development, found a bug (whoops), and had to start the signing process over again. <WHEW!>

But today, I got back (the second set of) signed bits from Microsoft, and now have **iPXE signed for Secure boot**. Whew! 

Time to release the product! SHIP IT!

## What is it?

[iPXE](https://ipxe.org) is a network boot loader. It's quick to download, only about 400kb to 500kb, and then it loads up the payload you need using Secure HTTPS. 
Like Linux Live CD or Microsoft WinPE (Windows Pre-installation Environment). 

Sure, iPXE can act as a replacement for standard PXE scenarios, since iPXE can download WinPE wim using HTTP which is faster than TFTP, but there are other scenarios that are more interesting.  *insert dramatic pause here*

Because iPXE can communicate over the internet using secure HTTPS, iPXE can now become the backbone of a true **cloud based deployment and recovery** solution. Think Work from Home users who need recovery, not to mention a **cloud** based environment would ideal for some Ransomware scenarios.

However, the binaries from ipxe.org are unsigned, you have to turn *OFF* SecureBoot in the BIOS in order to boot. Which sucks. It makes a true user serviced cloud recovery difficult. So...

I went ahead and got my version of iPXE signed by Microsoft, and I'm giving away one of the versions for free!

## Evaluating the product

Best way to get started is to try out our [Evaluation Guide](https://github.com/DeploymentLive/iPXEOps/blob/main/Docs/evalguide.md)

We even have a video walkthrough of some basic scenarios:

## Scenario 1 - DHCP Proxy.

I wanted to develop a full PXE Server stack for local use, but developing a full Windows Stack for DHCP and TFTP proved a pain.

So, earlier this year I modified my home Ubiquiti router to add the correct PXE settings into the `dnsmasq` utility. I wondered if I could just release a small linux version with minimal `dnsmasq` settings.

 Enter [Tiny Core linux](http://tinycorelinux.net/), [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq), and some glue scripts...

* Simply boot up to the Deployment Live iPXE Cloud environment with a Virtual Machine with Secure Boot off. 
* Select the `Advanced Tools Menu...`
* Select the `DHCP Proxy Server`
* Wait 1 minute... Done!

The server will now receive DHCP Boot requests, and send out **Deployment Live iPXE**, pointing to the cloud server. It works in parallel with your existing DHCP server.

There is even a menu for advanced features, simply press `Ctrl-C` while the log is running. More details on the docs page. 

`DANGER Will Robinson, Danger!!!  Startup DHCP Proxy in your home lab, but you should get permission from your Network Dept before installing at work.`

## Secnario 2 - Boot to Linux Live

One of the challenges in a Malware scenario is how to get everyone back to work quickly,
even if you don't know how EXACTLY how to fix their machines yet. There have been numerous cases of 
companies taking days or even weeks to get their machines back and running.

`Boot to a Backup Operating System` is a quick way to solve this. It's Linux Ubuntu KDE version 22.04. 
And it's running the correct SHIM64 so it can run on Secure Boot machines. 

Check out the [users guide](https://github.com/DeploymentLive/iPXEOps/blob/main/Docs/usersguide.md) where 
we show how users would start this process and what it looks like at the end.

## Scenario 3 - Deploy a new OS

Additionally we have a WinPE image that we can boot and start OS deployment.
The framework we are using here is [OSDCloud](https://osdcloud.com) from Dave Segura (thanks Dave!!)

The advantage here is that I don't need to host OS installation and Driver bits on my web servers. 
Instead OSDCloud can download bits directly from Microsoft. Which makes it faster too. 

There are ways to automate this process, including adding in your own Windows Deployment Package (*.ppkg) please check our admin guide for more information.

### Scenario 3.1 - HP machines

I did figure out how to boot to the HP recovery service, (it's just a WinPE file) so the HP Cloud Recovery System
will auto-light up the `Advanced Tools Menu` if you have a HP machine. ( Does Dell/HP have recovery system?)

## Scenario 4 - PC Recovery

Since we have a full version of WinRE running, next steps are to add some true recovery tools to WinRE. 

This would be for thinks like the CrowdStrike bug of 2024, where deleting a file can recover a machine. 

* Auto unlock of Bitlocker using credentials and a web service.
* Undelete files
* Mount Registry
* Various recovery scenarios....

This isn't fully done yet, but it's in progress.

## Limitations of Free vs Paid.

Right now the free version of **Deployment Live iPXE** trusts only **ONE** Certificate authority for HTTPS TLS/SSL communication. That's the `Deployment Live root CA`, so it can only connect to: https://boot.deploymentlive.com:8050 . 

You are more than welcome to use the **Deployment Live iPXE** binaries to download local files using unsecure HTTP.

However if you are looking to use SecureBoot signed iPXE to download files over HTTPS from **any** site, you will need the **Deployment Live iPXE Enterprise** version. Please contact us at info@deploymentlive.com or check out our [FAQ](https://github.com/DeploymentLive/iPXEOps/blob/main/Docs/faqguide.md).

Thanks!

-k