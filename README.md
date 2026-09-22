# SNN Roboter-Trainer – Anleitung

**MuJoCo-WASM · Franka Emika Panda · Spiking Neural Network · GEN-1.5-inspiriert (vereinfacht)**

Deine App: `SNN-Roboter-Trainer-v1.2.apk` (9,0 MB, signiert, läuft komplett offline auf dem Gerät)

---

## ⭐ Neu in v1.2 – Absturzsicherung, Auto-Backup, mehr Tempo

- **Auto-Backup (das Wichtigste)**: Datensatz **und** trainierte Gewichte werden jetzt automatisch im App-Speicher (IndexedDB) gesichert – fortlaufend während der Aufnahme, alle 25 Iterationen während des Trainings, beim Stopp und beim Verlassen der App. Nach einem Absturz, Hintergrund-Kill oder Neustart erscheint oben ein Banner **„Backup gefunden … wiederherstellen?“** – ein Tipp und alles ist zurück.
- **Absturzsicher**: Globale Fehler- und WebGL-Kontext-Überwachung. Grafik- oder Simulationsfehler beenden die App nicht mehr, sondern stoppen sauber, sichern das Backup und machen weiter. Beim Verlust des Grafikkontexts (typisch bei Speicherdruck) wird neu geladen – ohne Datenverlust.
- **Lag behoben**: Die Ursachen für das Ruckeln nach langen Aufnahmen/Trainings sind beseitigt – kleinere Speicherblöcke beim Aufnehmen (keine 113-MB-Allokationsspitzen mehr), **Puffer-Wiederverwendung im Training** (vorher ~10 MB neue Arrays pro Iteration → Garbage-Collector-Churn), Kamerabilder werden nur noch gelesen, wenn sie gebraucht werden.
- **RAM-Limit gesenkt** (420 → 320 MB): stabiler auf dem Handy; mit Auto-Backup geht trotzdem nichts mehr verloren.
- **Bugfix Gewichte-Export/Import**: In v1.1 konnte das Laden exportierter `.snnweights`-Dateien an einer Byte-Ausrichtung scheitern („start offset of Float32Array …“). Behoben – gilt auch für alte Dateien.

> ⚠️ **Wichtig beim Update von v1.1**: v1.2 ist mit einem neuen Signaturschlüssel signiert. Android verweigert das Überschreiben – bitte **einmalig die alte App deinstallieren**, dann v1.2 installieren. (Alte Daten sind davon nicht betroffen – v1.1 hatte noch kein Backup, v1.2 legt jetzt eigenes an.)

## Neu in v1.1

- **Stapeln funktioniert jetzt zuverlässig**: überarbeitete Greif-/Ablege-Logik (Finger-Stabilitätsprüfung, verstärkter Greifer-Servo, Yaw-Ausrichtung, kettenbasiertes Stapeln auf der gemessenen Turmspitze, kraftfreies Aufsetzen). In automatisierten Tests stapelt der Experte wiederholt alle 5 Würfel.
- **Neue Kameras**: zwei **feste externe Kameras** (Übersicht 92° + Nahaufnahme 60°) statt arm­montierter Kamera – der Greifer blockiert **nichts** mehr.
- **Farbe + höhere Auflösung**: 96×96 **RGB** pro Kamera (statt 64×64 Graustufen).
- **Datensatz v2**: RGB-Format. Alte v1-Datensätze (Graustufen) werden beim Import abgelehmt – bitte neu aufnehmen.

## 1. Installation (Samsung S24 Ultra / S26 Ultra)

1. APK auf das Handy übertragen (USB, Cloud, Bluetooth …)
2. APK antippen → Android fragt nach Erlaubnis für „Unbekannte Quellen" → für deinen Dateimanager erlauben
3. Installieren – fertig. Keine Berechtigungen nötig (kein Internet, komplett offline)

## 2. So funktioniert der Ablauf

