# Tagwerk – Projektkontext

Diese Datei fasst den Stand der App zusammen, damit eine neue Sitzung ohne Vorwissen weiterarbeiten kann.
**Bitte zusammen mit der aktuellen `index.html` und `firestore.rules` hochladen.** `config.js` wird nicht benötigt.

Stand: 12. Oktober 2026

---

## 1. Was ist Tagwerk?

Eine PWA für Aufgaben in der Familie. Es gibt persönliche Aufgaben, die nur man selbst sieht, und Familien-Aufgaben, die alle Mitglieder sehen. Dazu kommen Rechte, Zuweisungen, Projekte und Wiederholungen. Sprache der App und der Zusammenarbeit: Deutsch.

**Hosting & Technik**
- GitHub Pages, statisch, ohne Build-Schritt
- Firebase: anonyme Anmeldung (optional mit E-Mail/Passwort „gesichert“) und Firestore
- Firebase JS SDK 10.12.2, per `import()` von gstatic geladen
- Ohne gültigen API-Key (`DEIN_API_KEY` in `config.js`) läuft die App im **Demo-Modus** (localStorage, mehrere Personen simulierbar)

**Dateien**

| Datei | Inhalt |
|---|---|
| `index.html` | Die komplette App: CSS und Vanilla-JS in einer Datei, ca. 141 KB |
| `firestore.rules` | Sicherheitsregeln, die in der Firebase-Konsole veröffentlicht werden |
| `config.js` | Firebase-Zugangsdaten (`window.TAGWERK_FIREBASE`). **Nicht ändern.** |
| `manifest.webmanifest`, `sw.js`, `icon-192.png` | PWA-Dateien (existieren, wurden bisher nicht bearbeitet) |

## 2. Wünsche zur Zusammenarbeit

- Nur ändern, was nötig ist.
- Am Ende die geänderten Dateien **vollständig** zum Herunterladen liefern.
- Wenn sich die Sicherheitsregeln ändern, das **ausdrücklich sagen**, damit sie in Firebase neu veröffentlicht werden.
- Änderungen vorher im Demo-Modus im Browser testen (Playwright hat gut funktioniert, siehe Abschnitt 8).

## 3. Aufbau von `index.html`

In der Reihenfolge im Script:

1. **CSS**: Farb-Tokens auf `:root`, dazu Dunkelmodus per `prefers-color-scheme` und `data-theme`. `--accent` ist die Primärfarbe, auch für ausgewählte Chips.
2. **Datum-Helfer**: Alle Tage sind **ganze Tagesnummern** (UTC-Tage seit 1970 des lokalen Datums). Wichtige Funktionen: `todayNum()`, `parts()`, `iso()`, `fromIso()`, `fmtShort()`, `fmtRel()`.
3. **Zustand**:
   - `D` = zusammengeführte Sicht `{tasks, projects, log}`. Jeder Eintrag hat `scope: "me" | "fam"`.
   - `S` = Sitzung (uid, fid, members, requests, phase usw.)
   - `B` = Backend (Demo oder Firebase)
   - `ui` = Ansicht (`view`, `day`, `reorder`, `q`, `lf`, `lp`, `statDays`)
4. **Rechte und Zuständigkeit**: `R()`, `rightsOf()`, `memberActiveOn()`, `occAssignee()`, `canAct(t,o)`, `inToday(t,o,day)`, `assignable()`
5. **Pausen**: `pausedOn()`, `taskPaused()`, `setActive()`, `pauseTag()`, `activeBox()`, `pausesHtml()`
6. **Speichern**:
   - `change(fn)` macht einen Snapshot, führt `fn` aus und ruft `commit()` auf.
   - `commit()` vergleicht vorher und nachher. Persönliche Daten werden als *ein* Dokument (`private/{uid}`) geschrieben, Familien-Daten als einzelne Dokumente per Batch.
   - `undo()` stellt den letzten Snapshot wieder her.
