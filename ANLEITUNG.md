# Tagwerk für die Familie einrichten

Du brauchst ein Google-Konto und etwa 20 Minuten. Programmieren musst du nichts. Du klickst dich durch zwei Webseiten und kopierst an zwei Stellen Text. Der kostenlose Firebase-Tarif („Spark“) reicht für eine Familie völlig aus, eine Kreditkarte ist nicht nötig.

Die App besteht aus diesen Dateien:

| Datei | Wofür |
|---|---|
| `index.html` | die App selbst |
| `config.js` | deine Firebase-Zugangsdaten (trägst du in Schritt 2 ein) |
| `firestore.rules` | die Sicherheitsregeln (fügst du in Schritt 4 ein) |
| `sw.js`, `manifest.webmanifest`, `icon-*.png` | machen die App installierbar und offline nutzbar |

Solange `config.js` nicht ausgefüllt ist, läuft die App im **Demo-Modus**. Dann bleibt alles auf einem Gerät. Das ist praktisch zum Ausprobieren.

---

## 1. Firebase-Projekt anlegen

1. Öffne <https://console.firebase.google.com> und melde dich mit deinem Google-Konto an.
2. Klicke auf **Projekt erstellen** (oder „Neues Firebase-Projekt erstellen“).
3. Gib einen Namen ein, z. B. `tagwerk-familie`, und bestätige.
4. Google Analytics brauchst du nicht. Schalte es aus und klicke auf **Projekt erstellen**.

## 2. Die App registrieren und `config.js` ausfüllen

1. Klicke in der Projektübersicht auf das Symbol **`</>`** (Web).
2. App-Spitzname: `Tagwerk`. Den Haken bei „Firebase Hosting“ **nicht** setzen. Klicke auf **App registrieren**.
3. Du siehst jetzt einen Codeblock, der mit `const firebaseConfig = {` beginnt. Darin stehen sechs Zeilen wie `apiKey: "AIza…"`.
4. Öffne `config.js` mit einem einfachen Texteditor (Windows: Editor, Mac: TextEdit im Modus „Reiner Text“).
5. Ersetze die sechs Beispielzeilen zwischen `{` und `};` durch die sechs Zeilen aus Firebase. Speichern.

Später findest du diese Werte jederzeit unter ⚙️ **Projekteinstellungen → Allgemein → Meine Apps**.

> Diese Werte sind kein Passwort. Sie sagen der App nur, wo eure Datenbank liegt. Geschützt werden eure Daten durch die Sicherheitsregeln aus Schritt 4.

## 3. Anmeldung einschalten

1. Links im Menü: **Build → Authentication → Jetzt starten**.
2. Reiter **Anmeldemethode** (Sign-in method):
   - **Anonym** auswählen → aktivieren → speichern. *Das ist Pflicht.* Damit bekommt jedes Handy unsichtbar eine eigene Identität, ganz ohne Registrierung.
   - **E-Mail/Passwort** auswählen → aktivieren → speichern. *Empfohlen.* Damit kann jemand unter „Einstellungen“ seine Anmeldung sichern und nach einem Handywechsel weitermachen.

## 4. Datenbank anlegen und Sicherheitsregeln einfügen

1. Links im Menü: **Build → Firestore Database → Datenbank erstellen**.
2. Standort: **europe-west3 (Frankfurt)**. Der Standort lässt sich später nicht mehr ändern.
3. **Im Produktionsmodus starten** wählen und erstellen.
4. Oben auf den Reiter **Regeln** klicken.
5. Den gesamten Text dort löschen. Öffne `firestore.rules` im Texteditor, kopiere den **kompletten** Inhalt und füge ihn ein.
6. Auf **Veröffentlichen** klicken.

Damit prüft der Server bei jedem Zugriff die Rechte. Wer nur ansehen darf, kann auch mit technischen Tricks nichts abhaken. Persönliche Aufgaben kann niemand außer der Person selbst lesen. Die Ausnahme sind Admins, und auch die nur mit dem Admin-Passwort.

## 5. Die App ins Internet stellen

Damit die Handys die App installieren können, braucht sie eine Adresse mit `https://`. Am einfachsten geht das mit **Netlify**, ebenfalls kostenlos:

