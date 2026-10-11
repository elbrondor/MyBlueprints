# Home Assistant Blueprints

Dieses Repository enthält mehrere Home-Assistant-Blueprints für die automatische Steuerung von Licht, Rollläden, Klimaanlagen und Amazon-Echo-Wiedergabe.

## Blueprints

### Automatisches Licht mit Sperrzeiten

Lichtsteuerung über alle Bewegungs- und Belegungssensoren eines Bereichs. Die gemeinsame Nachleuchtzeit startet erst, wenn sämtliche Aktivität beendet ist; optionale Sperrzeiten, Abwesenheit, Dunkelheitsbedingung und manuelle Übersteuerung werden unterstützt.

- [Dokumentation zur Lichtsteuerung](LIGHT.md)
- [Blueprint: AutomaticLight.yaml](AutomaticLight.yaml)

### 🪟 Shutter Cover Automation

Automatische Rollladensteuerung mit Sonnenaufgang/Sonnenuntergang, Fenster- und Aussperrschutz, Regenschutz, manueller Übersteuerung und Globalbeschattung.

- [Dokumentation zur Coversteuerung](COVER.md)
- [Blueprint: ShutterCoverAutomation.yaml](ShutterCoverAutomation.yaml)

### ❄️ Climate Automation

Automatische Steuerung einer Klimaanlage pro Raum mit Temperaturstufen, Fenster-/Rolloschutz, Abwesenheits- und Gästemodus sowie manueller Übersteuerung.

- [Dokumentation zur Klimaanlagensteuerung](CLIMATE.md)
- [Blueprint: ClimateAutomation.yaml](ClimateAutomation.yaml)

### Amazon Echo Automation

Startet bei Aktivität einen Radiosender auf einem Amazon Echo, wechselt bei konfigurierbaren unerwünschten Programmen zu einer Alternative und stoppt die Wiedergabe nach anhaltender Inaktivität.

- [Dokumentation zur Echo-Steuerung](ECHO.md)
- [Blueprint: echo_automation.yaml](echo_automation.yaml)

## Installation

Die jeweilige YAML-Datei kann in Home Assistant als Blueprint importiert oder in das lokale Blueprint-Verzeichnis kopiert werden. Weitere Hinweise und die jeweiligen Funktionen stehen in den getrennten Dokumentationen.