7. **Aktionen**: `toggleDone`, `doMove`, `doSkip`, `doAssign`, `toggleItem`, `advanceAfter`, `reorder`
8. **Backends**: `demoBackend()`, `firebaseBackend(cfg)` mit denselben Methoden (`savePrivate`, `apply`, `createFamily`, `requestJoin`, `approve`, `updateMember`, …)
9. **Darstellung**:
   - `render()` = `renderMain()` + `syncHistory()`
   - Ansichten: `renderToday`, `renderDays`, `renderFamily` (inkl. `statsHtml`), `renderLib` (inkl. `libResultsHtml`, Suche und Filter)
10. **Termin-Logik**: `recMatch`, `recText`, `matcherFor`, `occurrences`, `dayModel`
11. **Bottom Sheets**:
    - `openSheet()` / `closeSheet()` / `renderSheet()`
    - Formulare: `taskForm`, `projectForm`, `occSheet`, `memberSheet`, `settingsSheet` u. a.
    - Entwürfe liegen in `sheet.draft`, Felder werden per `data-bind="pfad"` gebunden.
12. **Zurück-Taste (Android)**: `NAV`, `syncHistory()`, `popstate`-Handler
13. **Ereignisse**: ein zentraler Klick-Handler auf `document` mit `data-a="aktion"` und `data-v` / `data-k`. Dazu `input`- und `change`-Handler auf `$sheet` und `$app`.

## 4. Datenmodell

### Firestore-Pfade

```
users/{uid}                    {fid, familyName}
joinCodes/{code}               {fid, familyName, createdBy}
families/{fid}                 {name, code, createdBy, createdAt}
  members/{uid}                {name, color, rights{admin,check,edit,assign}, joinedAt,
                                active (false = inaktiv), away {from, to} | null}
  requests/{uid}               {name, at, code}
  secret/admin                 {pinHash}          (niemand darf lesen)
  unlocks/{uid}                {hash, until}      (Admin-Einsicht, 15 Min.)
  private/{uid}                {tasks, projects, log, rev}   (persönliche Daten, 1 Dokument)
  projects/{pid}               Projekt
  tasks/{tid}                  Aufgabe
  log/{taskId}_{orig}          {taskId, orig, d, m, s, by, c}
```

### Aufgabe (`task`)

| Feld | Bedeutung |
|---|---|
| `id`, `scope`, `title`, `prio` (1 hoch – 3 niedrig), `order` | Grunddaten und Reihenfolge |
| `projectId`, `offset` | Gehört zu einem Projekt; fällig `offset` Tage vor oder nach dem Projekttermin |
| `rec` | Wiederholung, siehe unten (bei Projekt-Aufgaben `null`) |
| `movable`, `follow` (`replace`/`keep`/`shift`), `replaceDays`, `shifts[{after,delta}]` | Verschieben und Folgetermine |
| `created` | Erster Tag, ab dem Termine zählen |
| `assignee`, `createdBy` | Feste Zuweisung (nur Familie) |
| `inactive`, `pauses[{from, to\|null}]` | Inaktiv bzw. Pausen (`to:null` = läuft, solange inaktiv) |
| `note`, `items[{id,text}]` | Notiz und Checkliste |
| `next` | Nur bei „nach Erledigung“: Tag des offenen Termins |
| `rotation[uid]`, `rotStart`, `rotN` | Rotation (dann ist `assignee = null`). `rotN` ist der Zähler bei „nach Erledigung“ |

### Projekt (`project`)

Felder: `id, scope, name, color, rec, created, prio, order, inactive, pauses`.

### Wiederholung (`rec`)

```
{type: "once"|"daily"|"weekly"|"monthly"|"yearly"|"after",
 start, end|null, every, weekdays[0=Mo..6=So], monthDays[1..31, 32=letzter], yearDates[{m,d}]}
```
- `"after"` = „X Tage nach Erledigung“ (`every` = X). Gibt es nur für Aufgaben ohne Projekt.

