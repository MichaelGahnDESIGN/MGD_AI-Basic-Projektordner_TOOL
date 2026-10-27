# Zugänge sicher einrichten – ganz einfach erklärt

## Worum geht es?

Du willst, dass dein KI-Helfer (Claude Code oder Codex) sich auf einem Server, in einem Shop oder bei WordPress anmelden kann.
Dein Passwort soll dabei **nicht im Chat** landen und **nicht auf GitHub**.

Stell dir dein Passwort wie einen Haustürschlüssel vor. Du gibst ihn nicht jedem in die Hand.
Stattdessen legst du ihn in einen kleinen Tresor (eine geschützte Datei auf deinem Rechner). Nur dein Rechner darf ihn dort herausnehmen.

Dafür gibt es zwei Hilfsmittel. Du kannst eines oder beide nutzen:

| Hilfsmittel | Wofür? | Pflicht? |
|---|---|---|
| **Skill `/zugang`** | Übernimmt deine Zugangsdaten in eine geschützte Datei und testet den Login | ja, das ist der Hauptweg |
| **Zugang-UI** | Kleine Webseite auf deinem Rechner, die einen SSH-Schlüssel für den Server erzeugt | nein, nur für Server mit SSH |

## Das brauchst du

- ☑️ Claude Code oder Codex ist installiert
- ☑️ Ein Projektordner auf deinem Rechner
- ☑️ Eine Textdatei mit deinen Zugangsdaten (siehe Schritt 2)
- ☑️ Nur für die Zugang-UI: Python 3. Test im Terminal: `python3 --version`. Kommt eine Zahl wie `3.11`, passt alles.

---

## Teil 1: Der Skill `/zugang`

### Schritt 1: Skill installieren (einmalig)

Kopiere den Skill-Ordner in den Skill-Ordner von Claude Code. Im Terminal:

```bash
mkdir -p ~/.claude/skills
cp -R ".agents/skills/zugang" ~/.claude/skills/zugang
```

Starte danach Claude Code neu (oder öffne einen neuen Chat). Jetzt kennt er den Befehl `/zugang`.

⚠️ Der Befehl kopiert nur den Skill. Er enthält keine Zugangsdaten.

### Schritt 2: Zugangs-Datei anlegen

Lege in deinem Projektordner eine Textdatei an, zum Beispiel `private/zugaenge.md`. Schreibe deine Daten hinein. Beispiel mit **erfundenen** Werten:

```text
## Server
Adresse: beispiel-server.de
Benutzer: mein-benutzer
Passwort: mein-geheimes-passwort

## Shop-Login
Adresse: https://mein-shop.de/admin/
Benutzer: admin
Passwort: noch-ein-geheimes-passwort
```

Die Datei muss nicht schön sein. Wichtig ist: Zu jedem Zugang gehören Adresse, Benutzer und Passwort.

### Schritt 3: Skill starten

Schreibe in Claude Code:

```text
/zugang private/zugaenge.md
```

Bei Codex: `$zugang private/zugaenge.md`.

### Was jetzt passiert

1. Der Helfer schaut, ob in deinem Projekt schon Schutzregeln für Git und Uploads existieren.
2. Er trägt deine Zugangs-Datei in die Liste „nicht hochladen“ ein (`.gitignore`). So landet sie nicht aus Versehen auf GitHub.
3. Er legt die geschützte Datei `.env` an. Gibt es sie schon, lässt er sie unverändert und nutzt `.env.access.local`.
4. Er sperrt die Dateien gegen versehentliches Ändern und Löschen.
5. Er testet **einmal**, ob der Login klappt. Passwort-Varianten probiert er nicht durch.
6. Du bekommst einen Bericht ohne Passwörter.

### Was der Skill nicht macht

- ❌ Er ändert keine Passwörter und keine Konten.
- ❌ Er legt keine neuen Schlüssel oder Tokens an.
- ❌ Er veröffentlicht nichts und deployt nichts.
- ❌ Er liest keine Kundendaten und keine Server-Logs.
- ❌ Er umgeht keine Zwei-Faktor-Abfrage und kein Captcha. Das machst du selbst.

### Bericht lesen

| Ergebnis | Bedeutung |
|---|---|
| ✅ erfolgreich | Der Login hat geklappt |
| ❌ abgelehnt | Der Server hat „nein“ gesagt. Passwort oder Benutzer prüfen |
| ⚠️ blockiert | Etwas steht im Weg (Zwei-Faktor, Captcha, Sperre). Du musst eingreifen |
| ☑️ ungetestet | Der Helfer durfte oder konnte nicht testen |

---

## Teil 2: Die Zugang-UI (optional, für Server mit SSH)

Manche Server erlauben den Login ohne Passwort, nur mit einem **Schlüssel**. Das ist sicherer. Die Zugang-UI erzeugt diesen Schlüssel für dich. Du klickst, statt Befehle zu tippen.

### Starten

Im Terminal, im Ordner des Repositorys:

```bash
./VORLAGE/AI/TOOLING/zugang-ui/start.command
```

Dein Browser öffnet sich. Die Seite läuft **nur auf deinem Rechner**. Kein Passwort verlässt ihn.

### Die fünf Kästchen

1. **Projekt:** Trage deinen Projektordner ein. Optional: den Pfad zu deiner Zugangs-Datei. „Aus .md übernehmen“ füllt viele Felder von selbst.
2. **Zugang:** Prüfe die Felder (Server-Adresse, Benutzer, Port) und klicke auf „Einrichten & Schlüssel erzeugen“.
3. **Schlüssel eintragen:** Du siehst eine lange Zeile, die mit `ssh-ed25519` beginnt. Das ist der **öffentliche** Schlüssel. Er ist ungefährlich. Klicke auf „Schlüssel kopieren“ und trage ihn bei deinem Hoster ein (siehe unten).
4. **Verbindung testen:** Ein Klick prüft den Server und deinen Schlüssel. Grün = alles gut.
5. **Selbstständig arbeiten:** Zeigt dir einen Kurznamen für den Server und die Regeln für Claude Code (siehe Teil 3).

