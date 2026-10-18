# Networking Fundamentals from an infrastructure perspective

LAN, WAN, VLANs, subnetting and firewalls: a knowledge portfolio.

This project explains how core networking building blocks work and how they fit together, and shows the understanding through explanation, worked subnetting problems, firewall rule design and fault diagnosis. It is written as a set of short documents, with diagrams drawn for the project.

**Author:** Joseph | **Version:** 0.1 | **Date:** 16/10/2026

> All IP addresses, hostnames and organisations used in examples are fictional, and the scenarios are written for learning purposes.

## Contents

1. [About this project](docs/01-about-this-project.md): Purpose, scope, how it was learned, and key terms
2. [LAN vs WAN](docs/02-lan-vs-wan.md): Ownership, scale and devices; where the OSI model fits
3. [VLANs vs subnets](docs/03-vlans-vs-subnets.md): Layer 2 vs Layer 3, why they are paired, inter-VLAN routing
4. [Subnetting](docs/04-subnetting.md): Binary, CIDR, host counts, VLSM, worked problems and an addressing plan
5. [Firewalls](docs/05-firewalls.md): Stateless vs stateful, core concepts, a rule table and rule-order effects
6. [Troubleshooting case studies](docs/06-troubleshooting-case-studies.md): Four fault scenarios with symptoms, diagnosis, cause and fix
7. [Self-test](docs/07-self-test.md): Ten questions with model answers
8. [Reflection](docs/08-reflection.md): Mistakes made and what I would explore next
9. [References](docs/09-references.md): Sources

## Figures

| Figure | Description | Editable source |
|---|---|---|
| 1 | Two LANs joined across a WAN | [lan-wan-diagram.drawio](diagrams/lan-wan-diagram.drawio) |
| 2 | One switch divided into three VLANs | [vlan-diagram.drawio](diagrams/vlan-diagram.drawio) |
| 3 | Firewall zones and traffic flows | [firewall-zones-diagram.drawio](diagrams/firewall-zones-diagram.drawio) |

The `.drawio` files open in [diagrams.net](https://app.diagrams.net). PNG and SVG exports sit alongside them in [`diagrams/`](diagrams).

## Repository layout

```
.
├── README.md
├── docs/        one file per section of the document
├── images/      figures as used in the documents
└── diagrams/    editable .drawio sources plus .png and .svg exports
```
