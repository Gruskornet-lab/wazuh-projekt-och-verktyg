# Wazuh SIEM – Installation och konfiguration

[← Tillbaka till README](../README.md)

## Vad är Wazuh?

Wazuh är en open source säkerhetsplattform för hotdetektering, incidentrespons och compliance. Det består av tre huvudkomponenter:

| Komponent | Funktion |
|---|---|
| Wazuh Manager | Samlar in och analyserar säkerhetshändelser från agenter |
| Wazuh Indexer | Lagrar och indexerar all data (OpenSearch-baserad) |
| Wazuh Dashboard | Webbgränssnitt för visualisering och analys |

## Resurser

| Resurs | Länk |
|---|---|
| Wazuh officiell sajt | https://wazuh.com |
| Wazuh dokumentation | https://documentation.wazuh.com |
| Wazuh paket | https://packages.wazuh.com |

---

## Förberedelser

### Lägg till Wazuh repository

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && sudo chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt update
```

### Generera SSL-certifikat

Ladda ner certifikatverktyget:
```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-certs-tool.sh
curl -sO https://packages.wazuh.com/4.12/config.yml
```

Redigera `config.yml` och ersätt alla node-namn och IP-adresser med serverns IP.

Generera certifikat:
```bash
bash wazuh-certs-tool.sh -A
```

---

## Installera Wazuh Manager

```bash
sudo apt install wazuh-manager -y
sudo systemctl daemon-reload
sudo systemctl enable wazuh-manager
sudo systemctl start wazuh-manager
```

---

## Installera Wazuh Indexer

```bash
sudo apt install wazuh-indexer -y
```

### Begränsa RAM-användning

> ⚠️ Viktigt på enheter med 4 GB RAM – Wazuh Indexer använder annars för mycket minne.

```bash
sudo nano /etc/wazuh-indexer/jvm.options
```

Ändra:
```
-Xms512m
-Xmx512m
```

### Kopiera certifikat

```bash
sudo mkdir -p /etc/wazuh-indexer/certs
sudo cp ~/wazuh-certificates/* /etc/wazuh-indexer/certs/
sudo chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/certs
```

### Uppdatera konfiguration

```bash
sudo nano /etc/wazuh-indexer/opensearch.yml
```

Se till att certifikatfilerna pekar på rätt namn (t.ex. `node-1.pem` istället för `indexer.pem`).

### Starta Wazuh Indexer

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-indexer
sudo systemctl start wazuh-indexer
```

### Initiera säkerhetsinställningar

```bash
sudo /usr/share/wazuh-indexer/bin/indexer-security-init.sh
```

---

## Installera Filebeat

Filebeat skickar data från Wazuh Manager till Wazuh Indexer.

```bash
sudo apt install filebeat -y
```

### Konfigurera Filebeat

```bash
sudo nano /etc/filebeat/filebeat.yml
```

Lägg till:
```yaml
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

filebeat.modules:
  - module: wazuh
    alerts:
      enabled: true
    archives:
      enabled: false

setup.template.json.enabled: true
setup.template.json.path: /etc/filebeat/wazuh-template.json
setup.template.json.name: wazuh
setup.template.overwrite: true
setup.ilm.enabled: false
```

### Kopiera certifikat

```bash
sudo mkdir -p /etc/filebeat/certs
sudo cp ~/wazuh-certificates/wazuh-1.pem /etc/filebeat/certs/
sudo cp ~/wazuh-certificates/wazuh-1-key.pem /etc/filebeat/certs/
sudo cp ~/wazuh-certificates/root-ca.pem /etc/filebeat/certs/
```

### Ladda ner Wazuh-mall

```bash
sudo curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/v4.12.0/extensions/elasticsearch/7.x/wazuh-template.json
```

### Ladda ner Wazuh-modul

```bash
sudo curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.4.tar.gz | sudo tar -xvz -C /usr/share/filebeat/module
```

### Starta Filebeat

```bash
sudo systemctl daemon-reload
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

Testa anslutningen:
```bash
sudo filebeat test output
```

---

## Installera Wazuh Dashboard

```bash
sudo apt install wazuh-dashboard -y
```

### Kopiera certifikat

```bash
sudo mkdir -p /etc/wazuh-dashboard/certs
sudo cp ~/wazuh-certificates/dashboard.pem /etc/wazuh-dashboard/certs/
sudo cp ~/wazuh-certificates/dashboard-key.pem /etc/wazuh-dashboard/certs/
sudo cp ~/wazuh-certificates/root-ca.pem /etc/wazuh-dashboard/certs/
sudo chown -R wazuh-dashboard:wazuh-dashboard /etc/wazuh-dashboard/certs
```

### Starta Dashboard

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-dashboard
sudo systemctl start wazuh-dashboard
```

Dashboard nås via webbläsaren på: `https://192.168.x.x`

---

## Lösenordshantering

> ⚠️ Använd alltid Wazuhs inbyggda verktyg för lösenordsbyten – det uppdaterar alla konfigurationsfiler automatiskt.

```bash
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh --change-all
```

---

## Flytta Wazuh-data till microSD

> Relevant om intern lagring är begränsad (t.ex. 32 GB eMMC).

```bash
sudo systemctl stop wazuh-manager
sudo mv /var/ossec /mnt/microsd/ossec
sudo ln -s /mnt/microsd/ossec /var/ossec
sudo systemctl start wazuh-manager
```
