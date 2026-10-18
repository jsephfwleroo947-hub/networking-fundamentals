[Back to index](../README.md)

# 5. Firewalls

## 5.1 What a firewall does

A firewall sits at the boundary between two networks and inspects the traffic trying to cross it. Common places are between a company's internal network and the internet, between a LAN and a DMZ, and between internal segments such as the office network and the server network. Think of it as a security checkpoint on the only road between two areas: everything has to pass through it, so everything can be checked.

For each piece of traffic, the firewall makes one decision: allow it or block it. It makes that decision by comparing the traffic against a list of rules, usually looking at where the traffic came from, where it is going, which port and protocol it is using, and, for a stateful firewall, whether it belongs to a connection that was already started from the inside. The rules are checked in order, the first one that matches wins, and anything that matches no rule is blocked. Most firewalls also log what they allowed and blocked, which is how problems are spotted and diagnosed afterwards.

## 5.2 Stateless vs stateful filtering

| **Aspect**                 | **Stateless**                                                                                                                                                           | **Stateful**                                                                                                                                                            |
|----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| How it decides             | Judges each packet on its own against the rules (IP’s, ports, protocol). Every packet is treated as if it were the first.                                               | Checks the first packet of connection against the rules, then judges later packets by whether they belong to a connection it already knows about.                       |
| What it remembers          | Nothing.                                                                                                                                                                | A state table of active connections (IPPs, ports, protocol and connection state).                                                                                       |
| Handling of return traffic | Needs its own explicit rule allowing replies in, usually a wide range of ports.                                                                                         | Allowed automatically if it matches a connection started from inside. Unsolicited inbound traffic is dropped.                                                           |
| Strengths and weaknesses   | Fast, simple and light on resources, but it can’t tell a real reply from a crafted lookalike – needs rules in both directions and tends to leave wide range ports open. | Tighter security and simpler rules since replies follow automatically, but it used memory from CPU for the state table which floods of connection attempts can exhaust. |

## 5.3 Core concepts

### 1. Default-deny vs default-allow

A firewall needs a rule for traffic that matches nothing else. With default-deny, anything not explicitly permitted is blocked, so you list only what you need and everything else is refused. With default-allow, anything not explicitly blocked is permitted, so you have to anticipate and block every threat. Default-deny is the safer approach because mistakes and forgotten services fail closed, and it's why the last row of a rule table is normally "Any, Any, Any, Deny".

Example: a web server needs inbound HTTPS only. Under default-deny, one rule allows TCP 443 to the server and everything else, including a forgotten database port or an unneeded remote-access service, is blocked automatically. Under default-allow, every one of those would have to be found and blocked individually.

### 2. Rule order and first-match processing

A firewall checks its rules from top to bottom and acts on the first rule that matches, ignoring the rest. This means the order matters as much as the content: a broad rule placed above a specific one can hide it completely, so specific rules usually go first and general ones later.

Example: the aim is to let the Staff VLAN reach the servers on port 443, except for a compromised PC at 10.10.10.50.

| Order | Rule                                    | Result                         |
|-------|-----------------------------------------|--------------------------------|
| 1     | Deny 10.10.10.50 to Servers, any port   | The compromised PC is blocked  |
| 2     | Allow 10.10.10.0/24 to Servers, TCP 443 | The rest of Staff gets through |
| 3     | Deny any to any                         | Default-deny                   |

If rules 1 and 2 were swapped, 10.10.10.50 would match the allow rule first and the deny would never be reached. (Section 5.5 works through this kind of swap in detail.)

### 3. NAT and port forwarding (and why NAT is not a security control by itself)

NAT (network address translation) rewrites addresses as traffic crosses a boundary. The most common form is source NAT with port translation (PAT, often called "overload"): many devices with private addresses share one public address, and the device keeps track of which source port belongs to which internal host so it can return replies to the right place. Port forwarding (destination NAT) works in the opposite direction: traffic arriving at the public address on a chosen port is rewritten and sent to a specific internal server.

Example: PC 10.10.10.5 and PC 10.10.10.6 both browse the web. The router translates both to the public address 203.0.113.2, using source ports 50000 and 50001 so it can tell the replies apart. A port forward rule such as "203.0.113.2 TCP 443 to 10.10.40.10 TCP 443" sends inbound HTTPS to a web server.

NAT is often mistaken for security because it has a side effect: with no translation entry, unsolicited inbound traffic has nowhere to go and is dropped. But that is an accident of how it works, not a designed control. NAT doesn't inspect or filter anything, port forwards deliberately open a path inwards, malware inside the network can still connect outwards, and IPv6 networks work without NAT at all. Treat NAT as an addressing tool and rely on firewall rules for security.

### 4. DMZ and why public-facing servers sit in it

A DMZ (demilitarised zone) is a separate network segment between the internet and the internal network, used for servers that must be reachable from outside, such as web, mail and DNS servers. These are the most exposed systems, so they are the most likely to be compromised. Putting them in their own segment means that if one falls, the attacker lands in the DMZ rather than on the internal network. The firewall rules enforce this: the internet may reach the DMZ only on specific ports, the DMZ may reach the internal network only where strictly necessary (or not at all), and the internal network may reach the DMZ for administration.

