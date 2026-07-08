# Static Routing VLSM Lab - Two Routers and Four LANs

## Lab Overview

This lab demonstrates how to configure **VLSM subnetting** and **static routing** between two Cisco routers.

The goal is to allow all PCs from different LANs to communicate using manually configured static routes.

Major network used:

```text
192.168.5.0/24
```

This `/24` network was subnetted using VLSM to support multiple LANs with different host requirements.

---

## Topology

```text
PC2 ---- SW ---- R1 ---- R2 ---- SW ---- PC3
              |       |
             PC1     PC4
```

### Devices Used

- 2 Cisco routers
- 4 PCs
- Switches for LAN connectivity
- Cisco Packet Tracer

---

## Addressing Plan

| Segment | Network | Mask | Usable Range | Gateway / Router IP |
|---|---:|---:|---:|---:|
| LAN2 | 192.168.5.0/25 | 255.255.255.128 | 192.168.5.1 - 192.168.5.126 | 192.168.5.126 |
| LAN1 | 192.168.5.128/26 | 255.255.255.192 | 192.168.5.129 - 192.168.5.190 | 192.168.5.190 |
| LAN3 | 192.168.5.192/28 | 255.255.255.240 | 192.168.5.193 - 192.168.5.206 | 192.168.5.206 |
| LAN4 | 192.168.5.208/28 | 255.255.255.240 | 192.168.5.209 - 192.168.5.222 | 192.168.5.222 |
| R1-R2 Link | 192.168.5.224/30 | 255.255.255.252 | 192.168.5.225 - 192.168.5.226 | Point-to-point |

---

## PC Addressing

| Device | IP Address | Subnet Mask | Default Gateway |
|---|---:|---:|---:|
| PC1 | 192.168.5.129 | 255.255.255.192 | 192.168.5.190 |
| PC2 | 192.168.5.1 | 255.255.255.128 | 192.168.5.126 |
| PC3 | 192.168.5.193 | 255.255.255.240 | 192.168.5.206 |
| PC4 | 192.168.5.209 | 255.255.255.240 | 192.168.5.222 |

---

## Router Interface Configuration

## R1 Interface Configuration

```cisco
enable
configure terminal

interface gigabitEthernet0/1
 ip address 192.168.5.126 255.255.255.128
 no shutdown
exit

interface gigabitEthernet0/0
 ip address 192.168.5.190 255.255.255.192
 no shutdown
exit

interface gigabitEthernet0/0/0
 ip address 192.168.5.225 255.255.255.252
 no shutdown
exit

end
write
```

### R1 Interface Verification

Command:

```cisco
show ip interface brief
```

Expected result:

```text
Interface              IP-Address      Status      Protocol
GigabitEthernet0/0     192.168.5.190   up          up
GigabitEthernet0/1     192.168.5.126   up          up
GigabitEthernet0/0/0   192.168.5.225   up          up
```

---

## R2 Interface Configuration

```cisco
enable
configure terminal

interface gigabitEthernet0/0
 ip address 192.168.5.206 255.255.255.240
 no shutdown
exit

interface gigabitEthernet0/1
 ip address 192.168.5.222 255.255.255.240
 no shutdown
exit

interface gigabitEthernet0/0/0
 ip address 192.168.5.226 255.255.255.252
 no shutdown
exit

end
write
```

### R2 Interface Verification

Command:

```cisco
show ip interface brief
```

Expected result:

```text
Interface              IP-Address      Status      Protocol
GigabitEthernet0/0     192.168.5.206   up          up
GigabitEthernet0/1     192.168.5.222   up          up
GigabitEthernet0/0/0   192.168.5.226   up          up
```

---

## Static Route Configuration

R1 knows its directly connected networks:

```text
192.168.5.0/25
192.168.5.128/26
192.168.5.224/30
```

R1 does not know the networks behind R2:

```text
192.168.5.192/28
192.168.5.208/28
```

So static routes are configured on R1.

### R1 Static Routes

```cisco
enable
configure terminal

ip route 192.168.5.192 255.255.255.240 192.168.5.226
ip route 192.168.5.208 255.255.255.240 192.168.5.226

end
write
```

---

R2 knows its directly connected networks:

```text
192.168.5.192/28
192.168.5.208/28
192.168.5.224/30
```

R2 does not know the networks behind R1:

```text
192.168.5.0/25
192.168.5.128/26
```

So static routes are configured on R2.

### R2 Static Routes

