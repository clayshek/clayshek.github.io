---
layout: post
title: Proxmox on Raspberry Pi
tags: proxmox raspberrypi homelab arm64 pimox7
excerpt_separator: <!--more-->
---

## **What**
* [Proxmox VE](https://www.proxmox.com/en/proxmox-ve) virtualization cluster on Raspberry Pi nodes, with some steps custom to my setup.

## **Why**
* Run container, or even VM, [ARM](https://en.wikipedia.org/wiki/ARM_architecture) workloads that always need to be running in a home lab environment
<!--more-->
* Low power consumption (compared to traditional server hardware)
* Simplicity & Familiarity - Another good platform solution for this would be [k3s](https://k3s.io/). However, I personally know Proxmox (my home lab hypervisor of choice), so the learning curve is low. K3s / K8s is on my list of technologies to learn, and this for me can be achieved more effeciently nesting on top of Proxmox
* Capability - this solution allows me to integrate Terraform and Ansible for provisioning and configuration, similar to how I've done prior with https://github.com/clayshek/homelab-monorepo 

## **How**
As Proxmox doesn't natively support the ARM processor architecture, will be using https://github.com/pimox/pimox7 to install latest version of Proxmox VE on Raspberry Pi 4

### Ras Pi Setup:
* Flash Ras Pi SD card with [latest Raspberry Pi OS](https://downloads.raspberrypi.org/raspios_arm64/) (raspios_arm64-2021-11-08 at time of writing) using [Raspberri Pi Imager](https://www.raspberrypi.com/software/).
* Create an empty file named `ssh` in the boot directory of the SD card to enable ssh. Boot the Pi.
* Ssh to whatever DHCP IP the Pi received. `ssh pi@ipaddr` using `raspberry` as password
* Change default password using `passwd`
* Run, in order:
```
sudo -s
apt update
apt upgrade
apt dist-upgrade
```
* Optional: using `raspi-config`: Set time zone, set System/Boot to "Console", disable Splash Screen
* Optional: if intending to use iSCSI storage, run `apt-get install open-iscsi`
* Run `curl https://raw.githubusercontent.com/pimox/pimox7/master/RPiOS64-IA-Install.sh > RPiOS64-IA-Install.sh` then `chmod +x RPiOS64-IA-Install.sh`
* Run `./RPiOS64-IA-Install.sh`, providing required inputs when prompted. Pi will reboot when script successfully completes.

### Proxmox Setup:
* Access Proxmox GUI at `https://<IP entered above>:8006`
* Add NAS storage: Datacenter -> Storage -> Add SMB/CIFS. Once per cluster.
* Add iSCSI SAN storage: Datacenter -> Storage -> Add iSCSI. ID=rpipve-iscsi, uncheck "Use LUNs directly". Once per cluster. If necessary, obtain initiator name with `sudo cat /etc/iscsi/initiatorname.iscsi`
* Add LVM storage on top of iSCSI LUN: Datacenter -> Storage -> Add LVM. ID=vm-store-iscsi, Volume Group=iscsi-vg
* Optional: Setup 2FA: Datacenter -> Permissions -> Two Factor


### To-Do:
* Turn all of the above into an Ansible playbook
