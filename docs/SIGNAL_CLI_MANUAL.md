# signal-cli Manuell Konfiguration

Detta dokument beskriver hur du konfigurerar Signal manuellt med signal-cli från kommandoraden, specifikt för onboarding av nya användare till Oden.

## Förutsättningar

### Java 25+

signal-cli 0.14.1 (som används av Oden) kräver **Java 25 eller senare**.

**Kontrollera din Java-version:**

```bash
java -version
```

**Installera Java:**

- **macOS (Homebrew):**
  ```bash
  brew install openjdk
  ```

- **Linux (Ubuntu/Debian):**
  ```bash
  sudo apt update
  sudo apt install openjdk-25-jre
  ```

- **Andra system:**
  Ladda ner från [Adoptium](https://adoptium.net/) eller [Oracle](https://www.oracle.com/java/technologies/downloads/)

### signal-cli Installation

**Automatisk installation (via Oden-skript):**

Oden's `scripts/run_mac.sh` laddar ner signal-cli automatiskt om det inte finns.

**Manuell installation:**

1. Ladda ner signal-cli 0.14.1:
   ```bash
   cd ~/Downloads
   wget https://github.com/AsamK/signal-cli/releases/download/v0.14.1/signal-cli-0.14.1.tar.gz
   ```

2. Extrahera:
   ```bash
   tar -xzf signal-cli-0.14.1.tar.gz
   ```

3. Flytta till önskad plats (t.ex. `/usr/local/`):
   ```bash
   sudo mv signal-cli-0.14.1 /usr/local/
   sudo ln -s /usr/local/signal-cli-0.14.1/bin/signal-cli /usr/local/bin/signal-cli
   ```

4. Verifiera installation:
   ```bash
   signal-cli --version
   ```

---

## Onboarding: Två Alternativ

Det finns två sätt att konfigurera ett Signal-konto för Oden:

1. **Länka till befintligt konto** (Rekommenderat)
2. **Registrera nytt telefonnummer**

### ⚠️ Viktiga varningar

- **Använd INTE ditt privata Signal-nummer** — skaffa ett dedikerat nummer för Oden
- **Rekommenderat:** Länka till ett befintligt konto istället för att registrera ett nytt nummer
- **Varning:** Om du registrerar ett nummer utan att först ha det i Signal-appen på en telefon blir Oden den enda enheten, vilket kan orsaka problem med meddelandesynkronisering och krypteringsnycklar

---

## Alternativ 1: Länka till Befintligt Konto (Rekommenderat)

Detta kopplar Oden som en länkad enhet till din befintliga Signal-installation (på telefon eller annan enhet).

### Steg 1: Starta länkningsprocessen

```bash
signal-cli link -n "Oden"
```

- `-n "Oden"` sätter enhetsnamnet till "Oden" (visas i Signal-appen under "Länkade enheter")
- En QR-kod visas i terminalen

### Steg 2: Scanna QR-koden

**På din Signal-telefon:**

1. Öppna Signal-appen
2. Gå till **Inställningar** → **Länkade enheter**
3. Tryck på **"+"** eller **"Länka ny enhet"**
4. Scanna QR-koden som visas i terminalen

### Steg 3: Vänta på länkning

signal-cli väntar automatiskt på att länkningen ska slutföras. När den är klar visas meddelandet:

```
Associated with: +46701234567
```

Detta är ditt telefonnummer som Oden kommer att använda.

### Steg 4: Konfigurera Oden

Om du kör Oden's setup-wizard kan du nu:
- Ange telefonnumret som visades
- Eller hoppa över detta steg om setup-wizarden automatiskt detekterade numret

---

## Alternativ 2: Registrera Nytt Telefonnummer

⚠️ **Varning:** Endast rekommenderat om du har ett dedikerat telefonnummer och förstår konsekvenserna av att inte ha numret i Signal-appen först.

### Steg 1: Registrera numret

**SMS-verifiering (standard):**

```bash
signal-cli -u "+46701234567" register
```

**Röstsamtal-verifiering:**

Om du inte kan ta emot SMS:

```bash
signal-cli -u "+46701234567" register --voice
```

**CAPTCHA-hantering:**

Signal kan begära CAPTCHA-verifiering. Om du får meddelandet:

```
Captcha required for verification (https://signalcaptchas.org/registration/generate.html)
```

1. Öppna länken i en webbläsare
2. Lös CAPTCHA-utmaningen
3. Kopiera den genererade token (börjar med `signalcaptcha://`)
4. Kör:
   ```bash
   signal-cli -u "+46701234567" register --captcha "signalcaptcha://signal-hcaptcha..."
   ```

### Steg 2: Verifiera med SMS-kod

Signal skickar en 6-siffrig verifieringskod till ditt telefonnummer via SMS (eller röstsamtal).

```bash
signal-cli -u "+46701234567" verify 123456
```

Ersätt `123456` med den kod du fick.

### Steg 3: Konfigurera profil (valfritt)

Sätt visningsnamn:

```bash
signal-cli -u "+46701234567" updateProfile --name "Oden Bot"
```

---

## Verifiera Konfigurationen

Efter framgångsrik länkning eller registrering kan du testa att signal-cli fungerar:

### Lista grupper

```bash
signal-cli -u "+46701234567" listGroups
```

### Skicka testmeddelande till dig själv

```bash
signal-cli -u "+46701234567" send -m "Test från Oden" "+46701234567"
```

### Starta daemon-läge (som Oden använder)

```bash
signal-cli -u "+46701234567" daemon --tcp 127.0.0.1:7583 --receive-mode on-connection
```

Tryck `Ctrl+C` för att stoppa daemon-läget.

---

## Oden-specifik Konfiguration

### Var sparas Signal-data?

signal-cli sparar all data (krypteringsnycklar, kontakter, meddelanden) i:

```
~/.oden/signal-data/data/+46701234567
```

Oden konfigurerar automatiskt miljövariabeln `SIGNAL_CLI_CONFIG_DIR` till `~/.oden/signal-data`.

### Manuell konfiguration

Om du kört signal-cli manuellt **utanför Oden** och vill flytta konfigurationen:

1. Hitta din signal-cli-data (vanligen `~/.local/share/signal-cli/data/`)
2. Kopiera till Oden's katalog:
   ```bash
   mkdir -p ~/.oden/signal-data
   cp -r ~/.local/share/signal-cli/data ~/.oden/signal-data/
   ```

### Anpassa signal-cli-sökväg

Om signal-cli är installerat på en icke-standard plats kan du konfigurera sökvägen i Oden:

**Via Web GUI:**
1. Öppna `http://127.0.0.1:8080`
2. Gå till **Konfiguration** → **Avancerat**
3. Sätt `signal_cli_path` till den fullständiga sökvägen (t.ex. `/usr/local/bin/signal-cli`)

**Via miljövariabel:**
```bash
export SIGNAL_CLI_PATH="/opt/signal-cli/bin/signal-cli"
python -m oden
```

---

## JSON-RPC Kommunikation

Oden kommunicerar med signal-cli via JSON-RPC 2.0 över TCP-socket (standard: `127.0.0.1:7583`).

### Exempel: Skicka meddelande

```json
{
  "jsonrpc": "2.0",
  "method": "send",
  "params": {
    "recipient": ["+46709876543"],
    "message": "Hej från Oden!"
  },
  "id": "msg-001"
}
```

### Exempel: Ta emot meddelanden

signal-cli skickar notifikationer i real-tid:

```json
{
  "jsonrpc": "2.0",
  "method": "receive",
  "params": {
    "envelope": {
      "source": "+46709876543",
      "sourceNumber": "+46709876543",
      "sourceUuid": "...",
      "timestamp": 1710534000000,
      "dataMessage": {
        "timestamp": 1710534000000,
        "message": "Rapporterar händelse X",
        "groupInfo": {
          "groupId": "...",
          "type": "DELIVER"
        }
      }
    },
    "account": "+46701234567"
  }
}
```

---

## Felsökning

### Problem: "Invalid ACI" eller "Unknown recipient"

**Orsak:** signal-cli's interna databas är out-of-sync med Signal-servern.

**Lösning:**

```bash
# Ta bort cachad kontaktinformation
rm -rf ~/.oden/signal-data/data/+46701234567/recipients-store
rm -rf ~/.oden/signal-data/data/+46701234567/groups.json

# Starta om Oden
python -m oden
```

### Problem: "Failed to send message" efter registrering

**Orsak:** Signal kräver att du först skickar ett meddelande från telefonen.

**Lösning:**

1. Installera Signal-appen på en telefon med samma nummer
2. Skicka ett meddelande till valfri kontakt
3. Detta aktiverar kontot fullständigt
4. Försök igen med signal-cli

### Problem: "Java version too old"

**Orsak:** signal-cli 0.14.1 kräver Java 25+.

**Lösning:**

```bash
# macOS
brew install openjdk

# Ubuntu/Debian
sudo apt install openjdk-25-jre

# Verifiera
java -version
```

### Problem: signal-cli hittas inte

**Orsak:** signal-cli finns inte i `PATH`.

**Lösning:**

```bash
# Hitta signal-cli
which signal-cli

# Om inget hittas, installera eller länka:
sudo ln -s /path/to/signal-cli-0.14.1/bin/signal-cli /usr/local/bin/signal-cli
```

### Problem: "Connection refused" när Oden startar

**Orsak:** signal-cli-daemon körs inte eller TCP-porten (7583) är blockerad.

**Lösning:**

1. Kontrollera att signal-cli startar:
   ```bash
   signal-cli -u "+46701234567" daemon --tcp 127.0.0.1:7583
   ```

2. Testa TCP-anslutning:
   ```bash
   nc -zv 127.0.0.1 7583
   ```

3. Kontrollera att ingen annan process använder port 7583:
   ```bash
   lsof -i :7583
   ```

---

## Avancerad Konfiguration

### Använd extern signal-cli (unmanaged mode)

Om du vill köra signal-cli separat (inte via Oden):

1. Starta signal-cli manuellt:
   ```bash
   signal-cli -u "+46701234567" daemon --tcp 127.0.0.1:7583 --receive-mode on-connection
   ```

2. Konfigurera Oden att inte hantera signal-cli:
   ```python
   # I config.db eller via Web GUI
   unmanaged_signal_cli = True
   ```

3. Starta Oden:
   ```bash
   python -m oden
   ```

Oden ansluter nu till den redan körande signal-cli-instansen.

### Ändra TCP-port

Standard-port är 7583. För att använda en annan port:

**Via Web GUI:**
1. Konfiguration → Avancerat
2. Sätt `signal_cli_port` till önskad port (t.ex. `8583`)

**Via miljövariabel:**
```bash
export SIGNAL_CLI_PORT=8583
python -m oden
```

**Starta signal-cli med anpassad port:**
```bash
signal-cli -u "+46701234567" daemon --tcp 127.0.0.1:8583
```

### Logga signal-cli-output

För felsökning kan du logga signal-cli's output:

**Via Web GUI:**
1. Konfiguration → Avancerat
2. Sätt `signal_cli_log_file` till sökväg (t.ex. `/tmp/signal-cli.log`)

**Manuellt:**
```bash
signal-cli -u "+46701234567" daemon --tcp 127.0.0.1:7583 2>&1 | tee /tmp/signal-cli.log
```

---

## Återställning och Migrering

### Backup av Signal-konfiguration

Signal-konfigurationen (krypteringsnycklar, kontakter, gruppcache) finns i:

```
~/.oden/signal-data/data/+46701234567/
```

**Backup:**

```bash
tar -czf signal-backup-$(date +%Y%m%d).tar.gz ~/.oden/signal-data/
```

**Återställ:**

```bash
tar -xzf signal-backup-20260315.tar.gz -C ~/
```

### Migrera från annan signal-cli-installation

Om du redan använder signal-cli någon annanstans:

1. **Hitta gammal konfiguration:**
   ```bash
   ls -la ~/.local/share/signal-cli/data/
   ```

2. **Kopiera till Oden:**
   ```bash
   mkdir -p ~/.oden/signal-data
   cp -r ~/.local/share/signal-cli/data ~/.oden/signal-data/
   ```

3. **Verifiera:**
   ```bash
   signal-cli -u "+46701234567" --config ~/.oden/signal-data listGroups
   ```

### Avlänka enhet

För att ta bort Oden från ditt Signal-konto:

**Från Signal-telefon:**
1. Inställningar → Länkade enheter
2. Välj "Oden"
3. Tryck "Ta bort enhet"

**Från signal-cli:**
```bash
# Detta raderar lokal konfiguration
rm -rf ~/.oden/signal-data/data/+46701234567/
```

---

## Kommandon för Utvecklare

### Testa JSON-RPC direkt

Du kan skicka JSON-RPC-kommandon manuellt till signal-cli via TCP:

**Exempel med `netcat`:**

```bash
# Starta signal-cli daemon i en terminal
signal-cli -u "+46701234567" daemon --tcp 127.0.0.1:7583

# I en annan terminal, skicka JSON-RPC
echo '{"jsonrpc":"2.0","method":"send","params":{"recipient":["+46709876543"],"message":"Test"},"id":"1"}' | nc 127.0.0.1 7583
```

**Exempel med Python:**

```python
import socket
import json

# Anslut till signal-cli
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('127.0.0.1', 7583))

# Skicka meddelande
request = {
    "jsonrpc": "2.0",
    "method": "send",
    "params": {
        "recipient": ["+46709876543"],
        "message": "Hej från Python!"
    },
    "id": "test-001"
}

sock.sendall((json.dumps(request) + "\n").encode("utf-8"))

# Ta emot svar
response = sock.recv(4096)
print(json.loads(response.decode("utf-8")))

sock.close()
```

### Debugging: Visa alla JSON-RPC-meddelanden

Starta signal-cli med verbose-läge:

```bash
signal-cli -v -u "+46701234567" daemon --tcp 127.0.0.1:7583
```

Detta visar alla inkommande och utgående JSON-RPC-meddelanden.

---

## Relaterad Dokumentation

- [SETUP_FLOW.md](SETUP_FLOW.md) — Oden's setup-wizard (web-baserad)
- [FEATURES.md](FEATURES.md) — Komplett funktionsspecifikation
- [WEB_GUI.md](WEB_GUI.md) — Web-gränssnitt och API
- [signal-cli GitHub](https://github.com/AsamK/signal-cli) — Officiell signal-cli-dokumentation
- [Signal Protocol](https://signal.org/docs/) — Signal's krypteringsprotokoll

---

## Support och Hjälp

**Problem med Oden:**
- [GitHub Issues](https://github.com/NicklasAndersson/oden/issues)

**Problem med signal-cli:**
- [signal-cli GitHub Issues](https://github.com/AsamK/signal-cli/issues)
- [signal-cli Wiki](https://github.com/AsamK/signal-cli/wiki)

**Signal-appen:**
- [Signal Support](https://support.signal.org/)
