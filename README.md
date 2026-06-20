RemoteDeck v3.15
================

Start:
1. RemoteDeck\RemoteDeck.exe starten.
2. Den QR-Code im unteren Bereich mit dem iPhone scannen.
3. Zertifikat-Warnung akzeptieren und optional zum Home-Bildschirm hinzufuegen.
4. Sobald das iPhone verbunden ist, minimiert sich die Windows-App automatisch
   in den Tray. Linksklick/Doppelklick auf das Tray-Icon oeffnet sie wieder,
   Rechtsklick zeigt ein Menue mit Oeffnen, QR Code zeigen und Beenden.

Windows-App Host:
- Die gebaute native App liegt in RemoteDeck\RemoteDeck.exe.
- Diese EXE nutzt ein eigenes Win32-Fenster ohne Browser/Tkinter, zeigt unten
  einen grossen QR-Code und startet den RemoteDeck-Server eingebettet.
- Fuer den schnellen iPhone-Test zeigt der QR-Code standardmaessig auf den
  lokalen HTTP-Link auf Port 8080. HTTPS auf Port 8443 bleibt parallel aktiv;
  wegen des selbstsignierten Zertifikats zeigt iOS dort erwartbar eine
  Sicherheitsabfrage, solange kein Zertifikat auf dem iPhone vertraut wird.
- Der Windows-Host ist kompakter, skaliert mit der Fenstergroesse, zeigt nur
  ruhige Statuspunkte und prueft den Status alle 30 Sekunden oder nach einer Aktion.
- Die QR-Ansicht zeigt eine Live-Diagnose fuer Remote verbunden/wartet,
  letzten Kontakt, QR-Protokoll und HTTPS-Status.
- /api/status antwortet schnell mit Verbindungsdaten und Cache-Werten; schwere
  System-, Audio-, Power- und Integrationschecks werden im Hintergrund
  nachgewarmt, damit der erste Handy-Aufruf nicht blockiert.
- Fenstergroesse und Position werden beim Verschieben/Schliessen gespeichert
  und beim naechsten Start wiederhergestellt.
- Beim Schliessen zeigt RemoteDeck den Hinweis "wird in den Tray minimiert",
  bleibt im Tray aktiv und kann dort wieder geoeffnet oder beendet werden.
- Die Mindestgroesse schuetzt die QR- und Setup-Leiste vor gequetschten
  Zwischenlayouts; groesseres Ziehen skaliert weiter ruhig mit.
- Config ist jetzt ein internes Setup-Menue fuer App-, Server-, Sicherheits-
  und Integrationsstatus. Pfade werden dort am PC gepflegt; der QR-Code bleibt
  ueber denselben Button sowie ueber das Tray-Menue erreichbar.
- Beim Start laeuft ein gecachter Systemcheck im Hintergrund. Er prueft
  Datenordner, Zertifikat, eingebettetes Python, VC-/OpenSSL-Runtime,
  PowerShell, Scheduled Tasks, shutdown.exe, Prozess-Scan und gebuendelte
  App-Dateien, ohne die normalen Statuspolls zu belasten.
- Die letzte Host-Statuskachel zeigt den Systemcheck; im Setup-Menue stehen
  Integrationen, Host und Check kompakt nebeneinander.
- RemoteDeck nutzt ein eigenes Icon fuer Programmfenster, EXE und Tray.
- RemoteDeck ist Single-Instance: ein zweiter Start oeffnet die vorhandene
  Instanz wieder, statt einen zweiten Server zu starten.
- Native-Patches liegen in patches/native_app.json. Damit koennen Titel,
  Untertitel, Akzentfarben, Auto-Tray und Refresh-Intervalle spaeter ohne
  Codeaenderung angepasst werden.
- Fuer den spaeteren Installer ist ein installer/-Ordner vorbereitet. Der
  Installer nimmt den kompletten PyInstaller-Ordner dist\RemoteDeck auf,
  nicht nur RemoteDeck.exe. Dadurch sind Python-DLL, OpenSSL-DLLs, VC-Runtime,
  Serverdatei, UI, Patches und Fan-Profile enthalten.
- HTTPS-Zertifikate werden pro Geraet im beschreibbaren Datenordner erzeugt.
  cert.pem/key.pem werden nicht mehr als gemeinsamer privater Schluessel ins
  Release-Bundle gelegt.
- Installierte Builds bekommen den Marker RemoteDeck.installed. Dann speichert
  RemoteDeck Config, Logs, Patches und Runtime-State in
  %LOCALAPPDATA%\RemoteDeck. Portable Builds ohne Marker bleiben weiterhin
  im eigenen Ordner portabel.
