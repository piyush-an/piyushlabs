---
title: "Day 0: Upgrading My Home Infrastructure"
date: 2026-03-22
draft: false
author: "Piyush Anand"
description: "Documenting my journey from a single Raspberry Pi to a full-fledged homelab with Proxmox clusters, exploring self-hosted solutions and open source tools."
tags: ["homelab", "self-hosting"]
categories: ["self-hosting"]
series: ["Homelab Journey"]
cover:
  image: "/images/hardware.JPG"
  caption: "My Homelab"
ShowToc: true
TocOpen: false
---

**TL;DR:** Upgraded from a single Raspberry Pi to a multi-node homelab featuring 3x HP ProDesk Mini PCs and a Lenovo ThinkCentre, running Proxmox clusters for virtualization, with Tailscale VPN for remote access. One node serves as a Windows 11 Pro workstation, while the other two form a Proxmox cluster for running self-hosted services and NAS.

<!--more-->

## The Journey Begins

This is my second attempt at running a homelab. I first started with a Raspberry Pi which I had running containers to. With this upgrade, I was thinking of documenting it as well to share it.

## The Hardware

Let's go through the hardware stuff. I had sourcing these before the memory and storage prices saw a steep rise in the last 10 months or so. I was lucky endough to source these from Facebook Marketplace.

### My Current Setup

**3x HP ProDesk 400 G5 Desktop Mini**
- CPU: Intel Core i3-9100T @ 3.10GHz
- RAM: 8GB / 16GB / 20GB (varies by unit)
- Storage: Varying from 256GB to 1TB

**1x Raspberry Pi 4 Model B**
- RAM: 8GB

**1x Lenovo ThinkCentre M910q Tiny Desktop Computer Mini PC**
- CPU: Intel Core i7-6700 @ 3.40GHz
- RAM: Empty

## Setup Strategy

### Machine 1: Primary Workstation (HP ProDesk)

I have set up one of the HP machines as a primary PC on which I have installed **Windows 11 Pro** for my day-to-day general usage. The Pro version enables me to access it remotely via Remote Desktop Protocol (RDP), which eliminates the need to switch between display, mouse, and keyboard. I can just access it via my laptop or even my mobile.

When I am away from home and out of reach via the local network, I have configured [Tailscale VPN](https://tailscale.com/) which helps me connect to my local network and continue with my work.

**Advantages:**
- Windows 11 Pro with RDP support
- Tailscale VPN for secure remote access
- Flexibility to access from any device

### Machines 2 & 3: Proxmox Cluster (2x HP ProDesk)

The last time with the Raspberry Pi, I had installed a Ubuntu Linux distribution directly on it, and I had limited experience with container setups. But this time, I was looking for a virtualization solution that would help me run containers and VMs with better isolation based on tool compatibility.

So I installed the [Proxmox VE](https://www.proxmox.com/en/products/proxmox-virtual-environment/overview) on both of the other HP machines and peered them together to create a cluster/node. This way, I can access both of my Proxmox instances via the same console.

**Cluster Purpose:**
- **Node 1**: Running and exploring services that I find interesting and helpful
- **Node 2**: Dedicated NAS setup predominantly

![Proxmox cluster summary](/images/pve_cluster_summary.png)

[![Proxmox cluster summary](/images/pve_cluster_summary.png)](/images/pve_cluster_summary.png)

<!-- *My Proxmox cluster dashboard showing both nodes* -->

## What's Next?

In the upcoming posts, I'll be documenting:
- Setting up various self-hosted services
- Specific tools and applications I'm running

---
