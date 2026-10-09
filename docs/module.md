# Modulvertrag v1

Jeder Eintrag in `startmenue/modules.json` enthält `id`, `title`, `description`, `path`, `directory`, `icon`. Das Startmenü lädt die lokale URL mit `?embedded=1` in ein eigenes iframe. Ein Modul muss eigenständig funktionieren und darf keine Dateien eines Nachbarmoduls importieren. Native ESM-Module oder vollständig gebaute Vite-Verzeichnisse sind geeignet.

Die optionalen Nachrichten verwenden `postMessage` mit `{protocol:'pi-display-v1',type,...}`. Sender und Empfänger prüfen `event.origin` und das konkrete Gegenfenster.

| Richtung | Typ | Inhalt |
|---|---|---|
| Modul → Start | `ready` | Modul ist bereit für Einstellungen |
| Start → Modul | `configure` | `settings`: `size`, `theme`, `weatherMode`, `idle` |
| Modul → Start | `home` | Startansicht öffnen |
| Modul → Start | `activity` | Bedienung, setzt den Standby-Zähler zurück |
| Modul → Start | `rendered` | Darstellung geändert; native Vorschau neu zeichnen |
| Wetter → Start → Radio | `appearance` | `theme`: `light` oder `dark`, Sonnenstand des gewählten Orts |
| Radio → Start | `radio-state` | `playing`, `station` |

Ein eingebettetes Modul rendert die Geräteoberfläche als `.device` ohne eigene Vorschauleisten. Es füllt den verfügbaren Viewport. Größe bleibt in nativen CSS-Pixeln; das Startmenü skaliert die gesamte Oberfläche und erzeugt bei Bedarf ein natives Canvas-Abbild. Die Original-DOM-Oberfläche bleibt für Maus/Touch bedienbar.

Module werden bei Navigation verborgen, nicht neu geladen. Das ermöglicht durchgehendes Radio und Wetteraktualisierung. Große zukünftige Module können eine eigene Pause-Nachricht ergänzen. Die Sammlung enthält keine Fernsteuerung und veröffentlicht keine Module ins Netz.
