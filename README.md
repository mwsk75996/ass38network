# Assignment 38 – SRX Tagged Interfaces and 802.1q trunk traffic

Konfiguration til to SRX240 i JSON: `r1-config.json` og `r2-config.json`. Bygger videre på [Assignment 36](https://github.com/mwsk75996/ass36network) og [Assignment 37](https://github.com/mwsk75996/ass37network) med de samme porte, VLANs og adresser.

## Routerne

- **R1 / SRX-C1** – switcher og router. De to VLANs SALES og OPERATIONS har hver sit virtuelle interface (`vlan.10` og `vlan.20`), som er gateway for alle PC'er på begge SRX'er. Zoner og policies er de samme som i Assignment 37.
- **R2 / SRX-C2** – kun switch. Den har de samme to VLANs, men ingen adresser, ingen zoner og ingen routing. Det er det alternative design fra opgaven, hvor kun SRX240-A router.
- **Trunk:** ge-0/0/5 på SRX-C1 er forbundet til ge-0/0/5 på SRX-C2. Porten er sat til `port-mode trunk` med begge VLANs, så rammerne sendes med 802.1Q-tag (VLAN ID 10 eller 20).
- **Måleport:** ge-0/0/2 på SRX-C1 er også en trunk-port med begge VLANs. Her sad PC3 med Linux og Wireshark, så vi kunne se den taggede trafik uden at bryde trunken til SRX-C2.
- Alle andre porte er access-porte. Der sendes rammerne uden tag.

## Ændringer fra Assignment 37

| | Assignment 37 | Assignment 38 |
|---|---|---|
| Antal SRX | 1 | 2 |
| Trunk | Ingen | ge-0/0/5 ↔ ge-0/0/5, tagged, VLAN 10 + 20 |
| Måleport | Ingen | ge-0/0/2 på SRX-C1, tagged |
| PC'er | PC1 og PC2 på SRX-C1 | PC1 på SRX-C1, PC2 på SRX-C2, PC3 til Wireshark |
| Routing | SRX-C1 | Stadig kun SRX-C1 |

Opgavens tegning bruger `vlan.5` og `vlan.6`, 192.168.4.0/24 og 10.1.2.0/24, og ge-0/0/6 som trunk. Vi har beholdt vores egne net og porte fra Assignment 36 og 37 og lagt trunken på ge-0/0/5. Vi bruger statiske adresser og ingen DHCP.

## Inventar

- R1: Juniper SRX240H-POE, hostname SRX-C1, serienr. AH4113AK0093, Junos 12.1X46-D55.3
- R2: Juniper SRX240H, hostname SRX-C2, serienr. AG4809AA0482, Junos 12.1X46-D30.2
- PC1: Skolens Lenovo, MAC 68:f7:28:be:46:85, statisk 192.168.10.10/24
- PC2: Sebastians laptop, MAC 34:5a:60:29:94:d8, statisk 192.168.20.5/24 (192.168.10.11/24 i test 8–9)
- PC3: Thomas' ThinkPad med Linux og Wireshark 4.7.4, MAC 90:2e:16:68:ae:cd, ingen adresse
- 4 netværkskabler (RJ45): PC1, PC2, PC3 og trunken
- 2 konsolkabler (USB-seriel, 9600 8N1) til at indlæse konfigurationerne
- Wi-Fi er slået fra på PC1 og PC2 under testen

## Adresser (LLD)

| Enhed | VLAN | ID | Port | Tagging | Tilsluttet | Adresse |
|---|---|---|---|---|---|---|
| SRX-C1 | SALES | 10 | vlan.10 | – | gateway | 192.168.10.1/24 |
| SRX-C1 | OPERATIONS | 20 | vlan.20 | – | gateway | 192.168.20.1/24 |
| SRX-C1 | SALES + OPERATIONS | 10 + 20 | ge-0/0/5 | tagged | SRX-C2 ge-0/0/5 | – |
| SRX-C1 | SALES + OPERATIONS | 10 + 20 | ge-0/0/2 | tagged | PC3 (Wireshark, test 10) | – |
| SRX-C1 | SALES | 10 | ge-0/0/6 | untagged | PC1 | 192.168.10.10/24 |
| SRX-C1 | SALES | 10 | ge-0/0/7 | untagged | PC3 (Wireshark, test 11) | – |
| SRX-C2 | SALES + OPERATIONS | 10 + 20 | ge-0/0/5 | tagged | SRX-C1 ge-0/0/5 | – |
| SRX-C2 | OPERATIONS | 20 | ge-0/0/3 | untagged | PC2 | 192.168.20.5/24 |
| SRX-C2 | SALES | 10 | ge-0/0/6 | untagged | PC2 i test 8–9 | 192.168.10.11/24 |

Fuld LLD i [LLD.md](LLD.md).

## Filer

| Fil | Indhold |
|---|---|
| `r1-config.json` | Konfiguration til R1 / SRX-C1 (JSON) |
| `r2-config.json` | Konfiguration til R2 / SRX-C2 (JSON) |
| `r1-config.conf` / `r2-config.conf` | Den kørende konfiguration hentet fra routerne med `show configuration` |
| `r1-config.set` / `r2-config.set` | Set-kommandoer, kan indlæses med `load set terminal` |
| `set-commands.md` | Samme set-kommandoer til læsning |
| `HLD.pdf` | Side 1: HLD med tagged/untagged på alle forbindelser. Side 2: logisk diagram med hosts |
| `topology.jpg` / `logic.jpg` | De to diagrammer som billeder |
| `LLD.md` | Interfaces, VLANs, adresser, zoner og policies |
| `testplan.md` | Testplan med resultater og Wireshark-måling |
| `wireshark-*.png` | Screenshots fra Wireshark: tagged på trunken (ID 10 og ID 20) og untagged på access-port |
| `trunk-ge002.pcapng` / `access-ge007.pcapng` | Selve optagelserne, kan åbnes i Wireshark |
| `srx-output.txt` | Råt output fra begge SRX'er |

## Test (kort)

11/11 tests bestået: trunk og access-porte på begge SRX'er, MAC-adresser lært over trunken, gateway og routing gennem trunken, policy mellem zonerne, samme VLAN over trunken, og Wireshark der viser 802.1Q-tag (ID 10 og 20) på trunken og ingen tag på en access-port. Se `testplan.md`.
