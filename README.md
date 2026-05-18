# 🛡️ Hemma-SOC med Wazuh och Suricata

> Ett hemmalabb-projekt där en gammal HP Chromebook x360 11 G1 EE omvandlats till en fullständig säkerhetsserver med SIEM och IDS – från Chrome OS till ett fungerande Security Operations Center.

---

## 📖 Bakgrund

Projektet startade med en gammal Chromebook som inte längre fick säkerhetsuppdateringar (Chrome OS EOL sedan juni 2024). Istället för att låta den samla damm omvandlades den till en dedikerad säkerhetsserver för hemmanätverket.

Målet var att bygga praktisk erfarenhet av verktyg som används i riktiga SOC-miljöer – Wazuh för SIEM och Suricata för nätverksövervakning.

---

## 🖥️ Hårdvara

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

## 🗺️ Projektöversikt

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

## ⚙️ Installationsprocess

### Fas 1 – Omvandla Chromebooken

**Problemet:** HP Chromebook x360 11 G1 EE hade sitt Chrome OS EOL i juni 2024 och fick inga säkerhetsuppdateringar längre.

**Lösningen:** Ersätt Chrome OS med Ubuntu Server via MrChromebox UEFI-firmware.

Steg:
1. Skapade Chrome OS Recovery USB som säkerhetsnät
2. Aktiverade Developer Mode (raderar all data)
3. Öppnade chassit och kopplade ur batteriet för att inaktivera hardware write protection (CR50-chip)
4. Körde MrChromebox firmware-skript via VT2-terminal och valde UEFI Full ROM
5. Återmonterade chassit och installerade Ubuntu Server 24.04 LTS från USB
6. Konfigurerade SSH för fjärrstyrning

**Utmaning:** Chromebooken saknar RJ45-port – löst med USB-till-Ethernet adapter för stabil serveranslutning.

**Utmaning:** 32 GB eMMC fylldes upp av Wazuh – löst genom att flytta `/var/ossec` till 128 GB microSD och skapa en symbolisk länk.

---

### Fas 2 – Suricata IDS

Suricata är ett open source Intrusion Detection System som analyserar nätverkstrafik i realtid och larmar vid misstänkta mönster.

**Installation:**
```bash
sudo apt install suricata -y
```

**Konfiguration:**
- Nätverksgränssnitt satt till `enx6c6e070fcb3d` (USB-Ethernet adapter)
- Regler uppdaterade via `suricata-update`

**Verifiering:**
```bash
sudo systemctl status suricata
sudo tail -f /var/log/suricata/stats.log
```

---

### Fas 3 – Wazuh SIEM

Wazuh är en open source säkerhetsplattform för hotdetektering, incidentrespons och compliance. Installerades med tre komponenter:

| Komponent | Funktion |
|---|---|
| Wazuh Manager | Samlar in och analyserar säkerhetshändelser |
| Wazuh Indexer | Lagrar och indexerar data (OpenSearch-baserad) |
| Wazuh Dashboard | Webbgränssnitt för visualisering |

**Utmaning:** Wazuh Indexer (OpenSearch) kraschade på grund av minnesbegränsning – löst genom att sätta JVM heap till 512MB i `/etc/wazuh-indexer/jvm.options`.

**Utmaning:** SSL-certifikat saknades vid första start – löst med `wazuh-certs-tool.sh` för att generera egensignerade certifikat.

**Utmaning:** Lösenordsbyte – löst med Wazuhs inbyggda verktyg:
```bash
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh --change-all
```

---

### Fas 4 – Filebeat

Filebeat fungerar som bryggan mellan Wazuh Manager och Wazuh Indexer – det skickar loggar och larm från Manager till Indexer för lagring och analys.

```bash
sudo apt install filebeat -y
```

---

### Fas 5 – Windows 11-agent

Wazuh-agent installerades på Windows 11-huvuddatorn för att skicka säkerhetshändelser till Wazuh Manager.

Agenten genererades via Wazuh Dashboard och installerades via PowerShell som administratör.

---

## 🏗️ Arkitektur

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

## 🗂️ Lagringslayout

| Plats | Innehåll |
|---|---|
| eMMC (32 GB) | Ubuntu Server, alla program, konfigfiler |
| microSD (128 GB) | `/var/ossec` (Wazuh-data och loggar) |

---

## 🔒 Säkerhet

- Alla tjänster lyssnar endast på lokalt nätverk – ingen exponering mot internet
- SSL/TLS med egensignerade certifikat för all intern kommunikation
- SSH-åtkomst via nyckel rekommenderas som nästa steg
- Brandvägg (ufw) planeras som nästa förbättring

---

## 📊 Aktiva övervakade enheter

| Enhet | OS | Status |
|---|---|---|
| Chromebook (server) | Ubuntu Server 24.04 | ✅ Aktiv |
| Huvuddator | Windows 11 | ✅ Agent aktiv |

---

## 🚀 Nästa steg

- [ ] Konfigurera `ufw` brandvägg
- [ ] SSH-nyckelautentisering istället för lösenord
- [ ] Integrera Suricata-larm med Wazuh
- [ ] Lägga till fler agenter
- [ ] Sätta upp automatiska e-postlarm vid kritiska händelser
- [ ] Dokumentera intressanta larm och händelser

---

## 🛠️ Använda verktyg

| Verktyg | Version | Syfte |
|---|---|---|
| Ubuntu Server | 24.04 LTS | Operativsystem |
| Wazuh | 4.x | SIEM |
| Suricata | Latest | IDS |
| Filebeat | Latest | Dataöverföring |
| MrChromebox | Latest | UEFI-firmware |

---

## 📚 Lärdomar

- **Hårdvarubegränsningar** – 32 GB eMMC är knappt för ett fullt Wazuh-stack. Extern lagring via microSD löste problemet.
- **RAM-begränsningar** – OpenSearch/Wazuh Indexer är minnesintensivt. JVM heap-storlek måste begränsas på hårdvara med 4 GB RAM.
- **Använd inbyggda verktyg** – Wazuhs `wazuh-passwords-tool.sh` hanterar lösenordsbyten korrekt i alla konfigurationsfiler automatiskt, till skillnad från manuell redigering.
- **Praktisk SOC-erfarenhet** – Att sätta upp och felsöka ett riktigt SIEM-system ger betydligt djupare förståelse än att bara läsa om det.

---

*Projekt av [@Gruskornet-lab](https://github.com/Gruskornet-lab)*