Example: the public web server at 10.10.40.10 sits in the DMZ subnet 10.10.40.0/27. The internet can reach it on TCP 443 only. It may reach one database server on the internal network on a single port, and nothing else inwards. If the web server is compromised, the attacker still can't scan the Staff or Servers VLANs.

### 5. Ingress vs egress filtering

Ingress filtering controls traffic entering a network, and egress filtering controls traffic leaving it. Most attention goes to ingress, but egress matters too. Controlling outbound traffic limits what malware can do once it's inside, because it can no longer freely contact its controller or send data out, and it stops your network sending spoofed traffic to others.

Example: an egress rule set might allow Staff to reach the internet on TCP 80 and 443 only, and permit DNS only to the organisation's own resolver. A compromised PC trying to send data over some unusual port, or to a different DNS server, is blocked and the attempt is logged. An ingress rule might drop any packet arriving from the internet with a source address in a private range (such as 10.x.x.x), since that can't be genuine.

## 5.4 Scenario and rule table

| **\#** | **Source**                | **Destination**                | **Port / protocol** | **Action** | **Reason**                                                                              |
|--------|---------------------------|--------------------------------|---------------------|------------|-----------------------------------------------------------------------------------------|
| 1      | Any (internet)            | 203.0.113.2 (firewall)         | UDP 51820           | Allow      | Lets remote workers establish the VPN tunnel                                            |
| 2      | Any (internet)            | 10.10.40.10 (web server)       | TCP 443             | Allow      | Public website, reached through a port forward from 203.0.113.2. Only HTTPS is exposed. |
| 3      | 10.10.40.10               | 10.10.20.10 (database)         | TCP 5432            | Allow      | The web server needs its database. One host, one port.                                  |
| 4      | 10.10.40.0/27 (DMZ)       | Staff, Servers and VPN subnets | Any                 | Deny       | If the DMZ is compromised, the attacker can't move into the internal networks.          |
| 5      | 10.10.10.0/24 (Staff)     | Internet                       | TCP 80, 443         | Allow      | Staff can browse the web. Limiting outbound ports is egress filtering.                  |
| 6      | 10.10.50.0/26 (VPN users) | 10.10.20.0/24 (Servers)        | TCP 443             | Allow      | Remote workers reach internal web applications only, not the whole network.             |

## 5.5 What if the rule order changes?

The swap. In the original table, rule 3 allows the web server (10.10.40.10) to reach the database (10.10.20.10) on TCP 5432, and rule 4 denies the whole DMZ from reaching any internal subnet. The firewall checks rules from top to bottom and stops at the first match. I'll swap them so the broad deny comes first.

| \#  | Source              | Destination                    | Port / protocol | Action |
|-----|---------------------|--------------------------------|-----------------|--------|
| 3   | 10.10.40.0/27 (DMZ) | Staff, Servers and VPN subnets | Any             | Deny   |
| 4   | 10.10.40.10         | 10.10.20.10                    | TCP 5432        | Allow  |

What happens to one packet. The web server sends a database request: source 10.10.40.10, destination 10.10.20.10, TCP 5432. Before the swap, it matched rule 3 and was allowed. After the swap, it matches the deny first, because the source is inside 10.10.40.0/27 and the destination is inside the Servers subnet. The firewall drops it and never looks at the allow rule below, which is now unreachable (this is called a shadowed rule). Because the connection never starts, there is no entry in the state table and no reply.

What breaks. The public website still answers on port 443, since rule 2 is untouched, so the site appears to be up. But any page that needs data from the database hangs or shows an error, while static pages still load. This is a confusing fault because the front door works and the failure is behind it. The firewall log would show repeated drops from 10.10.40.10 to 10.10.20.10 on port 5432 attributed to the deny rule, which is the clue that points to the cause. The fix is to move the specific allow back above the broad deny.

Why this failure is the safe kind. Nothing was exposed: the swap broke a function but the network stayed closed. That is the benefit of default-deny, because mistakes tend to fail by blocking things, which people notice and report. The dangerous direction is the reverse. In section 5.3, rule 1 denied the compromised PC 10.10.10.50 and rule 2 allowed the rest of the Staff subnet to the servers on port 443. If those two were swapped, 10.10.10.50 would match the allow rule first, and the deny would never be reached. Nothing would appear broken, the compromised PC would keep its access, and the mistake would only be found by someone reading the rules. A specific deny placed below a broader allow is how a firewall quietly exposes something.

## 5.6 Diagram

![Figure 3: a stateful firewall between the internet, a DMZ and three internal zones, with numbered traffic flows](../images/figure-3-firewall-zones.png)

*Figure 3: A stateful firewall separating the internet, a DMZ and three internal zones. Numbered flows match the rule table in 5.4, and the red cross marks traffic that is denied.*

---

[Previous: 4. Subnetting](04-subnetting.md) | [Next: 6. Troubleshooting case studies](06-troubleshooting-case-studies.md) | [Index](../README.md)
