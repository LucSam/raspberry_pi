# Raspberry Pi

Lokale Touch-Oberfläche mit Startmenü, Wetter, Radio und dunkler Standby-Uhr. Auf dem Intel-Mac im Browser gestaltbar; später auf Raspberry Pi OS mit Chromium verwendbar. Kein Konto, keine Cloud, keine laufenden API-Kosten.

## Start

Node.js ≥ 22.14.0. Erstinstallation von GitHub einschließlich aller Unterprojekte:

```sh
git clone --recurse-submodules https://github.com/LucSam/raspberry_pi.git
cd raspberry_pi
npm --prefix wetterwarte ci
npm run dev
```

Für die bereits vorhandene lokale Installation:

```sh
cd ~/Projects/raspberry_pi
npm run dev
```

**http://127.0.0.1:5173/** öffnen. Läuft die Vorschau schon, genügt **⌘R**. Nach Änderungen an Startmenü oder Radio neu laden; Wetter nutzt Vite mit automatischer Aktualisierung. Ctrl+C beendet beide Entwicklungsserver. Wetter-Vite belegt intern Port 5174.

Wurde ohne `--recurse-submodules` geklont, zunächst `git submodule update --init --recursive` ausführen. Startmenü und Radio benötigen keine npm-Abhängigkeiten.

Für den Pi oder eine offline nutzbare Produktionsvorschau:

```sh
npm run build
npm start
```

Vorher den Entwicklungsserver beenden. Gleiche Adresse und damit gleiche lokale Einstellungen und Daten. Ein vollständiger erster Online-Besuch speichert die Oberfläche; Radio benötigt stets Internet. Der lokale Server muss weiterhin laufen.

## Bedienung

- **Wetter / Radio:** Kacheln öffnen das jeweilige Modul. **Start** oben rechts führt zurück. Radio spielt beim Wechsel weiter; auf Start zeigt eine Schaltfläche den laufenden Sender.
- **Standby:** dunkle Uhr mit Systemzeit in Europe/Berlin. Antippen oder Esc führt zum Startmenü. Automatisch nach 5, 10 oder 30 Minuten ohne Bedienung; alternativ nur manuell. Radio läuft weiter.
- **Wetter:** zusätzliche Seite **Fokus** mit Wetter links und großer Regenkarte rechts. Die vorhandene Übersicht bleibt. Max/Min sind größer. Normale Herkunftszeilen und Info-Dialog entfallen; Demo-, Offline-, Fehler- und Veraltet-Hinweise bleiben. Methodik steht im [Wetter-README](wetterwarte/README.md).
- **Radar:** aktuelle Karte zuerst, dann automatische Vorbereitung der gesamten kommenden Bildfolge mit zwei parallelen Abrufen. Balken und Bildzähler zeigen den Fortschritt. Abspielen wartet auf alle Bilder; fehlende Bilder werden automatisch erneut angefragt und lassen sich manuell erneut laden. Abgelaufene Prognosen werden ausdrücklich gekennzeichnet. Der Binärcache ist auch für WebKit ausgelegt. Es werden ausschließlich Zeitpunkte desselben DWD-Modelllaufs verwendet.
- **Vorschau:** vier Formate, Originalgröße mit Linealkalibrierung und native Pixelansicht. Bei fehlgeschlagener Rasterdarstellung bleibt die aktuelle HTML-Oberfläche bedienbar; alte Rasterbilder werden entfernt. Vorhandene Kalibrierung und Wetterdaten bleiben auf derselben Browseradresse erhalten. **Geräteansicht** blendet die Gestaltungsregler aus (`?kiosk=1`). Auf dem echten Display Browserzoom 100 %, passende Auflösung wählen.

Standby ist eine Bildschirmansicht, kein Suspend und keine Regelung der Hintergrundbeleuchtung. Bei einem LCD spart Schwarz allein nicht wesentlich Energie. Eine echte Helligkeitssteuerung braucht einen zum gewählten Display passenden Pi-Treiber. Auf dem Mac wird Helvetica verwendet; auf einem Pi ohne diese Schrift greift der System-Fallback.

## Unabhängige Unterprojekte

```text
raspberry_pi/                 Sammlung, gemeinsame Startbefehle, Dokumentation
├── startmenue/               eigener Node-Server, Kacheln, Standby, Displayvorschau
├── wetterwarte/              eigenständige TypeScript-/Vite-Anwendung
├── radio/                    eigenständige Browseranwendung, ohne Bundler
└── docs/                     Pi-Einrichtung und Modulvertrag
```