### Log

- Im Speicher ist der Schlüssel `"taskId|orig"`, in Firestore die Dokument-ID `taskId_orig`.
- Inhalt eines Eintrags:
  - `d` = erledigt am (Tag)
  - `m` = verschoben auf
  - `s` = ausgelassen
  - `by` = wer erledigt hat (nur Familie)
  - `c` = abgehakte Checklisten-Punkte (IDs)
- Ladefenster: Das Familien-Log wird nur ab `orig >= heute-200` geladen. Persönliche Einträge, die älter als 150 Tage sind, werden beim Speichern aufgeräumt.

### Begriffe bei Terminen

- `orig` = Termin laut Regel
- `base` = `orig` plus Serienverschiebung
- `eff` = tatsächlicher Tag

## 5. Wichtige Logik

- **`occurrences(t, from, to)`** liefert die Termine einer Aufgabe. Bei `follow:"replace"` entfallen zu nahe Folgetermine.
  - Bei `"after"` gibt es immer genau einen offenen Termin (`next ?? rec.start`) plus die bekannten Termine aus dem Log.
- **`dayModel(day, filter)`** liefert `{due, overdue, missed}`. Der Filter wird **pro Termin** aufgerufen: `filter(t, o, day)`.
  - Termine in Pausen fallen weg.
  - Überfällige Termine erscheinen nur, wenn die Aufgabe heute nicht pausiert ist.
- **`occAssignee(t, o, day)`** bestimmt, wer für einen Termin zuständig ist (`null` = alle):
  - Mit Rotation: die Person an Position `rotIdx` in der Reihenfolge. Ist sie inaktiv oder im Urlaub, ist die nächste Person dran.
  - Mit fester Zuweisung: Ist die Person inaktiv oder im Urlaub, wird der Termin `null` (für alle offen).
- **`rotIdx`** zählt die Regeltermine seit `rotStart` (zwischengespeichert in `rotCache`). Bei „nach Erledigung“ wird stattdessen `rotN` verwendet.
- **`blocksHtml`** fasst in „Heute“ und der Tagesansicht **gleichnamige Projekte** zusammen, über persönlich und Familie hinweg. Groß- und Kleinschreibung sowie Leerzeichen am Rand zählen dabei nicht.
- **Zurück-Taste**: Solange ein Fenster offen ist oder ein anderer Reiter als „Heute“ angezeigt wird, liegt ein zusätzlicher History-Eintrag oben.
  - „Zurück“ schließt zuerst das Fenster und wechselt dann zu „Heute“.
  - Auf „Heute“ schließt „Zurück“ die App.

## 6. Änderungsverlauf

### Runde 1
1. Android-Zurück-Taste: Fenster schließen, dann zu „Heute“, dort App schließen
2. 20 Projektfarben (`COLORS`; die ersten 6 sind die alten)
3. Ausgewählte Ein/Aus-Chips (`.chip.on`, z. B. Wochentage) in Akzentfarbe statt fast schwarz. Die Schiebeschalter wurden nicht verändert.
4. Projekte und Aufgaben inaktiv schaltbar (Schalter „Aktiv“). Termine in der Pause entfallen.
5. Mitglieder für Familien-Aufgaben aktiv/inaktiv (`members.active`). Admins und die Person selbst dürfen das ändern.
6. Gleichnamige Projekte in der Tages- und Heute-Ansicht zusammengefasst

### Runde 2
1. Wiederholung **„Nach Erledigung“** (`rec.type "after"`, Feld `next`)
2. **Pausen mit Zeitraum** für Aufgaben und Projekte (`pauses`), **Urlaub** für Mitglieder (`members.away`, ein Zeitraum)
3. **Notiz und Checkliste** pro Aufgabe. Haken gelten pro Termin (`log.c`).
4. **Suche und Filter** unter „Aufgaben & Projekte“: Text, Aktiv / Inaktiv & Pausen, Person
5. **Rotation** („Abwechselnd“) mit Vorschau der nächsten Termine
6. **Statistik** „Erledigt“ unter „Familie“ (7, 30 oder 90 Tage, pro Person aufklappbar)

