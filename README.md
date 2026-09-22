# SNN Roboter-Trainer – Anleitung

**MuJoCo-WASM · Franka Emika Panda · Spiking Neural Network · GEN-1.5-inspiriert (vereinfacht)**

Deine App: `SNN-Roboter-Trainer-v1.4.apk` (9,0 MB, signiert, läuft komplett offline auf dem Gerät)

---

## ⭐⭐ Neu in v1.4 – Trainings-Stufen (wie GEN 1.5), Tempo-Beschleunigung & paralleles Sammeln

- **3-Stufen-Trainingskurrikulum** (Tab *SNN-Training*) – genau die GEN-1.5-Pipeline in Miniatur:
  - **Stufe 1 · Video-Vortraining (BC):** Verhaltens-Klonen auf dem gesammelten Datensatz (= wie *Gen 1*: Foundation-Policy per Behavior Cloning). Das bisherige Training – jetzt mit Stufen-Anzeige.
  - **Stufe 2 · Roboter-Finetuning (RL):** Das SNN steuert **selbst** den Roboter (mit Explorations-Rauschen, ~24× Echtzeit), bekommt **Belohnung** fürs Greifen & Stapeln und verfeinert sich daraus per **Advantage-gewichteter Regression + BC-Anker** gegen Vergessen (= wie *Gen 1.5*: RL-Post-Training auf dem BC-Fundament mit Hybrid-Loss). Einstellbar: RL-Episoden, Exploration σ, RL-Lernrate, BC-Anteil.
  - **Stufe 3 · Präzise Ausführung:** Deterministische Ausführung in Echtzeit **ohne Datensammlung** – reine Ausführung & Bewertung mit Erfolgsquote („5/5 gestapelt“), die über Neustarts erhalten bleibt.
- **SNN ausführen ohne Daten zu sammeln**: Der neue Stufe-3-Button (und der Aufnahme-Haken) trennt Ausführung und Aufnahme sauber – beim reinen SNN-Lauf wächst der Datensatz nicht mehr.
- **Tempo-Beschleunigung**: Neue Tempo-Stufen **8× und 16×**; im Modus **„Maximal“** laufen Physik und SNN synchron, aber so schnell das Gerät hergibt – größeres Zeitbudget pro Frame, gedrosseltes Rendern (je nach Gerät deutlich über 100× Echtzeit). Die Statistik zeigt jetzt echte **Ticks/s + Tempo-Faktor (×N)**.
- **Paralleles Sammeln (Worker)**: Im Tab *Datensatz* starten – jeder Worker ist ein eigener Thread mit **eigener MuJoCo-Physik, eigenem Experten und eigenen Kameras** (headless, ohne 3D-Anzeige) und sammelt mit maximalem Tempo. Alle Daten landen automatisch im selben Datensatz-Speicher (Episoden-Nummern werden eindeutig umnummeriert, Speicher-Quota wird überwacht). Auf dem S24U/S26U (8 Kerne) sind standardmäßig 3 Worker aktiv – der Datensatz wächst damit grob ×3 schneller.
- **Antwort auf „Geht paralleles Lernen?“**: Datensammlung jetzt echt parallel (mehrere Physik-Instanzen), das SNN-Training selbst läuft ohnehin parallel (gebündelte Batches auf der GPU/WebGPU), und Stufe 2 lernt aus eigenen Rollouts im Schnelltempo.

> ✅ **Update**: gleicher Signaturschlüssel wie v1.2/v1.3 – **direkt über die bestehende App installierbar**.

## ⭐ Neu in v1.3 – Grosser Speicher: 70.000+ Datensätze, 1 Mio.+ Trainingsiterationen

