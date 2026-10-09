# Prüfung der gemeinsamen Anwendung

Stand: 8.10.2026, Intel-Mac, Node.js 22.14.0, Chrome im Hintergrund. Die Tests haben keine Systemeinstellungen oder Bluetooth-Geräte verändert.

- Gemeinsamer Produktionsbuild erfolgreich, inklusive TypeScript-Prüfung und Cache-Version aus den tatsächlich gebauten Dateien.
- 15 Wetter-Rechentests erfolgreich.
- 11 vorhandene Wetter-Browserszenarien nach Anpassung an die entfernte Info-Oberfläche erfolgreich.
- Integrationstest: 480 × 320, 480 × 480, 800 × 480 und 1280 × 720; Wetteransichten, Fokus, Radio-Unterseiten, Start-Navigation, dunkle Standby-Uhr, native Retina-/RGB565-Vorschau. Reguläre Ansichten bei 480 × 320, 800 × 480 und 1280 × 720 ohne Inhalts-Scrollen. Das bisherige 480 × 480-Verhalten erlaubt in umfangreichen Wetterseiten weiterhin vertikales Scrollen.
- Eigene Radio-URL wird gespeichert; ein nicht erreichbarer Stream liefert einen sichtbaren Fehler. Zwei zusätzliche Tests prüfen Dateipfadbegrenzung und URL-Validierung.
- Echter radioeins-Stream: Chrome dekodiert Audio, Wiedergabezeit steigt; Radio läuft während Wetter und Standby weiter und stoppt auf Anforderung. Der Testbrowser war stummgeschaltet. Physischer Ton über die vorgesehenen Bluetooth-Boxen ist damit nicht geprüft.
- Offline-Produktion: Oberfläche nach Erstbesuch neu geladen, Demo-Wetter und historische Klimakarte verfügbar, Radio meldet fehlende Verbindung, Standby bleibt bedienbar.
- Radar mit kontrollierten Antwortzeiten: 23 eindeutige kommende Bilder automatisch vorgeladen, höchstens zwei Abrufe parallel, keine zusätzlichen Bildabrufe während der anschließenden Wiedergabe. Der Test belegt das Ladeverhalten; reale DWD-Antwortzeiten bleiben vom Server und Netzwerk abhängig.

Screenshots stehen in `startmenue/test-results/` und `wetterwarte/test-results/`. Beide Verzeichnisse sind von Git ausgeschlossen.

Zusätzliche Tests bei laufender Entwicklungsumgebung:

```sh
cd ~/Projects/raspberry_pi/startmenue
node tests/radar-buffer.mjs
node tests/radio-live.mjs          # kurzer echter Streamabruf, Browser stumm
node tests/production-offline.mjs # vorher in der Sammlung npm run build
```

Der Offline-Test verwendet vorübergehend Port 4173. Kein Raspberry Pi oder reales Display wurde getestet; insbesondere Hardwarebeschleunigung, Audio-Wiederverbindung, Touchdruck und Hintergrundbeleuchtung bleiben geräteabhängig. Die später ergänzten Kartenebenen werden im Nachtrag unten beschrieben.

## Reparaturrunden vom 9.10.2026

Sieben aufeinander aufbauende Prüf- und Korrekturrunden für die gemeldeten Radar- und Bedienfehler:

