# Hemma-SOC med Wazuh och Suricata

> Ett hemmalabb-projekt där en gammal HP Chromebook x360 11 G1 EE omvandlats till en fullständig säkerhetsserver med SIEM och IDS – från Chrome OS till ett fungerande Security Operations Center.

>  **Notering:** Detta projekt genomfördes med stöd av AI för vägledning och felsökning under installationsprocessen.

---

## Bakgrund

Projektet startade med en gammal Chromebook som inte längre fick säkerhetsuppdateringar (Chrome OS EOL sedan juni 2024). Istället för att låta den samla damm omvandlades den till en dedikerad säkerhetsserver för hemmanätverket.

Målet var att bygga praktisk erfarenhet av verktyg som används i riktiga SOC-miljöer – Wazuh för SIEM och Suricata för nätverksövervakning.

---

## Hårdvara

| Komponent | Specifikation |
|---|---|
| Enhet | HP Chromebook x360 11 G1 EE |
| CPU | Intel Celeron N3350 (dual-core, 1.1 GHz) |
| RAM | 4 GB LPDDR4 |
| Intern lagring | 32 GB eMMC |
| Extern lagring | 128 GB microSD (för loggdata) |
| OS | Ubuntu Server 24.04 LTS |
| Nätverksanslutning | USB-till-Ethernet adapter |

---

## Projektöversikt

```
Chromebook (Chrome OS EOL)
        ↓
Firmware ersatt med UEFI (MrChromebox)
        ↓
Ubuntu Server 24.04 LTS installerat
        ↓
Suricata IDS (nätverksövervakning)
        ↓
Wazuh SIEM (Manager + Indexer + Dashboard)
        ↓
Filebeat (dataöverföring)
        ↓
Windows 11-agent tillagd
        ↓
Hemma-SOC aktivt ✅
```

---

## Installationsprocess

### Fas 1 – Omvandla Chromebooken
 [Detaljerad guide](docs/chromebook-setup.md)

**Problemet:** HP Chromebook x360 11 G1 EE hade sitt Chrome OS EOL i juni 2024 och fick inga säkerhetsuppdateringar längre.

**Lösningen:** Ersätt Chrome OS med Ubuntu Server via [MrChromebox](https://mrchromebox.tech) UEFI-firmware.

Steg:
1. Skapade Chrome OS Recovery USB som säkerhetsnät
2. Aktiverade Developer Mode (raderar all data)
3. Öppnade chassit och kopplade ur batteriet för att inaktivera hardware write protection (CR50-chip)
4. Körde MrChromebox firmware-skript via VT2-terminal och valde UEFI Full ROM
5. Återmonterade chassit och installerade [Ubuntu Server 24.04 LTS](https://ubuntu.com/download/server) från USB
6. Konfigurerade SSH för fjärrstyrning

[Felsökning och problem vi stötte på](docs/felsökning.md)

---

### Fas 2 – Suricata IDS
[Detaljerad guide](docs/suricata-installation.md)

Suricata är ett open source Intrusion Detection System som analyserar nätverkstrafik i realtid och larmar vid misstänkta mönster.

```bash
sudo apt install suricata -y
sudo suricata-update
sudo systemctl restart suricata
```

---

### Fas 3 – Wazuh SIEM
[Detaljerad guide](docs/wazuh-installation.md)

Wazuh är en open source säkerhetsplattform för hotdetektering, incidentrespons och compliance.

| Komponent | Funktion |
|---|---|
| Wazuh Manager | Samlar in och analyserar säkerhetshändelser |
| Wazuh Indexer | Lagrar och indexerar data (OpenSearch-baserad) |
| Wazuh Dashboard | Webbgränssnitt för visualisering |

---

### Fas 4 – Windows 11-agent
[Detaljerad guide](docs/windows-agent.md)

Wazuh-agent installerades på Windows 11-huvuddatorn för att skicka säkerhetshändelser till Wazuh Manager.

---

## Arkitektur

```
[Windows 11 PC]
  Wazuh Agent
      |
      | (port 1514/1515)
      ↓
[Chromebook – Ubuntu Server]
  ┌─────────────────────────────┐
  │  Wazuh Manager              │
  │  ↓                          │
  │  Filebeat                   │
  │  ↓                          │
  │  Wazuh Indexer (OpenSearch) │
  │  ↓                          │
  │  Wazuh Dashboard (port 443) │
  └─────────────────────────────┘
      |
  Suricata (passiv nätverksövervakning)
```

---

## Lagringslayout

| Plats | Innehåll |
|---|---|
| eMMC (32 GB) | Ubuntu Server, alla program, konfigfiler |
| microSD (128 GB) | `/var/ossec` (Wazuh-data och loggar) |

---

## Säkerhet

- Alla tjänster lyssnar endast på lokalt nätverk
- SSL/TLS med egensignerade certifikat för all intern kommunikation
- Lösenordshantering via Wazuhs inbyggda verktyg

---

## Aktiva övervakade enheter

| Enhet | OS | Status |
|---|---|---|
| Chromebook (server) | Ubuntu Server 24.04 | ✅ Aktiv |
| Huvuddator | Windows 11 | ✅ Agent aktiv |

---

## Nästa steg

- [ ] Konfigurera `ufw` brandvägg
- [ ] SSH-nyckelautentisering
- [ ] Integrera Suricata-larm med Wazuh
- [ ] Lägga till fler agenter
- [ ] Automatiska e-postlarm vid kritiska händelser

---

## Användbara länkar

| Resurs | Länk |
|---|---|
| Ubuntu Server nedladdning | https://ubuntu.com/download/server |
| MrChromebox firmware | https://mrchromebox.tech |
| MrChromebox dokumentation | https://docs.mrchromebox.tech |
| Wazuh dokumentation | https://documentation.wazuh.com |
| Wazuh nedladdning | https://packages.wazuh.com |
| Suricata dokumentation | https://suricata.readthedocs.io |
| Balena Etcher | https://www.balena.io/etcher |

---

*Projekt av [@Gruskornet-lab](https://github.com/Gruskornet-lab)*
