# SNN Roboter-Trainer – Anleitung

**MuJoCo-WASM · Franka Emika Panda · Spiking Neural Network · GEN-1.5-inspiriert (vereinfacht)**

Deine App: `SNN-Roboter-Trainer-v1.0.apk` (8,6 MB, signiert, läuft komplett offline auf dem Gerät)

---

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
- Tipp: Im Tab *Steuerung* die Zeitlupe auf **„Maximal"** stellen → die Simulation rendert schneller als Echtzeit, der Datensatz wächst schneller. Zum Zusehen zurück auf „Echtzeit".

### Schritt 2: SNN auf dem Handy trainieren
- Tab **SNN-Training**:
  - Lernrate 0.002, Batch 4, Sequenzlänge 12 sind gute Startwerte.
  - **Iterationen**: 500–2000 für erste Erfolge (je nach Datenmenge). Der Loss-Chart sollte fallen.
  - **⏬ Training starten** – das Training nutzt WebGPU (bzw. fällt auf WebGL/CPU zurück; das aktive Backend steht oben im Kopf).
- Architektur (fix): LIF-Conv-SNN – 4×Conv+LIF (12/24/24/24 Kanäle) → Dense 128 (LIF) → 8 Ausgänge mit tanh. ≈ 65.000 Parameter, Surrogat-Gradient (ATan-artig), BPTT über die Zeit, Adam.

### Schritt 3: SNN testen (wie GEN-1.5)
- **🧠 SNN in Sim testen** – ab jetzt steuert **nur noch das Netz** den Roboter: Input = ausschließlich die beiden Kameras (Greifer-Kamera + Weitwinkel, 64×64 Graustufen), Output = 8 PWM-artige Motorbefehle (7 Gelenkgeschwindigkeiten + Greifer).
- **Zurück zum Experten** wechselt jederzeit zurück.
- Nach mehr Trainingsdaten + Iterationen wird das SNN besser – einfach öfter wiederholen (Daten sammeln → trainieren → testen).

### Export / Import
- **Datensatz exportieren** → `.snnpack`-Datei (gzip) landet in *Downloads*. Import fügt Daten hinzu (z. B. Sitzungen vom PC übertragen).
- **Gewichte** exportieren/importieren als `.snnweights` – so nimmst du trainierte Netze zwischen Geräten mit (S24 Ultra ↔ S26 Ultra).

## 3. Bedienoberfläche

| Element | Bedeutung |
|---|---|
| ▶ Start / ⏹ Stopp | Simulation + Aufnahme gemeinsam |
| ⟲ Reset | Aktuelle Runde neu würfeln (Würfel neu platzieren) |
| Zeitlupe | Echtzeit / 2× / 4× / Maximal (Sammeltempo) |
| Politik | Skript-Experte oder SNN |
| PiP-Bilder | Greifer-Kamera (68°) + Weitwinkel (118°) – genau das, was das SNN sieht |
| Chips oben | Sim-Status, Aufnahme-Aktiv, TF.js-Backend |

## 4. Technik & GEN-1.5-Anlehnung

- **Simulation:** MuJoCo (aktueller Main-Stand), als Single-Thread-WebAssembly selbst gebaut, läuft in der WebView offline
- **Roboter:** Franka Emika Panda (MuJoCo Menagerie), Positionsservos; die Politik gibt normalisierte **Geschwindigkeits-/PWM-äquivalente Befehle** (−1…+1) aus, die zu Ziellagen integriert werden
- **Kameras:** über dem Greifer montiert (11,5 cm / 15 cm hinter dem Handursprung), Offscreen-Rendering 64×64 Graustufen mit Three.js
- **IK:** Resolved-Rate (Jacobian + gedämpfte kleinste Quadrate + Nullraum-Haltung)
- **SNN:** Input nur Kameras → Output Motoren, wie GEN-1.5 (Generalist AI) – nur eben als spikendes Netz und direkt on-device trainiert (Verhaltens-Klonen des Experten statt 500.000 h Real-Daten)

## 5. Selbst bauen / ändern

```bash
# Web-App bauen (Projekt liegt in robotapp/)
cd robotapp && npm install && npx vite build

# APK bauen (Projekt liegt in android/, Assets werden automatisch synchronisiert)
cd ../android && ./gradlew assembleRelease
# → app/build/outputs/apk/release/app-release.apk
```
- Signatur: `android/app/release.keystore` (Alias `snn`, Passwort `snn2026` – für eigene Builds bitte ersetzen!)
- Benötigt: Android SDK (Plattform 35), JDK 17+

## 6. Grenzen & Tipps

- Das SNN sieht **nur** die Kameras – Farbwahl der Würfel ist irrelevant, das Netz lernt Positionen & Bewegungen.
- Bei Speicherwarnung: Datensatz exportieren und leeren, dann weiter sammeln (Limit ≈ 420 MB).
- Erste SNN-Erfolge brauchen echte Datenmenge + Iterationen – der kurze Smoke-Test im Auslieferungszustand war nur ein Funktionstest.
- Performance: auf dem S24U/S26U läuft die Physik in Echtzeit; „Maximal" entkoppelt von der Anzeige.
