# Spezifikation: VfB Ulm Video-Tagger

## Ziel
Ein Offline-Tool zur Videoanalyse eigener Floorball-Spiele (Großfeld und Kleinfeld).
Das Video läuft in einem eigenen Fenster auf Bildschirm 2, die Erfassung im
Steuerfenster auf Bildschirm 1. Jede Eingabe erhält automatisch die exakte
Videozeit. Erfasst werden Reihenwechsel, Schüsse, Tore, Bullys, Strafen und
Ballbesitzketten (wer passt zu wem), daraus entstehen Spielerstatistik,
Passmatrix, Passnetzwerk pro Reihe und Reihenvergleich.

Ziel-Aufwand: ein komplettes Spiel in zwei Durchgängen, insgesamt unter 3 Stunden.

## Umsetzung in vier Schritten
Bitte in dieser Reihenfolge bauen und nach jedem Schritt auf mein OK warten:
1. Videofenster und Tastatursteuerung
2. Setup-Bildschirm und Durchgang 1
3. Durchgang 2 (inkl. automatischer Reihenanzeige)
4. Auswertung und Export

Weise vor Schritt 1 auf Unklarheiten oder technische Probleme in dieser
Spezifikation hin (insbesondere Zwei-Fenster-Prinzip unter file://).

## Technik
- Eine einzelne HTML-Datei (HTML/CSS/JS, keine Build-Tools), läuft in Chrome.
- Komplett offline. Videos werden lokal per Dateiauswahl geöffnet (liegen auf
  D:\VfB_Ulm_Videos), nichts wird hochgeladen.
- Eigenständig: Der Tagger läuft auf jedem Gerät nur mit `videotagger.html`
  und `exceljs.min.js`. Kein Zugriff auf andere Ordner oder Projekte,
  keine gemeinsamen Dateien. Einzige optionale Online-Funktion ist der
  Abrufversuch beim Saisonmanager-Import; alles funktioniert auch ohne
  Internet.
- Falls Chrome bei Start per file:// Probleme mit dem zweiten Fenster oder den
  Video-Blob-URLs macht: Start über einen lokalen Server
  (z. B. `python -m http.server`) dokumentieren und eine start.bat beilegen.

## Zwei-Fenster-Architektur
- Steuerfenster = Hauptseite. Button „Videofenster öffnen“ öffnet per
  window.open('', 'video') ein leeres Fenster, dessen DOM das Steuerfenster
  selbst aufbaut (ein <video>-Element, schwarzer Hintergrund, Overlay mit
  Videozeit, aktuellem Drittel und aktueller Reihe).
- Das Steuerfenster hält eine direkte Referenz auf das <video>-Element und
  steuert es (play, pause, currentTime, playbackRate). Kein postMessage nötig.
- Tastenkürzel müssen in BEIDEN Fenstern funktionieren: keydown-Listener in
  beiden Fenstern registrieren, die dieselbe Funktion aufrufen. Egal welches
  Fenster den Fokus hat, jede Eingabe wirkt.
- Wird das Videofenster geschlossen, kann es neu geöffnet werden, ohne dass
  Daten verloren gehen; Position im Video bleibt erhalten.

## Ein-Bildschirm-Modus
- Schalter in der Kopfzeile „Zwei Bildschirme | Ein Bildschirm“. Die Wahl
  wird pro Gerät gemerkt (localStorage `vfbTagger.geraet.v1`, try/catch),
  nicht im Spiel. Meldet der Browser nur einen Bildschirm
  (`screen.isExtended === false`), schlägt das Tool beim ersten Start einmal
  „Ein Bildschirm“ vor (ohne Berechtigungsabfrage).
- Wiedergabeschicht gemeinsam: `wirePlayer()` verbindet ein `<video>` und
  die Overlay-Elemente mit dem Player-Zustand, egal ob im Videofenster oder
  im Hauptfenster. Play, Pause, Springen, Tempo, Overlay (Videozeit,
  Drittel, Reihe, laufende Kette, Unterbrechung) und Einblendungen laufen
  über dieselben Funktionen. Tastenkürzel sind identisch.
- Wechsel: Position, Tempo und Play/Pause bleiben erhalten; ein offenes
  Videofenster wird geschlossen, das Video läuft im Hauptfenster an
  derselben Stelle weiter (dieselbe Datei, kein erneutes Wählen). Zurück zu
  „Zwei Bildschirme“ über „Videofenster öffnen“; das Video geht dort an
  derselben Stelle weiter. Die laufende Kette und alle Daten bleiben
  unberührt.
- Layout: Video links (breites Fenster) bzw. oben (hochkant/schmal), so
  groß wie möglich (bei 1366 × 768 etwa 63 % der Breite, bei 1920 × 1080
  etwa 74 %). Daneben die Erfassung kompakt (Spielerbuttons in Formation,
  Start/Ende, laufende Kette, letzte Aktion, Reihen- und Ereignisbuttons,
  Hilfetexte ausgeblendet; Zeitanzeige und Fortschrittsbalken in einer
  Zeile). Die Ereignisliste ist eine einklappbare Schublade von rechts
  (Knopf „Liste ▸“, Standard: eingeklappt). Geprüft: Durchgang 1 und 2
  passen bei 1366 × 768 (auch mit nur ~650 px Innenhöhe des Browsers)
  ohne Scrollen.
- Das Video liegt außerhalb der Reiter und wird nur per CSS positioniert
  (Chrome hält ein Video an, wenn es im DOM verschoben wird). Setup und
  Auswertung verdecken es; zurück in der Erfassung steht es wieder an
  derselben Stelle. Läuft eine Playlist (Experiment), schwebt das Video in
  der Auswertung oben rechts über der Playlist.
- Theatermodus: Taste `Z` (ohne Strg; Strg+Z bleibt Rückgängig) oder
  Doppelklick aufs Video vergrößert das Video auf fast das ganze Fenster,
  unten bleibt eine schmale Leiste mit Drittel, Reihe, laufender Kette und
  letzter Aktion. Alle Erfassungstasten wirken weiter; nochmal `Z` zurück.
  Nur im Ein-Bildschirm-Modus in der Erfassung; im Videofenster bleibt
  Doppelklick = Vollbild. Für mehr Fläche zusätzlich F11 (Browser-Vollbild).

## Standard- und Experimentmodus
- Schalter in der Kopfzeile „Standard | Experiment“, Standard ist
  voreingestellt; pro Gerät gemerkt (localStorage, try/catch). Im
  Experimentmodus steht das Abzeichen „β Experiment“ in der Kopfzeile, und
  jede experimentelle Spalte, Zeile oder Fläche trägt „β“ (Bildschirm und
  Excel).
- Standard: genau die Funktionen der Analyse Heidelberg-Mannheim
  (20.10.2026) mit denselben Zahlen. Experiment: zusätzlich alle neuen,
  noch nicht im Echtbetrieb getesteten Funktionen. Neue Funktionen kommen
  immer zuerst in den Experimentmodus und werden einzeln übernommen.
- Ein Modus steuert nur Darstellung und Berechnung. Spieldaten (Ereignisse,
  Setup, JSON-Sicherung) sind in beiden Modi identisch; ein Wechsel ist
  jederzeit möglich, ohne Datenverlust und ohne Neuladen des Videos. Die
  Setup-Felder Drittellänge/Verlängerung sind im Standard ausgeblendet, ihre
  Werte bleiben gespeichert.
- Excel im Experimentmodus: Dateiname mit `_experiment`, in jedem Blatt
  oben rot „Experimentmodus – enthält ungetestete Kennzahlen“ (Statistik-
  Blätter und Passmatrix am Titel in A1, Kennzahlen-Blätter in Zeile 4,
  Ereignisse in H1).
- Funktions-Schalter im Code: zentrale Liste `FEATURES`; eine Funktion ist
  aktiv, wenn ihr Wert `'standard'` ist oder der Experimentmodus läuft.
  Übernehmen in den Standard = Wert auf `'standard'` setzen.

| Schalter | Modus | Inhalt |
|---|---|---|
| `erweiterteKennzahlen` | Experiment | Unterreiter „Erweiterte Kennzahlen“ (alle Blöcke); Spielertabelle CF, CA, CF %, CF % rel., Ballbesitzanteil, Konter verursacht; Reihenvergleich CF %, Ballbesitzanteil, längste Phase, Konter gegen uns, Gegenpressing, CA/10 je Schichtphase; Passnetzwerk „beteiligt an %“; Plausibilitätscheck „> 8 s zwischen zwei Kontakten“ |
| `spielzeitGeschaetzt` | Experiment | Setup-Felder Drittellänge/Verlängerung; Spalte „Spielzeit (gesch.)“; „/10“ auf geschätzte Spielzeit (sonst Eiszeit); Spaltenname „Eiszeit (Video)“ (sonst „Eiszeit“). Die Erweiterten Kennzahlen rechnen intern immer mit der Schätzung |
| `autoTorPause` | Experiment | Tor/Gegentor bis Bully zählt nicht als Eiszeit (sonst nur O-Unterbrechungen); wirkt auf Eiszeit, Ø Einsatz, Schichtliste, Drittel-Faktor |
| `playlist` | Experiment | Zahlen anklickbar, Schnellauswahl, Playlist mit N/V/Esc, CSV- und ffmpeg-Export |
| `stichprobeN` | Experiment | „n=…“ und Graufärbung bei n < 5 (Bildschirm und Excel inkl. n-Spalten, Zeile „n Ballbesitzketten“, „Anteil“, Pass % der Passmatrix) |
| `kennzahlenExport` | Experiment | Excel: Kennzahlen-Block in den Statistik-Blättern, Reihenvergleich-Spalten ab M, Blätter Corsi/Ballbesitz/Umschalten/Spielstand/Duos/Über-Unterzahl, neue Legendentexte |
| `gegentorAlsSchuss` | Standard | Gegentor ohne Gegnerschuss ±3 s zählt als Gegnerschuss der Reihe (Heidelberg-Mannheim: kein Fall, keine Zahl verändert) |
| `plausiTorOhneBully` | Standard | Plausibilitätscheck „Tor ohne Bully in 3 min“ (nur Hinweis, keine Zahl) |
| `gegnerschussErgebnis` | Experiment | Nach X Ergebnis (gehalten/geblockt/vorbei/Tor) und Blocker erfassen, Nachtragen mit K, Bearbeiten in der Ereignisliste; Blocks, On-Field aufs Tor/FA, Tabellen „Gegnerschüsse nach Reihe“ und „Torhüter“, Fenwick für beide Seiten, Plausibilitätschecks, Excel-Zeilen/-Spalten (siehe „Ergebnis von Gegnerschüssen“) |

Ebenfalls im Standard (harmlos, ohne Schalter): Bully-Warntext „kein
Bullyspieler“; Ein-Bildschirm-Modus (unabhängig vom Betriebsmodus).

### Prüfung (27.10.2026)
- Standard, Sicherung Heidelberg-Mannheim vom 20.10., Excel gegen
  `VfB_Ulm_2026-10-18_SG_Heidelberg-Mannheim_Auswertung.xlsx`: Blätter
  Gesamt und 1.–3. Drittel – alle ganzen Zahlen (Einsätze, Ballgewinne,
  Ballverluste, Schüsse, Bullys, Pässe, Fehlpässe, +/−, Reihenvergleich)
  identisch; Zeiten ≤ 0,29 s und /10′ ≤ 0,005 abweichend (Rundung der
  Referenz, die aus dem alten Export erzeugt wurde). Passmatrix:
  Gesamtmatrix identisch; Matrizen je Reihe weichen wie seit dem 20.10.
  gewollt ab (Reihe beim Pass statt beim Kettenbeginn). 706 Formeln ohne
  Fehler (HyperFormula).
- Experiment: Prüfwerte vom 27.10. unverändert (Faktoren 0,840/0,842/0,872,
  Schüsse 12/16 · 17/5 · 8/22, CF % 43/77/27, Ballbesitz 8,23/7,33/6,95 min,
  längste Phase 43,1 s, 11 Rückeroberungen, 3 Gegentore ≤ 20 s,
  Gegenpressing 22/87); 782 Formeln ohne Fehler. Eiszeit Reihe 1: Standard
  20:12,8, Experiment 20:10,1 (Tor-Pause).
- Sechsmal Modus und zweimal Anzeige gewechselt: Spieldaten und
  gespeicherter Stand identisch.

## Videodatei (eine Datei pro Spiel)
- Pro Spiel gibt es genau EINE Videodatei (MP4, H.264). Die Kameraabschnitte
  werden vorher zusammengeschnitten. Keine Playlist, kein Dateiwechsel.
- Jedes Ereignis speichert die Videozeit (time = video.currentTime).
- Nach Neuladen der Seite: Video erneut auswählen lassen, anhand von
  Dateiname und Größe den gespeicherten Daten zuordnen und an die letzte
  Position springen.
- Klickbarer Fortschrittsbalken über die gesamte Spieldauer.

## Videosteuerung (Tastatur)
- Leertaste: Play/Pause
- Pfeil links/rechts: 2 s zurück/vor; mit Umschalt: 10 s
- Komma / Punkt: Tempo runter/hoch (0,5x · 0,75x · 1x · 1,25x · 1,5x · 2x)
- Z: Theatermodus (nur Ein-Bildschirm-Modus, siehe dort)
- Tempo und Position immer im Steuerfenster sichtbar.

## Setup-Bildschirm (vor dem Spiel)
- Spielinfo: Datum, Gegner, Heim/Auswärts, Modus GF oder KF.
- (Experiment) „Drittellänge (min)“ (Standard 20, für KF frei einstellbar) und
  „Verlängerung (min)“ (Standard 10). Grundlage der geschätzten Spielzeit
  (siehe „Kennzahlen – Definitionen“). Leeres Feld = Faktor 1 für dieses
  Drittel, mit Hinweis in der Auswertung.
- Kader: Name, Rückennummer, Position. Torwart separat.
- Spielsystem (Formation) einstellbar, getrennt für GF und KF. Die Formation
  bestimmt die Positionen einer Reihe, deren Anordnung auf dem Bildschirm,
  die Tastenbelegung in Durchgang 2 und die Punkte im Passnetzwerk.
  Vorlagen:
  - GF 2-2-1 mit Top (STANDARD): LV, RV (Verteidiger), LF, RF (Flügel/
    Mittelfeld), TOP (vorderster Spieler)
  - GF 2-1-2 (klassisch): LV, RV, C, LS, RS
  - KF 1-2 (STANDARD): D (Defender), LF, RF (zwei Flügel)
  - KF 1-1-1: D, C, S
  Positionsnamen sind im Setup umbenennbar. Datenstruktur generisch halten
  (Formation = Liste von Positionen mit { id, label, x, y, taste }), damit
  weitere Systeme ohne Codeänderung ergänzt werden können.
- Reihen anlegen: Spieler den Positionen der gewählten Formation zuordnen.
  Pro Reihe einen Standard-Bullyspieler festlegen (im 2-2-1 nicht
  zwingend der Top). Zusätzlich Sonderreihen ÜZ, UZ und Spezial mit frei
  wählbarer Spielerzahl und eigener Formation.
- Kader und Reihen als Vorlage speichern und für das nächste Spiel laden.

### Spielbericht aus dem Saisonmanager (optional)
- Feld für den Link (…/spiel/<ID>) oder die Spiel-ID aus saisonmanager.de.
- Geprüft (19.10.2026): Ein direkter Abruf aus dem Tagger funktioniert
  NICHT. Die API (`/api/v2/games/<ID>.json`) verlangt einen API-Schlüssel
  („API key required“, auch beim direkten Aufruf im Browser) und sendet
  keine CORS-Freigabe für lokale Dateien. Der Schlüssel der Website wird
  bewusst nicht verwendet. Der Button „Laden“ versucht den Abruf trotzdem
  (falls sich das ändert) und klappt bei Fehler den Ausweichweg auf.
- Ausweichweg mit Anleitung im Tool: Spiel auf saisonmanager.de in Chrome
  öffnen → F12 → Netzwerk → Filter „games“ → F5 → Rechtsklick auf
  „<ID>.json“ → Kopieren → Antwort kopieren → im Tool einfügen und
  „Übernehmen“. Alternativ die Antwort als .json-Datei laden.
- Vor dem Übernehmen ein Dialog: „Unser Team“ (automatisch, wenn genau ein
  Teamname „Ulm“ enthält) und Auswahl der Teile:
  - Spielinfo: Datum, Gegner, Heim/Auswärts.
  - Kader abgleichen per Rückennummer: Namen aktualisieren, neue Spieler
    ergänzen (bestehende Spieler-IDs bleiben, damit Ereignisse gültig
    bleiben); Torhüter erkennen (goalkeeper / Position „Tor“).
  - Tore (Drittel, Zeit, Schütze, Vorlage, Team, Stand) und Strafen
    (Drittel, Zeit, Team, Nummer, Art, Grund) als Referenzliste; Timeouts
    werden mitgespeichert.
  - Startaufstellung als Reihe 1, nur wenn der Bericht eine enthält
    (starting_players); sonst bleiben die Reihen manuell.
- Alles wird im Spiel gespeichert (`game.bericht`, auch in der Sicherung);
  danach braucht das Tool kein Internet mehr, auch nicht nach Neuladen.
- Beim Erfassen eines Tors (J) wird der Schütze (und die Vorlage) des n-ten
  Ulmer Tors laut Bericht vorgeschlagen und im Ereignis gespeichert
  (bearbeitbar).
- Ohne Internet oder bei Fehlern funktioniert die manuelle Eingabe wie
  bisher.

## Durchgang 1 (Spielgerüst, höheres Tempo)
Modus-Umschalter oben: „Durchgang 1“ / „Durchgang 2“.

Tasten in Durchgang 1:
- 1, 2, 3: Reihe 1/2/3 kommt aufs Feld
- 4: ÜZ, 5: UZ, 6: Spezial
- X: Gegnerschuss (wird automatisch der Reihe auf dem Feld zugeordnet;
  Experiment: danach Ergebnis und ggf. Blocker, siehe unten)
- K: (Experiment) nächster Gegnerschuss ohne Ergebnis (Nachtragen)
- H: Gegentor
- J: Tor Ulm
- B: Bully gewonnen, N: Bully verloren (Bullyspieler = Standard-
  Bullyspieler der aktuellen Reihe, per Klick änderbar)
- P: Strafe (Dialog: Team, Spieler, Dauer)
- D: Drittelbeginn / Drittelende
- O: Spielunterbrechung Beginn / Ende (siehe „Spielunterbrechungen“;
  wirkt in beiden Durchgängen)

Einzelwechsel: Rechtsklick auf einen Spielerbutton öffnet eine Bankliste,
der gewählte Spieler ersetzt ihn ab der aktuellen Videozeit.

Bei Tor und Gegentor wird automatisch gespeichert, welche Spieler laut
aktuellem Stand auf dem Feld stehen (für Plus/Minus). Der Stand wird im
Ereignis angezeigt und ist bearbeitbar.

### Ergebnis von Gegnerschüssen (Experiment)
Schalter `gegnerschussErgebnis`. Im Standardmodus verhält sich X wie
bisher (keine Auswahl), alle Zahlen bleiben gleich.
- X legt den Gegnerschuss sofort an (Videozeit, Reihe). Danach erscheint in
  der Erfassung (und im Video-Overlay) die Ergebnisauswahl; das Video läuft
  weiter, wie bei der Schussauswahl in Durchgang 2:
  - 2 = gehalten (aufs Tor, Torwart hält; auch ein abgefälschter Schuss,
    der trotzdem aufs Tor kommt)
  - 3 = geblockt (von einem Ulmer Feldspieler aufgehalten, bevor er das Tor
    erreicht)
  - 4 = vorbei (inkl. Pfosten/Latte)
  - 1 = Tor: der eben angelegte Gegnerschuss wird zum Gegentor (wie H, mit
    den Spielern auf dem Feld für Plus/Minus) – kein zweites Ereignis
  - Esc oder Enter = unbekannt (Schuss bleibt ohne Ergebnis)
- Nach 3 folgt die Blocker-Auswahl: Positionstasten der Formation der Reihe
  auf dem Feld (wie in Durchgang 2, z. B. GF 2-2-1: 1 LV, 2 RV, 3 LF, 4 RF,
  5 TOP), Buttons anklickbar, 0 = anderer Spieler (Kader-Dialog, hält das
  Video kurz an), Esc/Enter = Blocker unbekannt. Der Torwart ist kein Blocker.
- Solange die Auswahl offen ist, wirken Ziffern nur dort (keine
  Reihenwechsel; andere Ziffern werden ignoriert). Ein Buchstabe (X, H, B,
  D, O …) schließt die Auswahl (Ergebnis unbekannt) und wirkt dann normal.
- Letzte Aktion z. B. „Gegnerschuss – gegen Reihe 2 · geblockt · #7 Müller“.
- Ergebnis und Blocker sind in der Ereignisliste bearbeitbar (dort auch
  „Tor – wird zum Gegentor“); Strg+Z nimmt Ergebnis, Blocker und die
  Umwandlung zum Gegentor jeweils einzeln zurück.
- Nachtragen: Knopf „Nächster Gegnerschuss ohne Ergebnis“ und Taste K
  (in keinem Modus anders belegt) springen 3 s vor den nächsten offenen
  Gegnerschuss ab der aktuellen Videozeit (am Ende wieder von vorn),
  spielen ab und öffnen die Auswahl; danach geht es automatisch weiter.
  Enter = unbekannt lassen und weiter, Esc = Nachtragen beenden. Anzeige
  „noch x von y offen“.
- Ein Gegnerschuss mit Gegentor ±3 s (X und H zur selben Szene, wie in
  älteren Spielen) gilt als Tor: nicht „offen“, wird beim Nachtragen
  übersprungen.

### Reihenbesetzung pro Drittel
Oft fehlt ein Spieler und jemand hilft in einer zweiten Reihe aus, pro
Drittel ein anderer. Deshalb:
- Beim Erfassen von „Drittelbeginn“ (Taste D) erscheint der Dialog
  „Besetzung für dieses Drittel“ mit allen Reihen, vorausgefüllt mit der
  Besetzung des vorherigen Drittels (beim 1. Drittel: Setup), per Klick
  änderbar. Enter übernimmt. Abbrechen erfasst den Drittelbeginn nicht.
- Die Besetzung wird als eigenes Ereignis `aufstellung` mit Zeitstempel
  gespeichert (zusammen mit dem Drittelbeginn ein Schritt für Strg+Z).
  Das Setup bleibt unverändert und gilt nur, solange es noch keine
  Besetzung gibt.
- Reihenwechsel (Tasten 1–6) übernehmen immer die Besetzung, die zur
  Videozeit des Wechsels gilt, auch beim nachträglichen Erfassen in einem
  früheren Drittel. Eine neue Besetzung gilt ab Drittelbeginn auch für die
  Reihe, die gerade auf dem Feld steht.
- Wird eine Besetzung nachträglich geändert (Ereignisliste, ✎), bietet das
  Tool an, die Wechselereignisse dieses Drittels anzupassen. Einzelwechsel
  bleiben erhalten.
- „Auch in Reihe übernehmen“ beim Einzelwechsel ändert die Besetzung des
  laufenden Drittels (ohne Besetzung: das Setup).

## Durchgang 2 (Ballbesitzketten, normales Tempo)
Das Tool ermittelt aus den Wechseln von Durchgang 1, welche Spieler zur
aktuellen Videozeit auf dem Feld stehen, und zeigt automatisch deren
Buttons. Das funktioniert auch beim Zurückspulen.

Bildschirm: Spielerbuttons so angeordnet wie die gewählte Formation auf
dem Feld (Spielrichtung nach oben, Torwart unten mittig). Beispiel 2-2-1:
TOP oben mittig, LF/RF in der Mitte links/rechts, LV/RV unten links/rechts.
Darunter die laufende Kette als Text, z. B.
„Ballgewinn → Chris → Ole → Chris → Franzi“.

Tasten in Durchgang 2:
- Spieler nach Position (fest, unabhängig vom Namen), Belegung kommt aus
  der Formation:
  GF 2-2-1: 1 = LV, 2 = RV, 3 = LF, 4 = RF, 5 = TOP, 6 = Torwart
  GF 2-1-2: 1 = LV, 2 = RV, 3 = C, 4 = LS, 5 = RS, 6 = Torwart
  KF 1-2:   1 = D, 2 = LF, 3 = RF, 6 = Torwart
  KF 1-1-1: 1 = D, 2 = C, 3 = S, 6 = Torwart
  Die zugehörige Taste steht klein auf jedem Spielerbutton.
- Start eines Ballbesitzes:
  A = Ballgewinn, S = Bully, D = Freischlag, F = Torwart, G = loser Ball
- Ende eines Ballbesitzes:
  Q = Fehlpass, W = Ballverlust (Zweikampf/Dribbling), E = Schuss,
  R = weggeschlagen, T = Unterbrechung
- Nach E öffnet sich die Schussauswahl: 1 = Tor, 2 = gehalten,
  3 = geblockt, 4 = vorbei
- 0: „Anderer Spieler“ (Auswahl aus dem ganzen Kader, falls jemand
  angespielt wird, der laut Tool nicht auf dem Feld ist)
- Esc: laufende Kette verwerfen

Regeln:
- Jeder Spielerklick hängt den Spieler an die Kette an und speichert die
  Videozeit. Derselbe Spieler zweimal hintereinander wird ignoriert.
- Startcode-Vorauswahl, wenn vor dem ersten Spieler kein Start gedrückt
  wurde: nach einem Bully-Ereignis aus Durchgang 1 in den letzten 3 s →
  Bully; nach einem Gegnerschuss (letzte 5 s) → Torwart; sonst →
  „unbekannt“. Die Vorauswahl ist hervorgehoben und jederzeit per
  Starttaste änderbar. Gespeichert wird `startVorauswahl = true`.
- Ballgewinn wird nie vorausgewählt, sondern nur per Taste A erfasst.
  Eine Kette mit Start „unbekannt“ zählt nicht als Ballgewinn.
- Ein Endcode schließt den Ballbesitz ab und speichert ihn.
- Ballgewinn-Definition: siehe „Entscheidungen“ (Hilfetext im Tool).

## Spielunterbrechungen (Timeouts, lange Unterbrechungen)
- Taste O startet eine Spielunterbrechung, nochmal O beendet sie. Die
  Taste wirkt in beiden Durchgängen (auch als Button „Unterbrechung“).
- Beim Start fragt ein kleiner Dialog die Art ab: Timeout VfB, Timeout
  Gegner, Verletzung, Sonstiges (Enter = Timeout VfB). Das Video pausiert
  währenddessen.
- Gespeichert als Ereignis `unterbrechung` mit art = beginn | ende und
  grund. Erscheint in der Ereignisliste (Filter „Unterbrechungen“) und im
  Excel-Blatt „Ereignisse“.
- Die Zeit zwischen Beginn und Ende zählt nicht als Eiszeit, nicht als
  Einsatzdauer und nicht in die Pro-10-Minuten-Werte. Eine Unterbrechung
  teilt einen Einsatz nicht: Bleibt dieselbe Reihe danach auf dem Feld,
  läuft der Einsatz weiter; ein Reihenwechsel während der Unterbrechung
  startet wie immer neue Einsätze (siehe „Einsätze“ in der Auswertung).
- Eine laufende Unterbrechung ist im Videofenster-Overlay (roter Balken,
  z. B. „⏸ TIMEOUT VfB“) und im Steuerfenster (rotes Badge mit Dauer)
  deutlich sichtbar.
- Nicht zu verwechseln mit dem Endcode T („Unterbrechung“) einer
  Ballbesitzkette in Durchgang 2.

## Korrektur und Sicherheit
- Strg+Z: Rückgängig, beliebig oft (auch Wechsel und Kettenschritte).
- Ereignisliste am rechten Rand, neueste oben, filterbar nach Typ.
  Klick auf eine Zeile springt im Video 3 s vor das Ereignis.
  Jede Zeile bearbeitbar und löschbar.
- Automatisches Speichern nach jeder Eingabe (localStorage oder IndexedDB).
- Button „Sicherung exportieren“ (JSON) und „Sicherung laden“.

## Auswertung (eigener Tab)
Oben ein Umschalter: Gesamt · 1. Drittel · 2. Drittel · 3. Drittel (plus
Verlängerung, falls erfasst). Alle Tabellen, die Passmatrix und das
Passnetzwerk richten sich danach; die Plausibilitätschecks gelten immer
für das ganze Spiel. Zuordnung zum Drittel über die Videozeit zwischen
Drittelbeginn und Drittelende; eine Kette zählt zu dem Drittel, in dem sie
beginnt; die Eiszeit wird am Drittelende abgeschnitten.

1. Spielertabelle im Stil der bisherigen Excel-Statistik:
   Reihe (Besetzung im gewählten Drittel; bei „Gesamt“ alle Reihen des
   Spielers, z. B. „R2 / R3“, ergänzt um tatsächliche Einsätze laut Wechseln),
   Eiszeit gesamt und pro Einsatz, Ballgewinne, Schüsse (Tor, gehalten,
   geblockt, vorbei, gesamt), Bullys (gewonnen, verloren, Quote),
   Pässe (erfolgreich, Fehlpass, Quote), Ballverluste, Plus/Minus.
   Eiszeit = Videozeit auf dem Feld ohne O-Unterbrechungen.
   (Experiment) dazu Spielzeit (geschätzt), CF, CA, CF %, CF % relativ,
   Ballbesitzanteil, Konter verursacht (Definitionen siehe „Kennzahlen –
   Definitionen“); „Eiszeit (Video)“ = ohne Stillstände (O-Unterbrechungen
   und Tor-Pausen).
   Erfolgreicher Pass = zwei aufeinanderfolgende Spieler in einer Kette.
   Fehlpass = letzter Spieler einer Kette mit Ende Q.
   Ballgewinn = erster Spieler einer Kette mit Start = Ballgewinn und
   startVorauswahl = false (Taste A gedrückt bzw. im Ereignis bestätigt).
   Einsätze = Reihen-Einsätze (nicht die zusammenhängende Zeit auf dem
   Feld), Tooltip: „Jeder Reihen-Einsatz zählt einzeln, auch ohne Verlassen
   des Feldes.“ Regeln:
   - Jeder Reihenwechsel (1–6) startet für alle Feldspieler der neuen Reihe
     einen neuen Einsatz, auch für Spieler, die schon auf dem Feld standen
     (Aushelfer von Reihe 1 direkt in Reihe 2 = 2 Einsätze; Wechsel in
     ÜZ/UZ und zurück zählen jeweils neu).
   - Doppeldruck derselben Reihe, während sie auf dem Feld steht: kein neuer
     Einsatz (nur für Spieler, die dabei neu dazukommen).
   - Einzelwechsel: nur der eingewechselte Spieler beginnt einen neuen
     Einsatz, die anderen laufen weiter.
   - Unterbrechungen (O) und die Zeit zwischen Tor und Anspiel teilen einen
     Einsatz nicht.
   - Drittelbeginn startet immer neue Einsätze.
   - Torhüter: neue Einsätze nur bei Torwartwechsel bzw. Drittelbeginn.
   - Ø Einsatz = Eiszeit / Einsätze. Gilt für alle Ansichten (Gesamt, pro
     Drittel) und den Excel-Export; berechnet rückwirkend aus den
     Ereignissen.
2. Passmatrix als Heatmap (Zeilen = Passgeber, Spalten = Empfänger),
   nach Reihen sortiert, Filter nach Reihe.
3. Passnetzwerk pro Reihe als SVG: Spieler als Punkte an ihrer
   Position laut Formation (x/y aus der Formation), Linienstärke = Passanzahl (beide Richtungen getrennt oder
   zusammengefasst umschaltbar), Punktgröße = Ballkontakte.
4. Reihenvergleich: Eiszeit, eigene Schüsse, zugelassene Gegnerschüsse
   (inkl. Gegentore ohne erfassten Gegnerschuss ±3 s), Tore, Gegentore,
   jeweils absolut und pro 10 Minuten Eiszeit. (Experiment) Spielzeit
   (geschätzt) und „/10“ auf geschätzte Spielzeit; dazu CF %,
   Ballbesitzanteil, längste Phase, Konter gegen uns, Gegenpressing-Erfolg,
   CA/10 je Schichtphase.
5. Plausibilitätschecks mit Warnhinweisen: Anzahl Tore aus Durchgang 1
   vs. Schüsse mit Ergebnis Tor aus Durchgang 2; Zeitabschnitte ohne Reihe
   auf dem Feld; nicht abgeschlossene Ketten; mit Spielbericht: Anzahl und
   Reihenfolge (Team, Drittel) der Tore und Strafen laut Bericht gegen die
   in Durchgang 1 erfassten, Schützen der Ulmer Tore; Unterbrechung ohne Ende;
   Unterbrechung über eine Drittelgrenze hinweg; Ballbesitzketten während
   einer Unterbrechung; Tor ohne Bully in den folgenden 3 min;
   (Experiment) Ketten mit mehr als 8 s zwischen zwei Kontakten;
   (Experiment) Gegnerschüsse ohne Ergebnis (Anzahl, Sprung zum ersten),
   geblockt ohne Blocker, Blocker laut Wechseln nicht auf dem Feld.
6. (Experiment) Unterreiter „Übersicht | Erweiterte Kennzahlen“. Die Erweiterten
   Kennzahlen zeigen die Blöcke Grundlagen (Faktoren je Drittel, Hinweis
   zur Schätzung), Corsi (Situation umschaltbar), Ballbesitzzeit (mit
   Prüfliste „> 8 s zwischen zwei Kontakten“, klickbar), Umschalten,
   Schichtverlauf (Tabelle + kleines Balkendiagramm), Bully-Wert,
   Spielstand, Duos (filterbar nach Spieler), Über- und Unterzahl, Rollen im
   Passnetzwerk. Der Drittel-Umschalter gilt auch hier. Im Passnetzwerk
   Punktgröße umschaltbar: Ballkontakte | beteiligt an %.
7. (Experiment) Clip-Playlist: jede Zahl ist anklickbar (Definition der Belege siehe
   „Kennzahlen – Definitionen“, Abschnitt 10) und öffnet unten rechts eine
   Playlist; Schnellauswahl „▶ Alle Tore“, „▶ Alle Gegentore“, „Alle
   Schüsse von …“.
8. (Experiment) Ergebnis von Gegnerschüssen: Spielertabelle zusätzlich
   Blocks, Blocks/10, Gegner aufs Tor (Feld) und FA (Feld); darunter die
   Tabellen „Gegnerschüsse nach Reihe β“ (Gegnerschüsse, aufs Tor,
   geblockt, vorbei, unbekannt, aufs Tor/geblockt/vorbei je 10 min, Anteil
   aufs Tor, Blockquote, FF, FA, FF %) und „Torhüter β“ (Paraden,
   Gegentore, Fangquote). Definitionen siehe „Kennzahlen – Definitionen“,
   Abschnitt 13.

## Entscheidungen (geklärt nach Plan-Review)
- Startart: file:// per Doppelklick.
- Excel-Export: ExcelJS als zweite Datei `exceljs.min.js` neben der HTML
  (früher SheetJS `xlsx.full.min.js`, entfernt, weil es keine Formatierung
  schreiben kann).
- Ballgewinn-Definition (Hilfetext): „Gewinn durch Zweikampf oder Abfangen
  von Pässen. Gezählt nur, wenn der Ball nicht sofort wieder verloren wurde.“
- Tore: Durchgang 1 (J/H) ist maßgeblich (Feldbesetzung für Plus/Minus),
  Durchgang 2 dient nur dem Abgleich.
- Strafen: ÜZ/UZ-Reihe wird manuell gesetzt (4/5); nach P erscheint nur ein
  Hinweis „ÜZ/UZ-Reihe setzen?“.
- Torwartwechsel wie Einzelwechsel (Rechtsklick auf Torwart-Button);
  gezogener Torwart über Reihe „Spezial“ mit 6 Feldspielern.

## Export
Excel-Export im Stil der VfB-Vorlage
(`VfB_Ulm_2026-10-18_SG_Heidelberg-Mannheim_Auswertung.xlsx`, Logik aus
`Referenz_Excel-Layout.py` nach JavaScript übertragen). Bibliothek: ExcelJS
(`exceljs.min.js` neben der HTML, offline), weil die freie SheetJS-Version
keine Formatierung schreibt. Dateiname: `VfB_Ulm_<Datum>_<Gegner>_Auswertung.xlsx`.

Blätter: „Gesamt“, „1. Drittel“, „2. Drittel“, „3. Drittel“ (plus
„Verlängerung“, falls erfasst), „Passmatrix“, (Experiment) „Corsi“,
„Ballbesitz“, „Umschalten“, „Spielstand“, „Duos“, „Über-Unterzahl“,
dann „Ereignisse“. Im Experimentmodus Dateiname
`…_Auswertung_experiment.xlsx` und Hinweis in jedem Blatt. Die früheren
Rohdaten-Blätter entfallen; Formeln verweisen direkt auf Zellen der
formatierten Blätter. Jede Formel wird mit ihrem Ergebnis gespeichert, damit
auch Programme ohne Neuberechnung die Werte zeigen.

### Statistik-Blätter (Gesamt und je Drittel)
- Spieler als Spalten ab E, gruppiert nach Stammreihe laut Setup (erste
  Reihe 1–3, in der ein Spieler steht; dann ggf. „Weitere“, dann Torwart),
  jede Gruppe mit dickem Kasten; danach „Alle“ (Formeln) und „Anteil“.
- Kopf: Zeile 1 Vorname (Kommentar „#Nr Name (Position)“), Zeile 2 Nr
  (Format `"#"@`), Zeile 3 tatsächlich gespielte Reihen im Blatt-Zeitraum
  (z. B. „R1 / R2“).
- Zeilenblöcke (Füllfarbe): Shiftlänge `FFFF00` (Zeit gesamt, /Einsatz,
  Einsätze) · Ballbesitz `F4B183` (Ballgewinn, Ballverlust, Bilanz) ·
  Schuss `FF99FF` (ins Tor, aufs Tor, geblockt, vorbei, Gesamt) · Bully
  `92D050` (gewonnen, nicht) · Pässe `9DC3E6` (erfolgreich, nicht, gesamt,
  erfolgreich %) · +/− `D9D9D9` · (Experiment) Kennzahlen β `D9C3E9` (Spielzeit gesch.,
  CF, CA, CF % als Formel, CF % rel. in Prozentpunkten, Ballbesitzanteil
  (Netto-Video), n Ballbesitzketten, Konter verursacht; Torhüter ohne Corsi; „Alle“ nur bei Konter, weil
  CF/CA für alle Spieler auf dem Feld zählen). Schrift durchgehend
  Calibri 11 (wie die Vorlage-Datei).
- Zeiten als Excel-Zeit `[m]:ss`, Quoten `0%`, Bilanz und +/− `+0;-0;0`.
  Rohwerte als Zahlen; Summen, Quoten, /Einsatz, Bilanz, „Alle“ und
  „Anteil“ (Schussarten, Bully) als Formeln.
- Legende: Tor `FFC000`, Gegentor `FF7C80`, Drittelbeginn fett
  unterstrichen; Hinweis „Zeiten = Videozeit ohne Unterbrechungen
  (Timeouts, Tor bis Anspiel, Verletzungen)“. (Experiment) „Zeit gesamt =
  Videozeit ohne Stillstände …; Spielzeit (gesch.) = × Faktor des Drittels“
  und Hinweise zu Stichprobe und Ballbesitzanteil.
- Reihenvergleich (Reihe 1–3, ÜZ, UZ, ggf. Spezial/ohne Reihe, Gesamt):
  Eiszeit, Schüsse, Gegnerschüsse, Tore, Gegentore und je /10′ als
  Formel bezogen auf die Eiszeit in Spalte C (`IFERROR(E/(C*144),"–")`);
  Farbskala bei Gegnerschüssen /10′ in Reihe 1–3 (grün `C6EFCE` = wenig,
  rot `FFC7CE` = viel).
- (Experiment) Reihenvergleich ab Spalte M: M Spielzeit (gesch.), /10′
  dann auf Spalte M (`IFERROR(E/(M*144),"–")`); N CF %, O n (CF+CA),
  P Ballbesitzanteil (Netto-Video), Q n (Ketten), R längste Phase,
  S Konter gegen uns, T Gegenpressing, U n (Verluste), V–X CA/10′ je
  Schichtphase (0–45 s, 45–90 s, über 90 s). Die Spalten werden je
  Schalter fortlaufend vergeben (ohne `stichprobeN` entfallen die
  n-Spalten).
- (Experiment) Quoten mit Stichprobe n < 5 grau und kursiv (auch „Anteil“,
  Pass % in der Passmatrix).
- Schichtliste je Reihe, Überzahl, Unterzahl (ggf. Spezial): Nettodauer
  ohne Unterbrechungen. Eine Schicht beginnt mit einem Reihenwechsel
  (Doppeldruck zählt nicht) oder am Drittelbeginn für die Reihe, die noch
  auf dem Feld steht. Tor-/Gegentor-Schichten eingefärbt, erste Schicht
  jedes Drittels fett unterstrichen, bei ÜZ/UZ die Spieler daneben, unten
  Σ und Ø als Formeln.
- Fixierung bei E4, Zoom 90 %, Querformat, eine Seite breit.

### Passmatrix
- Erklärungstext oben.
- Gesamtmatrix als Heatmap (Farbskala weiß → `BDD7EE` → `2E75B6`, Nullen
  leer, Diagonale grau), sortiert nach Stammreihen, Kästen um die Blöcke
  einer Stammreihe; rechts gespielt, Fehlpässe, Pass-%; unten „erhalten“.
- Je eine Matrix für Reihe 1, 2, 3 auf dem Feld mit allen Spielern, die in
  dieser Reihe gespielt haben (inkl. Aushelfer), plus Torwart. Jeder Pass
  zählt zu der Reihe, die zum Zeitpunkt des Passes auf dem Feld stand
  (genauer als die Vorlage, die die Reihe beim Kettenbeginn nimmt);
  Fehlpässe nur in dieser Reihe. Darunter: alle Pässe dieser Reihe und
  davon mit Spielern außerhalb der Reihe (Wechsel während Ballbesitz).
  Zusätzlich eine Matrix „Überzahl“, falls dort Pässe erfasst sind.
- Top-15-Passduos (A → B, B → A, gesamt, Stammreihen), Werte als
  Verweise auf die Gesamtmatrix; Sortierung nach Anzahl, bei Gleichstand
  wie die Vorlage.

### Kennzahlen-Blätter (Experiment: Corsi, Ballbesitz, Umschalten, Spielstand, Duos, Über-Unterzahl)
- Gleiche Tabellen (Spalten, Zeilen, Werte) wie unter „Erweiterte
  Kennzahlen“ in der Auswertung; die Bildschirmtabellen und der Export
  nutzen dieselben Funktionen.
- Je Blatt untereinander die Blöcke Gesamt, 1., 2., 3. Drittel (plus
  Verlängerung); nicht erfasste Drittel mit „nicht erfasst“.
- Corsi: je Situation (5 gegen 5 bzw. gleiche Stärke, Überzahl,
  Unterzahl, Spezial) Reihen mit Team-Summe und Feldspieler. Ballbesitz:
  Reihen mit Team, Spieler (Feldspieler, Torhüter). Umschalten: Reihen mit
  Team, Spieler. Spielstand: Team und Reihen nach Spielstand. Duos: alle
  Paare (ohne Filter). Über-Unterzahl: Überzahl und Unterzahl.
- Werte als Zahlen mit Format: Zeiten `[m]:ss`, Anteile `0%`, CF % rel.
  `+0" PP"`, /10-Werte `0.0`; fehlende Werte „–“.
- Rechts neben jeder Quote eine Spalte „n“ (Stichprobe); Quote und n grau
  und kursiv bei n < 5.

### Ereignisse
- Alle Rohdaten (Zeit, Sekunden, Drittel, Typ, Details, Reihe auf dem
  Feld), Kopfzeile fett/grau und fixiert, Autofilter, lesbare
  Spaltenbreiten.
- (Experiment, `gegnerschussErgebnis`) Spalten G „Ergebnis β“ und
  H „Blocker β“; Experiment-Hinweis dann in J1.

### Ergebnis von Gegnerschüssen im Excel (Experiment)
- Statistik-Blätter, Kennzahlen-Block: Blocks (mit „Alle“), Blocks /10′,
  Gegner aufs Tor (Feld), Fenwick gegen (Feld), Paraden (TW),
  Fangquote (TW, grau bei n < 5).
- Reihenvergleich, fortlaufend nach den übrigen Kennzahl-Spalten (bei
  Heidelberg-Mannheim Y–AM): Gegner aufs Tor, geblockt, vorbei, Ergebnis
  unbekannt (mit Summe), je /10′, Anteil aufs Tor und Blockquote (je mit n),
  FF, FA, FF % (mit n).

### Prüfung (20.10.2026)
- Mit dem Spiel Heidelberg-Mannheim (nachgebaut aus dem Blatt
  „Ereignisse“ des bisherigen Exports) stimmen alle Werte mit der
  Referenzdatei überein (Abweichungen nur durch Rundung der alten
  Rohdaten: Zeiten ≤ 0,4 s, /10′-Werte ≤ 0,006); Layout (Farben, Rahmen,
  Schrift, Formate, Fixierung, Farbskalen) identisch. Alle Formeln ohne
  Fehler nachgerechnet (HyperFormula).
- Im Experimentmodus weichen Eiszeit (Tor-Pausen abgezogen) und /10′
  (geschätzte Spielzeit statt Eiszeit) bewusst von der Referenzdatei ab;
  im Standardmodus nicht (siehe „Standard- und Experimentmodus“, Prüfung
  27.10.2026).

- JSON-Export aller Rohdaten (für späteren Import ins Analytics Center),
  unverändert: Kennzahlen werden immer neu berechnet.

## Kennzahlen – Definitionen
(Experiment) Alle Kennzahlen dieses Abschnitts sind im Experimentmodus,
außer in Abschnitt 11 „Gegentor zählt als Gegnerschuss“, die Bully-Warnung
und der Check „Tor ohne Bully“ (Standard); die automatische Tor-Pause aus
Abschnitt 11 ist Experiment (`autoTorPause`).

Alle Kennzahlen werden ausschließlich aus bereits erfassten Daten
berechnet (Wechsel, Besetzungen, Unterbrechungen, Tore, Strafen, Bullys,
Gegnerschüsse, Ballbesitzketten mit Zeit jedes Kontakts sowie
tStart/tEnde), rückwirkend auch für ältere Spiele. Keine neuen Tasten oder
Eingaben. Die Zeitfenster stehen als Konstanten `KZ` oben im Code
(nicht im Setup änderbar). Unter „Festlegung“ stehen die bei der Umsetzung
geklärten Punkte.

### 0. Grundlagen (für alle Kennzahlen)
**Auf dem Feld:** Wer zu einem Zeitpunkt t auf dem Feld steht, ergibt sich
aus Wechseln, Einzelwechseln und der Besetzung des Drittels
(`lineupAt(t)`). Ein Ereignis zählt für alle Spieler, die zum Zeitpunkt
des Ereignisses auf dem Feld stehen (eigener Schuss: Zeit des Schusses =
tEnde der Kette; Gegnerschuss/Gegentor: Ereigniszeit). Torhüter werden in
Spielertabellen getrennt ausgewiesen.

**Geschätzte Spielzeit (Netto):** Bei Ball im Aus läuft die Uhr nicht. Die
Videozeit (auch nach Abzug der O-Unterbrechungen) ist deshalb länger als
die Spielzeit.
- Setup: „Drittellänge (min)“ und „Verlängerung (min)“.
- Pro Drittel: Faktor = Drittellänge ÷ Netto-Videozeit des Drittels
  (Drittelbeginn bis Drittelende minus O-Unterbrechungen).
- Geschätzte Spielzeit eines Spielers/einer Reihe = Summe (Netto-Videozeit
  je Drittel × Faktor des Drittels).
- In der Auswertung beide Spalten: „Eiszeit (Video)“ und „Spielzeit
  (geschätzt)“. Alle Werte „pro 10 min“ beziehen sich auf die geschätzte
  Spielzeit.
- Hinweistext: „Spielzeit geschätzt: Stillstand bei Ball im Aus wird
  gleichmäßig über das Drittel verteilt angenommen.“
- Faktor je Drittel in der Auswertung anzeigen (typisch 0,8–0,9).
- Festlegung: Standard Drittellänge 20 min (GF und KF), Verlängerung 10 min.
  Leeres Feld → Faktor 1 mit Hinweis. Die Tor-Pause (Tor/Gegentor bis zum
  nächsten Bully, höchstens 3 min) zählt wie eine O-Unterbrechung auch bei
  der Netto-Videozeit für den Faktor; Überschneidungen mit O werden nicht
  doppelt abgezogen.

**Ballbesitzphase:** Zwei Ketten werden zu einer Phase zusammengefasst, wenn
die erste mit Fehlpass oder Ballverlust endet und die nächste eigene Kette
höchstens 2 s nach deren Ende beginnt, ohne dass dazwischen ein
Gegnerschuss, Gegentor, eine Unterbrechung oder ein Drittelende liegt.
Ketten bleiben in den Rohdaten unverändert; Phasen sind nur eine
Auswertungssicht. Beide Zählungen (Ketten und Phasen) ausweisen.
- Festlegung: Phasendauer = Beginn der ersten bis Ende der letzten Kette
  (inkl. Lücken ≤ 2 s) minus O-Unterbrechungen. Phasenwerte zählen für die
  Reihe bzw. die Spieler auf dem Feld bei Phasenbeginn.

**Spielstand:** Aus Tor/Gegentor für jeden Zeitpunkt den Spielstand
berechnen (Führung / Gleichstand / Rückstand aus Ulmer Sicht).
- Festlegung: Ein Tor zählt noch zum Spielstand davor.

### 1. Corsi (Schussversuche für/gegen auf dem Feld)
Pro Spieler (Feldspieler) und pro Reihe:
- CF = eigene Schüsse (alle Ergebnisse: Tor, gehalten, geblockt, vorbei),
  während er auf dem Feld steht.
- CA = Gegnerschüsse + Gegentore (ein Gegentor nicht doppelt zählen, wenn
  ±3 s ein Gegnerschuss erfasst ist), während er auf dem Feld steht.
- CF % = CF ÷ (CF + CA).
- CF/10 und CA/10 pro 10 min geschätzter Spielzeit.
- CF % relativ = CF % mit ihm auf dem Feld − CF % ohne ihn (gleiche Drittel).
- Zusätzlich für eigene Schüsse: Schüsse aufs Tor % = (Tor + gehalten) ÷
  alle eigenen Schüsse, geblockt %.

Getrennt nach Situation: 5 gegen 5 / Überzahl / Unterzahl (aus der Reihe
auf dem Feld). Hauptwert in der Spielertabelle: 5 gegen 5.
- Festlegung: Eigene Schüsse = Ketten aus Durchgang 2 mit Ende Schuss plus
  Ulmer Tore aus Durchgang 1 ohne Torschuss in Durchgang 2 (±3 s). Reihe
  „Spezial“ ist eine eigene Situation (nicht 5 gegen 5); im KF heißt
  5 gegen 5 „gleiche Stärke“. CF % relativ: nur Drittel, in denen er
  gespielt hat, gleiche Situation, in Prozentpunkten. Torhüter ohne Corsi.
- Festlegung: Gegnerschüsse sind alle Versuche (auch geblockt/vorbei).
  Ohne den Schalter `gegnerschussErgebnis` sind sie ohne Ergebnis erfasst;
  dann für den Gegner kein Fenwick und kein „aufs Tor %“, Hinweis
  „Ergebnis der Gegnerschüsse nicht erfasst“. Mit Schalter: siehe
  Abschnitt 13. Die Corsi-Spalten heißen „eigene aufs Tor %“ und
  „eigene geblockt %“.

### 2. Ballbesitzzeit
- Dauer jeder Kette = tEnde − tStart minus Überschneidung mit
  O-Unterbrechungen.
- Pro Reihe und pro Spieler (auf dem Feld): Ballbesitzzeit gesamt, Anteil an
  der geschätzten Spielzeit (Ballbesitzzeit mit dem gleichen Drittel-Faktor
  umrechnen), Ø und Median der Phasendauer, längste Phase, Phasen ≥ 20 s.
- Bei Ketten über einen Reihenwechsel: die Zeit anteilig der Reihe
  zuordnen, die zum jeweiligen Kontaktzeitpunkt auf dem Feld stand.
- Prüfliste: Ketten mit mehr als 8 s zwischen zwei Kontakten (klickbar,
  springt ins Video), weil dort eine Unterbrechung in der Kette stecken kann.
- Festlegung (27.10.2026): Rechnung bleibt (Faktor auf beiden Seiten, also
  Ballbesitz ÷ Netto-Videozeit auf dem Feld). Spaltenüberschrift „Anteil an
  Netto-Videozeit“ statt „Anteil an Spielzeit“, Hinweis: „Stillstand bei
  Ball im Aus nicht herausgerechnet; exakt erst mit Spieluhr.“ Gilt für alle
  Ballbesitzanteile (Spieler-, Reihen-, Spielstand-, Über-/Unterzahl-
  Tabellen) auf dem Bildschirm und im Excel.

### 3. Umschalten
- Konter gegen uns: Gegnerschuss oder Gegentor innerhalb von 10 s nach dem
  Ende einer eigenen Kette mit Fehlpass oder Ballverlust (ohne eigene Kette
  dazwischen). Pro Reihe und pro Spieler, der den Ball verloren hat
  (Anzahl, und davon Gegentore). Gegentore zusätzlich mit Fenster 20 s.
- Eigene Konter: Kette mit Start „Ballgewinn“ oder „loser Ball“, die
  innerhalb von 10 s nach tStart mit einem Schuss endet. Pro Reihe und pro
  Ballgewinner/Schütze.
- Gegenpressing-Erfolg: Anteil der eigenen Fehlpässe/Ballverluste, nach
  denen innerhalb von 5 s wieder eine eigene Kette beginnt (ohne
  Gegnerschuss dazwischen). Pro Reihe.
- Festlegung: Konter zählen je Ballverlust (höchstens einmal); die
  20-s-Gegentore stehen in einer eigenen Spalte. „Ballgewinn“ als Start nur
  bei ausdrücklich gedrückter Taste A, „loser Ball“ immer. Alle Fenster
  netto (ohne O-Unterbrechungen). Gegenpressing: außer Gegnerschuss
  unterbrechen auch Gegentor, Unterbrechung und Drittelende (wie bei
  Phasen).
- Festlegung (27.10.2026): Beim Gegenpressing zählt als Rückeroberung nur
  eine neue Kette mit Start „Ballgewinn“ (Taste A) oder „loser Ball“, kein
  Freischlag, kein Torwart, kein unbekannter Start. Eigene Konter mit Start
  „Torwart“ (auch vorausgewählt nach Gegnerschuss) werden zusätzlich
  gezählt, aber in eigenen Spalten: „Starts Torwart“, „Konter nach
  Torwart-Start“ (Reihe) und „Konter nach Torwart-Start als Schütze“
  (Spieler); die bisherigen Spalten enthalten sie nicht.

### 4. Schichtverlauf (Ermüdung)
Pro Reihe: CF/10 und CA/10 getrennt nach Schichtphase 0–45 s / 45–90 s /
über 90 s (Netto-Sekunden seit Schichtbeginn, O-Unterbrechungen
abgezogen). Dazu die Spielzeit je Phase (Zellen mit < 2 min Spielzeit
grau/kursiv). Als kleines gruppiertes Balkendiagramm in der Auswertung.

### 5. Bully-Wert
Pro Bullyspieler und pro Reihe:
- gewonnen → eigener Schuss innerhalb 10 s
- verloren → Gegnerschuss (oder Gegentor) innerhalb 10 s
- dazu Bullys nach Anlass: Drittelbeginn / nach Gegentor / nach eigenem
  Tor / sonstige (Anlass = letztes Drittelbeginn/Tor/Gegentor seit dem
  vorherigen Bully).

### 6. Spielstand
CF %, Ballbesitzanteil und Gegnerschüsse/10 getrennt nach Führung /
Gleichstand / Rückstand (Team gesamt und pro Reihe), jeweils mit
Spielzeit in diesem Zustand.

### 7. Duos (mit/ohne)
Für jedes Spielerpaar mit mindestens 3 min gemeinsamer geschätzter
Spielzeit: gemeinsame Spielzeit, CF % zusammen, CF % Spieler A ohne B,
CF % Spieler B ohne A, Pässe zwischen beiden. Tabelle sortiert nach
gemeinsamer Spielzeit, filterbar nach Spieler.
- Festlegung: nur Feldspieler, Zeiten und CF % bei 5 gegen 5.

### 8. Über- und Unterzahl
Aus Strafen und den Reihen ÜZ/UZ:
- Anzahl Überzahlen / Unterzahlen, Spielzeit, Schüsse für/gegen, Tore
  für/gegen.
- Überzahlquote = ÜZ-Tore ÷ Überzahlen, Unterzahl überstanden % =
  1 − UZ-Gegentore ÷ Unterzahlen.
- Pässe pro Ballbesitz und Ballbesitzanteil in Überzahl.
- Festlegung: 2 min, 2+2 min, 5 min und Matchstrafe zählen je als eine
  Über- bzw. Unterzahl, 10 min nicht. Spielzeit = solange die ÜZ- bzw.
  UZ-Reihe auf dem Feld steht; endet eine Über- oder Unterzahl vorzeitig
  durch ein Tor, zählt nur die tatsächlich gespielte Zeit.

### 9. Rollen im Passnetzwerk
Pro Spieler: Pässe gespielt, Pässe erhalten, Anteil an allen Pässen des
Teams (gespielt + erhalten), Anzahl verschiedener Passpartner, häufigster
Passpartner. Im Passnetzwerk optional die Knotengröße nach „beteiligt
an %“.

### 10. Szenen per Klick abspielen (Clip-Playlist)
- Jede Zahl in der Auswertung ist anklickbar (Spielertabelle,
  Reihenvergleich, Corsi, Konter, Bully-Wert, Passmatrix-Zelle usw.). Ein
  Klick öffnet eine Playlist der zugrunde liegenden Ereignisse und spielt
  sie im Videofenster nacheinander ab.
- Vorlauf/Nachlauf je Ereignistyp (Konstante `CLIP_VN`): Tor / Gegentor
  7 s / 3 s · Schuss / Gegnerschuss 5 s / 2 s · Ballverlust / Fehlpass
  4 s / 4 s · Bully 2 s / 6 s · Ballbesitzkette tStart − 2 s bis tEnde + 2 s ·
  sonst 4 s / 3 s. Überlappende Clips werden zusammengefasst.
- Steuerung: nächste/vorherige Szene (N/V), Pause, Liste der Szenen mit
  Zeit und Beschreibung, aktuelle Szene hervorgehoben.
- Export der Playlist als CSV-Schnittliste (Start, Ende, Beschreibung) und
  als `.bat` für ffmpeg (schneidet und fügt zu einer Datei zusammen; ffmpeg
  muss lokal installiert sein, Kurzanleitung im Tool).
- Schnellauswahl „Alle Tore“, „Alle Gegentore“, „Alle Schüsse von …“.
- Festlegung: Anzahlen → die gezählten Ereignisse; Zeiten → die Schichten
  bzw. Ketten; Quoten → Zähler und Nenner; Passmatrix-Zelle → die Ketten
  mit diesem Pass. Solange die Playlist offen ist, steuern N/V/Esc die
  Playlist, die Videotasten funktionieren, alle anderen Erfassungstasten
  sind gesperrt. Die `.bat` erwartet die Videodatei im selben Ordner und
  kodiert neu (libx264, CRF 20), damit die Schnitte bildgenau sind.

### 11. Kleinere Punkte
- Ein Gegentor zählt automatisch auch als Gegnerschuss der Reihe auf dem
  Feld (nicht doppelt, wenn ±3 s ein Gegnerschuss erfasst ist).
- Zeit zwischen Tor/Gegentor und dem nächsten Bully zählt nicht als
  Eiszeit (wie eine O-Unterbrechung); bereits mit O markierte Tor-Pausen
  werden nicht doppelt abgezogen. Plausibilitätscheck, wenn nach einem Tor
  3 min lang kein Bully erfasst ist.
- Bully-Warnung „kein Bullyspieler“.

### 12. Stichprobe n
Neben jeder Quote und jedem Prozentwert steht die Anzahl n, auf der er
beruht; Werte mit n < 5 (Konstante `KZ.MIN_N`) grau/kursiv, auf dem
Bildschirm und im Excel. n je Quote:
- CF %, CF % relativ, CF % in Duos/Spielstand: CF + CA (mit ihm bzw. in der
  Situation)
- eigene aufs Tor %, eigene geblockt %: eigene Schüsse (CF)
- Bully %: Bullys; Pass %: Pässe + Fehlpässe; Gegenpressing: Fehlpässe +
  Ballverluste; Bully-Wert-Quoten: gewonnene bzw. verlorene Bullys
- Ballbesitzanteil: Ballbesitzketten mit Kontakt der Reihe/des Spielers
  bzw. im Spielstand (Team: alle Ketten); Über-/Unterzahl: Ketten mit
  Beginn in ÜZ/UZ
- Überzahlquote / Unterzahl überstanden: Anzahl Strafen
- beteiligt an % (Passnetzwerk): Pässe des Teams
- Anteil aufs Tor, Blockquote (Gegner): Gegnerschüsse mit bekanntem
  Ergebnis; FF %: FF + FA; Fangquote: Paraden + Gegentore

### 13. Ergebnis von Gegnerschüssen (Experiment, `gegnerschussErgebnis`)
- Ergebnis je gegnerischem Schussversuch: Gegentor (auch Gegnerschuss mit
  Gegentor ±3 s) = Tor; sonst gehalten / geblockt / vorbei wie erfasst;
  fehlt es: unbekannt. Gehalten = aufs Tor, Torwart hält (auch
  abgefälscht, wenn er trotzdem aufs Tor kommt); geblockt = von einem
  Ulmer Feldspieler aufgehalten, bevor er das Tor erreicht; vorbei inkl.
  Pfosten/Latte.
- Zuordnung zur Reihe wie bei „Gegnerschüsse“ im Reihenvergleich (Reihe auf
  dem Feld beim Gegnerschuss; einzelnes Gegentor: Reihe zur Torzeit). Damit
  gilt je Reihe: Gegnerschüsse = aufs Tor + geblockt + vorbei + unbekannt;
  ein Gegentor zählt genau einmal (X → 1 erzeugt kein zweites Ereignis).
- Aufs Tor = gehalten + Gegentore. Anteil aufs Tor = aufs Tor ÷ Schüsse mit
  bekanntem Ergebnis; Blockquote = geblockt ÷ Schüsse mit bekanntem
  Ergebnis (Gegentore zählen als bekannt). /10 je 10 min Spielzeit.
- Fenwick: FA = Gegnerschüsse ohne geblockte (inkl. Gegentore und
  unbekannte), FF = eigene Schüsse ohne geblockte, FF % = FF ÷ (FF + FA).
  Reihen: alle Situationen (wie CF % im Reihenvergleich).
- Spieler: Blocks = geblockte Gegnerschüsse mit ihm als Blocker (alle
  Situationen), Blocks /10 je 10 min Spielzeit; On-Field (5 gegen 5, wie
  CF/CA): Gegner aufs Tor und FA, solange er auf dem Feld steht.
- Torhüter (wer laut Wechseln im Tor steht): Paraden = gehalten,
  Gegentore, Fangquote = Paraden ÷ (Paraden + Gegentore).
- Datenmodell: gegnerschuss `{ reiheAufFeld, ergebnis?: 'gehalten' |
  'geblockt' | 'vorbei', blocker?: spielerId }`; fehlendes Ergebnis =
  unbekannt. Alte Sicherungen bleiben gültig, das JSON hängt nicht vom
  Modus ab.

### Prüfwerte Heidelberg-Mannheim (27.10.2026)
- Faktoren 0,840 / 0,842 / 0,872 (erwartet ca. 0,84 / 0,84 / 0,87).
- Schüsse/Gegnerschüsse R1 12/16, R2 17/5, R3 8/22, CF % 43/77/27 %
  (wie erwartet).
- Ballbesitz nach Kettenbeginn 8,33 / 7,47 / 6,62 min (erwartet 8,3 / 7,5 /
  6,6); nach Kontakt (gewählte Definition) 8,23 / 7,33 / 6,95 min.
  Längste Kette 43,1 s (R1). Rückeroberungen ≤ 2 s: 11.
- Gegentore ≤ 20 s nach Fehlpass/Ballverlust: 3 von 5 (erwartet 3).
- Gegenpressing nach neuer Startregel 22 von 87 (vorher 23; R2 7 statt 8
  von 25, die eine Rückeroberung war ein Freischlag). Starts Torwart 9,
  davon 0 Konter.

### Prüfung Ergebnis von Gegnerschüssen (02.11.2026)
- Standard: X ohne Auswahl (danach wirkt 2 als Reihenwechsel); Excel gegen
  die Referenz unverändert (ganze Zahlen identisch, Zeiten ≤ 0,29 s),
  706 Formeln ohne Fehler; Ereignis-Blatt mit 6 Spalten.
- Experiment: Prüfwerte vom 27.10. unverändert; 46 Gegnerschüsse, davon 5
  mit Gegentor (= Tor), 41 offen zum Nachtragen.
- Testkopie mit Ergebnissen und Blockern: je Reihe und Drittel
  Gegnerschüsse = aufs Tor + geblockt + vorbei + unbekannt, FA =
  Gegnerschüsse − geblockt, Summe Blocks = geblockte mit Blocker,
  Gegentore = „Tor“ (keine Doppelzählung). Tastenfolgen bei laufendem
  Video (X → 2, X → 3 → 1, X → 1, X → Enter, X → 6 → B), Rückgängig
  Schritt für Schritt, Bearbeiten (inkl. Tor → Gegentor), Nachtragen mit K
  (überspringt Schüsse mit Gegentor) geprüft. Moduswechsel ändert das
  JSON nicht. Excel 810 Formeln ohne Fehler.

## Bekannte Grenzen
- Gleichzeitige Strafen (4 gegen 4) und angezeigte Strafen werden nicht
  gesondert erfasst; maßgeblich ist die gesetzte Reihe (ÜZ/UZ/1–3).
- Spielzeit ist geschätzt (Faktor je Drittel = Drittellänge ÷
  Netto-Videozeit); Stillstand bei Ball im Aus wird gleichmäßig verteilt
  angenommen. Exakt erst mit Spieluhr.
- Ergebnis der Gegnerschüsse: im Standardmodus nicht erfasst (kein Fenwick
  und kein „aufs Tor %“ für den Gegner). Im Experimentmodus erfassbar
  (inkl. Nachtragen); Quoten zählen nur Schüsse mit bekanntem Ergebnis, FA
  enthält auch unbekannte.

## Datenmodell (Vorschlag)
- game: { id, datum, gegner, modus, video: { name, size, duration },
  kader[], reihen[], bericht? }
  - bericht (Saisonmanager): { spielId, heimTeam, gastTeam, unser, datum,
    ergebnis, tore: [{ periode, zeit, team, schuetze, vorlage, stand }],
    strafen: [{ periode, zeit, team, nummer, art, grund }], timeouts[] }
- events[]: jedes Ereignis mit { id, typ, time, ...typabhängige Felder }
  - wechsel: { reihe, spieler[] } bzw. einzelwechsel: { raus, rein }
  - gegnerschuss: { reiheAufFeld, ergebnis?: gehalten | geblockt | vorbei,
    blocker?: spielerId } (Ergebnis/Blocker optional, Experiment)
  - tor / gegentor: { aufDemFeld[], schuetze?, vorlage? } (Schütze/Vorlage
    bei Ulmer Toren, vorgeschlagen aus dem Spielbericht)
  - bully: { spieler, gewonnen }
  - strafe: { team, spieler, dauer }
  - drittel: { nummer, beginn|ende }
  - unterbrechung: { art: beginn|ende, grund: timeoutVfb | timeoutGegner |
    verletzung | sonstiges }
  - aufstellung: { drittel, reihen: { <Reihe>: { slots: [{ pos, id }],
    bully, ohneTorwart } } } – Besetzung ab Drittelbeginn
  - ballbesitz: { start (ballgewinn | bully | freischlag | torwart |
    loserBall | unbekannt), startVorauswahl, kette: [{ spieler, time }],
    ende, schussErgebnis?, tStart, tEnde }
- Auf-dem-Feld-Stand zu Zeit t wird immer aus allen Wechselereignissen
  ≤ t berechnet (nicht als Zustand gespeichert), damit Zurückspulen und
  nachträgliche Korrekturen konsistent bleiben.
- Gültige Reihenbesetzung zu Zeit t = letztes `aufstellung`-Ereignis ≤ t,
  sonst die Reihen aus dem Setup.

## Später, nicht in Version 1
- Zonen per Klick auf eine Spielfeldskizze
- Vergleich mehrerer Spiele / Saisonwerte
- Tastenbelegung im Tool anpassbar