- **Kein RAM-Limit mehr**: Der Datensatz liegt nicht mehr im Arbeitsspeicher (vorher Limit ≈ 320 MB ≈ 6.000 Schritte), sondern wird **dauerhaft in Chunks auf dem Gerätespeicher (IndexedDB)** abgelegt. **70.000+ Schritte** (≈ 3,9 GB) sind problemlos möglich – der RAM-Verbrauch bleibt dabei konstant niedrig (Schreibpuffer + ~190 MB Lese-Cache).
- **Dauerhaft & absturzsicher**: Jeder volle Chunk (256 Schritte) wird sofort gesichert, der laufende Rest alle 60 Sekunden. Nach Absturz, Hintergrund-Kill oder Neustart ist der komplette Datensatz **automatisch wieder da** – ohne Banner-Klick.
- **1.000.000+ Trainingsiterationen**: Das Training liest die Daten jetzt **chunk-weise von der Platte** (gemischte Reihenfolge pro Epoche, RAM-Cache für heiße Chunks). Puffer werden wiederverwendet, der neue Zähler **„Iterationen gesamt“** speichert deinen Gesamtfortschritt über alle Sitzungen hinweg – auch nach App-Neustart.
- **Speicher-Monitor**: Im Tab *Datensatz* zeigt „Gerätespeicher“ jetzt belegte GB + freien Anteil. Läuft der Speicher voll (> 92 %), stoppt die Aufnahme sauber mit Hinweis – nichts geht kaputt.
- **Streaming-Export**: `.snnpack`-Export läuft jetzt als **Stream** direkt auf die Platte/Downloads – auch 70.000 Schritte exportieren, ohne den RAM zu füllen. Import ebenso (liest blockweise, erkennt v1.1/v1.2-Dateien automatisch).
- **Bugfix Import**: v1.1/v1.2 konnten Dateien mit mehr als einem Block beim Import **falsch einlesen** (fehlinterpretierte Blockgrenzen). v1.3 liest beide Altformate korrekt (Automatische Erkennung 512er/2048er-Blöcke) und schreibt ein sauberes v3-Format.
- **v1.2-Backup wird übernommen**: Beim ersten Start erscheint ein Banner, falls noch ein v1.2-Backup (alte „block_“-Sicherung) existiert – ein Tipp auf *Wiederherstellen* migriert es in den neuen Dauerspeicher und räumt die alte Kopie weg.

> ✅ **Update von v1.2**: gleicher Signaturschlüssel wie v1.2 – v1.3 lässt sich **direkt über die bestehende App installieren**, ohne Deinstallation. (Nur beim Sprung von v1.1 war Deinstallieren nötig.)

## Neu in v1.2

- **Auto-Backup**: Gewichte werden laufend im App-Speicher gesichert (alle 25 Iterationen beim Training, bei Stopp und Verlassen). Wiederherstellungs-Banner nach dem Start.
- **Absturzsicher**: Globale Fehler- und WebGL-Kontext-Überwachung; Fehler stoppen sauber mit Sicherung statt Absturz.
- **Lag behoben**: Puffer-Wiederverwendung im Training, kleinere Speicherblöcke, Kamera-Readback nur bei Bedarf.
- **Bugfix Gewichte-Export/Import** (Float32-Ausrichtung).

## Neu in v1.1

- **Stapeln funktioniert zuverlässig**: überarbeitete Greif-/Ablege-Logik; in Tests stapelt der Experte wiederholt alle 5 Würfel.
- **Neue Kameras**: zwei **feste externe Kameras** (Übersicht 92° + Nahaufnahme 60°) statt arm­montierter Kamera – der Greifer blockiert **nichts** mehr.
- **Farbe + höhere Auflösung**: 96×96 **RGB** pro Kamera (statt 64×64 Graustufen).

## 1. Installation (Samsung S24 Ultra / S26 Ultra)

1. APK auf das Handy übertragen (USB, Cloud, Bluetooth …)
2. APK antippen → Android fragt nach Erlaubnis für „Unbekannte Quellen“ → für deinen Dateimanager erlauben
3. Installieren – fertig. Keine Berechtigungen nötig (kein Internet, komplett offline)

## 2. So funktioniert der Ablauf

### Schritt 1: Datensatz sammeln
- App öffnen → **▶ Start** drücken.
- Der Skript-Experte arbeitet selbstständig: **kein Würfel sichtbar → Suchen**, **Würfel sichtbar → greifen & anheben**, dann **auf den Stapel legen**. Alle 5 Würfel gestapelt → neue Runde mit zufälligen Positionen.
- Die Aufnahme läuft **so lange, bis du ⏹ Stopp drückst** – jetzt **stundenlang möglich**: 70.000+ Schritte ≈ 3,9 GB auf dem Gerät. Alles ist laufend gesichert; du kannst die App jederzeit schließen, nach Neustart geht es automatisch weiter.
- **⚡ Paralleles Sammeln (neu in v1.4):** Statt Start einfach *Parallel sammeln* drücken – mehrere Worker (eigene Physik + Experte je Worker) füllen den Datensatz gleichzeitig, headless mit maximalem Tempo. Ideal für große Stufe-1-Datenmengen.
- **Gerätespeicher im Blick**: Tab *Datensatz* → „Gerätespeicher“ zeigt belegte GB und freien Anteil. Über 92 % Belegung stoppt die Aufnahme sauber.
- Tipp: Tempo auf **„Maximal“** → Simulation schneller als Echtzeit (je nach Gerät über 100×), Datensatz wächst entsprechend schneller. Zum Zusehen zurück auf „Echtzeit“.