### Bewusst nicht umgesetzt (nicht gewünscht)
- Punkte und Belohnungen für Kinder
- Kalender-Export (.ics)
- Push-Erinnerungen
- Foto als Nachweis

## 7. Sicherheitsregeln – aktueller Stand (Kurzfassung)

- **`can(fid, r)`**: Mitglied und (Admin oder Recht `r`).
- **Mitglieder**:
  - Admin darf alles ändern.
  - Die Person selbst darf nur `name`, `color`, `active` (bool) und `away` (map oder null) ändern.
- **Aufgaben anlegen**: nur mit `edit`. `assignee` und `rotation` nur zusätzlich mit `assign`.
- **Aufgaben ändern**, drei erlaubte Wege:
  1. Mit `edit` alles, aber `assignee` und `rotation` nur mit `assign`
  2. Mit `assign` nur `assignee`, `rotation`, `rotStart`, `rotN`
  3. Mit `check` nur `shifts`, `next`, `rotN`, und nur wenn die Aufgabe niemandem, mir, einer inaktiven bzw. abwesenden Person zugewiesen ist (oder ich Admin bin)
- **`assigneeAway()`**: `active == false` oder Urlaub heute. Der Urlaub wird mit `request.time` geprüft, mit ±1 Tag Spielraum wegen der Zeitzone.
- **Log**: schreiben, wenn `mayCheckTask` erfüllt ist, gleiche Bedingung wie oben. Löschen zusätzlich mit `edit`.
- **Persönliche Daten** (`private/{uid}`): nur die Person selbst. Admins dürfen mit Admin-Passwort-Freischaltung (`unlocks`) lesen.

## 8. Bekannte Einschränkungen & Hinweise

- **Rotation wird nur in der App durchgesetzt.** Rotations-Aufgaben haben `assignee = null`. Die Regeln erlauben deshalb jeder Person mit `check`, sie abzuhaken. Die Regeln können nicht berechnen, wer gerade dran ist.
- Die Regeln wurden **nicht im Firestore-Emulator** getestet, weil der Download in der Sandbox gesperrt war.
- Die Statistik hängt am Ladefenster des Logs (200 Tage) und ist daher auf höchstens 90 Tage begrenzt.
- Mitglieder können nur **einen** Urlaubszeitraum haben. Aufgaben und Projekte können mehrere Pausen haben.
- `normalize()` ergänzt fehlende Felder (`prio`, `order`) lokal. Fehlen sie bei alten Familien-Dokumenten in Firestore, kann ein reines Abhaken durch Mitglieder mit nur `check`-Recht an Regel 3 scheitern. Das war schon vorher so und ist bisher nicht aufgetreten.
- Ob der Wunsch „on/off-Buttons heller“ die Chips oder die Schiebeschalter meinte, ist nicht bestätigt. Umgesetzt wurden die Chips.

## 9. Testen

- **Demo-Modus**: lokal eine `config.js` mit `window.TAGWERK_FIREBASE={apiKey:"DEIN_API_KEY"};` anlegen und `index.html` öffnen.
- Unter „Einstellungen“ → „Ansehen als“ kann man die Person wechseln. Unter „Familie“ lassen sich Beitrittsanfragen simulieren.
- Nützlich in der Browser-Konsole bzw. in Playwright: `B.simulateRequest()`, `B.approve(uid,name,rights)`, `B.switchTo(uid)`, `B.updateMember(uid,patch)`, `ui.view='lib';render()`.
- JS-Syntax prüfen: Script aus `index.html` extrahieren und `node --check` ausführen.
