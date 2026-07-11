# Lab 02 – Ethernet Switching

## Overview

This lab demonstrates basic Layer 2 Ethernet switching, including MAC address learning, frame forwarding, and communication between devices in the same LAN.

## Objectives

- Build a basic switched LAN topology
- Connect multiple PCs through Cisco switches
- Configure IPv4 addresses on end devices
- Observe how switches learn MAC addresses
- Verify end-to-end connectivity using ping

## Topology

- **Switches:** 2 × Cisco 2960
- **End devices:** 4 × PCs
- **Network:** `192.168.1.0/24`
- **Simulation software:** Cisco Packet Tracer

## Skills Practiced

- Connecting PCs to switches
- Connecting two switches together
- Configuring IPv4 addresses
- Understanding Layer 2 frame forwarding
- Checking the switch MAC address table
- Testing connectivity between hosts

## Verification Commands

```text
show mac address-table
show interfaces status
