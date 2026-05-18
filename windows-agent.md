# Wazuh Windows 11-agent – Installation

[← Tillbaka till README](../README.md)

## Förberedelser

Innan du installerar agenten behöver du:
- Wazuh Manager igång på servern
- Administratörsbehörighet på Windows-datorn
- Serverns IP-adress

---

## Generera installationskommando

1. Öppna Wazuh Dashboard i webbläsaren: `https://192.168.x.x`
2. Gå till **Agents** → **Deploy new agent**
3. Välj **Windows**
4. Fyll i serverns IP-adress och ett namn för agenten
5. Kopiera det genererade PowerShell-kommandot

---

## Installera agenten

Öppna **PowerShell som administratör** på Windows-datorn och kör kommandot från Dashboard. Det ser ut ungefär så här:

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.x.x-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='192.168.x.x' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='dittAgentnamn'
```

---

## Hämta autentiseringsnyckel

SSH:a in på servern och kör:

```bash
sudo /var/ossec/bin/manage_agents
```

- Välj **A** för att lägga till agent
- Fyll i namn och IP för Windows-datorn
- Välj **E** för att exportera nyckeln
- Kopiera nyckeln och klistra in den i installationsfönstret på Windows

---

## Verifiera

Kontrollera i Wazuh Dashboard under **Agents** att agenten visas som aktiv.

---

## Felsökning

Om agenten inte syns i Dashboard:

```powershell
# Kontrollera om installationsfilen finns
Test-Path "$env:tmp\wazuh-agent"

# Kontrollera om tjänsten finns
Get-Service | Where-Object {$_.DisplayName -like "*ossec*" -or $_.DisplayName -like "*Wazuh*"}

# Kontrollera installationsmappen
Test-Path "C:\Program Files (x86)\ossec-agent"
```