1. Klickpositionen der Rasteransicht mit den tatsächlichen DOM-Flächen verglichen. Der gemeldete Versatz war in einer frisch geladenen Ansicht nicht durchgehend reproduzierbar; gemessene Zentren lagen in Chrome und WebKit weniger als einen Pixel auseinander.
2. Echtes DWD-RV-PNG und offizielles WMS-GetStyles-SLD untersucht: Die magentafarbene `ras:DataContour`-Linie gehört nicht zur Regenpalette. Ihre Mischpixel werden als nicht auswertbar maskiert. 14.559 Regenpixel und 124.171 Maskenpixel des gespeicherten Prüfbildes werden in beiden Engines identisch verarbeitet; unbekannte Farben werden weiterhin abgewiesen.
3. Veraltete Rasterbilder bei Änderungen und fehlgeschlagenem Neuaufbau sofort entfernt. Ein gezielt ausgelöster Bilddekodierungsfehler prüft diesen Rückfall auf das aktuelle HTML. Eigene Ortsauswahl ersetzt das native Auswahlfeld. Tests klicken auf die im Raster ermittelten Positionen von Ort, Uhr, Start, Abspielen und Weiter.
4. Abgelaufene Prognosen ohne irreführende Jetzt-Achse; fehlendes Bild ohne überlagerte Legende und wiederholte Fehlermeldung. Ungültiger Bildcache wird erst nach erfolgreicher Bildprüfung ersetzt. Fehleransichten in allen vier Größen geprüft.
5. Größere Pfeilflächen (mindestens 44 Pixel hoch); Radar-Ladestatus und Radio-Fußzeile im 480×320-Format eingepasst. Integration in Chrome und WebKit einschließlich Wetter-Unterseiten, Start, Radio-Unterseiten, Standby und Retina-Vorschau.
6. WebKit-Bildcache auf ArrayBuffer umgestellt; vorhandene Blob-Caches bleiben lesbar. Echter radioeins-Stream dekodiert in WebKit und läuft bei Wetter/Standby weiter. Aktueller DWD-Lauf: 23 Bilder automatisch vorbereitet, Wiedergabe ohne weitere Bildabrufe und erneutes Anzeigen nach Neuladen ohne DWD-Verbindung.
7. Abschließender TypeScript-/Produktionsbuild, 16 Rechentests, 11 Wetter-Browserszenarien, Server-/Integrationstests und zusätzliche Radar-/Klick-Regressionssuite. Offline-Produktion in Chrome und WebKit ebenfalls erfolgreich: Neuladen, Demo-Wetter, Klimakarte, Radiofehler und Standby. Die zusätzliche Regressionstestsuite besteht vollständig.

Beim WebKit-Offline-Test verursacht der Protokollschalter „offline“ vor einer Navigation einen internen Browserfehler. Deshalb wird zum Neuladen zuerst der lokale Testserver gestoppt (echter Service-Worker-Rückfall); anschließend wird für den Radiotest zusätzlich der Offline-Schalter gesetzt. Der Entwicklungsserver auf Port 5173 bleibt dabei an.

WebKit ist die Safari-Engine im Playwright-Test, nicht das persönliche Safari-Profil des Benutzers. Die Rasterfehler-Reproduktion erfolgt durch kontrollierte Fehlerzufuhr. Lautsprecherton und Bluetooth-Kopplung bleiben Hardwareprüfungen; Audiotests sind stumm.

```sh
# Einmalig für die zusätzlichen Safari-Engine-Prüfungen:
cd ~/Projects/raspberry_pi/wetterwarte
npx playwright install webkit
cd ..
npm run test:regression  # lokale Entwicklungsvorschau muss laufen
npm run test:safari
cd startmenue
node tests/radio-live.mjs webkit
node tests/radar-live.mjs webkit
node tests/production-offline.mjs webkit # nach npm run build in der Sammlung
```

Das Radar-Prüfbild und seine offizielle Stilbeschreibung liegen unter `startmenue/tests/fixtures/`. Quelle: DWD, `maps.dwd.de`, WMS-Layer `dwd:Niederschlagsradar`, Bezugslauf 8.10.2026, 20:25 UTC; Stilabruf am 8.10.2026. Referenzdateien sind keine aktuellen Wetterdaten der Anwendung.

## Zweiter Durchgang am 9.10.2026: Safari-Zoom, Radio und weitere Karten

Sieben Prüfbereiche mit Korrekturen und anschließender Wiederholungsprüfung:

1. Randklicks statt nur Buttonzentren ergänzt; Lautstärke-Slider entfernt, Sendernavigation auf beschriftete 44-Pixel-Schaltflächen umgestellt.
2. Im installierten Safari 26.6.2 bei gespeichertem Seitenzoom 85 % unterschiedliche Koordinaten von Rasterbild und eingebettetem Dokument nachgewiesen. Die Eingabefläche misst beide Bereiche und rechnet Berührungspunkte um. Ein vollständiger Basisdurchlauf in Safari bestand bei 15 %, 50 % und 85 % der Buttonbreite: Stumm, Abspielen/Stoppen, Senderseiten, Formular öffnen/schließen, Mond, Uhr ohne Startfunktion und Start. SafariDriver benötigt zusätzlich eine eigene Kalibrierung seiner Testkoordinaten; diese steht ausschließlich im Testskript.
3. COSMO, Beats Radio, pure fm Berlin und radioeins als erste vier Presets. DWD-Temperatur und DWD-Windwahrscheinlichkeit als echte WMS-Flächen; CAMS-Luftqualität als getrennte Ortswerte. Echte PNG-Größe, Stunden-/Sechsstundenschritte und explizites Windprodukt geprüft. Cache-Wiederherstellung bei blockierter DWD-Verbindung erfolgreich.
4. Layout- und Navigationstests in Chrome und WebKit über alle vier Auflösungen; alle Kartenebenen auswählbar. Der neue 44-Pixel-Ebenenwähler teilt sich die verfügbare Radarfläche mit der Karte; deren Mindesthöhe im 800-Pixel-Test liegt deshalb nun bei 240 Pixeln. Geometrische Inhaltsgrenzen werden zusätzlich geprüft.
5. Erweiterte sichtbare Klicktests bei 8 %, 50 % und 92 % der Buttonbreite in Chrome und WebKit: Stumm/an, Senderwahl, Weiter/Zurück sowie Eingabe und Speicherung eines eigenen Senders über echte Tastatureingabe. Jeweils physische Vorschau und Pixelraster. Diese vier Testkombinationen bestehen. Die erweiterte Prüfung im installierten Safari wurde wegen des störenden Eingriffsdialogs auf Benutzerwunsch beendet; sie wird nicht als vollständig bestanden gewertet. Safari und WebKit sind ausdrücklich getrennte Prüfumgebungen. Keine Standardsuite startet sichtbare Safari-Automation.
6. Alle vier gewünschten echten Streams dekodieren in WebKit, ihre Wiedergabezeit steigt. radioeins läuft beim Wechsel zu Wetter und Standby weiter. Audioausgabe im Test stumm; Bluetooth-Boxen nicht physisch geprüft. Radar: 23 kommende Bilder, höchstens zwei parallele Abrufe, keine doppelten Bilder oder Netzwerkabrufe während Wiedergabe. Zusätzlich bleibt die gemessene Kartenhöhe während des verzögerten Vorladens konstant.
7. Produktionsbuild einschließlich TypeScript, 16 Rechentests, elf Wetter-Browserszenarien und die erweiterte Radar-/Klick-Regressionssuite bestanden. Die gemeinsame Oberfläche wurde in Chrome und WebKit geprüft. Offline-Neuladen des fertigen Builds, Demo-Wetter/Klimakarte, Radiofehlermeldung und Standby bestanden abschließend in beiden Engines.

Die neue Eingabeschicht (`startmenue/src/input-surface.js`) ist im Produktionscache enthalten. Eingaben in der Rastervorschau verwenden direkt bedienbare Felder über dem Raster. Während reiner Datenaktualisierungen bleibt das bisherige Raster sichtbar; bei Navigation oder fehlgeschlagener Rasterisierung wird es verworfen.

Zusatzprüfungen, laufende Entwicklungsumgebung vorausgesetzt:

