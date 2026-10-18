[Back to index](../README.md)

# 3. VLANs vs subnets

## 3.1 What a VLAN does

A VLAN (virtual LAN) is a way of splitting one physical switch, or a group of switches, into several separate logical networks. It works at Layer 2, the Data Link layer, where switches forward frames using MAC addresses. By default, every port on a switch is in the same broadcast domain, meaning a broadcast sent by one device (such as an ARP request asking "who has this IP address?") is flooded out of every other port. On a large flat network this wastes bandwidth and lets every device see every other device's broadcast traffic. A VLAN confines broadcasts to the ports that belong to it, so devices in one VLAN never receive broadcasts from another, and cannot communicate with it directly, even if they are plugged into the same switch.

The effect is that one physical switch behaves as if it were several separate switches. For example, staff computers, servers and guest devices can all be plugged into the same hardware, yet each group sits in its own VLAN and is kept apart. Groups can be organised by function or department rather than by physical location, because membership is set by configuration rather than by cabling.

Switch ports operate in one of two main modes. An access port belongs to a single VLAN and connects to an end device such as a PC or printer. The device is unaware of VLANs, and frames arrive and leave untagged. A trunk port connects switches to each other, or a switch to a router or firewall, and carries traffic for several VLANs over one link. To keep the VLANs distinct on that shared link, each frame is labelled using the IEEE 802.1Q standard, which inserts a small tag into the Ethernet frame containing the VLAN ID. The receiving switch reads the tag, forwards the frame only within the correct VLAN, and removes the tag before delivering it out of an access port. VLAN IDs are 12 bits long, giving usable IDs from 1 to 4094.

## 3.2 What a subnet does

A subnet is a smaller network carved out of a larger block of IP addresses. It works at Layer 3, the Network layer, where devices are identified by IP addresses rather than MAC addresses. Each subnet is defined by a network address and a subnet mask (or the equivalent CIDR prefix, such as /24), which splits every address into two parts: the network portion, shared by every device in the subnet, and the host portion, which identifies the individual device.

For example, 192.168.10.0/24 means the first 24 bits identify the network and the last 8 bits identify the host, giving addresses 192.168.10.1 to 192.168.10.254 for devices. Subnetting lets an organisation divide its address space into right-sized groups instead of one large network, which uses addresses efficiently and gives routers and firewalls a clear boundary to work with.

The mask is also how a host decides where to send traffic. Before sending a packet, the host compares its own network portion with the destination's, in effect applying its mask to both addresses. If they match, the destination is on the same subnet, so the host finds its MAC address (using ARP[^1]) and sends the frame directly. If they do not match, the destination is on another network, so the host sends the packet to its default gateway, the router interface on its own subnet, and the router takes it from there. A host with no gateway configured can only talk to devices in its own subnet.

## 3.3 Side-by-side comparison

| **Aspect**                           | **VLAN**                                                                                                                                                         | **Subnet**                                                                                                                           |
|--------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| OSI layer                            | Layer 2 (Data link) Frames are tagged and switched using **MAC addresses**.                                                                                      | Layer 3 (Network). Packets are addressed and routed using **IP addresses**.                                                          |
| What it separates                    | Broadcast domains, devices on different VLANs can’t see each other’s broadcasts even on the same physical switch.                                                | IP address ranges. Hosts decide, using a subnet mask, whether a destination is on their own network or needs to go via a gateway.    |
| Problem it solves                    | It stops one big flat network sharing broadcasts and access. It groups devices logically i.e. by department or function regardless of where they are plugged in. | Divides an address space into right-sized networks, so addresses are not wasted and routing and filtering can happen between groups. |
| How it is defined                    | Configured on a switch; A VLAN ID; 1 to 4094 assigned to access ports and carried between switches over trunk ports using 802.1Q tags[^2].                       | Defined by a network address and mask such as 10.10.30.0/24, configured on hosts and on the gateway interface.                       |
| What lets traffic cross between them | A layer 3 device: a router or a layer 3 switch. A layer 2 switch alone can never move traffic between VLANs.                                                     | A router. Traffic leaving the subnet goes to the default gateway which routes it onward and then a firewall can inspect it there.    |

