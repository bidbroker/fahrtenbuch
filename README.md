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

- Fahrten erfassen (Start, Ziel, km, Uhrzeit, Fahrer, Mitfahrer, Bemerkung)
- Tankstopps während einer Fahrt eintragen (Ort, km, Liter)
- Automatische Verbrauchsberechnung (L/100km)
- Fahrten bearbeiten und löschen
- Jahresansicht mit Filterung
- Drucken / PDF-Export
- Textexport (komplettes Fahrtenbuch als .txt)
- **GitHub-Sync:** Daten werden automatisch in diesem Repository gespeichert → geräteübergreifend verfügbar

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

## Fahrtenstatistik (Stand 2026-07-20)

| Jahr | Fahrten |
|---|---|
| 2023 | 9 |
| 2024 | 30 |
| 2025 | 19 |
| 2026 | 14 |
| **Gesamt** | **72** |
