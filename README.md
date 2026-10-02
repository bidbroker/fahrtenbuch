# Fahrtenbuch

Digitales Fahrtenbuch für den VW Buggy Oldtimer (Ledl Europa 2001).
Single-Page-App — eine einzige HTML-Datei, keine Installation, kein Server nötig.

---

## Fahrzeug

| Feld | Wert |
|---|---|
| Bezeichnung | Ledl Europa 2001 |
| Kennzeichen | WU-327AF |
| FIN/VIN | 1102716349 |
| Motor | 1192er Mexico D1 508739 / D1 |
| Beschreibung | VW Buggy Rot / Historisch |

---

## Features

- **⚡ Schnell-Tab (Standard-Startseite mobil):** Ausfahrt mit minimalen Eingaben starten
  - KM-Start automatisch mit letztem Stand vorbelegt
  - Startort per Geolocation (📍) → Reverse-Geocoding (OpenStreetMap Nominatim)
  - Zweck per Schnellwahl (Einkaufen / Sightseeing / Ausfahrt) oder Freitext
  - Nur der Zielort muss getippt werden
  - Ankunft: KM-Ende tippen → fertig
- Fahrten erfassen (Start, Ziel, km, Uhrzeit, Fahrer, Mitfahrer, Bemerkung)
- Tankstopps während einer Fahrt eintragen (Ort, km, Liter)
- Automatische Verbrauchsberechnung (L/100km)
- Fahrten bearbeiten und löschen
- Jahresansicht mit Filterung (neueste Fahrt oben)
- Drucken / PDF-Export
- Textexport (komplettes Fahrtenbuch als .txt)
- **GitHub-Sync:** Daten werden automatisch in diesem Repository gespeichert → geräteübergreifend verfügbar
- **Offline-fähig:** Fahrtendaten sind in die HTML eingebettet → App startet auch ohne Netz, Token oder localStorage

---

## Nutzung am iPhone

Einfachster Weg — direkt über GitHub Pages öffnen:

```
https://bidbroker.github.io/fahrtenbuch/fahrtenbuch.html
```

In Safari → Teilen → „Zum Home-Bildschirm" → App-Icon am Homescreen.

> **Standort (📍):** iOS-Einstellungen → Datenschutz → Ortungsdienste → Safari-Websites → „Beim Verwenden". HTTPS (GitHub Pages) ist Voraussetzung — bei lokaler `file://`-Datei funktioniert Geolocation/Reverse-Geocoding nicht.

---

## Datenspeicherung

- **Lokal:** `localStorage` im Browser (sofort, ohne Netzwerk)
- **GitHub:** `fahrtenbuch_data.json` in diesem Repository (nach jeder Fahrt automatisch)
- Format: JSON mit den Feldern `fahrten`, `settings`, `offeneFahrt`, `lastSync`

### GitHub-Sync Konfiguration (einmalig im Browser)
1. App öffnen → Einstellungen → GitHub-Sync
2. GitHub-User: `bidbroker`
3. Repository: `fahrtenbuch`
4. Personal Access Token mit Scope `repo` eintragen
5. „Verbinden & testen" klicken

---

## Datenstruktur (`fahrtenbuch_data.json`)

```json
{
  "fahrten": [
    {
      "id": 1783415095282,
      "datum": "2026-07-07",
      "zweck": "Gemeindeamt",
      "von": "Gablitz",
      "bis": "Gablitz",
      "kmStart": 26651,
      "kmEnd": 26653,
      "strecke": 2,
      "timeStart": "11:04",
      "timeEnd": "11:30",
      "fahrer": "Christian Rapberger",
      "mitfahrer": "",
      "bemerkung": "",
      "tankstopps": [],
      "gesamtLiter": 0,
      "verbrauch": null
    }
  ],
  "settings": { "title": "...", "kz": "...", "vin": "...", "motor": "...", "sub": "..." },
  "offeneFahrt": null,
  "lastSync": "2026-07-20T19:00:00.000Z"
}
```

---

## Bekannte Probleme & Fixes

### KM-Scan (OCR) verworfen (2026-10-02)

**Idee:** Tacho fotografieren → KM-Stand automatisch per OCR (Tesseract.js) übernehmen.

