---
title: Proxmox VE
summary: Proxmox VE
authors:
    - Andrzej Kaczynski
date: 2024-01-05
---

## Introduction

I decided to use [Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview) for my Protectli Virtualization Platform. The main reason was ease of setup and good support and documented deployments from Protectli. I'm sure it is going to meet all my home lab requirements. There tons of documents in the Internet with tutorials for Proxmox. That was another reason of choosing this OS software.

## Installation

Normally, the Proxmox VE installation is very trivial. It is based on Debian Linux and comes with really handy installer. You simply need to download the official ISO file from the [Proxmox website](https://www.proxmox.com/en/downloads) and prepare the USB stick. 

However, with my Protectli I had few challenges. Mainly because I wanted to use its 8GB eMMC Storage for the purpose of OS installation. During installation, I was able to select my eMMC storage however the installer was failing with an error:

```bash
Unable to get device for partition 1 on device /dev/mmcblk0
```

Luckily I found this article [Install Proxmox VE on eMMC](https://ibug.io/blog/2022/03/install-proxmox-ve-emmc/), which helped me address the issue. It seems Proxmox and its installed do not like **/dev/mmcblk*** devices, probably considering the eMMC as not good place for hosting the OS. 

## Post-Installation

There were a few things to be configured after the installation like:
- Configure necessary repositories (By default Proxmox uses the ones which requires subscription. I wanted to stay with OS only).
- Install basic packages (VIM)
- Update *GRUB* to enable PCI-Passthrough/IOMMU
- etc.

I've created my ansible role & playbook for manage this part. You can find the source code on my github repo [https://github.com/andrewkaczynski/homelab-ansible](https://github.com/andrewkaczynski/homelab-ansible).