```sh
cd ~/Projects/raspberry_pi/startmenue
node tests/fields-live.mjs          # reale DWD-/CAMS-Abrufe, externer Ausfall + Cache
node tests/radio-live.mjs webkit   # vier echte Streams, stumm
```

## 9.10.2026 · Menüzeile, Lesbarkeit, Kartenwiedergabe und Radio

Änderungen mit iterativen Prüfungen und Korrekturen:

- Eine Navigationsebene mit Auswahlmenüs; bei 1280 Pixeln zusätzlich direkt erreichbare Hauptansichten in derselben Zeile. Kartenlegenden als separate Spalte. Fokus-Zustand, Stunden und Tagesminimum in der 3,5-Zoll-Ansicht mindestens 16 Pixel und Schriftgewicht 600; °C ausdrücklich angegeben.
- Layout- und Integrationstests für 480×320, 480×480, 800×480 und 1280×720 in Headless-Chrome und Playwright-WebKit bestanden. Neue Tests prüfen die tatsächlichen Rechtecke von Karte und Legende, eine einzige Menüzeile ohne Überlauf, die drei zusätzlichen Zeitvorschauen sowie Radio-Titel und Ausfallzustände.
- 11 Wetter-Browsertests und 16 Berechnungstests bestanden. Radar-Farbfixture und Fehlerwiederherstellung in beiden Engines bestanden. Radar puffert im Test 23 eindeutige Bilder mit höchstens zwei gleichzeitigen Anfragen; Wiedergabe erzeugt keine weiteren Abrufe.
- Die sichtbaren Klickpositionen bei 8 %, 50 % und 92 % der Buttonbreite wurden erneut in Chrome und WebKit geprüft, jeweils mit Pixelraster und kalibrierter physischer Vorschau. Senderauswahl einschließlich Dropdown, Stummschaltung, Start, eigene Sender und Tastatureingabe funktionieren. Kein sichtbares Safari-Automationsfenster wurde verwendet.
- Neue Temperatur-/Wind-Pufferprüfung mit neun PNG-Fixtures im echten Format 880×448: höchstens zwei gleichzeitige Anfragen, feste Kartenhöhe, gezieltes Wiederholen eines fehlgeschlagenen Bildes, Wiedergabe ohne neue Netzwerkabrufe und gespeicherte Bilder offline. Dabei wurde eine beim Ansichtswechsel verlorene Lückenanzeige korrigiert. Zeit-/Quellen- und Statusbereiche reservieren auch vor der ersten Antwort ihren Platz.
- Echte DWD-Abrufe liefern Temperatur stündlich und die explizit angeforderte Windwahrscheinlichkeit sechsstündlich. CAMS liefert drei Ortsreihen. Titeladapter mit vier eigenen Tests für fragmentierte ICY-Angaben, Umlaute, fehlende Metadaten, Unix-Zeitstempel, veraltete Beats-Titel und unbekannte Sender geprüft. Zusammen mit den vorhandenen Servertests: sechs Tests bestanden. Live-Stream-Metadaten von COSMO, pure fm Berlin und radioeins abrufbar; es kann eine Sendungs- oder Stationsangabe statt eines Songtitels sein.

Reproduzierbare Zusatzprüfungen bei laufender Vorschau:

```sh
npm run test:regression
npm run test:safari
node --test radio/server/metadata.test.mjs
```

Die tatsächliche Lesbarkeit, Blickwinkel und Helligkeit des physischen 3,5-Zoll-Panels sowie dessen Touchdruck und die Bluetooth-Ausgabe sind hier weiterhin nicht hardwareseitig geprüft.

Abschluss: gemeinsamer Produktionsbuild erfolgreich. Der neue Build lädt in Chrome und Playwright-WebKit auch nach Ausfall der lokalen Quelle offline: Startmenü, Demo-Wetter einschließlich Klimaraster, Radio mit erklärter Offline-Meldung und Standby. Die neu hinzugefügten Titelinformationen setzen einen erreichbaren lokalen Adapter und Internetzugang voraus.

