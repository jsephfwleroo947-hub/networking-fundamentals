[Back to index](../README.md)

# 1. About this project

## 1.1 Purpose and scope

This document covers the core building blocks of computer networking: LANs and WANs, VLANs, subnetting and firewalls and how they fit together. I wrote it to build a real, practical understanding of these fundamentals and to demonstrate that understanding through explanation, worked problems and fault diagnosis.

After reading it, a reader should be able to explain how a LAN differs from a WAN, why VLANs and subnets are different mechanisms that are usually paired, how to plan and calculate an addressing scheme and how a firewall decides what traffic to allow.

## 1.2 How I approached it

I learned this material from a mix of video courses, hands-on learning platforms, reference documents and practice problems. For the theory, I followed Professor Messer's CompTIA Network+ course and Jeremy's IT Lab CCNA course on YouTube and used Practical Networking's video series to reinforce how addressing and switching fit together.

I used TryHackMe's networking rooms for hands-on practice with concepts such as LANs, the OSI model and packets, and I checked definitions and standards against primary sources such as RFC 1918 and the IEEE 802.1Q standard for VLAN tagging.

I also worked through subnetting problems by hand until I could do them without a calculator, and I include those with full working in section 4. The document is organised from the broad to the specific: LANs and WANs first, then VLANs versus subnets, then subnetting calculations, then firewalls, followed by troubleshooting case studies and a self-test that apply everything together.

## 1.3 Understanding Key Terms 

| **Term** | **My definition**                                                                                                                                                                                                       |
|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| LAN      | A **Local Area Network** is a network that connects devices in one enclosed place, for example a school, office or home, wherein they can share files, printers and internet access.                                    |
| WAN      | A **Wide Area Network** connects networks or devices over a larger distance such as linking offices in different cities or connecting a home network to the internet.                                                   |
| VLAN     | A **Virtual LAN** splits one physical network into separate logical networks so devices can be grouped securely even if they use the same switches.                                                                     |
| Subnet   | A **Subnet** is a smaller section of a larger IP network – it helps organise devices and lets routers decide whether traffic stays local or needs to be sent elsewhere.                                                 |
| Router   | A **router** sends data packets between different networks such as between LAN and the internet, using IP addresses.                                                                                                    |
| Switch   | A **switch** connects devices within the same local network and forwards data to the correct device using MAC addresses.                                                                                                |
| Firewall | A **firewall** is a security system that monitors network traffic and allows or blocks it based on security rules.                                                                                                      |
| NAT      | **Network Address Translation** lets multiple devices on a private network share one public IP address when communicating with the internet. The router changes private addresses to the public address and back again. |
| DMZ      | A **Demilitarized Zone** is a separate, partly exposed network area for public facing services such as a web server, so that if it is attacked, the main private LAN is better protected.                               |

---

[Next: 2. LAN vs WAN](02-lan-vs-wan.md) | [Index](../README.md)
