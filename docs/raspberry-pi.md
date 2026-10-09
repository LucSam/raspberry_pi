# Raspberry Pi einrichten

Empfohlen für diesen Prototyp: Raspberry Pi OS **mit Desktop**, WLAN und Bluetooth. Die Desktop-Ausgabe bringt die Audio-Infrastruktur mit. Die Lite-Ausgabe benötigt zusätzliche Audio-/Bluetooth-Einrichtung. Das konkret gewählte Display braucht einen passenden Treiber; insbesondere GPIO-TFTs sind nicht wie HDMI-Displays austauschbar.

1. WLAN und Systemzeit einrichten. Zeitzone Europe/Berlin wählen; NTP-Zeitsynchronisation aktiviert lassen.
2. Bluetooth-Lautsprecher in den Kopplungsmodus bringen. In der Pi-Desktopleiste das Bluetooth-Menü öffnen, Gerät hinzufügen und koppeln.
3. In der Audioauswahl den Lautsprecher als Ausgabe wählen, mit angemessener Lautstärke testen. Wenn mehrere Profile erscheinen, das Wiedergabe-/Stereo-Profil verwenden.
4. Node.js ≥ 22.14.0 und die drei Projektordner auf dem Pi bereitstellen. Im Sammlungsordner `npm --prefix wetterwarte ci`, `npm run build`, anschließend `npm start` ausführen.
5. Chromium mit `http://127.0.0.1:5173/?kiosk=1` öffnen. Optional nach erfolgreichem Test: `chromium --kiosk http://127.0.0.1:5173/?kiosk=1`. Die tatsächliche Desktopauflösung muss dem ausgewählten Profil entsprechen. Im Startmenü Radio öffnen und Sender antippen.

Der Browser spielt über die ausgewählte Systemausgabe. Kopplung, Wiederverbindung nach Neustart, Display-Treiber und Standby-Helligkeit müssen am tatsächlichen Pi geprüft werden. Hier wurden keine Autostartdateien oder Systemkonfigurationen auf dem Mac installiert.

Standby hält die Systemuhr sichtbar und lässt das Radio weiterlaufen. Er schaltet weder WLAN noch Bluetooth ab und dimmt nicht die physische LCD-Hintergrundbeleuchtung. Eine solche Erweiterung folgt erst mit feststehender Hardware. Touch weckt nur die Ansicht auf; der Pi läuft weiter.

Dokumentation: [Raspberry Pi: Konfiguration und Bluetooth](https://www.raspberrypi.com/documentation/computers/configuration.html), [offizielle Übersicht der Audioausgaben](https://pip-assets.raspberrypi.com/categories/1259-audio-camera-and-display/documents/RP-008124-WP-1-Choosing%20an%20Audio%20option.pdf).