### Schritt 2: SNN auf dem Handy trainieren (Stufe 1 – Video-Vortraining)
- Tab **SNN-Training**:
  - Lernrate 0.002, Batch 4, Sequenzlänge 12 sind gute Startwerte.
  - **Iterationen**: auch **1.000.000+ sind möglich** – der Fortschritt wird laufend gesichert (Gewichte alle 25 Iterationen) und der Zähler „Iterationen gesamt“ überlebt Neustarts. Für erste Erfolge: 2.000–20.000, dann steigern. Der Loss-Chart sollte fallen.
  - **⏬ Stufe 1 starten (Video-BC)** – nutzt WebGPU (bzw. WebGL/CPU-Fallback; aktives Backend oben im Kopf).
- Architektur (fix): LIF-Conv-SNN – 4×Conv+LIF (12/24/24/24 Kanäle, 6-Kanal-Farbinput) → Dense 128 (LIF) → 8 Ausgänge (tanh). ≈ 126.000 Parameter, Surrogat-Gradient (ATan-artig), BPTT, Adam.

### Schritt 2b: Stufe 2 – Roboter-Finetuning (RL, neu in v1.4)
- **🚀 Stufe 2 starten (RL)**: Das SNN übernimmt die Steuerung (mit σ-Rauschen zur Exploration), sammelt Belohnung (Greifen +1, Stapeln +3, Runde komplett +10, Fortschritts-Shaping) und lernt nach jeder Episode daraus (24 RL-Updates + wählbarer BC-Anteil aus Stufe 1 gegen Vergessen). σ nimmt pro Episode automatisch ab (0,96×).
- Empfehlung: zuerst ≥ 2.000–10.000 Schritte Stufe-1-Daten und ein paar tausend Iterationen, dann 10–50 RL-Episoden. Die Statuskarten zeigen Episode, Reward, beste Stapel und Updates.

### Schritt 3: Ausführen & Bewerten (Stufe 3 – ohne Datensammlung)
- **🎯 SNN ausführen (ohne Datensammlung)**: Deterministisch (kein Rauschen), Echtzeit, schreibt **nichts** in den Datensatz. Zählt Runden komplett gestapelt / gesamt → Erfolgsquote bleibt über Neustarts erhalten.
- **🧠 SNN in Sim testen** bleibt als schneller Test (mit Aufnahme, falls Haken gesetzt) – **Zurück zum Experten** wechselt jederzeit zurück.
- Nach mehr Daten + Stufe-2-Finetuning wird das SNN präziser – simply wiederholen (sammeln → Stufe 1 → Stufe 2 → Stufe 3).

### Export / Import
- **Datensatz exportieren** → `.snnpack` (gzip) landet als Stream in *Downloads* – auch mehrere GB ohne RAM-Probleme. Import fügt Daten hinzu und erkennt auch v1.1/v1.2-Dateien.
- **Gewichte** exportieren/importieren als `.snnweights` – z. B. S24 Ultra ↔ S26 Ultra. Zusätzlich automatisch im App-Backup gesichert.

## 3. Bedienoberfläche

| Element | Bedeutung |
|---|---|
| ▶ Start / ⏹ Stopp | Simulation + Aufnahme gemeinsam (Stopp beendet auch Stufen & Worker) |
| ⟲ Reset | Aktuelle Runde neu würfeln (Würfel neu platzieren) |
| Tempo | Echtzeit / 2× / 4× / 8× / 16× / Maximal (Physik + SNN synchron beschleunigt) |
| Politik | Skript-Experte oder SNN |
| ⚡ Parallel sammeln | Worker-Anzahl (Auto = 3 auf 8-Kern-Geräten) + Start/Stopp, live Schritte/s |
| Stufe 1/2/3 | Video-BC → RL-Finetuning → präzise Ausführung; erreichte Stufe wird gespeichert |
| PiP-Bilder | Übersicht (92°) + Nahaufnahme (60°), fest montiert, 96×96 Farbe – genau das, was das SNN sieht |
| „Gerätespeicher“ | Belegter Speicher + freier Anteil (Datensatz-Tab) |
| „Iterationen gesamt“ | Kumulierter Trainingsfortschritt über alle Sitzungen (SNN-Tab) |
| Chips oben | Sim-Status, Aufnahme-Aktiv, TF.js-Backend |

