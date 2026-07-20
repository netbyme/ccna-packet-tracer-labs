# Lab 14 - Rapid Spanning Tree Protocol (RSTP)

## About This Lab

This Packet Tracer lab helped me understand how Rapid Spanning Tree Protocol prevents Layer 2 loops while keeping backup links available.

The topology contains redundant links between switches and a shared segment connected through a hub. This makes it possible to see the main RSTP port roles, port states, and link types in one lab.

## Lab File

`14-Lab - Rapid STP.pkt`

## What I Practiced

- Understanding how the root bridge is elected
- Identifying root, designated, alternate, and backup ports
- Checking forwarding, learning, and discarding states
- Understanding edge, point-to-point, and shared link types
- Seeing how a hub creates a shared Ethernet segment
- Observing a backup port on SW1
- Manually changing the RSTP link type on an interface
- Verifying the spanning-tree topology from the CLI

## Important RSTP Port Roles

| Port role | Purpose |
| --- | --- |
| Root port | The best path from a non-root switch toward the root bridge |
| Designated port | The forwarding port for a network segment |
| Alternate port | A backup path toward the root bridge |
| Backup port | A backup for another port on the same shared segment |

## RSTP Port States

RSTP uses three port states:

- **Discarding:** The port does not forward normal traffic or learn MAC addresses.
- **Learning:** The port learns MAC addresses but does not forward normal traffic yet.
- **Forwarding:** The port learns MAC addresses and forwards traffic.

## RSTP Link Types

| Link type | Typical connection |
| --- | --- |
| Edge | A switch port connected to an end device |
| Point-to-point | A full-duplex link between two switches |
| Shared | A half-duplex shared segment, normally using a hub |

## Main Commands

Enable Rapid PVST+ globally:

```text
enable
configure terminal
spanning-tree mode rapid-pvst
end
```

Manually configure a link type when required:

```text
configure terminal
interface fastEthernet 0/1
spanning-tree link-type point-to-point
exit
interface fastEthernet 0/2
spanning-tree link-type shared
end
```

Verify RSTP:

```text
show spanning-tree
show spanning-tree summary
show spanning-tree detail
show spanning-tree interface fastEthernet 0/1 detail
```

## How to Test the Lab

1. Use `show spanning-tree` and identify the root bridge.
2. Check the root and designated ports that are forwarding.
3. Find the alternate or backup port that is discarding.
4. Shut down an active switch link.
5. Confirm that the alternate path quickly changes to forwarding.
6. Use `no shutdown` to restore the link and check the topology again.

Example failure test:

```text
configure terminal
interface fastEthernet 0/1
shutdown
end
show spanning-tree
configure terminal
interface fastEthernet 0/1
no shutdown
end
```

## Optional Access-Port Protection

PortFast and BPDU Guard can be added to ports connected only to end devices:

```text
configure terminal
interface range fastEthernet 0/3-10
spanning-tree portfast
spanning-tree bpduguard enable
end
```

Do not enable PortFast on normal links between switches.

## What I Learned

RSTP keeps the network loop-free like classic STP, but it reacts much faster when the topology changes. Alternate and backup ports remain ready, so they can replace a failed forwarding path without waiting through the long classic STP timer process.

The shared hub segment was especially useful because it showed the difference between a shared link and a normal point-to-point switch link.

## Conclusion

This lab gave me practical experience reading an RSTP topology instead of only memorizing the theory. I can now identify the root bridge, understand why each port received its role, recognize the different RSTP link types, and test what happens when an active link fails.