- installer\smoke_test_bundle.ps1 prueft vor dem Installer-Bau, ob alle
  noetigen Runtime-Dateien im Bundle liegen. BUILD_REMOTEDECK_EXE.bat fuehrt
  diesen Check nach PyInstaller automatisch aus.
- installer\RemoteDeck.iss ist fuer Inno Setup vorbereitet. Inno Setup wird
  nur auf der Build-Maschine benoetigt; der spaetere Nutzer muss weder Python
  noch Inno Setup, pip, OpenSSL oder eine VC-Runtime separat installieren.
- Wenn die laufende App dist\RemoteDeck sperrt, kann ein neuer Build in einen
  Staging-Ordner gebaut werden. installer\build_installer.ps1 akzeptiert dafuer
  -BundleDir, z.B. dist_stage\RemoteDeck. BUILD_REMOTEDECK_EXE.bat erkennt
  diesen Fall automatisch und baut dann nach dist_stage\RemoteDeck.
- installer\promote_staged_build.ps1 uebernimmt einen geprueften Staging-Build
  nach dist\RemoteDeck und kann mit -StopRunning die dort laufende alte App
  gezielt beenden.
- Ohne Inno Setup kann installer\package_portable_zip.ps1 ein vollstaendiges
  Portable-Zip aus dist\RemoteDeck oder dist_stage\RemoteDeck erstellen.
- Fuer spaetere Updates bereitet die App %LOCALAPPDATA%\RemoteDeck\updates vor:
  staged\ nimmt einen neuen Bundle-Stand auf, RemoteDeckApplyUpdate.cmd wartet
  auf das Beenden der App, kopiert Dateien unsichtbar per Batch und startet die
  EXE erneut. Pairing-State, Logs, lokale Zertifikate und Runtime-Dateien werden
  dabei nicht ueberschrieben.
- installer\sign_release.ps1 ist als Signatur-Schritt vorbereitet. Lokale
  Testsignaturen sind nur fuer Smoke-Tests sinnvoll; fuer oeffentliches Vertrauen
  ist SignPath Foundation fuer geeignete Open-Source-Projekte oder spaeter
  Microsoft Artifact Signing / ein CA-Zertifikat vorgesehen.
- .github\workflows\signpath-release.yml ist fuer GitHub-Releases vorbereitet:
  EXE signieren, Installer mit signierter EXE bauen, Setup signieren,
  Checksummen/Release Notes/latest.json erzeugen und optional als Draft-Release
  auf GitHub veroeffentlichen.
- docs\GITHUB_RELEASE.md beschreibt die Veroeffentlichungs-Checkliste. Vor dem
  ersten oeffentlichen Upload muessen Lizenz, SignPath-Secrets und Draft-Pruefung
  bewusst gesetzt werden.
- Beim ersten Start uebernimmt die EXE fehlende Defaults aus ui/, patches/
  und fan_profiles/ in den beschreibbaren Runtime-Ordner. Bestehende
  Nutzeranpassungen werden dabei nicht ueberschrieben.
- START_SERVER_V3.bat und START_WINDOWS_APP.bat sind nur noch Weiterleitungen
  zur Standalone-EXE, damit alte Verknuepfungen nicht ins Leere laufen.
- Der Host startet server_v3.py bei Bedarf im Hintergrund, zeigt die iPhone-URL
  samt lokal erzeugtem QR-Code und fragt /api/status fuer Server, SignalRGB,
  FanControl, iCUE und Sicherheit ab.
- Apps suchen prueft in der Windows-App Standardpfade fuer SignalRGB,
  FanControl, iCUE und Gigabyte und speichert gefundene Pfade in
  remote_config.json.
- Start- und Fehlerdetails landen in remote_host.log, damit der Batch nicht
  mehr still verschwindet.
- ui/ und patches/ sind vorbereitet, damit Design- und Animationsaenderungen
  spaeter ohne kompletten EXE-Neubuild getestet werden koennen.
- Wenn ui/index.html vorhanden und gueltig ist, liefert der Server diese Datei
  als Web-App aus. Ist sie fehlerhaft oder fehlt sie, nutzt RemoteDeck weiter
  die eingebettete PWA aus server_v3.py.

Bedienung:
- Unten sind vier auswaehlbare Tabs: Home, Audio, Apps und Setup.
- Home ist wie ein kompaktes iOS-Control-Center aufgebaut: Windows-Verbindung,
  Shutdown-Status, Systemwerte, Lautstaerke, Helligkeit und Power.