## 9.10.2026 · COSMO-Playlist und Radio Swiss Jazz

- Radio Swiss Jazz als fünfter Sender ergänzt, die ersten vier Plätze bleiben unverändert. Offizielle HTTPS-MP3-Adresse mit 128 kbit/s und echten ICY-Titelangaben geprüft.
- COSMO liest die WDR-Ergebnistabelle: Titel, Interpret und Berliner Sendezeit. Kennzeichnung „Zuletzt gespielt“, Alter und Ausfall der Playlist bleiben sichtbar. Die Stream-Metadaten dienen als ausdrücklich benannter Rückfall. Ein vollständiger Ausfall beim späteren Aktualisieren entfernt die bisherige Zeitangabe.
- Neun Adaptertests bestanden, einschließlich Sommer-/Winterzeit, Tageswechsel, Zeitumstellung, zukünftigem/zu altem Eintrag, fehlender Tabelle und getrennten Ausfällen der beiden Quellen.
- Neue Browserprüfung in Headless-Chrome und Playwright-WebKit bestanden: Titel und Interpret, Sendezeit, vier Displaygrößen ohne vertikalen Überlauf, Senderreihenfolge, Wiedergabe mit Audiofixture, gespeicherte Auswahl sowie Ausfälle beim Aktualisieren. In die gemeinsame Regressionstestsuite aufgenommen; keine sichtbare Safari-Automation.
- Zusätzlich echte COSMO-Tabelle und Swiss-Jazz-Stream im Browser geprüft: aktuelle Titel abrufbar, Audio dekodiert, Wiedergabezeit steigt. Tonausgabe im Test stumm. Abschließender gemeinsamer Produktionsbuild erfolgreich. Wetter wurde in diesem Durchgang nicht geändert; dessen gesamte frühere Testsuite wurde deshalb nicht erneut ausgeführt.

```sh
node --test radio/server/metadata.test.mjs
cd startmenue
node tests/radio-playlist.mjs
node tests/radio-playlist.mjs webkit
```

## 9.10.2026 · Regenradar auf Berlin und Brandenburg konzentriert

- Der regionale Ausschnitt umfasst beide Länder vollständig; zusätzliche Breite zeigt deren Umfeld. Bei 480×320 misst die Karte 400×205 Pixel, Brandenburg darin 186 Pixel in der Höhe. Bei 800×480 sind es 702×345 Pixel Kartenfläche, bei 1280×720 1174×581. Die Intensitätsskala bleibt 58–64 Pixel breit und verdeckt keine Daten. Die Karte nimmt in jeder Größe mehr als die Hälfte der gesamten Displayfläche ein.
- Neuer Geometrietest in Headless-Chrome und Playwright-WebKit: vier Auflösungen, vollständige Landesgrenzen, Kartenanteil, Legendenfläche, tatsächliche Größe der Ortsbeschriftung, kein Scrollen und mindestens 44 Pixel hohe Schaltflächen. Zusätzlich kalibrierte Rastervorschau mit Startmenü und sichtbaren Klickpositionen.
- Vorladeprüfung: 23 eindeutige Bilder, höchstens zwei parallele Abrufe, gleichbleibende Kartenhöhe während des Ladens. Wiedergabe und Jetzt bestehen. Fehlende/veraltete Bilder und Wiederherstellung eines ungültigen Caches bestehen in beiden Engines. Bestehende Chrome-Prüfung für Menüs, andere Karten und Lesbarkeit bestanden.
- Echter DWD-Lauf: 23 kommende Radarbilder vorbereitet; Wiedergabe ohne weitere Radarabfragen, erneutes Anzeigen aus dem Cache bei blockierter DWD-Verbindung. Der Netzwerktest zählt jetzt nur den Regen-Layer, damit unabhängige Temperatur-/Windabrufe nicht fälschlich als Nachladen des Radars gelten.