**Ergebnis:** Funktioniert mit diesem Fahrzeug **nicht** zuverlässig und wurde wieder entfernt.

**Warum gescheitert:**
- Der VDO-Walzenzähler des Oldtimers hat beige Ziffern auf hellgrauem Grund (minimaler Kontrast)
- Die Tausenderstelle wird vom Tachozeiger gekreuzt/verdeckt
- Mechanische Walzen-Ziffern haben einen eigenen Font (keine gedruckten Zahlen)
- Das Gesamtbild enthält ~170 Zahlen von der Tacho-Skala (20–150) → OCR ertrinkt im Rauschen

**Getestet (mit echtem Foto, lokal verifiziert):**
- Ganzes Bild → unbrauchbar (170+ Zahlen)
- Zuschnitt aufs Zählwerk + Hochskalierung + Kontrast + PSM 8 → bestes Ergebnis `24668` statt echtem `27006` (2 von 5 Ziffern falsch)
- Plausibilitätsprüfung gegen letzten Stand + Tausenderstellen-Rekonstruktion halfen nicht genug

**Entscheidung:** KM-Stand wird **manuell getippt** (mit Vorbelegung des letzten Standes).
Bei diesem Tacho ist Tippen schneller und fehlerfrei. Nicht erneut mit Tesseract versuchen —
eine Verbesserung bräuchte Apple Live Text / Vision, was nur in einer nativen iOS-App geht.

### App hing komplett / Tabs reagierten nicht (2026-10-02)

**Ursache:** Verwaister Code-Block + zerbrochenes `if/else` in `mRenderSchnell()` erzeugten einen
JavaScript-**Syntaxfehler**. Ein einziger Syntaxfehler legt das gesamte Script still → kein Tab reagierte.

**Lektion:** Bei „nichts funktioniert mehr" immer zuerst Syntax prüfen, nicht an Einzelsymptomen arbeiten.
Verifikation per `node --check` (JS aus HTML extrahieren) + jsdom-Ladetest (alle Tabs, 0 Fehler).

### Duplikate in der Datenbank (behoben 2026-07-20)

**Ursache:** Beim Bearbeiten eines Eintrags und gleichzeitigem oder wiederholtem GitHub-Sync
wurden Einträge mehrfach ins Array geschrieben statt überschrieben.
Konkret: 6 IDs waren doppelt/mehrfach vorhanden (eine ID sogar 5×), davon ein Eintrag
mit `kmEnd: null` (abgebrochene Bearbeitung).

**Symptom:** Fahrtenbuch für 2026 nicht mehr anzeigbar, App verhält sich fehlerhaft.

**Fix:** Funktion `dedupFahrten()` eingebaut — wird jetzt automatisch aufgerufen:
- beim **Laden** von GitHub (`loadFromGitHub`)
- beim **Speichern** zu GitHub (`saveToGitHub`)

**Logik:**
- Pro `id` wird nur ein Eintrag behalten
- Vollständige Einträge (`kmEnd != null`) haben Vorrang vor unvollständigen
- Bei mehreren vollständigen: letzter gewinnt
- Ergebnis wird nach Datum + Startzeit sortiert

**Manuelle Bereinigung:** 81 → 72 Einträge, alle 2026-Daten wiederhergestellt.

---

## Datenbankpflege

Falls die Daten manuell repariert werden müssen:

```bash
# Repo klonen
git clone https://github.com/bidbroker/fahrtenbuch.git
cd fahrtenbuch

# Duplikate prüfen
python3 -c "
import json
from collections import Counter
data = json.load(open('fahrtenbuch_data.json'))
ids = Counter(f['id'] for f in data['fahrten'])
print({id_: n for id_, n in ids.items() if n > 1})
"

# Nach Reparatur pushen
git add fahrtenbuch_data.json
git commit -m "Fix: Datenbank bereinigt"
git push
```

---

## Fahrtenstatistik (Stand 2026-10-02)

| Jahr | Fahrten |
|---|---|
| 2023 | 9 |
| 2024 | 30 |
| 2025 | 19 |
| 2026 | 25 |
| **Gesamt** | **83** |
