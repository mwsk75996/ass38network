# LLD – Assignment 38 (SRX Tagged Interfaces and 802.1q trunk traffic)

Enhederne svarer til [HLD'en](HLD.pdf).

## R1 / SRX-C1

| Interface | Mode | VLAN | Tagging | Tilsluttet | Adresse | Zone |
|---|---|---|---|---|---|---|
| vlan.10 | l3-interface | SALES (10) | – | gateway | 192.168.10.1/24 | SALES |
| vlan.20 | l3-interface | OPERATIONS (20) | – | gateway | 192.168.20.1/24 | OPERATIONS |
| ge-0/0/2 | trunk | SALES (10) + OPERATIONS (20) | tagged | PC3 (Wireshark, test 10) | – | – |
| ge-0/0/3 | access | OPERATIONS (20) | untagged | ubrugt | – | – |
| ge-0/0/4 | access | OPERATIONS (20) | untagged | ubrugt | – | – |
| ge-0/0/5 | trunk | SALES (10) + OPERATIONS (20) | tagged | SRX-C2 ge-0/0/5 | – | – |
| ge-0/0/6 | access | SALES (10) | untagged | PC1 | – | – |
| ge-0/0/7 | access | SALES (10) | untagged | PC3 (Wireshark, test 11) | – | – |

## R2 / SRX-C2

| Interface | Mode | VLAN | Tagging | Tilsluttet |
|---|---|---|---|---|
| ge-0/0/3 | access | OPERATIONS (20) | untagged | PC2 |
| ge-0/0/4 | access | OPERATIONS (20) | untagged | ubrugt |
| ge-0/0/5 | trunk | SALES (10) + OPERATIONS (20) | tagged | SRX-C1 ge-0/0/5 |
| ge-0/0/6 | access | SALES (10) | untagged | PC2 i test 8–9 |
| ge-0/0/7 | access | SALES (10) | untagged | ubrugt |

SRX-C2 har ingen l3-interfaces, ingen adresser og ingen security-konfiguration.

## PC'er

| Enhed | Sidder i | VLAN | Adresse | Gateway |
|---|---|---|---|---|
| PC1 | SRX-C1 ge-0/0/6 | SALES (10) | 192.168.10.10/24 | 192.168.10.1 |
| PC2 | SRX-C2 ge-0/0/3 | OPERATIONS (20) | 192.168.20.5/24 | 192.168.20.1 |
| PC2 i test 8–9 | SRX-C2 ge-0/0/6 | SALES (10) | 192.168.10.11/24 | 192.168.10.1 |
| PC3 | SRX-C1 ge-0/0/2 (test 10) og ge-0/0/7 (test 11) | trunk / SALES | ingen, lytter kun | – |

## VLANs

| VLAN | ID | l3-interface | Porte på SRX-C1 | Porte på SRX-C2 |
|---|---|---|---|---|
| SALES | 10 | vlan.10 (kun SRX-C1) | ge-0/0/6, ge-0/0/7, ge-0/0/5 (t), ge-0/0/2 (t) | ge-0/0/6, ge-0/0/7, ge-0/0/5 (t) |
| OPERATIONS | 20 | vlan.20 (kun SRX-C1) | ge-0/0/3, ge-0/0/4, ge-0/0/5 (t), ge-0/0/2 (t) | ge-0/0/3, ge-0/0/4, ge-0/0/5 (t) |

(t) = tagged.

## Routing

Kun SRX-C1 router. Den har kun direkte forbundne netværk og ingen statiske ruter.

| Destination | Next-hop |
|---|---|
| 192.168.10.0/24 | vlan.10 (direkte) |
| 192.168.20.0/24 | vlan.20 (direkte) |

## Security policies (SRX-C1)

| Fra zone | Til zone | Policy | Applikationer | Handling |
|---|---|---|---|---|
| OPERATIONS | SALES | Allow_HTTP_and_ping | junos-http, junos-icmp-ping | permit |
| SALES | OPERATIONS | deny_all_traffic | any | deny |

Host-inbound-traffic: begge zoner tillader ping til SRX'en. Trafik inden for samme VLAN bliver switchet og rammer ikke en policy, heller ikke når den går over trunken.