### Schlüssel beim Hoster eintragen

- **all-inkl (KAS):** Im Kundenmenü auf *Tools → SSH-Schlüssel*. Beim richtigen SSH-Benutzer einfügen und speichern.
- **Andere Hoster:** Suche im Hilfebereich nach „SSH-Schlüssel hinzufügen“.

Der **private** Schlüssel bleibt im Ordner `.ssh/` deines Projekts. Gib ihn nie weiter und lade ihn nirgendwo hoch.

---

## Teil 3: Claude selbstständig arbeiten lassen (optional)

Damit Claude Code nicht bei jedem Handgriff nachfragt, kannst du **bewusst** festlegen, was er ohne Nachfrage darf. Das machst du selbst, damit du die Kontrolle behältst.

Lege im Projektordner die Datei `.claude/settings.json` an. Beispiel für einen Server mit dem Kurznamen `meinserver`:

```json
{
  "permissions": {
    "allow": [
      "Bash(ssh meinserver cat:*)",
      "Bash(ssh meinserver ls:*)",
      "Bash(ssh meinserver tail:*)",
      "Bash(ssh meinserver grep:*)"
    ],
    "ask": [
      "Bash(ssh meinserver rm:*)",
      "Bash(ssh meinserver mysql:*)",
      "Bash(ssh meinserver chmod:*)",
      "Bash(ssh meinserver mv:*)"
    ]
  }
}
```

- `allow` heißt: Das darf Claude ohne Nachfrage (nur ansehen und suchen).
- `ask` heißt: Hier fragt Claude **immer vorher** nach (löschen, Datenbank, Rechte, verschieben).

---

## Das bleibt immer deine Entscheidung

- 🚫 Etwas löschen (Dateien, Datenbank, Konten)
- 🚫 Passwörter ändern
- 🚫 Etwas live schalten oder veröffentlichen
- 🚫 Zwei-Faktor-Code oder Captcha lösen

Vor jedem Löschen sollte ein Backup liegen. Frage den Helfer: „Mach vorher ein Backup.“

---

## Kleines Wörterbuch

| Wort | Bedeutung |
|---|---|
| **Server** | Der Computer im Internet, auf dem deine Webseite liegt |
| **Hoster** | Die Firma, die den Server vermietet (zum Beispiel all-inkl, Hostinger) |
| **SSH** | Eine sichere „Fernbedienung“ für den Server per Text |
| **Schlüssel (öffentlich / privat)** | Wie ein Schloss und ein Schlüssel. Das Schloss (öffentlich) steht beim Hoster. Den Schlüssel (privat) behältst nur du |
| **Fingerprint** | Der „Fingerabdruck“ des Servers. Damit prüfst du, dass du mit dem richtigen Server sprichst |
| **`.env`** | Eine kleine Textdatei mit geheimen Werten. Sie bleibt nur auf deinem Rechner |
| **`.gitignore`** | Eine Liste: „Diese Dateien niemals auf GitHub hochladen“ |
| **Webroot** | Der Ordner auf dem Server, aus dem deine Webseite angezeigt wird |
| **Zwei-Faktor (MFA)** | Zusätzlicher Code beim Login, zum Beispiel per App |
| **Token** | Ein Passwort-Ersatz für Programme, oft mit eingeschränkten Rechten |

---

## Wenn etwas nicht klappt

| Du siehst | Das bedeutet | Das hilft |
|---|---|---|
| `/zugang` wird nicht erkannt | Der Skill ist nicht installiert oder Claude wurde nicht neu gestartet | Schritt 1 wiederholen, Claude Code neu starten |
| Login „abgelehnt“ | Benutzer oder Passwort stimmen nicht | Daten in der Zugangs-Datei prüfen |
| „blockiert“ | Zwei-Faktor, Captcha oder Sperre | Login einmal selbst im Browser machen, dann nochmal versuchen |
| `Permission denied (publickey)` | Der Schlüssel ist beim Hoster nicht (richtig) eingetragen | Beim richtigen SSH-Benutzer eintragen, ein paar Minuten warten |
| Der Fingerabdruck stimmt nicht | Falscher Server oder jemand fängt die Verbindung ab | **Nicht weitermachen.** Adresse prüfen |
| `python3: command not found` | Python fehlt | Python 3 installieren (python.org) |

---

## Aufräumen, wenn das Projekt fertig ist

1. Öffentlichen Schlüssel beim Hoster wieder entfernen.
2. Passwörter ändern, wenn sie je im Klartext in einer Datei oder einem Chat standen.
3. Die Zugangs-Datei sicher aufbewahren (zum Beispiel im Passwortmanager) oder löschen. Vorher Backup.

---

## Was der Schutz kann und was nicht

- `.gitignore` verhindert das normale Hochladen zu GitHub. Es entfernt aber keine Daten, die schon einmal hochgeladen wurden.
- Dateisperren verhindern **versehentliches** Ändern. Sie sind kein Backup.
- Die geschützten Dateien liegen im Klartext auf deinem Rechner. Wer deinen Rechner öffnen darf, kann sie lesen.
- Manuelle Uploads, ZIP-Dateien und Cloud-Ordner beachten `.gitignore` nicht automatisch. Prüfe sie vorher.

Der Skill ist unter der MIT-Lizenz frei nutzbar. Echte Zugangsdaten gehören nie in ein öffentliches Repository.