- Audio zeigt Lautstaerke, Mute-Status, Windows-Ausgabegeraete und Media-Tasten.
- Apps startet Programme, steuert SignalRGB/iCUE, durchsucht installierte
  SignalRGB-Effekte und schaltet FanControl-Profile.
- Setup auf dem Handy zeigt nur sinnvolle Remote-Einstellungen: Design,
  Systemcheck, Verbindungsstatus, Integrationsstatus und Sicherheit als
  Status. Programm-Pfade, Scans und Sicherheits-/Serveroptionen werden nicht
  mehr auf dem Mobilgeraet bearbeitet.
- Setup in der Windows-App ist der Ort fuer automatische Suche und Pfadpflege.
- Design kann auf Handy und Windows zwischen Dunkel, Hell und System gewechselt
  werden. Der Systemmodus folgt der Windows- bzw. Geraete-Einstellung.
- Die Windows-App-Settings enthalten jetzt echte Host-Steuerungen fuer
  QR-Protokoll HTTP/HTTPS, Auto-Tray, privaten LAN-Modus und einen direkten
  Sprung zu den Windows-Farbeinstellungen.
- Die Windows-App-Settings sind in Host, Apps und Diagnose gegliedert.
- Diagnose enthaelt jetzt echte PC-Werkzeuge: Autostart setzen, Startmenue-
  Verknuepfung erzeugen, Datenordner/Log/Patchdatei oeffnen, Log leeren und
  HTTP/HTTPS-Portcheck ausfuehren.
- Der Portcheck prueft lokal 8080 und 8443, ohne das Handy zu belasten. Damit
  ist schnell sichtbar, ob iPhone-HTTP und interner HTTPS-Server erreichbar
  sind.
- patches/native_app.json kann direkt aus der Windows-App geoeffnet werden.
  Dadurch bleiben spaetere UI-/Refresh-/Akzent-Anpassungen patchbar, ohne
  dass auf dem Handy Pfade oder technische Host-Optionen auftauchen.
- Die Windows-App zeigt beim manuellen Schliessen den Tray-Hinweis auch dann
  zuverlaessig, wenn kurz vorher schon eine Tray-Meldung angezeigt wurde.
- Die Pfadfelder im Windows-Setup sind farblich an Hell/Dunkel angepasst,
  haben Innenabstand und liegen auf gezeichneten Glasrahmen statt auf harten
  Windows-Standardrahmen.
- Die Handy-Settings enthalten nun eine reine Verbindungskarte fuer iPhone,
  QR-Protokoll und HTTP/HTTPS-Ports. PC-Pfade und Host-Schalter bleiben
  weiterhin nur in der Windows-App.
  Apps bietet nicht-destruktive Tests fuer SignalRGB, FanControl, iCUE und
  Systemcheck; Diagnose bietet Autostart, Datenordner, Log-Oeffnen und
  Log-Leeren.
- Die UI nutzt unmittelbare Touch-Down-Haptik, kurze Spring-Animationen,
  Ripple/Toast-Feedback und Value-Pop fuer Zahlen. SignalRGB-Effekte faerben
  den Control-Center-Hintergrund passend und bleiben nach Refresh/Tabwechsel
  erhalten, Shutdown zeigt eine kurze Warnwelle.
- Sicherheit: Der Server akzeptiert standardmaessig nur Loopback und private
  Netzwerk-Adressen wie 192.168.x.x, 10.x.x.x oder 172.16-31.x.x.

Power:
- Restart wurde entfernt.
- Shutdown wird ausschliesslich ueber eine Benutzereingabe in Minuten gesetzt.
- Vor dem Setzen sind zwei Bestaetigungen erforderlich.
- Timer werden als Windows Scheduled Task angelegt.
- Der laufende Status wird aus dem gespeicherten Timer und der geplanten Task
  gelesen, auch nach einem Server-Neustart.
- Abbrechen entfernt die geplante Task und ruft zusaetzlich shutdown /a auf.

Audio:
- Lautstaerke und Mute werden nach jeder Aktion erneut gelesen.
- Ausgabegeraete werden ueber Windows Core Audio aufgelistet.
- Das Standard-Ausgabegeraet kann direkt in der Audio-Ansicht gewechselt werden.
- Medienzuordnung: Previous, Play/Pause, Next, Stop, leiser, lauter.
- Medientasten werden primaer als globale Windows-Media-Keys gesendet
  (SendInput) und fallen nur bei einem technischen Fehler auf APPCOMMAND
  zurueck. Lautstaerke/Mute lesen danach den Audiostatus neu.
