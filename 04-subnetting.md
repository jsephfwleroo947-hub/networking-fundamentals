[Back to index](../README.md)

# 4. Subnetting

## 4.1 Theory notes

### 1. Binary and dotted decimal; how a mask splits network bits from host bits

An IPv4 address is really a 32-bit binary number. For readability it's written as four 8-bit groups (octets) in decimal, each from 0 to 255. Each bit in an octet has a place value of 128, 64, 32, 16, 8, 4, 2, 1 from left to right. A subnet mask is also 32 bits: a run of 1s followed by a run of 0s. The 1s mark the network portion of the address and the 0s mark the host portion. Applying the mask to an address (a bitwise AND) gives the network address.

*Example:* 192.168.10.70 with mask 255.255.255.192.

|            | Binary                              | Decimal         |
|------------|-------------------------------------|-----------------|
| Address    | 11000000.10101000.00001010.01000110 | 192.168.10.70   |
| Mask       | 11111111.11111111.11111111.11000000 | 255.255.255.192 |
| AND result | 11000000.10101000.00001010.01000000 | 192.168.10.64   |

The mask has 26 ones, so there are 26 network bits and 6 host bits, and the host is on the network 192.168.10.64.

### 2. CIDR notation, block sizes and the "magic number" method

CIDR notation[^1] writes the mask as a slash and the number of 1 bits, so 255.255.255.192 becomes /26. The "magic number" method is a shortcut for finding subnet boundaries without writing out binary. Find the "interesting" octet (the one where the mask isn't 255 or 0), then subtract its mask value from 256. The result is the block size and subnets start at multiples of it.

*Example:* for /26 the interesting octet is the fourth, with mask value 192. The block size is 256 - 192 = 64, so the subnets start at .0, .64, .128 and .192. The address .70 falls between .64 and .128, so it's in the .64 subnet, which matches the binary result above.

| CIDR | Mask            | Block size | Usable hosts |
|------|-----------------|------------|--------------|
| /24  | 255.255.255.0   | 256        | 254          |
| /25  | 255.255.255.128 | 128        | 126          |
| /26  | 255.255.255.192 | 64         | 62           |
| /27  | 255.255.255.224 | 32         | 30           |
| /28  | 255.255.255.240 | 16         | 14           |
| /29  | 255.255.255.248 | 8          | 6            |
| /30  | 255.255.255.252 | 4          | 2            |

### 3. Network address, broadcast address, usable host range

Every subnet has two special addresses. The network address has all host bits set to 0 and identifies the subnet itself. The broadcast address has all host bits set to 1 and reaches every device in the subnet. Everything between the two is the usable host range, meaning the addresses you can assign to devices.

*Example:* 192.168.10.64/26. The network address is 192.168.10.64 (host bits 000000). The broadcast address is 192.168.10.127 (host bits 111111), which you can also find as 64 + 64 - 1. The usable range is 192.168.10.65 to 192.168.10.126.

### 4. Hosts per subnet = 2^h - 2 (and why the 2 is subtracted)

With h host bits there are 2^h possible combinations, but two of them are reserved: all zeros (the network address) and all ones (the broadcast address). Neither can be given to a device, so usable hosts = 2^h - 2. To size a subnet, find the smallest h where 2^h - 2 is at least the number of hosts you need.

*Example:* the /26 has h = 6, so 2^6 - 2 = 62 usable hosts. If you need 50 hosts, h = 5 gives only 30, which is too few, while h = 6 gives 62, so you choose /26. A /30 has h = 2, giving 2 usable hosts, which is why it's the standard size for a point-to-point WAN link like the ones in Figure 1 (203.0.113.0/30 has usable addresses .1 and .2).

One exception worth a sentence: a /31 (RFC 3021) is allowed on point-to-point links and uses both addresses, and a /32 identifies a single host.

### 5. Fixed-length vs variable-length (VLSM) subnetting

With fixed-length subnetting, every subnet uses the same mask, so every subnet is the size needed by the largest group and the rest waste addresses. With VLSM, each subnet gets the mask that fits its own needs, so the address space is used efficiently. The usual method is to allocate the largest requirement first, then the next, placing each one on the next free block boundary so nothing overlaps.

*Example:* split 192.168.10.0/24 for groups of 50, 25 and 10 hosts.

| Group | Hosts needed | Host bits    | Mask | Subnet           | Usable range |
|-------|--------------|--------------|------|------------------|--------------|
| A     | 50           | 6 (62 hosts) | /26  | 192.168.10.0/26  | .1 to .62    |
| B     | 25           | 5 (30 hosts) | /27  | 192.168.10.64/27 | .65 to .94   |
| C     | 10           | 4 (14 hosts) | /28  | 192.168.10.96/28 | .97 to .110  |

Fixed-length would need three /26 blocks (192 addresses), whereas VLSM uses 64 + 32 + 16 = 112, leaving 192.168.10.112 onwards free for growth.

## 4.2 Worked problem template

| **Step**                          | **Working**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Problem                           | The block 172.16.4.0/23 must be split into 4 equal subnets, each able to hold at least 100 hosts.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Host bits needed                  | Usable hosts = 2^h - 2. h = 6 gives 62, which is too few. h = 7 gives 126, which is enough. So each subnet needs 7 host bits.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Resulting mask                    | Network bits = 32 - 7 = 25, so the mask is /25 (255.255.255.128). The block starts at /23, so 2 extra bits are borrowed (25 - 23), which gives 2^2 = 4 subnets, as required.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Binary working                    | The /23 mask is 11111111.11111111.11111110.00000000, so the third octet of 172.16.4.0 is 00000100, with its last bit belonging to the host part. The two borrowed bits are that last bit of the third octet and the top bit of the fourth octet. Their four values give the four subnets: **00** gives third octet 00000100 (4) and fourth octet 00000000 (0), so 172.16.4.0. **01** gives third octet 4 and fourth octet 10000000 (128), so 172.16.4.128. **10** gives third octet 00000101 (5) and fourth octet 0, so 172.16.5.0. **11** gives third octet 5 and fourth octet 128, so 172.16.5.128. The block size is 256 - 128 = 128 in the fourth octet. |
| Answer                            | **1:** 172.16.4.0/25, usable 172.16.4.1 to 172.16.4.126, broadcast 172.16.4.127. **2:** 172.16.4.128/25, usable 172.16.4.129 to 172.16.4.254, broadcast 172.16.4.255. **3:** 172.16.5.0/25, usable 172.16.5.1 to 172.16.5.126, broadcast 172.16.5.127. **4:** 172.16.5.128/25, usable 172.16.5.129 to 172.16.5.254, broadcast 172.16.5.255.                                                                                                                                                                                                                                                                                                                  |
| Check (does it fit? any overlap?) | Each subnet holds 126 usable hosts, which is at least 100. The four ranges run end to end with no overlap and no gaps. 4 x 128 = 512, which equals the size of the original /23, so the block is used completely. Each network address is a multiple of 128 within its octet, and the first subnet starts at the original network address, 172.16.4.0.                                                                                                                                                                                                                                                                                                       |

## 4.3 VLSM worked example

| **Subnet**   | **Hosts needed** | **Mask**              | **Network address** | **Usable range**                 | **Broadcast**  |
|--------------|------------------|-----------------------|---------------------|----------------------------------|----------------|
| A            | 50               | /26 (255.255.255.192) | 192.168.10.0        | 192.168.10.1 to 192.168.10.62    | 192.168.10.63  |
| B            | 25               | /27 (255.255.255.224) | 192.168.10.64       | 192.168.10.65 to 192.168.10.94   | 192.168.10.95  |
| C            | 10               | /28 (255.255.255.240) | 192.168.10.96       | 192.168.10.97 to 192.168.10.110  | 192.168.10.111 |
| D (WAN link) | 2                | /30 (255.255.255.252) | 192.168.10.112      | 192.168.10.113 to 192.168.10.114 | 192.168.10.115 |

## 4.4 Addressing plan for an imaginary organisation

| **VLAN ID**    | **Purpose**    | **Subnet / mask**               | **Gateway** | **Usable range**           | **Justification**                                                                                              |
|----------------|----------------|---------------------------------|-------------|----------------------------|----------------------------------------------------------------------------------------------------------------|
| 10             | Staff          | 10.10.10.0/24 (255.255.255.0)   | 10.10.10.1  | 10.10.10.1 to 10.10.10.254 | About 120 devices (PC’s, phones, printers) A /24 gives 254 addresses, roughly double what's needed.            |
| 20             | Servers        | 10.10.20.0/24 (255.255.255.0)   | 10.10.20.1  | 10.10.20.1 to 10.10.20.254 | About 15 servers. Deliberately generous so each service can have its own address and the plan is easy to read. |
| 30             | Guests         | 10.10.30.0/24 (255.255.255.0)   | 10.10.30.1  | 10.10.30.1 to 10.10.30.254 | Up to 60 visitor devices at busy times. The /24 absorbs spikes without reconfiguration.                        |
| 40             | DMZ            | 10.10.40.0/27 (255.255.255.224) | 10.10.40.1  | 10.10.40.1 to 10.10.40.30  | About 6 public facing servers – kept small on purpose since fewer addresses means a smaller attack surface.    |
| n/a (VPN pool) | Remote workers | 10.10.50.0/26 (255.255.255.192) | 10.10.50.1  | 10.10.50.1 to 10.10.50.62  | About 25 remote users, each needing an address while connected. A /26 gives 62.                                |
| 99             | Management     | 10.10.99.0/28 (255.255.255.240) | 10.10.99.1  | 10.10.99.1 to 10.10.99.14  | About 8 infrastructure devices (switches, firewall, monitoring). A /28 gives 14.                               |

I sized each subnet from an assumed device count, then chose the smallest standard block that comfortably exceeds it. The Staff, Servers and Guests networks use /24s, which give 254 usable addresses. That is roughly twice the expected size for Staff and four times for Guests, and for Servers it is far more than needed, but a /24 is easy to read and removes the risk of running out. The DMZ is a /27 and the management network a /28 because both should stay small: fewer addresses means fewer places for an unauthorised device to hide. The VPN pool is a /26, giving 62 addresses for about 25 remote workers, which leaves room for the team to more than double.

The plan also leaves most of the address space unused. All six subnets sit inside 10.10.0.0/16, each in its own third octet, and the VLAN ID matches that octet (VLAN 10 is 10.10.10.0, VLAN 20 is 10.10.20.0, and so on), which makes the plan easy to remember and troubleshoot. Most of the third octets are free, so new VLANs can be added without renumbering anything. The tightest subnet is Management, with 14 usable addresses for about 8 devices. If it grew beyond that, 10.10.99.0/28 could be widened to a /27 without touching any other subnet, because the addresses directly after it are free.

[^1]: Classless Inter-Domain Routing

---

[Previous: 3. VLANs vs subnets](03-vlans-vs-subnets.md) | [Next: 5. Firewalls](05-firewalls.md) | [Index](../README.md)