## 3.4 Why they are paired one-to-one

A VLAN and a subnet are separate mechanisms, but they are normally paired one-to-one: each VLAN carries exactly one subnet, and each subnet lives in exactly one VLAN.The reason is that the two boundaries need to agree.

A host uses its subnet mask to decide which devices are "local", meaning it can reach them directly with ARP and a frame, and which are "remote", meaning it must send them to the default gateway. That decision only works if the devices the host considers local are really in the same broadcast domain. When one VLAN holds one subnet, the Layer 3 view and the Layer 2 view match: everything in the subnet receives the host's broadcasts, and everything outside it must go through the gateway.

The pairing also makes the network simpler to run. Each VLAN has a single gateway (a router interface or Layer 3 switch interface) on its subnet, for example VLAN 30 with 10.10.30.0/24 and gateway 10.10.30.1. A firewall rule written for a subnet therefore applies to exactly one VLAN, and the addressing plan, the switch configuration and the troubleshooting all point to the same thing.

When the two do not line up, things inevitably break. If two subnets share one VLAN, devices in both subnets receive each other's broadcasts, but each group believes the other is remote. Traffic between neighbours is sent to the gateway, which must have an address in both subnets to route it, and then bounces straight back out of the same VLAN. If the gateway is missing an address for one subnet, those devices cannot communicate at all, even though they share a switch. Security is also weaker, because the VLAN provides no separation between the two groups and anyone can join the other subnet just by setting a static address. If one subnet is spread across two VLANs, the opposite happens: hosts think their peers are local, so they send an ARP broadcast, but the broadcast never leaves their own VLAN, the ARP never gets an answer, and the devices cannot reach each other despite sharing a subnet.

## 3.5 Inter-VLAN routing

Devices in different VLANs cannot talk to each other directly, so something at Layer 3 has to route between them. Each VLAN has its own subnet, and the device that routes between them owns the gateway address for each one. Two common designs do this.

In router-on-a-stick, a single physical router interface connects to a switch over a trunk port. The router divides that interface into logical subinterfaces, one per VLAN, and each is configured with the matching VLAN ID (using 802.1Q) and an IP address, which becomes that VLAN's default gateway. For example, a PC in VLAN 10 (10.10.10.5) sending to a PC in VLAN 20 (10.10.20.5) sees that the destination is on another subnet, so it sends the frame to its gateway, 10.10.10.1. The switch tags the frame for VLAN 10 and sends it up the trunk, the router receives it on the VLAN 10 subinterface, looks up the destination, and sends it back down the same cable tagged for VLAN 20. The switch then delivers it to the second PC. The name comes from the shape of the design: the traffic goes up to the router and straight back on one link.

A Layer 3 switch does the same job inside the switch. Instead of subinterfaces, it has a switch virtual interface (SVI) for each VLAN, an IP address assigned to the VLAN itself, and routing is enabled on the switch. Traffic between VLANs is routed internally, in hardware, without leaving the device. This is faster and scales better, which is why it is the usual choice inside larger networks, while router-on-a-stick suits small networks because it needs only a router and a trunk.

## 3.6 Diagram

![Figure 2: one switch divided into three VLANs, with a router on a trunk link providing the gateways](../images/figure-2-vlans.png)

*Figure 2: One switch divided into three VLANs. Each VLAN has its own subnet, and a router on a trunk link provides the gateway for each.*

[^1]: Address Resolution Protocol

[^2]: 4-byte headers inserted into Ethernet frames to identify VLAN membership and prioritize traffic.

---

[Previous: 2. LAN vs WAN](02-lan-vs-wan.md) | [Next: 4. Subnetting](04-subnetting.md) | [Index](../README.md)