- YouTube-Tasten senden gezielte Keyboard-Shortcuts an das aktive Fenster
  (K, J, L, M, F, C, T, I, Shift+P/N und Tempo +/-). Der Browser/Player
  muss am PC im Fokus sein.
- Maus und Tastatur laufen ueber /api/input. Touchpad-Bewegungen werden
  clientseitig gebuendelt, damit das Netzwerk nicht mit Pointer-Events
  ueberlastet wird. Pfeiltasten, Enter, Escape, Tab, Backspace, Delete und
  Texteingabe gehen an das aktive Windows-Fenster.

Helligkeit:
- Interne Displays werden ueber WMI gelesen und gesetzt.
- Mehrere WMI-Displays werden als Geraete gefuehrt; der angezeigte Wert ist
  der Mittelwert der aktiven Displays.

Performance:
- Statusabrufe sind gedrosselt. Die UI aktualisiert etwa alle 15 Sekunden und
  pausiert normale Abrufe, wenn die Seite im Hintergrund ist.
- Die Handy-App erkennt Android, iOS, Save-Data, wenig RAM/CPU und Reduced
  Motion. Auf leichteren Geraeten werden Blur, Ripple und Hintergrundanimationen
  automatisch reduziert, ohne Funktionen abzuschalten.
- Mobile Rendering nutzt dynamische Viewport-Hoehe, sichere Tap-Ziele,
  stabile Reiterwechsel und gebuendelte Render-Updates per requestAnimationFrame.
- Touchpad-Bewegungen werden visuell per Animation-Frame und netzwerkseitig
  gebuendelt gesendet; Android nutzt etwas ruhigere Intervalle.
- API-Requests haben kurze Timeouts, damit die UI bei WLAN-Haengern nicht
  blockiert.
- System, Audio, Helligkeit, FanControl, SignalRGB und iCUE werden serverseitig
  gecached, damit PowerShell/COM und App-Status nicht dauerhaft laufen.
- Dependency-Checks werden serverseitig lange gecached und beim Host-Start
  vorgewaermt. Dadurch sieht der Nutzer spaeter Hinweise, ohne dass jedes
  Handy-Refresh PowerShell oder Dateisystemscans startet.
- FanControl-Temperaturen werden aus der aktiven FanControl-Konfiguration
  abgeleitet und nur kurz gecached.
- Die SignalRGB-Effektliste wird lokal aus App-, User- und Cache-Effekten
  gelesen und nur etwa alle 120 Sekunden neu aufgebaut.
- Lautstaerke-, Helligkeits- und SignalRGB-Helligkeitsregler senden verzoegert.
- Die Windows-App cached Config und QR-Code im Paint-Loop und verhindert
  parallele Status-Refresh-Threads.
- Die native Glas-UI cached GDI-Brushes/Pens, nutzt ClearType-Fonts und zeichnet
  Host-Buttons direkt im doppelt gepufferten Canvas. Das reduziert
  Schriftwackeln, Resize-Jitter und Button-Artefakte bei Zwischenskalierungen.
- Im Tray wird die Host-App seltener aktualisiert und invalidiert das Fenster
  nicht unnoetig. Sichtbar bleibt der schnelle Status, versteckt reichen ruhige
  Hintergrundchecks.

Konfiguration:
- signalrgb_path: Pfad zum SignalRgbLauncher.exe, Standard ist
  %LOCALAPPDATA%\VortxEngine\SignalRgbLauncher.exe.
- signalrgb_api_base: lokale SignalRGB API, Standard ist
  http://127.0.0.1:16038/api/v1.
- signalrgb_off_effect: Effekt, den der Launcher nutzt, wenn die Local API
  nicht verfuegbar ist. Standard ist Solid Color mit color=#000000 und
  breathe=0; bei aktiver API wird zuerst Canvas aus genutzt.
- signalrgb_effects: Favoritenliste fuer die Effektbuttons. Bei aktiver Local
  API koennen Namen oder IDs genutzt werden; beim Launcher-Fallback sind echte
  lokal installierte Effektnamen am zuverlaessigsten. RemoteDeck liest den
  lokalen SignalRGB-Effects-Ordner zusaetzlich automatisch ein.
- icue_path: kompletter Pfad zur iCUE.exe.
- icue_lights_off_command: optionales Kommando oder Script, das in iCUE ein
  Licht-aus-Profil aktiviert.
- fancontrol_path: kompletter Pfad zur FanControl.exe.
- fan_profiles: Pfade zu silent/balanced/gaming JSON-Profilen.
- quick_launch: Liste mit id, label und command fuer Schnellstarts.
- ui.theme: dark, light oder system fuer das Control-Center-Design.
- security.private_network_only: true laesst nur lokale/private Netzwerke zu.
- security.allow_loopback: true erlaubt den Aufruf direkt auf dem PC.