### Schritt 1: Datensatz sammeln
- App öffnen → **▶ Start** drücken.
- Der Skript-Experte arbeitet jetzt selbstständig: **kein Würfel sichtbar → Suchen** (Kamerafahrt), **Würfel sichtbar → greifen & anheben**, dann **auf den Stapel legen**. Alle 5 Würfel gestapelt → neue Runde mit zufälligen Positionen.
- Die Aufnahme läuft **so lange, bis du ⏹ Stopp drückst** (Kontrollkästchen „Aufnahme" im Tab *Datensatz*).
- Faustregel für einfache Qualität: **10–30 Minuten** Sammeln (ca. 12.000–36.000 Schritte ≈ 100–290 MB im RAM). Der Fortschritt steht im Tab *Datensatz*.
- **Auto-Backup läuft mit**: Im Tab *Datensatz* unter „Sicherung (Auto-Backup)“ siehst du den Stand des letzten Backups. Auf Nahme und Training sind automatisch gesichert – du kannst die App jederzeit schließen, ein Absturz verliert nichts mehr. „💾 Jetzt sichern“ erzwingt einen manuellen Sicherungspunkt, „🗑 Backup löschen“ räumt den App-Speicher frei.
- Tipp: Im Tab *Steuerung* die Zeitlupe auf **„Maximal"** stellen → die Simulation rendert schneller als Echtzeit, der Datensatz wächst schneller. Zum Zusehen zurück auf „Echtzeit".

### Schritt 2: SNN auf dem Handy trainieren
- Tab **SNN-Training**:
  - Lernrate 0.002, Batch 4, Sequenzlänge 12 sind gute Startwerte.
  - **Iterationen**: 500–2000 für erste Erfolge (je nach Datenmenge). Der Loss-Chart sollte fallen.
  - **⏬ Training starten** – das Training nutzt WebGPU (bzw. fällt auf WebGL/CPU zurück; das aktive Backend steht oben im Kopf).
- Architektur (fix): LIF-Conv-SNN – 4×Conv+LIF (12/24/24/24 Kanäle, 6-Kanal-Farbinput) → Dense 128 (LIF) → 8 Ausgänge mit tanh. ≈ 126.000 Parameter, Surrogat-Gradient (ATan-artig), BPTT über die Zeit, Adam.

### Schritt 3: SNN testen (wie GEN-1.5)
- **🧠 SNN in Sim testen** – ab jetzt steuert **nur noch das Netz** den Roboter: Input = ausschließlich die beiden festen Kameras (Übersicht + Nahaufnahme, 96×96 **Farbe**), Output = 8 PWM-artige Motorbefehle (7 Gelenkgeschwindigkeiten + Greifer).
- **Zurück zum Experten** wechselt jederzeit zurück.
- Nach mehr Trainingsdaten + Iterationen wird das SNN besser – einfach öfter wiederholen (Daten sammeln → trainieren → testen).

### Export / Import
- **Datensatz exportieren** → `.snnpack`-Datei (gzip) landet in *Downloads*. Import fügt Daten hinzu (z. B. Sitzungen vom PC übertragen).
- **Gewichte** exportieren/importieren als `.snnweights` – so nimmst du trainierte Netze zwischen Geräten mit (S24 Ultra ↔ S26 Ultra). In v1.2 zusätzlich automatisch im App-Backup gesichert (Checkpoint alle 25 Iterationen, nach Abschluss und bei Fehlern).

## 3. Bedienoberfläche

| Element | Bedeutung |
|---|---|
| ▶ Start / ⏹ Stopp | Simulation + Aufnahme gemeinsam |
| ⟲ Reset | Aktuelle Runde neu würfeln (Würfel neu platzieren) |
| Zeitlupe | Echtzeit / 2× / 4× / Maximal (Sammeltempo) |
| Politik | Skript-Experte oder SNN |
| PiP-Bilder | Übersicht (92°) + Nahaufnahme (60°), fest montiert, 96×96 Farbe – genau das, was das SNN sieht |
| Chips oben | Sim-Status, Aufnahme-Aktiv, TF.js-Backend |

## 4. Technik & GEN-1.5-Anlehnung

- **Simulation:** MuJoCo (aktueller Main-Stand), als Single-Thread-WebAssembly selbst gebaut, läuft in der WebView offline
- **Roboter:** Franka Emika Panda (MuJoCo Menagerie), Positionsservos; die Politik gibt normalisierte **Geschwindigkeits-/PWM-äquivalente Befehle** (−1…+1) aus, die zu Ziellagen integriert werden
- **Kameras:** zwei feste externe Kameras (nicht am Arm!) – Übersicht (92°) + Nahaufnahme (60°), Offscreen-Rendering 96×96 **Farbe (RGB)** mit Three.js
- **IK:** Resolved-Rate (Jacobian + gedämpfte kleinste Quadrate + Nullraum-Haltung), phasenweise Stellgeschwindigkeits-Begrenzung, Gripp-Yaw-Anpassung an die Würfel-Ausrichtung
- **SNN:** Input nur Kameras → Output Motoren, wie GEN-1.5 (Generalist AI) – nur eben als spikendes Netz und direkt on-device trainiert (Verhaltens-Klonen des Experten statt 500.000 h Real-Daten)

## 5. Selbst bauen / ändern

- Die App ist eine WebView-Hülle (Android) um die Web-App in `assets/www` (HTML + JS, unminifiziert lesbar als `assets/app.js`). Einfachste Anpassung: APK entpacken, `assets/www` ändern, mit [apktool](https://apktool.org) neu bauen, mit `zipalign` + `apksigner` (Android Build-Tools) neu signieren.
- v1.2-Signaturschlüssel: `v12-release.jks` (Alias `v12`, Passwort `SNNtrainer2026` – für eigene Builds bitte ersetzen!). Wer den Schlüssel behalten will, sollte ihn sichern – nur damit signierte Updates installierbar sind.
- Benötigt: JDK 17+, apktool 2.10, Android Build-Tools 34

## 6. Grenzen & Tipps

- Das SNN sieht **nur** die beiden festen Kameras (96×96 RGB) – über Farbe sind die Würfel jetzt unterscheidbar, Positionen & Bewegungen werden aus zwei Blickwinkeln gelernt.
- Bei Speicherwarnung: Datensatz exportieren und leeren, dann weiter sammeln (Limit ≈ 320 MB, Backup entfällt daraus nicht).
- Erste SNN-Erfolge brauchen echte Datenmenge + Iterationen – der kurze Smoke-Test im Auslieferungszustand war nur ein Funktionstest.
- Performance: auf dem S24U/S26U läuft die Physik in Echtzeit; „Maximal" entkoppelt von der Anzeige. v1.2 reduziert die Last zusätzlich (Kamera-Readback nur bei Bedarf, weniger Speicherallokationen).