Jedes Unterprojekt hat eigene Startbefehle, README, `.gitignore` und ein eigenes GitHub-Repository. Die Sammlung bindet sie als Git-Submodule ein und hält damit genau die zusammen geprüften Commits fest. Es bestehen keine symbolischen Verknüpfungen und keine gemeinsamen `node_modules`. Werden die Projekte einzeln geklont, als Nachbarordner anordnen; Mounts stehen in `startmenue/modules.json`.

| Verzeichnis | Repository |
| --- | --- |
| Sammlung | [LucSam/raspberry_pi](https://github.com/LucSam/raspberry_pi) |
| `startmenue/` | [LucSam/pi-startmenue](https://github.com/LucSam/pi-startmenue) |
| `wetterwarte/` | [LucSam/wetterwarte](https://github.com/LucSam/wetterwarte) |
| `radio/` | [LucSam/pi-radio](https://github.com/LucSam/pi-radio) |

## Versionsstand

Die erste gemeinsam versionierte Ausgabe trägt in allen vier Repositories den annotierten Git-Tag **`v0.1.0`**. Die Sammlung referenziert die passenden Modul-Commits. Zum späteren Wiederherstellen dieses Standes in einem frischen Klon:

```sh
git switch --detach v0.1.0
git submodule update --init --recursive
npm --prefix wetterwarte ci
npm run build
npm start
```

Für eine Aktualisierung auf `main`: `git pull --ff-only`, danach `git submodule update --init --recursive` und die Abhängigkeiten beziehungsweise den Build aktualisieren. Eigene Moduländerungen zuerst im jeweiligen Unterprojekt committen und hochladen, anschließend dessen neuen Commit in der Sammlung festhalten. Tags bestehender Ausgaben bleiben unverändert.

Neue Module ergänzen ausschließlich einen Eintrag in dieser Datei und den dokumentierten [Modulvertrag](docs/module.md). Der Radioplayer bleibt in einem eigenen eingebetteten Dokument geladen, damit Navigation die Audiowiedergabe nicht beendet. Die Wetteranwendung bleibt separat lauffähig.

## Bluetooth und Erweiterungen

Bluetooth-Boxen im Betriebssystem koppeln und als Tonausgabe wählen. Die [Pi-Anleitung](docs/raspberry-pi.md) beschreibt den Ablauf; die App gibt keine erfolgreiche Kopplung vor. Auf dem Mac dieselbe Auswahl unter Systemeinstellungen → Bluetooth bzw. Ton.

Sinnvolle nächste Module: Timer/Wecker, Kalender sowie VBB-Abfahrten für Elstal. Diese sind noch nicht implementiert. Temperatur- und Windkarten sind als DWD-WMS-Flächendaten verfügbar; CAMS/Open-Meteo liefert modellierte Luftqualität. Unter Radar sind Regen, Temperatur, Wind und Luftqualität auswählbar. Temperatur: ICON-EU, 2 m, 0,0625°, stündlich. Wind: ICON-EPS, 0,25°, sechsstündlich, Wahrscheinlichkeit für mehr als 36 km/h. Luftqualität: CAMS Europa, drei Ortswerte (EAQI und PM₂,₅); keine Flächeninterpolation. Die Karten zeigen ihre eigenen Modellzeiten und Quellen.

Quellen: [DWD-Geodienste](https://www.dwd.de/DE/leistungen/geodienste/help/nutzung_geodienste.html), [Open-Meteo Luftqualität](https://open-meteo.com/en/docs/air-quality-api).

## Prüfung

```sh
npm --prefix wetterwarte test
# Gemeinsame Vorschau muss laufen:
WEATHER_TEST_URL=http://127.0.0.1:5173/wetter/ npm --prefix wetterwarte run test:browser
npm test
```

Browserprüfungen verwenden installiertes Google Chrome und das Playwright-Paket aus Wetterwarte. Die Tests des Startmenüs prüfen Integration, Formate, Standby, native Rasterdarstellung und Radiofehler. Zusätzliche Radar- und Klicktests: `npm run test:regression`; WebKit-Integration: `npm run test:safari` (einmalig `cd wetterwarte && npx playwright install webkit`). Details und Ergebnisse stehen im [Prüfprotokoll](docs/pruefung.md). Ein physischer Pi und Bluetooth-Lautsprecher wurden hier nicht getestet.
