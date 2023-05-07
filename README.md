# Hosthead infrastructure and software choices

**DRAFT. This document is partially complete.**

Standardizing infrastructure where possible is generally a good thing. The choies made here do not necessarily represent the best infrastructure, the best software, or the best design patterns and automation. The choices made rather represent a standardization that can be assumed when following other hosthead repositories.

# Networking

## Hardware

A switch supporting VLAN tagging will be useful if you have multiple physical servers supporting your infrastructure.

If you are defining a storage area network by using a technology such as NFS, Ceph, Gluster, or another filesystem, plan to add at least 1 extra physical link to each server for the storage area network.

At this time the hardware choices here do not affect the deployments so long as you can configure VLAN tagging and trunking appropriately on your device.

## Software

### Firewall

Software firewalls provide a lot of flexibility. BSD* powered firewalls are common due to their use of pf. OPNsense is the recommended choice here for a software firewall. OPNsense is an alternative to pfsense, another popular choice, forked for security and modernization.

[OPNsense Project Link](https://opnsense.org/)

### Server to Server VPN service tunnels

Wireguard is a lightweight alternative to OpenVPN. While not as flexible, it's arguably more secure. Wireguard does not offer configurable ciphers, it only uses secure ciphers. Wireguard operates on a state machine that does not change state until a valid authenticated packet is received, preventing disclosure of the running service to invalid requests as well as preventing traffic on the wire until there is a need to transmit data.

[Wireguard Project Link](https://www.wireguard.com/)

See also [Tunnels and Tubes](tunnels-and-tubes/)

### Client to Server VPN remote access tunnels

OpenVPN remains one of the most flexible VPN tools. OPNsense makes configuring OpenVPN fairly easy and supports a highly available setup.

# Linux Software

## Operating System

For the Linux based deployments the latest Ubuntu LTS is used unless there is an issue with the latest release, in which case the last LTS is used or Debian is used. LTS releases of Ubuntu have 5 years of support and an extra 5 of extended security maintenance, but running the latest LTS release is generally a good idea.

[Ubuntu Server LTS](https://ubuntu.com/download/server)

## Automation

Ansible is used for things such as patching the servers automatically.

Ansible can be installed with

    sudo apt install ansible

As of writing, Ubuntu 22.04 LTS is shipping out Ansible 2.10, a fairly modern release.

[Ansible documentation](https://docs.ansible.com/ansible/2.9/)
