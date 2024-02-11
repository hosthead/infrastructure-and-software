# Virtualization

One of the best things since sliced bread for the self hoster is virtualization. Take a single server and slice it up into multiple operating systems, managed by an out of band (from the perspective of the VMs) console.

Also take advantage of network segmentation for security by virtualizing your network and firewall, for advanced users.

## What is Proxmox?

Per [proxmox.com](https://www.proxmox.com/en/): 

> *Proxmox Virtual Environment is a complete, open-source server management platform for enterprise virtualization. It tightly integrates the KVM hypervisor and Linux Containers (LXC), software-defined storage and networking functionality, on a single platform. With the integrated web-based user interface you can manage VMs and containers, high availability for clusters, or the integrated disaster recovery tools with ease.)*

In my own words, it's a zero cost VM hypervisor built on top of mostly open source components. There are some proprietary bits, but the software is very friendly to the self hoster. You will get the occasional prompt to purchase support, which can be remedied with an edit to the javascript for the web interface on the proxmox install.

## Getting Proxmox

You can install proxmox on any unused server or PC. You just need to download the [latest ISO](https://www.proxmox.com/en/downloads) and burn it to a disc or flash it to a bootable USB drive.

## Standard tooling used

Proxmox uses standard linux virtualization tooling as well as supports standard storage tooling.

Virtualization & Containers:

* QEMU/KVM
* LXC

Storage:

* ZFS
* LVM
* Ceph/CephFS
* NFS
* SMB
* iSCSI
* GlusterFS
* Local Directory
