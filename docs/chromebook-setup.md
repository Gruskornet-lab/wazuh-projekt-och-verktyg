# Chromebook Setup – Från Chrome OS till Ubuntu Server

[← Tillbaka till README](../README.md)

## Verktyg och resurser

| Verktyg | Länk |
|---|---|
| MrChromebox firmware | https://mrchromebox.tech |
| MrChromebox dokumentation | https://docs.mrchromebox.tech |
| Ubuntu Server 24.04 LTS | https://ubuntu.com/download/server |
| Balena Etcher (skriva USB) | https://www.balena.io/etcher |
| Chromebook Recovery Utility | Chrome Web Store |

---

## Steg 1 – Skapa Chrome OS Recovery USB

Innan något annat – skapa ett säkerhetsnät.

1. Öppna Chrome-webbläsaren på huvuddatorn
2. Installera **Chromebook Recovery Utility** från Chrome Web Store
3. Starta verktyget → klicka Kom igång
4. Skriv in modellnamnet: `SNAPPY`
5. Välj USB-minne och låt det skriva klart

---

## Steg 2 – Aktivera Developer Mode

⚠️ **All data på Chromebooken raderas.**

1. Stäng av Chromebooken
2. Håll inne **ESC + Refresh (F3) + Power**
3. Tryck **CTRL + D** på recovery-skärmen
4. Tryck **Enter** för att bekräfta
5. Vänta 5–10 minuter

---

## Steg 3 – Inaktivera Write Protection

Din enhet använder ett CR50-chip. Write protection inaktiveras genom att koppla ur batteriet fysiskt.

1. Stäng av Chromebooken och koppla ur laddaren
2. Öppna chassit (Torx T5-skruvar på undersidan)
3. Koppla ur batterikontakten från moderkortet
4. Sätt tillbaka bottenplattan löst
5. Koppla in USB-C laddaren (ersätter batteriet som strömkälla)

---

## Steg 4 – Kör MrChromebox-skriptet

1. Starta Chromebooken med laddaren inkopplad
2. Gå igenom Chrome OS-setup minimalt
3. På inloggningsskärmen – tryck **CTRL + ALT + F2**
4. Logga in som `root`
5. Kör:

```bash
cd /tmp; curl -LO mrchromebox.tech/firmware-util.sh
sudo install -Dt /usr/local/bin -m 755 firmware-util.sh
sudo firmware-util.sh
```

6. Välj **2) Install/Update UEFI (Full ROM) Firmware**
7. Stäng av enheten när det är klart

> ⚠️ Notering: Skriptet kan inte köras från Crostini (Chrome OS Linux-beta) – måste köras från VT2-terminalen.

---

## Steg 5 – Återmontera och skapa Ubuntu USB

1. Återanslut batterikontakten
2. Sätt tillbaka bottenplattan och skruva i alla skruvar
3. På huvuddatorn – ladda ner [Ubuntu Server 24.04 LTS](https://ubuntu.com/download/server)
4. Använd [Balena Etcher](https://www.balena.io/etcher) för att skriva ISO till USB

> 💡 Tips: Windows kommer fråga om du vill formatera USB-minnet efter att Etcher är klart – välj NEJ.

---

## Steg 6 – Installera Ubuntu Server

1. Sätt i USB-minnet i Chromebooken
2. Starta – tryck **F10** eller **ESC** för bootmeny
3. Välj USB-minnet
4. Välj **Try or Install Ubuntu Server**

Inställningar under installation:

| Val | Inställning |
|---|---|
| Språk | English |
| Tangentbord | Swedish |
| Installation type | Ubuntu Server (inte Minimized) |
| Storage | Use entire disk → `mmcblk1` (eMMC) |
| SSH | ✅ Install OpenSSH server |
| Snap packages | Hoppa över |

> ⚠️ Välj `mmcblk1` (eMMC, 29GB) och INTE `mmcblk0` (microSD).

---

## Steg 7 – Konfiguration efter installation

### Förhindra viloläge vid stängt lock

```bash
sudo nano /etc/systemd/logind.conf
```

Ändra/lägg till:
```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
```

```bash
sudo systemctl restart systemd-logind
```

### Hitta IP-adress

```bash
ip a
```

### SSH in från huvuddatorn

```bash
ssh användarnamn@192.168.x.x
```