## 4. Technik & GEN-1.5-Anlehnung

- **Simulation:** MuJoCo (aktueller Main-Stand), als Single-Thread-WebAssembly selbst gebaut, läuft in der WebView offline
- **Roboter:** Franka Emika Panda (MuJoCo Menagerie), Positionsservos; die Politik gibt normalisierte **Geschwindigkeits-/PWM-äquivalente Befehle** (−1…+1) aus, die zu Ziellagen integriert werden
- **Kameras:** zwei feste externe Kameras (nicht am Arm!) – Übersicht (92°) + Nahaufnahme (60°), Offscreen-Rendering 96×96 **Farbe (RGB)** mit Three.js
- **Datensatz-Speicher (neu in v1.3):** Chunk-Speicher auf IndexedDB – 256 Schritte pro Chunk (~14 MB), Bilder + Posen getrennt von den schlanken Trainings-Metadaten; LRU-Cache (~192 MB) für schnelle Trainingsbatches; `navigator.storage`-Quota-Überwachung
- **IK:** Resolved-Rate (Jacobian + gedämpfte kleinste Quadrate + Nullraum-Haltung), phasenweise Stellgeschwindigkeits-Begrenzung, Gripper-Yaw-Anpassung an die Würfel-Ausrichtung
- **SNN:** Input nur Kameras → Output Motoren, wie GEN-1.5 (Generalist AI) – nur eben als spikendes Netz und direkt on-device trainiert. **Stufen wie das Vorbild:** Gen 1 = Behavior Cloning auf Demonstrationen (unsere Stufe 1); Gen 1.5 = RL-Post-Training auf dem BC-Fundament mit Hybrid-Loss (unsere Stufe 2: Advantage-gewichtete Regression auf eigenen Rollouts + BC-Anker); Stufe 3 = Bewertung/Ausführung. Vereinfachungen: kein Aktions-Chunking, kein Multi-Embodiment, kleinere Netze – aber dieselbe Reihenfolge.
- **Paralleles Sammeln (neu in v1.4):** Web-Worker (Modul-Worker) mit je eigener MuJoCo-WASM-Instanz + OffscreenCanvas-Kameras + Experte; Zero-Copy-Transfers (ArrayBuffer) zurück in den Haupt-Thread; Backpressure über die Schreib-Warteschlange; Speicher-Quota-Überwachung.

## 5. Selbst bauen / ändern

- Die App ist eine WebView-Hülle (Android) um die Web-App in `assets/www` (HTML + JS, unminifiziert lesbar als `assets/app.js`). Einfachste Anpassung: APK entpacken, `assets/www` ändern, mit [apktool](https://apktool.org) neu bauen, mit `zipalign` + `apksigner` (Android Build-Tools) neu signieren.
- Signaturschlüssel: `v12-release.jks` (Alias `v12`, Passwort `SNNtrainer2026` – für eigene Builds bitte ersetzen!). Wer Updates über bestehende Installationen installieren will, sollte ihn sichern.
- Benötigt: JDK 17+, apktool 2.10, Android Build-Tools 34

## 6. Grenzen & Tipps

- Das SNN sieht **nur** die beiden festen Kameras (96×96 RGB) – über Farbe sind die Würfel unterscheidbar, Positionen & Bewegungen werden aus zwei Blickwinkeln gelernt.
- **Speicher voll?** („Gerätespeicher“-Anzeige) → Datensatz exportieren und leeren, dann weiter sammeln. Die Aufnahme stoppt bei > 92 % Belegung automatisch sauber.
- Erste SNN-Erfolge brauchen echte Datenmenge + Iterationen – am besten lang sammeln (mehrere 10.000 Schritte) und dann lange trainieren (Zehntausende bis Millionen Iterationen).
- Lange Trainingsläufe: Gerät ans Ladegerät und Display anlassen (Android pausiert Hintergrund-Apps) – der Fortschritt ist trotzdem laufend gesichert.
- Performance: auf dem S24U/S26U läuft die Physik in Echtzeit; „Maximal“ entkoppelt von der Anzeige. Das Training nutzt WebGPU, wo verfügbar.
