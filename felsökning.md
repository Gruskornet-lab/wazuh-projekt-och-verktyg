# Felsökning – Problem och lösningar

[← Tillbaka till README](../README.md)

En dokumentation av alla problem vi stötte på under projektet och hur de löstes.

---

## MrChromebox-skriptet

### Problem: "Read-Only file system" vid nedladdning
```
Warning: failed to open the file firmware-util.sh: Read-Only file system
curl: (23) Failure writing output to destination
```

**Orsak:** Rotkatalogen i Chrome OS VT2-skalet är skrivskyddad.

**Lösning:** Kör från `/tmp` istället:
```bash
cd /tmp; curl -LO mrchromebox.tech/firmware-util.sh
sudo install -Dt /usr/local/bin -m 755 firmware-util.sh
sudo firmware-util.sh
```

---

## Ubuntu Server installation

### Problem: Kan inte hitta USB i UEFI
Chromebooken startade till UEFI Shell (`Shell>`) istället för USB-minnet.

**Lösning:** Skriv `exit` i Shell för att komma till UEFI boot-menyn, välj sedan **Boot Manager** och välj USB-minnet.

### Problem: Backslash `\` fungerar inte på Chromebook med engelskt tangentbord
**Lösning:** Prova **AltGr + -** eller navigera i UEFI-menyn utan att skriva sökvägar manuellt.

---

## Disk full (eMMC)

### Problem: "No space left on device"
```
Fel: Skrivfel - write (28: No space left on device)
```

**Orsak:** Wazuh Manager tog upp 6.4 GB i `/var/ossec` på 32 GB eMMC.

**Lösning:** Flytta Wazuh-data till microSD:
```bash
# Montera microSD
sudo mkfs.ext4 /dev/mmcblk0p1
sudo mkdir -p /mnt/microsd
sudo mount /dev/mmcblk0p1 /mnt/microsd

# Flytta data och skapa symbolisk länk
sudo systemctl stop wazuh-manager
sudo mv /var/ossec /mnt/microsd/ossec
sudo ln -s /mnt/microsd/ossec /var/ossec
sudo systemctl start wazuh-manager
```

Frigör även utrymme med:
```bash
sudo apt clean
sudo journalctl --vacuum-size=100M
sudo apt autoremove -y
```

---

## Wazuh Indexer startar inte

### Problem 1: SSL-certifikat saknas
```
Unable to read the file /etc/wazuh-indexer/certs/root-ca.pem
```

**Orsak:** SSL-certifikat hade inte genererats och kopierats.

**Lösning:** Generera certifikat med `wazuh-certs-tool.sh` och kopiera till rätt plats:
```bash
bash wazuh-certs-tool.sh -A
sudo mkdir -p /etc/wazuh-indexer/certs
sudo cp ~/wazuh-certificates/* /etc/wazuh-indexer/certs/
sudo chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/certs
```

### Problem 2: Fel certifikatnamn i konfigurationen
```
Unable to read the file /etc/wazuh-indexer/certs/indexer.pem
```

**Orsak:** Konfigurationsfilen refererade till `indexer.pem` men certifikaten genererades som `node-1.pem`.

**Lösning:** Uppdatera `/etc/wazuh-indexer/opensearch.yml`:
```bash
sudo nano /etc/wazuh-indexer/opensearch.yml
```
Ändra `indexer.pem` → `node-1.pem` och `indexer-key.pem` → `node-1-key.pem`.

### Problem 3: För lite RAM
```
1.1G memory peak – process exited with error code
```

**Orsak:** Wazuh Indexer (OpenSearch) försökte använda mer RAM än tillgängligt.

**Lösning:** Begränsa JVM heap i `/etc/wazuh-indexer/jvm.options`:
```
-Xms512m
-Xmx512m
```

---

## Filebeat konfigurationsfil

### Problem: Filebeat.yml innehöll felmeddelande
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Error><Code>AccessDenied</Code><Message>Access Denied</Message></Error>
```

**Orsak:** URL:en för att ladda ner Filebeat-konfigurationen returnerade ett felmeddelande istället för konfigurationsfilen.

**Lösning:** Skriv konfigurationen manuellt:
```bash
sudo tee /etc/filebeat/filebeat.yml > /dev/null << 'EOF'
output.elasticsearch:
  hosts:
    - https://192.168.x.x:9200
  protocol: https
  username: admin
  password: "dittLösenord"
  ssl.certificate_authorities:
    - /etc/filebeat/certs/root-ca.pem
  ssl.certificate: /etc/filebeat/certs/wazuh-1.pem
  ssl.key: /etc/filebeat/certs/wazuh-1-key.pem
...
EOF
```

---

## Wazuh Dashboard – Status 500

### Problem: Dashboard returnerade HTTP 500
```
Error: Authentication Exception
```

**Orsak:** Dashboard kunde inte autentisera mot Wazuh Indexer – lösenordet stämde inte.

**Lösning:** Använd Wazuhs inbyggda lösenordsverktyg som uppdaterar ALLA konfigurationsfiler på en gång:
```bash
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh --change-all
```

> 💡 Lärdom: Försök ALDRIG ändra Wazuh-lösenord manuellt genom att redigera hashfiler. Använd alltid det inbyggda verktyget.

---

## Windows-agent installeras inte

### Problem: Wazuh-tjänsten hittades inte efter installation
```
Cannot find any service with service name 'Wazuh'
```

**Orsak:** Installationen misslyckades tyst utan felmeddelande.

**Lösning:** Kör installationen manuellt med loggning:
```powershell
msiexec.exe /i $env:tmp\wazuh-agent /l*v $env:tmp\wazuh-install.log WAZUH_MANAGER='192.168.x.x' WAZUH_AGENT_NAME='agentnamn'
```

Agenten behöver också en autentiseringsnyckel från Wazuh Manager:
```bash
sudo /var/ossec/bin/manage_agents
```

---

## Wazuh Dashboard – Inga index-mönster

### Problem: "No template found for the selected index-pattern"

**Orsak:** Inga larmindex existerade ännu – Filebeat hade inte skickat data till Indexer.

**Lösning:** Kör säkerhetsinitieringen och installera Filebeat korrekt:
```bash
sudo /usr/share/wazuh-indexer/bin/indexer-security-init.sh
```