1. Konto anlegen auf <https://app.netlify.com> (Anmeldung mit Google geht).
2. **Add new site → Deploy manually** wählen.
3. Den ganzen Ordner `tagwerk-familie` (mit der ausgefüllten `config.js`) in das Feld ziehen.
4. Nach ein paar Sekunden bekommst du eine Adresse wie `https://zauberhaft-123.netlify.app`. Unter **Site configuration → Change site name** kannst du sie z. B. in `huber-tagwerk.netlify.app` ändern.

Zur Sicherheit trägst du diese Adresse auch in Firebase ein: **Authentication → Einstellungen → Autorisierte Domains → Domain hinzufügen**.

## 6. Familie gründen

1. Öffne die Adresse auf deinem Handy.
2. Installiere die App:
   - **iPhone (Safari):** Teilen-Symbol → **Zum Home-Bildschirm**
   - **Android (Chrome):** Menü ⋮ → **App installieren** bzw. **Zum Startbildschirm hinzufügen**
3. Öffne die App vom Startbildschirm und tippe auf **Familie gründen**. Gib den Familiennamen, deinen Namen und ein Admin-Passwort ein.
4. Hast du die bisherige Tagwerk-Version im selben Browser benutzt, kannst du die alten Aufgaben direkt übernehmen. Bei der Version aus Claude geht das so: dort im Menü **Speichern, Sichern & Darstellung → Sicherung in die Zwischenablage kopieren**. Danach fügst du sie in der neuen App unter **Menü → Einstellungen & Sicherung → Persönliche Aufgaben importieren** ein.

## 7. Die anderen einladen

1. Menü → **Familie → Einladung kopieren** und z. B. per WhatsApp schicken.
2. Die Person öffnet die Adresse, installiert die App und tippt auf **Einer Familie beitreten**. Dann gibt sie den Code und ihren Namen ein.
3. Du bekommst die Anfrage unter **Familie** angezeigt, auch mit einer Zahl am Menü-Button. Tippe sie an, lege die Rechte fest und klicke auf **Aufnehmen**.

**Tipp:** Lass alle unter **Einstellungen → Anmeldung sichern** eine E-Mail und ein Passwort hinterlegen. Wer das nicht macht und die App löscht oder das Handy wechselt, muss neu beitreten. Dabei wären die persönlichen Aufgaben weg.

---

## Die Rechte im Überblick

| Recht | Was die Person bei Familien-Aufgaben darf |
|---|---|
| *(keins)* | nur ansehen |
| Abhaken & verschieben | Aufgaben erledigen, verschieben, auslassen, wenn sie ihr oder niemandem zugewiesen sind |
| Anlegen & bearbeiten | Familien-Projekte und -Aufgaben erstellen, ändern, löschen, sortieren |
| Zuweisen | Aufgaben einer Person geben, die abhaken darf |
| Admin | alles, dazu Mitglieder aufnehmen und entfernen, Rechte vergeben, mit Admin-Passwort persönliche Aufgaben ansehen |

Persönliche Aufgaben verwaltet jede Person immer selbst, egal welche Rechte sie in der Familie hat.

Eine Familien-Aufgabe ohne Zuweisung ist für alle da. Wer sie zuerst abhakt, erledigt sie für alle.

## Wenn etwas nicht klappt

| Meldung / Verhalten | Lösung |
|---|---|
| Die App zeigt „Demo-Modus“ | `config.js` ist nicht ausgefüllt oder wurde nicht mit hochgeladen (Schritt 2 und 5). |
| „Die anonyme Anmeldung ist … nicht eingeschaltet“ | Schritt 3: „Anonym“ aktivieren. |
| „fehlende Berechtigung“ beim Speichern | Wurden die Regeln aus Schritt 4 vollständig eingefügt und **veröffentlicht**? Hat die Person das nötige Recht? |
| „Anmeldung sichern“ schlägt fehl | Schritt 3: „E-Mail/Passwort“ aktivieren. |
| Admin-Passwort vergessen | Ein anderer Admin kann es unter **Familie → Admin-Passwort ändern** neu setzen. |

## Später Änderungen einspielen

Wenn du eine neue Version von `index.html` bekommst, ersetze nur diese Datei im Ordner. Deine `config.js` behältst du. Dann ziehst du den Ordner bei Netlify erneut unter **Deploys** in das Feld. Die Handys holen sich die neue Version beim nächsten Öffnen.
