# Climate Automation

`ClimateAutomation.yml` ist ein Home-Assistant-Blueprint zur automatischen Steuerung einer Klimaanlage pro Raum.

## Funktionen

- Globaler Hauptschalter mit höchster Priorität
- 10 Minuten Freigabeverzögerung nach Einschalten des Hauptschalters
- 10 Minuten Schutzzeit nach Home-Assistant-Start
- Raumtemperaturabhängige Kühlung mit drei Lüfterstufen
- Konfigurierbare Solltemperatur
- Fenster-/Türschutz
- Rollo kann ein offenes Fenster freigeben, wenn `current_position` klein genug ist
- Abwesend-/Urlaubsmodus
- Gästemodus
- Manuelle Übersteuerung durch ausgewählte Automationen und Buttons/Events
- Übersteuerungsdauer konfigurierbar, Default 60 Minuten
- Mindestlaufzeit und Mindestausschaltzeit
- Außentemperatur verhindert unnötiges Einschalten
- Verzögertes Ausschalten bei dauerhaft niedriger Außentemperatur

## Standardwerte

| Einstellung | Default |
|---|---:|
| Solltemperatur | 21 °C |
| Temp0 / Ausschalten | 20 °C |
| Temp1 / LOW | 23 °C |
| Temp2 / MEDIUM | 25 °C |
| Temp3 / HIGH | 27 °C |
| Rollo gilt geschlossen bis | 5 % |
| Mindestlaufzeit | 10 min |
| Mindestausschaltzeit | 10 min |
| Außentemperatur-Ausschaltverzögerung | 3 h |
| Manuelle Übersteuerung | 60 min |
| Fan LOW | `low` |
| Fan MEDIUM | `medium` |
| Fan HIGH | `high` |

## Prioritäten

Die Regeln werden in dieser Reihenfolge behandelt:

1. Globaler Hauptschalter AUS → keine Aktion.
2. Abwesend/Urlaub → Klimaanlage AUS.
3. Fenster offen und Rollo nicht ausreichend geschlossen → Klimaanlage AUS.
4. Gästemodus → keine automatische Änderung.
5. Manuelle Übersteuerung → keine automatische Änderung.
6. Außentemperaturregeln.
7. Mindestlauf-/Mindestausschaltzeit.
8. Raumtemperatur und Lüfterstufen.

Der Fensterschutz bleibt auch während einer manuellen Übersteuerung aktiv.

## Manuelle Übersteuerung

Im einklappbaren Bereich **Übersteuerung** gibt es zwei optionale Listen:

- **Übersteuernde Automationen**: Home-Assistant-Automationen. Maßgeblich ist deren `last_triggered`.
- **Übersteuernde Buttons / Events**: `event`, `button` oder `input_button`.

Sind beide Listen leer, ist die Override-Prüfung wirkungslos. Nach einer manuellen Aktion greift die normale Temperatursteuerung für die konfigurierte Zeit nicht ein. Der Default beträgt 60 Minuten. Jede weitere manuelle Aktion startet die Zeit erneut.

## Fenster und Rollo

Ein Fenster-/Türsensor mit Zustand `on` gilt als offen. Ist mindestens ein Fenster offen, wird die Klimaanlage ausgeschaltet.

Ausnahme: Ein konfiguriertes Rollo hat `current_position` kleiner oder gleich dem eingestellten Grenzwert. Standardmäßig sind das 5 %.

| Fenster | Rollo | Klima erlaubt |
|---|---:|---|
| geschlossen | 100 % | ja |
| offen | 100 % | nein |
| offen | 6 % | nein |
| offen | 5 % | ja |
| offen | 0 % | ja |

## Temperatursteuerung

Bei den Standardwerten gilt:

- Raumtemperatur ≤ 20 °C → ausschalten, Mindestlaufzeit beachten.
- 20 °C < Raumtemperatur < 23 °C → aktuellen Zustand beibehalten.
- 23 °C bis < 25 °C → Kühlen, Soll 21 °C, LOW.
- 25 °C bis < 27 °C → Kühlen, Soll 21 °C, MEDIUM.
- ab 27 °C → Kühlen, Soll 21 °C, HIGH.

Die Fan-Mode-Namen sind frei konfigurierbar, damit unterschiedliche Climate-Integrationen unterstützt werden.

## Außentemperatur

Ist die Klimaanlage AUS und die Außentemperatur kleiner oder gleich der Solltemperatur, wird sie nicht automatisch eingeschaltet.

Läuft die Klimaanlage bereits und bleibt die Außentemperatur ununterbrochen für die konfigurierte Dauer kleiner oder gleich der Solltemperatur, wird die Anlage ausgeschaltet. Default: 3 Stunden.

> Hinweis: Home Assistants `for`-Zeit bei Triggern wird bei einem Home-Assistant-Neustart bzw. beim Neuladen der Automationen zurückgesetzt. Danach muss die Bedingung erneut für die vollständige konfigurierte Dauer erfüllt sein.

## Installation

Datei in das Blueprint-Verzeichnis kopieren, zum Beispiel:

```text
config/blueprints/automation/elbrondor/ClimateAutomation.yml
```

Danach in Home Assistant die Automationen/Blueprints neu laden oder Home Assistant neu starten. Anschließend unter **Einstellungen → Automationen & Szenen → Blueprints** aus dem Blueprint eine Automation pro Raum erzeugen.

## Hinweise

Die Mindestlauf- und Mindestausschaltzeit verwenden den Zeitpunkt der letzten Zustandsänderung der `climate`-Entität. Manuelle Änderungen des HVAC-Modus werden dadurch ebenfalls berücksichtigt.

Der Blueprint setzt für den Kühlbetrieb den HVAC-Modus `cool`. Die verwendete Climate-Integration muss diesen Modus sowie `climate.set_temperature` und `climate.set_fan_mode` unterstützen.

[← Zurück zur Übersicht](README.md)
