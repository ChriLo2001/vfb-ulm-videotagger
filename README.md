# VfB Ulm Video-Tagger

Ein Werkzeug zur Videoanalyse von Floorballspielen (Großfeld und Kleinfeld), entwickelt für
VfB Ulm Floorball. Du schaust das Spielvideo auf einem Bildschirm und erfasst auf dem anderen
(oder beides in einem Fenster) per Tastatur, was passiert: Reihenwechsel, Schüsse, Tore, Strafen, Bullys und jede Ballbesitzkette
(wer passt zu wem). Daraus rechnet das Tool automatisch Eiszeiten, Passquoten, Passmatrix und
Passnetzwerk und – im Experimentmodus – Corsi, Ballbesitzzeit, Konter, Schichtverlauf und vieles
mehr. Dort lässt sich auch jede Zahl anklicken und spielt die passenden Szenen ab.

**Direkt im Browser öffnen:** https://chrilo2001.github.io/vfb-ulm-videotagger/

Es wird nichts installiert und nichts hochgeladen. Video und Daten bleiben auf deinem Rechner.

---

## Inhalt

- [Voraussetzungen](#voraussetzungen)
- [Schnellstart](#schnellstart)
- [Ein-Bildschirm-Modus](#ein-bildschirm-modus)
- [Standard- und Experimentmodus](#standard--und-experimentmodus)
- [Setup vor dem Spiel](#setup-vor-dem-spiel)
- [Durchgang 1: Spielgerüst](#durchgang-1-spielgerüst)
- [Durchgang 2: Ballbesitzketten](#durchgang-2-ballbesitzketten)
- [Tasten auf einen Blick](#tasten-auf-einen-blick)
- [Auswertung](#auswertung)
- [Szenen abspielen und Clips exportieren](#szenen-abspielen-und-clips-exportieren)
- [Export und Sicherung](#export-und-sicherung)
- [Regeln für saubere Daten](#regeln-für-saubere-daten)
- [Datenschutz](#datenschutz)
- [Bekannte Grenzen](#bekannte-grenzen)
- [Lokal nutzen und weiterentwickeln](#lokal-nutzen-und-weiterentwickeln)

---

## Voraussetzungen

- **Google Chrome** (andere Browser sind nicht getestet)
- **Ein Video pro Spiel** als MP4 mit H.264. Kameraabschnitte vorher zusammenschneiden,
  Vor- und Nachlauf sowie Drittelpausen am besten entfernen. H.265/HEVC läuft in Chrome nicht
  zuverlässig.
- **Zwei Bildschirme** empfohlen: Video auf dem einen, Erfassung auf dem anderen. Ein Bildschirm
  geht auch (Ein-Bildschirm-Modus), z. B. am Laptop.
- Für den Clip-Export optional: [ffmpeg](https://ffmpeg.org/download.html)

Ob ein Video funktioniert, prüfst du am schnellsten, indem du es in ein leeres Chrome-Fenster ziehst.

## Schnellstart

1. Tool öffnen (Link oben oder lokal `videotagger.html`).
2. **Setup:** Spiel anlegen, Video auswählen, Kader und Reihen eintragen (oder aus dem
   Saisonmanager laden), Spielsystem wählen.
3. **„Videofenster öffnen“** klicken, das Fenster auf den zweiten Bildschirm ziehen,
   Doppelklick für Vollbild. Nur ein Bildschirm: oben rechts „Ein Bildschirm“ wählen.
4. **Durchgang 1** bei 1,25- bis 1,5-facher Geschwindigkeit: Drittel, Wechsel, Tore, Strafen,
   Bullys, Gegnerschüsse.
5. **Durchgang 2** bei normaler Geschwindigkeit: alle eigenen Ballbesitzketten.
6. **Auswertung** ansehen, Excel exportieren, JSON-Sicherung speichern.

Ein komplettes Großfeldspiel dauert in beiden Durchgängen zusammen etwa 3 Stunden.

## Ein-Bildschirm-Modus

Oben rechts zwischen **„Zwei Bildschirme“** und **„Ein Bildschirm“** umschalten (wird pro Rechner
gemerkt). Im Ein-Bildschirm-Modus liegt das Video im selben Fenster links (bzw. oben bei einem
schmalen Fenster), die Erfassung kompakt daneben. Die Ereignisliste klappt über „Liste ▸“ von
rechts aus. Funktioniert ab 1366 × 768 ohne Scrollen.

- **`Z`** (oder Doppelklick aufs Video): Theatermodus – Video fast im ganzen Fenster, unten eine
  schmale Leiste mit Reihe, laufender Kette und letzter Aktion. Alle Tasten wirken weiter.
  Nochmal `Z` zurück. Für noch mehr Fläche: F11.
- Umschalten geht jederzeit, auch mitten in einer Kette: Position, Tempo und Abspielen bleiben
  erhalten, es geht nichts verloren, und das Video muss nicht neu gewählt werden.

## Standard- und Experimentmodus

Oben rechts **„Standard“ / „Experiment“** (Standard ist voreingestellt, pro Rechner gemerkt).

- **Standard** enthält genau die Funktionen, die in echten Analysen geprüft sind, mit denselben
  Zahlen wie bisher.
- **Experiment** zeigt zusätzlich neue, noch nicht im Echtbetrieb getestete Funktionen. Sie sind
  mit **β** gekennzeichnet, oben steht „β Experiment“. Die Excel-Datei heißt dann
  `…_Auswertung_experiment.xlsx` und trägt in jedem Blatt einen Hinweis.
- Die Spieldaten und die JSON-Sicherung sind in beiden Modi gleich. Umschalten ändert nur, was
  angezeigt und berechnet wird.

Zurzeit experimentell: Erweiterte Kennzahlen, geschätzte Spielzeit (Drittellänge), automatische
Tor-Pause, Szenen per Klick und Clip-Export, Stichprobe n, Kennzahlen im Excel, Ergebnis und Blocker
bei Gegnerschüssen. Neue Funktionen
kommen immer zuerst in den Experimentmodus.

## Setup vor dem Spiel

- **Spielinfo:** Datum, Gegner, Heim/Auswärts, Modus GF oder KF
- **Drittellänge und Verlängerung** (β Experiment): für die geschätzte Spielzeit (siehe Auswertung)
- **Kader:** Name, Rückennummer, Position, Torwart separat. „Torwart bei Spielbeginn“ nicht
  vergessen.
- **Spielsystem (Formation):** Vorlagen GF 2-2-1 mit Top, GF 2-1-2, KF 1-2, KF 1-1-1.
  Positionsnamen sind umbenennbar. Die Formation bestimmt die Anordnung der Spielerbuttons, die
  Tasten in Durchgang 2 und das Passnetzwerk.
- **Reihen:** Spieler den Positionen zuordnen, pro Reihe einen Bullyspieler festlegen. Dazu die
  Sonderreihen Überzahl, Unterzahl und Spezial (z. B. ohne Torwart).
- Kader und Reihen lassen sich als Vorlage speichern und im nächsten Spiel laden.

### Spielbericht aus dem Saisonmanager (optional)

Mit Link oder Spiel-ID übernimmt das Tool Spielinfo, Kader (Abgleich per Rückennummer) sowie
Tore und Strafen als Referenz. Damit prüft die Auswertung, ob alle Tore und Strafen erfasst sind,
und bei einem Ulmer Tor werden Schütze und Vorlage vorgeschlagen.

Weil die Saisonmanager-Schnittstelle direkte Abrufe nicht erlaubt, gibt es einen Ausweichweg,
der im Tool erklärt ist: Spielbericht in Chrome öffnen → F12 → Reiter „Netzwerk“ → Filter
`games` → F5 → Rechtsklick auf `<ID>.json` → Kopieren → Antwort kopieren → im Tool einfügen.

## Durchgang 1: Spielgerüst

Ziel: Wer stand wann auf dem Feld, und was waren die großen Ereignisse.

- **Drittel:** `D` beim Anpfiff öffnet den Dialog „Besetzung für dieses Drittel“
  (vorausgefüllt, Enter übernimmt). Hier trägst du Aushelfer ein, wenn jemand in zwei Reihen
  spielt. `D` am Drittelende beendet das Drittel. **Erst `D`, dann die Reihe drücken.**
- **Reihenwechsel:** `1` `2` `3`, Überzahl `4`, Unterzahl `5`, Spezial `6`. Einzelne Spieler
  tauschen: Rechtsklick auf den Spielerbutton.
- **Gegnerschuss** `X` (jeder Schussversuch, auch geblockt oder vorbei), **Gegentor** `H`,
  **Tor** `J`. Bei Toren speichert das Tool, wer auf dem Feld stand (Plus/Minus).
- **Ergebnis des Gegnerschusses** (β, nur im Experimentmodus): nach `X` erscheint eine Auswahl,
  das Video läuft weiter. `2` gehalten · `3` geblockt · `4` vorbei (auch Pfosten/Latte) · `1` Tor
  (wird direkt zum Gegentor) · `Enter`/`Esc` unbekannt. Nach `3` den Blocker per Positionstaste
  wählen (wie in Durchgang 2, `0` = anderer Spieler, `Enter` = unbekannt). Solange die Auswahl
  offen ist, wechseln Ziffern keine Reihe.
- **Nachtragen** (β): `K` springt zum nächsten Gegnerschuss ohne Ergebnis und öffnet die Auswahl,
  danach geht es automatisch weiter („noch x von y offen“). So lassen sich ältere Spiele in
  wenigen Minuten ergänzen.
- **Bully** gewonnen `B` / verloren `N`, **Strafe** `P`.
- **Unterbrechung** `O` (Timeout, Verletzung, Sonstiges), nochmal `O` beendet sie. Die Zeit
  zählt nicht als Eiszeit. Im Experimentmodus (β) zieht das Tool die Zeit zwischen Tor und
  Anspiel zusätzlich automatisch ab.

## Durchgang 2: Ballbesitzketten

Das Tool weiß aus Durchgang 1, welche Spieler gerade auf dem Feld stehen, und zeigt deren
Buttons. Du erfasst nur noch den Ball.

1. **Start (optional):** `A` Ballgewinn · `S` Bully · `D` Freischlag · `F` Torwart · `G` loser Ball.
   Ohne Starttaste wählt das Tool nach einem Bully „Bully“, nach einem Gegnerschuss „Torwart“,
   sonst „unbekannt“. Ein Ballgewinn zählt nur mit gedrückter Taste `A`.
2. **Jeder Ballkontakt:** Spieler per Positionstaste (`1`–`5`, Torwart `6`, anderer Spieler `0`).
   Erst drücken, wenn der Ball wirklich angekommen ist.
3. **Ende (Pflicht):** `Q` Fehlpass · `W` Ballverlust · `E` Schuss (danach `1` Tor,
   `2` gehalten, `3` geblockt, `4` vorbei) · `R` weggeschlagen · `T` Unterbrechung.

Beispiel: Ballgewinn durch den linken Verteidiger, über Flügel und Top zum Schuss auf das Tor:
`A` `1` `3` `5` `4` `E` `2`

## Tasten auf einen Blick

| Bereich | Tasten |
|---|---|
| Video | Leertaste Play/Pause · `←` `→` 2 s · Umschalt + `←` `→` 10 s · `,` `.` Tempo |
| Durchgang 1 | `D` Drittel · `1`–`6` Reihen · `X` Gegnerschuss · `H` Gegentor · `J` Tor · `B`/`N` Bully · `P` Strafe |
| Durchgang 1 (β) | nach `X`: `1` Tor · `2` gehalten · `3` geblockt (dann Blocker per Positionstaste, `0` anderer) · `4` vorbei · `Enter` unbekannt · `K` nächster Gegnerschuss ohne Ergebnis |
| Durchgang 2 | `1`–`6` Spieler nach Position · `0` anderer Spieler · `A S D F G` Start · `Q W E R T` Ende · nach `E`: `1`–`4` Schussergebnis |
| Immer | `O` Unterbrechung · `Strg+Z` rückgängig · `Esc` (oder Rücktaste) Kette verwerfen |
| Ein Bildschirm | `Z` Theatermodus an/aus |
| Playlist (β) | `N` nächste Szene · `V` vorherige · `Esc` schließen |

Spielerbelegung im GF 2-2-1: `1` LV · `2` RV · `3` LF · `4` RF · `5` TOP · `6` Torwart.
Die Taste steht klein auf jedem Spielerbutton.

## Auswertung

Alle Werte gibt es für das Gesamtspiel und für jedes Drittel einzeln.

**Übersicht**
- **Plausibilitätschecks:** Abgleich mit dem Spielbericht, Lücken ohne Reihe auf dem Feld,
  offene Ketten, Unterbrechungen ohne Ende, Tore ohne Bully, Ketten mit langen Lücken (β)
- **Spielertabelle:** Eiszeit, Einsätze, Ballgewinne, Schüsse nach Ergebnis, Bullys, Pässe,
  Fehlpässe, Ballverluste, Plus/Minus; im Experimentmodus zusätzlich Spielzeit (geschätzt), CF,
  CA, CF %, Ballbesitzanteil, Konter verursacht (β)
- **Reihenvergleich:** Eiszeit, Schüsse und Gegnerschüsse, Tore, Werte pro 10 Minuten Eiszeit; im
  Experimentmodus pro 10 Minuten geschätzter Spielzeit und weitere Kennzahlen (β)
- **Passmatrix** (Heatmap) und **Passnetzwerk** pro Reihe
- **Gegnerschüsse nach Ergebnis** (β): Spielertabelle mit Blocks und zugelassenen Schüssen aufs
  Tor bzw. Fenwick gegen (auf dem Feld); Tabelle „Gegnerschüsse nach Reihe“ (aufs Tor, geblockt,
  vorbei, unbekannt, Anteil aufs Tor, Blockquote, FF / FA / FF %) und „Torhüter“ (Paraden,
  Fangquote). Plausibilitätschecks für offene Ergebnisse und Blocker.

**Erweiterte Kennzahlen** (β, nur im Experimentmodus)

| Kennzahl | Bedeutung |
|---|---|
| Spielzeit (geschätzt) | Videozeit pro Drittel auf die echte Drittellänge umgerechnet, weil bei Ball im Aus die Uhr steht. Alle Werte „pro 10 min“ beziehen sich darauf. |
| Corsi (CF, CA, CF %) | Schussversuche für und gegen, während ein Spieler oder eine Reihe auf dem Feld steht. Dazu CF % relativ zum Team. |
| Ballbesitz | Ballbesitzzeit, Anteil an der Netto-Videozeit, Phasendauer, längste Phase |
| Umschalten | Konter gegen uns nach eigenem Ballverlust, eigene Konter, Gegenpressing-Erfolg |
| Schichtverlauf | Schüsse für und gegen nach Schichtdauer (0–45 s, 45–90 s, über 90 s) |
| Bully-Wert | Was in den 10 Sekunden nach einem Bully passiert |
| Spielstand | Werte bei Führung, Gleichstand, Rückstand |
| Duos | Corsi von Spielerpaaren zusammen und getrennt |
| Über- und Unterzahl | Überzahlquote, Unterzahl überstanden, Schüsse, Ballbesitz |
| Rollen im Passnetzwerk | Wer verteilt, wer wird angespielt, Anteil an allen Pässen |

Neben jeder Quote steht die Stichprobengröße n. Werte mit n < 5 sind grau. Die genauen
Definitionen stehen in [SPEC.md](SPEC.md), Abschnitt „Kennzahlen – Definitionen“.

## Szenen abspielen und Clips exportieren

(β, nur im Experimentmodus) **Jede Zahl in der Auswertung ist anklickbar.** Ein Klick öffnet eine
Playlist der zugehörigen Szenen und spielt sie im Videofenster (bzw. im Ein-Bildschirm-Modus im
eingebetteten Video) ab, mit passendem Vor- und Nachlauf (z. B. Tore 7 s davor,
3 s danach). Dazu gibt es eine Schnellauswahl „Alle Tore“, „Alle Gegentore“, „Alle Schüsse
von …“.

Jede Playlist lässt sich exportieren:
- als **CSV-Schnittliste** (Start, Ende, Beschreibung)
- als **.bat-Datei für ffmpeg**, die die Szenen aus dem Video schneidet und zu einem Clip
  zusammenfügt. Die .bat und das Video müssen im selben Ordner liegen.

## Export und Sicherung

- **Excel:** Statistik-Blätter für Gesamtspiel und jedes Drittel im Stil der Vereinsvorlage,
  Passmatrix und alle Ereignisse; im Experimentmodus zusätzlich Kennzahlen-Blätter (Corsi,
  Ballbesitz, Umschalten, Spielstand, Duos, Über-Unterzahl) und weitere Spalten (β)
- **JSON:** komplette Rohdaten. Damit kann ein Spiel auf einem anderen Rechner weiterbearbeitet
  oder später neu ausgewertet werden.

**Wichtig:** Die Daten liegen während der Arbeit nur im Browser-Speicher. Speichere nach jedem
Drittel eine JSON-Sicherung. Leerst du den Browser-Cache oder wechselst du den Rechner, ist der
Stand sonst weg.

## Regeln für saubere Daten

- **Erst `D`, dann die Reihe** zu jedem Drittelbeginn
- **Ball im Aus:** Kette mit `T` beenden, beim Freischlag neu mit `D` beginnen. Sonst enthält
  die Ballbesitzzeit Stillstand.
- **Spieler erst drücken, wenn der Ball angekommen ist.** Ein Pass, der nicht ankommt, ist ein
  Fehlpass (`Q`) des Passgebers.
- **Fliegende Wechsel** immer zum gleichen Moment erfassen, z. B. wenn der erste neue Spieler
  das Feld betritt
- **Video nicht umbenennen oder verschieben**, nachdem du mit einem Spiel begonnen hast. Das Tool
  erkennt es an Name und Größe.

## Datenschutz

- Das Video wird **nur lokal** im Browser abgespielt und nie hochgeladen.
- Alle Spieldaten bleiben im Browser des jeweiligen Rechners und in den Dateien, die du selbst
  exportierst.
- In diesem Repository liegen **keine Spieldaten, Videos oder Auswertungen**. Bitte auch keine
  hochladen (die `.gitignore` schließt sie aus).

## Bekannte Grenzen

- Die Spielzeit ist **geschätzt**: Stillstand bei Ball im Aus wird gleichmäßig über das Drittel
  verteilt angenommen. Exakt wäre es erst mit der Spieluhr aus dem Video.
- Im Standardmodus wird bei Gegnerschüssen kein Ergebnis erfasst, für den Gegner gibt es dort kein
  „aufs Tor %“ und kein Fenwick. Im Experimentmodus (β) lässt sich das Ergebnis erfassen und
  nachtragen; Quoten zählen nur Schüsse mit bekanntem Ergebnis.
- Gleichzeitige Strafen (4 gegen 4) und angezeigte Strafen werden nicht gesondert erfasst.
- Getestet mit Google Chrome unter Windows.

## Lokal nutzen und weiterentwickeln

Ohne Internet: Repository herunterladen (grüner Button „Code“ → „Download ZIP“), entpacken und
`videotagger.html` per Doppelklick in Chrome öffnen. `exceljs.min.js` muss im selben Ordner
liegen.

| Datei | Inhalt |
|---|---|
| `videotagger.html` | das komplette Tool (HTML, CSS, JavaScript in einer Datei) |
| `exceljs.min.js` | Bibliothek für den Excel-Export |
| `index.html` | Startseite für den Weblink, leitet auf das Tool weiter |
| `SPEC.md` | Spezifikation mit allen Funktionen, Tastenbelegung und Kennzahl-Definitionen |

Änderungen bitte zuerst in `SPEC.md` beschreiben und dann umsetzen. Das Tool wurde mit
Claude Code entwickelt; die SPEC ist die Grundlage für jede Erweiterung.

Ideen für später stehen am Ende der SPEC, u. a. Schussort per Klick, Zonenauswertung mit
Kamerakalibrierung und das Auslesen der Spieluhr.

## Lizenz

Code: [MIT](LICENSE). Enthält [ExcelJS](https://github.com/exceljs/exceljs) (MIT), siehe
[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
