# Lab 13 - Configuring Spanning Tree Protocol

## About This Lab

In this lab, I practiced configuring **Spanning Tree Protocol (STP)** on a network with four switches and redundant links.

STP prevents Layer 2 loops by blocking one of the redundant paths. If the active path fails, STP can use the backup path.

## Lab Objectives

- Check the current STP topology.
- Identify the root bridge for VLANs 1 and 2.
- Identify root, designated, and blocked ports.
- Configure primary and secondary root bridges.
- Change STP interface cost.
- Change STP port priority.
- Configure PortFast and BPDU Guard.
- Verify the final STP topology.

## Root Bridge Plan

| VLAN | Primary Root | Secondary Root |
| --- | --- | --- |
| VLAN 1 | SW1 | SW2 |
| VLAN 2 | SW2 | SW1 |

## Configuration

### SW1

```cisco
enable
configure terminal

spanning-tree vlan 1 root primary
spanning-tree vlan 2 root secondary

interface fastEthernet 0/1
 spanning-tree vlan 1 port-priority 240

end
copy running-config startup-config
```

### SW2

```cisco
enable
configure terminal

spanning-tree vlan 2 root primary
spanning-tree vlan 1 root secondary

end
copy running-config startup-config
```

### SW3

PortFast and BPDU Guard were enabled on the port connected to an end device.

```cisco
enable
configure terminal

interface fastEthernet 0/3
 spanning-tree portfast
 spanning-tree bpduguard enable

end
copy running-config startup-config
```

### SW4

The STP cost of FastEthernet 0/2 was increased for VLAN 1. This makes the link less preferred by STP.

```cisco
enable
configure terminal

interface fastEthernet 0/2
 spanning-tree vlan 1 cost 100

interface fastEthernet 0/3
 spanning-tree portfast
 spanning-tree bpduguard enable

end
copy running-config startup-config
```

> PortFast and BPDU Guard should only be enabled on ports connected to end devices, not on switch-to-switch links.

## Verification Commands

```cisco
show spanning-tree
show spanning-tree vlan 1
show spanning-tree vlan 2
show spanning-tree interface fastEthernet 0/3 detail
show running-config | include spanning-tree
```

## What I Verified

- SW1 is the root bridge for VLAN 1.
- SW2 is the root bridge for VLAN 2.
- Every non-root switch has one root port for each VLAN.
- Redundant paths are blocked to prevent Layer 2 loops.
- Changing interface cost affects STP path selection.
- Port priority can influence STP tie-breaking.
- PortFast and BPDU Guard are enabled on the correct access ports.
- End devices can communicate successfully.

## What I Learned

- STP selects the root bridge using the lowest Bridge ID.
- The path with the lowest root-path cost is preferred.
- A lower port-priority value is preferred during a tie.
- Increasing an interface cost makes that path less attractive.
- PortFast allows an access port to enter forwarding quickly.
- BPDU Guard protects an access port if it receives a BPDU.
- STP blocks the redundant path but keeps it available as a backup.

## Lab File

- `13-Lab-Configuring Spanning Tree.pkt`

## Final Result

The network is loop-free and still has redundant links available. SW1 and SW2 are the planned root bridges for different VLANs, while STP cost and port priority control path selection.

The end-device ports are also protected using PortFast and BPDU Guard.

