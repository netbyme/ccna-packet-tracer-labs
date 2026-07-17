# Lab 06 – Life of a Packet

## About This Lab

This lab shows what really happens when one PC sends a ping to another PC.

There was no new configuration to complete. The goal was to use Cisco Packet Tracer's **Simulation Mode** and follow the packet from the source device to the destination.

The main lesson is simple:

- The source and destination **IP addresses stay the same** from beginning to end.
- The source and destination **MAC addresses change at every router**.
- A switch forwards the frame without changing its Layer 2 addresses.

## Objectives

- Follow an ICMP packet through switches and routers
- Identify the source and destination IP addresses
- Identify the source and destination MAC addresses on each network segment
- Understand how ARP finds the MAC address of the next hop
- See how switches and routers handle frames differently
- Compare communication inside one LAN with communication between different networks

## Topology

```text
PC1 and PC3                                            PC4
192.168.1.0/24                                  192.168.3.0/24
      |                                                |
     SW1 ---- R1 -------- R2 -------- R3 ---- SW2
             G0/0 G0/1  G0/0 G0/1  G0/0 G0/1
```

The packet crosses three routers when PC1 communicates with PC4.

## Important IP Addresses

| Device | IPv4 Address | Purpose |
|---|---|---|
| PC1 | `192.168.1.1/24` | Source in Questions 1 and 2 |
| PC3 | `192.168.1.3/24` | Same-LAN destination |
| PC4 | `192.168.3.1/24` | Remote-network destination |

## MAC Address Reference

The lab uses easy-to-recognize MAC address endings.

| Device and Interface | MAC Ending |
|---|---|
| PC1 FastEthernet0 | `1111` |
| PC3 FastEthernet0 | `3333` |
| PC4 FastEthernet0 | `4444` |
| R1 G0/0 | `AAAA` |
| R1 G0/1 | `BBBB` |
| R2 G0/0 | `CCCC` |
| R2 G0/1 | `DDDD` |
| R3 G0/0 | `EEEE` |
| R3 G0/1 | `FFFF` |

The full MAC addresses can be checked inside Packet Tracer with `ipconfig /all` on a PC or `show interfaces` on a router.

## Question 1 – PC1 Pings PC4

PC1 and PC4 are in different networks:

```text
Source IP:      192.168.1.1
Destination IP: 192.168.3.1
```

These IP addresses remain unchanged for the complete journey.

The Ethernet MAC addresses are different on each routed segment:

| Network Segment | Source MAC | Destination MAC |
|---|---|---|
| PC1 → SW1 | PC1 `1111` | R1 G0/0 `AAAA` |
| SW1 → R1 | PC1 `1111` | R1 G0/0 `AAAA` |
| R1 → R2 | R1 G0/1 `BBBB` | R2 G0/0 `CCCC` |
| R2 → R3 | R2 G0/1 `DDDD` | R3 G0/0 `EEEE` |
| R3 → SW2 | R3 G0/1 `FFFF` | PC4 `4444` |
| SW2 → PC4 | R3 G0/1 `FFFF` | PC4 `4444` |

### What Happens

1. PC1 sees that `192.168.3.1` is outside its local network.
2. PC1 sends the frame to its default gateway, R1 G0/0.
3. SW1 forwards the frame but does not change its MAC addresses.
4. R1 removes the old Ethernet header, checks its routing table, and creates a new frame for R2.
5. R2 repeats the same routing process and forwards the packet to R3.
6. R3 sees that PC4's network is directly connected and creates a frame addressed to PC4.
7. SW2 forwards the frame to PC4 without changing the MAC addresses.

## Question 2 – PC1 Pings PC3

PC1 and PC3 are both in the `192.168.1.0/24` network.

Because the destination is local, PC1 sends the frame directly to PC3. It does not send it to the default gateway.

| Network Segment | Source MAC | Destination MAC |
|---|---|---|
| PC1 → SW1 | PC1 `1111` | PC3 `3333` |
| SW1 → PC3 | PC1 `1111` | PC3 `3333` |

SW1 forwards the frame to PC3 and keeps the same source and destination MAC addresses.

## Question 3 – PC4 Pings PC1

This is the return journey from PC4 to PC1.

```text
Source IP:      192.168.3.1
Destination IP: 192.168.1.1
```

| Network Segment | Source MAC | Destination MAC |
|---|---|---|
| PC4 → SW2 | PC4 `4444` | R3 G0/1 `FFFF` |
| SW2 → R3 | PC4 `4444` | R3 G0/1 `FFFF` |
| R3 → R2 | R3 G0/0 `EEEE` | R2 G0/1 `DDDD` |
| R2 → R1 | R2 G0/0 `CCCC` | R1 G0/1 `BBBB` |
| R1 → SW1 | R1 G0/0 `AAAA` | PC1 `1111` |
| SW1 → PC1 | R1 G0/0 `AAAA` | PC1 `1111` |

## Why the First Ping May Fail

Before a device can send an Ethernet frame, it must know the MAC address of the next hop.

ARP is used to learn that address. While PCs and routers are completing ARP, the first ICMP request may time out. After the ARP tables and switch MAC address tables are populated, the next pings should succeed.

This behavior is normal in Packet Tracer.

## How I Verified the Packet Flow

1. From PC1, send a ping to PC4:

```text
ping 192.168.3.1
```

2. Send the ping a second time so that ARP and MAC learning are already complete.
3. Change Packet Tracer from **Realtime** to **Simulation** mode.
4. Keep ARP and ICMP visible in the event filters.
5. Send the ping again.
6. Use **Capture/Forward** to move the packet one device at a time.
7. Open each PDU and compare the **Inbound** and **Outbound** Layer 2 information.

## Useful Verification Commands

### On a PC

```text
ipconfig /all
arp -a
ping 192.168.3.1
ping 192.168.1.3
```

### On a Router

```cisco
show interfaces g0/0
show interfaces g0/1
show ip arp
show ip route
```

### On a Switch

```cisco
show mac address-table
```

## Key Difference Between a Switch and a Router

| Device | What It Does |
|---|---|
| Switch | Forwards the existing Ethernet frame and keeps the same source and destination MAC addresses |
| Router | Removes the old Layer 2 header, checks the destination IP, and builds a new Layer 2 frame for the next hop |

## What I Learned

- IP addresses identify the original source and final destination.
- MAC addresses are used only for delivery across the current local network segment.
- A host sends remote traffic to its default gateway.
- A host sends local traffic directly to the destination host.
- ARP maps a local or next-hop IPv4 address to a MAC address.
- Switches learn source MAC addresses and forward frames.
- Routers make Layer 3 forwarding decisions and rewrite the Layer 2 header.