SignalRGB:
- OpenRGB wurde entfernt.
- RemoteDeck nutzt zuerst die offizielle lokale SignalRGB API:
  GET /lighting fuer Status, PATCH /lighting/enabled fuer Canvas an/aus,
  PATCH /lighting/global_brightness fuer Helligkeit und POST
  /lighting/effects/:id/apply fuer Effekte. Wegen uneinheitlicher Doku wird
  bei 404 zusaetzlich /lighting/effect/:id/apply versucht.
- Wenn die Local API 403 liefert oder nicht verfuegbar ist, nutzt RemoteDeck
  SignalRgbLauncher.exe --url=... mit -silentlaunch- als Fallback.
- Die UI zeigt den Modus als Zustand: API = Vollzugriff, Launcher/URL-Modus =
  Effekte ohne Helligkeit/Canvas-API, Offline = SignalRGB-Aktionen aus.
- api_requires_pro ist kein Fehler. RemoteDeck markiert nur den Pro/API-Bereich,
  Effekte per Launcher bleiben nutzbar.
- Apps > Beleuchtung kann SignalRGB Canvas aus/an, installierte Effekte mit
  Suche, Previous, Next, Shuffle, SignalRGB-Helligkeit, Effektlisten-Refresh
  und direkte SignalRGB-Ansichten wie Customize/Devices steuern.
- Launcher-Effektfavoriten duerfen optional Query-Parameter enthalten, z.B.
  Rainbow Rise?reverse=false&speed=3.
- Setup zeigt SignalRGB-Prozess, API-Status, aktuellen Effekt und Canvas-State.

FanControl:
- FanControl ist auf C:\Program Files (x86)\FanControl\FanControl.exe gesetzt.
- Die RemoteDeck-Profile liegen in fan_profiles:
  - remote_silent.json
  - remote_balanced.json
  - remote_gaming.json
- Apps > FanControl verbindet/startet FanControl, falls der Pfad stimmt.
- Silent/Balanced/Gaming setzen das jeweilige Profil ueber FanControl.exe -c.
- active_profile kommt aus FanControls CACHE-Datei. Wenn FanControl nicht
  laeuft, zeigt die UI den zuletzt von RemoteDeck gesetzten Profilnamen als
  Fallback.
- Ein UAC-Prompt ist kein Fehlerzustand: Die UI markiert die Aktion als
  gesendet und zeigt "Am PC bestaetigen".
- Falls die RemoteDeck-Profilkopien fehlen, werden automatisch die Profile aus
  C:\Program Files (x86)\FanControl\Configurations genutzt.
- Setup zeigt, ob FanControl vorhanden ist, laeuft, welche Profile fehlen und
  wie viele aktive Controls/Kurven die Profile enthalten.
- Apps zeigt die Temperaturquellen der aktiven FanControl-Kurven und die
  aktuell lesbaren Werte. NVIDIA-GPU-Werte kommen ueber nvidia-smi; CPU- und
  weitere Sensoren werden ueber Libre/OpenHardwareMonitor-WMI gematcht, wenn
  diese Quelle verfuegbar ist.
- Stop beendet FanControl.exe ueber taskkill.

iCUE und Beleuchtung:
- Apps > Alle Lichter aus ruft SignalRGB Licht aus und danach optional den
  konfigurierten iCUE Licht-aus-Befehl auf.
- Der iCUE-Licht-aus-Button wird ausgeblendet, wenn kein externer Befehl
  konfiguriert ist.
- iCUE hat kein eigenes "Licht wieder an"; RemoteDeck bietet dafuer iCUE neu
  starten an.
- iCUE Start/Stop bleiben separat, damit die Beleuchtung ausgeschaltet werden
  kann, ohne iCUE zwangsweise zu beenden.
- Setup zeigt, ob iCUE laeuft und ob ein Licht-aus-Befehl verfuegbar ist.

Homescreen:
- /manifest.webmanifest, /icon.svg, /apple-touch-icon.png und /icon-512.png
  werden direkt vom Server ausgeliefert.
- Auf iOS kann die Seite mit RemoteDeck-Logo zum Home-Bildschirm hinzugefuegt
  werden.
- Android/Chrome bekommt ein Standalone-PWA-Manifest mit maskierbarem 512px
  Icon, Theme-Color und stabiler App-ID.

Hinweis:
Temperaturen erscheinen nur, wenn OpenHardwareMonitor oder LibreHardwareMonitor
Sensoren per WMI bereitstellt.