```cisco
enable
configure terminal

ip route 192.168.5.0 255.255.255.128 192.168.5.225
ip route 192.168.5.128 255.255.255.192 192.168.5.225

end
write
```

---

## Routing Table Verification

### R1 Routing Table

Command:

```cisco
show ip route
```

Expected important routes:

```text
C    192.168.5.0/25 is directly connected, GigabitEthernet0/1
C    192.168.5.128/26 is directly connected, GigabitEthernet0/0
S    192.168.5.192/28 [1/0] via 192.168.5.226
S    192.168.5.208/28 [1/0] via 192.168.5.226
C    192.168.5.224/30 is directly connected, GigabitEthernet0/0/0
```

### R2 Routing Table

Command:

```cisco
show ip route
```

Expected important routes:

```text
S    192.168.5.0/25 [1/0] via 192.168.5.225
S    192.168.5.128/26 [1/0] via 192.168.5.225
C    192.168.5.192/28 is directly connected, GigabitEthernet0/0
C    192.168.5.208/28 is directly connected, GigabitEthernet0/1
C    192.168.5.224/30 is directly connected, GigabitEthernet0/0/0
```

---

## Connectivity Tests

### Router-to-Router Test

From R1:

```cisco
ping 192.168.5.226
```

Successful result:

```text
Success rate is 100 percent (5/5)
```

From R2:

```cisco
ping 192.168.5.225
```

Successful result:

```text
Success rate is 100 percent (5/5)
```

---

## PC Connectivity Tests

From PC1:

```text
ping 192.168.5.190
ping 192.168.5.225
ping 192.168.5.226
ping 192.168.5.209
```

Successful result:

```text
Reply from 192.168.5.226
Reply from 192.168.5.209
```

This confirms that PC1 can reach R2 and PC4 through static routing.

---

## Troubleshooting Notes

During the lab, several mistakes were identified and fixed.

### Mistake 1: Wrong Next-Hop Address

Wrong command:

```cisco
ip route 192.168.5.192 255.255.255.240 192.168.5.225
```

Problem:

```text
192.168.5.225 is R1's own IP address.
```

Correct command:

```cisco
ip route 192.168.5.192 255.255.255.240 192.168.5.226
```

The next-hop must be the IP address of the neighboring router.

---

### Mistake 2: Wrong Subnet Mask on R2 Link

Wrong command:

```cisco
ip address 192.168.5.226 255.255.255.192
```

Problem:

```text
255.255.255.192 = /26
```

That created an overlapping subnet:

```text
192.168.5.192/26
```

Correct command:

```cisco
ip address 192.168.5.226 255.255.255.252
```

The R1-R2 point-to-point link uses:

```text
192.168.5.224/30
```

---

### Mistake 3: Missing Return Route on R2

PC1 could ping R1, but it could not ping R2.

Working:

```text
PC1 -> R1
```

Not working:

```text
PC1 -> R1 -> R2
```

Reason:

```text
R2 did not know how to return traffic to PC1's network.
```

Fix:

```cisco
ip route 192.168.5.128 255.255.255.192 192.168.5.225
```

---

### Mistake 4: Typing Wrong Network in Static Route

Wrong route:

```cisco
ip route 192.168.128.0 255.255.255.192 192.168.5.225
```

Problem:

```text
PC1 is not in 192.168.128.0/26.
PC1 is in 192.168.5.128/26.
```

Correct route:

```cisco
ip route 192.168.5.128 255.255.255.192 192.168.5.225
```

After fixing this, PC1 successfully pinged R2 and PC4.

---

## Key Lessons Learned

- Static routes need a destination network, subnet mask, and next-hop address.
- The next-hop IP must be the neighboring router, not the local router.
- Point-to-point router links commonly use `/30` networks in CCNA labs.
- Routers need routes in both directions.
- A ping can fail even if the forward path is correct, because the return path may be missing.
- Overlapping subnets are not allowed on router interfaces.
- Always verify using:
  - `show ip interface brief`
  - `show ip route`
  - `ping`
  - PC IP configuration

---

## Final Result

The lab was completed successfully.

Confirmed successful pings:

```text
PC1 -> R2: 192.168.5.226
PC1 -> PC4: 192.168.5.209
```

Successful output:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

---

## Save Configuration

On both routers:

```cisco
write
```

or:

```cisco
copy running-config startup-config
```

---

## Author

Created as part of my CCNA static routing and VLSM practice labs.
