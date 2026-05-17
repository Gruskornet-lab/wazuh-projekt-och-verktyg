# Hemma-SOC med Wazuh och Suricata

## Översikt
Ett hemmalabb-projekt där en HP Chromebook x360 11 G1 EE 
omvandlats till en säkerhetsserver med Wazuh SIEM och 
Suricata IDS.

## Hårdvara
- HP Chromebook x360 11 G1 EE (Intel Celeron N3350, 4GB RAM)
- 128GB microSD för logglagring
- Ubuntu Server 24.04 LTS

## Komponenter
- **Wazuh Manager** – samlar säkerhetshändelser
- **Wazuh Indexer** – lagrar data
- **Wazuh Dashboard** – visualiserar larm
- **Suricata** – nätverksövervakning och IDS
- **Filebeat** – skickar data från Manager till Indexer

## Agenter
- Windows 11 (huvuddator)

## Syfte
Praktisk erfarenhet av SIEM och IDS-verktyg som används 
i SOC-miljöer.