```sh
cd startmenue
node tests/radar-layout.mjs
node tests/radar-layout.mjs webkit
node tests/radar-buffer.mjs
node tests/radar-recovery.mjs
node tests/radar-live.mjs
```

## 9.10.2026 · Gemeinsame Oberfläche für alle vier Karten

- Regen, Temperatur, Wind und Luftqualität nutzen dieselben Komponenten für Kartenrahmen, vertikale Legende, Zeitzeile und Wiedergabe. Feste Zeilen in der Legende verhindern Versatz durch unterschiedlich lange Erläuterungen. Alle Karten verwenden denselben Ausschnitt für ganz Berlin und Brandenburg.
- Neuer Vergleichstest in Chrome und Playwright-WebKit über alle vier Displaygrößen: gleiche Position und Abmessungen für Karte, Farbskala, Zeitzeile, Ladebalken und Bedienung; kein Scrollen; Vorschau und Jetzt funktionieren. Morgen/übermorgen erhalten explizite Datumsangaben.
- Luftqualität: drei Ortsmarker mit EAQI, keine erfundene Flächenkarte. Fehlende Angaben, Klassengrenzen und PM₂,₅ des ausgewählten Ortes geprüft. Zahlen bleiben kontrastreich, Farben kennzeichnen die Marker und Skala.
- Vorladeprüfung für Temperatur/Wind bestanden: höchstens zwei parallele Anfragen, unveränderte Kartenhöhe, gezieltes Wiederholen eines fehlenden Bildes, Wiedergabe ohne weitere Bildabrufe und Offline-Cache. Radar-Geometrie einschließlich kalibrierter Pixelvorschau in Chrome sowie Fehlerwiederherstellung in Chrome/WebKit bestanden.

```sh
cd startmenue
node tests/maps-consistency.mjs
node tests/maps-consistency.mjs webkit
```

## 9.10.2026 · Bildwechsel und Wischbedienung, Version v0.1.0

- Die native Vorschau behält das vollständige Bild bis zum fertigen Ersatz. Während eines strukturellen Wechsels aktivieren Eingaben keine unter dem alten Bild liegenden neuen Bedienelemente. Bei fehlgeschlagener Rasterisierung wird weiterhin die aktuelle Browserdarstellung freigegeben.
- Chrome und Playwright-WebKit: Übergänge pro Animationsframe, konstante Vorschauabmessungen beim Menüwechsel, langsame Bilddekodierung und Standalone-Wetter geprüft. Im Test wird die initiale ready/configure-Kommunikation der eingebetteten App abgewartet.
- Wischbedienung in allen vier Größen, einschließlich kalibrierter Originalgröße und Pixelraster: Hauptansichten über der Menüzeile, Unterseiten im Inhalt. Kurze, vertikale, abgebrochene und Mehrfinger-Gesten lösen keinen Seitenwechsel aus. Zeitregler und geöffnete Menüs behalten ihre Bedienung. Zusätzlich echte Browser-Touch-Ereignisse in Chromium geprüft.
- Sichtbare Klickpositionen und Texteingabe in WebKit/Originalgröße sowie Chrome/Pixelraster einschließlich Radio, Start, Ortsauswahl und Rasterfehler bestanden. Gemeinsame Kartengeometrie über vier Produkte und vier Formate in WebKit bestanden. Keine sichtbare Safari-Automation verwendet.
- Gemeinsamer Produktionsbuild mit TypeScript-Prüfung erfolgreich. Vor dem Tag erneut 16 Wetter-Berechnungstests sowie elf Server-/Radio-Tests bestanden. Der Tag enthält Quellcode, mitgelieferte Daten/Assets und Test-Fixtures; installierte Abhängigkeiten, erzeugte Builds und lokale Testaufnahmen sind ausgeschlossen.

```sh
cd startmenue
node tests/preview-navigation.mjs
node tests/preview-navigation.mjs webkit
```
