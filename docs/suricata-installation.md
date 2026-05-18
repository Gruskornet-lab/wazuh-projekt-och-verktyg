# Suricata IDS – Installation och konfiguration

[← Tillbaka till README](../README.md)

## Vad är Suricata?

Suricata är ett open source Intrusion Detection System (IDS) som analyserar nätverkstrafik i realtid. Det jämför trafiken mot ett stort bibliotek av regler och larmar när det upptäcker misstänkta mönster.

## Resurser

| Resurs | Länk |
|---|---|
| Suricata officiell sajt | https://suricata.io |
| Suricata dokumentation | https://suricata.readthedocs.io |
| Suricata regler (Emerging Threats) | https://rules.emergingthreats.net |

---

## Installation

```bash
sudo apt update
sudo apt install suricata -y
```

Verifiera installation:
```bash
suricata --version
```

---

## Konfiguration

### Hitta nätverksgränssnitt

```bash
ip a
```

Leta efter gränssnittet med din lokala IP-adress (t.ex. `enx6c6e070fcb3d` för USB-Ethernet adapter).

### Uppdatera konfigurationsfilen

```bash
sudo nano /etc/suricata/suricata.yaml
```

Hitta och ändra:
```yaml
af-packet:
  - interface: enx6c6e070fcb3d  # Ersätt med ditt gränssnittsnamn
```

---

## Uppdatera regler

```bash
sudo suricata-update
sudo systemctl restart suricata
```

---

## Starta och aktivera

```bash
sudo systemctl enable suricata
sudo systemctl start suricata
```

---

## Verifiera att Suricata fungerar

Kontrollera status:
```bash
sudo systemctl status suricata
```

Kontrollera att trafik fångas:
```bash
sudo tail -f /var/log/suricata/stats.log
```

Leta efter `capture.kernel_packets` – om värdet ökar fungerar Suricata.

Larmlogg i realtid:
```bash
sudo tail -f /var/log/suricata/fast.log
```

> 💡 Tom `fast.log` är ett gott tecken – det betyder ingen misstänkt trafik har detekterats.
