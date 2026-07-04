# Lab05 - Static Routing Troubleshooting

## Objective

Troubleshoot a multi-router static routing topology where PC1 and PC2 cannot ping each other.

The goal is to identify and fix the wrong static routes and incorrect interface IP addressing, then verify full end-to-end connectivity.

---

## Topology

```text
PC1 ---- SW1 ---- R1 ---- R2 ---- R3 ---- SW2 ---- PC2
```

---

## Networks

| Network | Description |
|---|---|
| 192.168.1.0/24 | PC1 LAN |
| 192.168.12.0/24 | R1 to R2 link |
| 192.168.13.0/24 | R2 to R3 link |
| 192.168.3.0/24 | PC2 LAN |

---

## IP Addressing Table

| Device | Interface | IP Address | Description |
|---|---|---|---|
| PC1 | FastEthernet0 | 192.168.1.1/24 | PC1 LAN |
| PC1 | Default Gateway | 192.168.1.254 | R1 LAN interface |
| R1 | G0/1 | 192.168.1.254/24 | PC1 LAN gateway |
| R1 | G0/0 | 192.168.12.1/24 | Link to R2 |
| R2 | G0/0 | 192.168.12.2/24 | Link to R1 |
| R2 | G0/1 | 192.168.13.2/24 | Link to R3 |
| R3 | G0/0 | 192.168.13.3/24 | Link to R2 |
| R3 | G0/1 | 192.168.3.254/24 | PC2 LAN gateway |
| PC2 | FastEthernet0 | 192.168.3.1/24 | PC2 LAN |
| PC2 | Default Gateway | 192.168.3.254 | R3 LAN interface |

---

## Problem

PC1 and PC2 were unable to ping each other.

Initial test from PC1:

```bash
ping 192.168.3.1
```

Result:

```text
Request timed out.
Request timed out.
Request timed out.
Request timed out.
```

---

## Troubleshooting Process

### Step 1 - Check PC1 IP Configuration

On PC1:

```bash
ipconfig
```

PC1 had the correct IP address:

```text
IP Address: 192.168.1.1
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.1.254
```

Then I tested the default gateway:

```bash
ping 192.168.1.254
```

Result:

```text
Reply from 192.168.1.254
Reply from 192.168.1.254
Reply from 192.168.1.254
Reply from 192.168.1.254
```

This confirmed that PC1, SW1, and the R1 LAN interface were working correctly.

---

## Misconfiguration 1 - Missing or Wrong Static Route on R1

R1 must know how to reach the PC2 LAN network.

Correct route on R1:

```bash
conf t
ip route 192.168.3.0 255.255.255.0 192.168.12.2
end
```

### Explanation

R1 is not directly connected to the `192.168.3.0/24` network.

To reach PC2, R1 must forward the packet to R2 using the next-hop IP address:

```text
192.168.12.2
```

---

## Misconfiguration 2 - Wrong Static Route Direction on R2

On R2, the route to the PC2 LAN was pointing toward the wrong exit interface.

Wrong route example:

```bash
ip route 192.168.3.0 255.255.255.0 GigabitEthernet0/0
```

`GigabitEthernet0/0` points back toward R1, not toward R3.

Correct route:

```bash
conf t
no ip route 192.168.3.0 255.255.255.0 GigabitEthernet0/0
ip route 192.168.3.0 255.255.255.0 192.168.13.3
end
```

### Explanation

R2 must send traffic for `192.168.3.0/24` toward R3.

The next-hop IP address is:

```text
192.168.13.3
```

---

## Misconfiguration 3 - Wrong IP Address on R3 Interface

R3 had the wrong IP address on the link toward R2.

Wrong IP example:

```text
192.168.23.3/24
```

The correct topology uses this network:

```text
192.168.13.0/24
```

Correct R3 configuration:

```bash
conf t
interface g0/0
no ip address
ip address 192.168.13.3 255.255.255.0
no shutdown
end
```

After this fix, R2 and R3 were in the same subnet:

```text
R2 G0/1: 192.168.13.2/24
R3 G0/0: 192.168.13.3/24
```

---

## Verification

### Test PC1 to PC2

```bash
ping 192.168.3.1
```

Successful result:

```text
Reply from 192.168.3.1
Reply from 192.168.3.1
Reply from 192.168.3.1
Reply from 192.168.3.1
```

### Test PC2 to PC1

```bash
ping 192.168.1.1
```

Successful result:

```text
Reply from 192.168.1.1
Reply from 192.168.1.1
Reply from 192.168.1.1
Reply from 192.168.1.1
```

---

## Important Note About ARP

The first ping after fixing the network may fail once or twice because ARP needs to resolve MAC addresses.

Example:

```text
Request timed out.
Request timed out.
Reply from 192.168.3.1
Reply from 192.168.3.1
```

This is normal behavior.

---

## Useful Commands

```bash
ipconfig
ping <destination-ip>
show ip interface brief
show ip route
show running-config | include ip route
conf t
ip route <network> <mask> <next-hop>
no ip route <network> <mask> <wrong-next-hop-or-interface>
```

---

## What I Learned

- How to troubleshoot static routing step by step.
- Always test the default gateway first.
- If the gateway works, the local LAN is probably correct.
- Static routes must point in the correct direction.
- A wrong exit interface can send packets backward.
- Router interfaces on the same link must be in the same subnet.
- Next-hop IP is better than only using an exit interface on Ethernet links.
- End-to-end connectivity requires both forward and return routes.

---

## Final Result

PC1 and PC2 can successfully ping each other.

Lab completed successfully.
