# Lab 04 – Interface Configuration

## About This Lab

In this lab, I practiced how to configure router interfaces using the Cisco IOS command line.

I assigned IP addresses to the interfaces, turned them on, and checked that they were working correctly.

## What I Learned

- How to enter privileged mode with `enable`
- How to enter configuration mode
- How to select a router interface
- How to assign an IP address and subnet mask
- How to turn on an interface with `no shutdown`
- How to check the status of router interfaces
- How to test the connection with `ping`
- How to save the configuration

## Main Commands Used

### Enter configuration mode

```text
Router> enable
Router# configure terminal
```

### Configure an interface

```text
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address <IP-ADDRESS> <SUBNET-MASK>
Router(config-if)# no shutdown
Router(config-if)# exit
```

I repeated these commands for the other interfaces using the correct IP addresses from the topology.

### Check the interfaces

```text
Router# show ip interface brief
```

When an interface shows `up/up`, it means that the interface is working correctly.

### Check the routing table

```text
Router# show ip route
```

The router automatically adds directly connected networks to its routing table.

### Test the connection

```text
Router# ping <DESTINATION-IP>
```

If the ping is successful, the devices can communicate.

### Save the configuration

```text
Router# copy running-config startup-config
```

## Interface Status

| Status | Meaning |
| --- | --- |
| `up/up` | The interface is working correctly |
| `administratively down/down` | The interface is turned off and needs `no shutdown` |
| `down/down` | There may be a cable or connection problem |

## Troubleshooting

If the connection does not work, I check:

1. The IP address and subnet mask.
2. The interface name.
3. Whether `no shutdown` was entered.
4. Whether the cable is connected correctly.
5. Whether the interface on the other device is also turned on.

## Lab File

The completed Packet Tracer lab is included in this folder:

```text
04-Lab-Interface Configuration.pkt
```

## Tools Used

- Cisco Packet Tracer
- Cisco IOS CLI

---

This lab is part of my CCNA 200-301 study journey.
