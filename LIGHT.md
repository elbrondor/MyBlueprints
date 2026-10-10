# Automatic Light

`AutomaticLight.yaml` ist ein Home-Assistant-Blueprint zur automatischen Steuerung von Licht- und Schalter-Entitaeten anhand aller Bewegungs- und Belegungssensoren eines Bereichs.

**Aktuelle Blueprint-Version: 1.1.0**

## Voraussetzungen

- Home Assistant 2026.7.0 oder neuer
- Mindestens ein Bewegungs- oder Belegungssensor im ausgewaehlten Bereich
- Mindestens eine zu steuernde `light`- oder `switch`-Entitaet
- Korrekte Bereichszuordnung der Sensoren in Home Assistant

Der Blueprint verwendet die Home-Assistant-Geraeteklassen `motion` und `occupancy`. Sensoren ohne passende Geraeteklasse werden nicht als Aktivitaetssensoren erkannt.

## Funktionen

- Einschalten bei Bewegung oder Belegung
- Gemeinsame Auswertung aller passenden Sensoren eines Bereichs
- Konfigurierbare Nachleuchtzeit nach Ende der gesamten Aktivitaet
- Steuerung mehrerer Licht- und Schalter-Entitaeten
- Optionale Sperrzeiten ueber einen Zeitplan
- Sperre des automatischen Einschaltens bei Abwesenheit oder Urlaub
- Optionale Dunkelheitsbedingung
- Zeitlich begrenzte manuelle Uebersteuerung ueber lokale Schalteingaenge
- Periodische Sicherheitspruefung fuer manuell eingeschaltete Leuchten

## Sensorlogik

Bewegungs- und Belegungssensoren sind gleichwertige Aktivitaetsquellen:

- Meldet mindestens ein Bewegungs- oder Belegungssensor Aktivitaet, darf das Licht automatisch eingeschaltet werden.
- Solange mindestens ein Sensor Bewegung oder Belegung meldet, bleibt das Licht eingeschaltet.
- Die Nachleuchtzeit beginnt erst, wenn alle Bewegungs- und alle Belegungssensoren im Bereich inaktiv sind.
- Erneute Bewegung oder Belegung setzt die Nachleuchtzeit zurueck.
- Das Licht wird erst ausgeschaltet, wenn beide Sensorgruppen fuer die vollstaendige Nachleuchtzeit inaktiv waren.

Bereiche duerfen nur Bewegungsmelder, nur Belegungssensoren oder beide Sensortypen enthalten.

## Eingaben

### Bereich und Lichtaktoren

| Einstellung | Erforderlich | Standard | Beschreibung |
|---|---|---|---|
| Bereich fuer Bewegung und Belegung | ja | - | Bereich, dessen Bewegungs- und Belegungssensoren ausgewertet werden |
| Zu schaltende Entitaeten | ja | - | Eine oder mehrere `light`- oder `switch`-Entitaeten |

### Einschaltbedingungen

| Einstellung | Erforderlich | Standard | Beschreibung |
|---|---|---|---|
| Sperr-Entitaet | nein | `input_boolean.abwesend_urlaub` | Bei Zustand `on` wird nicht automatisch eingeschaltet |
| Sperrzeiten | nein | leer | Ein aktiver `schedule` verhindert das automatische Einschalten |
| Helligkeit aussen (IsDark) | nein | leer | Wenn gesetzt, muss der binaere Sensor den Zustand `on` besitzen |

Die Einschaltbedingungen sperren ausschliesslich das automatische Einschalten. Das automatische Ausschalten bleibt aktiv.

### Manuelle Uebersteuerung

| Einstellung | Erforderlich | Standard | Beschreibung |
|---|---|---|---|
| Lokale Lichtschalter | nein | leer | Binaere Schalteingaenge, deren Zustandsaenderung als manuelle Bedienung gilt |
| Dauer der Uebersteuerung | nein | 15 min | Zeitraum, in dem der aktuelle Lichtzustand nicht automatisch geaendert wird |

Jede Zustandsaenderung eines ausgewaehlten Schalteingangs startet die Uebersteuerungszeit neu. Waehrend dieser Zeit bleibt der manuell gewaehlte Zustand erhalten.

### Ausschaltverhalten

| Einstellung | Erforderlich | Standard | Beschreibung |
|---|---|---|---|
| Nachleuchtzeit | nein | 5 min | Dauer, fuer die alle Bewegungs- und Belegungssensoren inaktiv sein muessen |

## Ablauf und Prioritaeten

Die Steuerung verarbeitet Ereignisse in dieser Reihenfolge:

1. Eine aktive manuelle Uebersteuerung behaelt den aktuellen Lichtzustand bei.
2. Bewegung oder Belegung kann das Licht einschalten, sofern keine Einschaltbedingung sperrt.
3. Das Ende einer Sensorgruppe allein schaltet das Licht nicht aus, solange die andere Gruppe noch Aktivitaet meldet.
4. Erst nach gemeinsamer Inaktivitaet und abgelaufener Nachleuchtzeit wird ausgeschaltet.
5. Eine Sicherheitspruefung alle 10 Minuten schaltet ebenfalls nur aus, wenn alle Sensoren bereits fuer die vollstaendige Nachleuchtzeit inaktiv waren.

## Dunkelheit und Sperren

Die optionale Dunkelheitsbedingung wird nur beim automatischen Einschalten ausgewertet:

- Leeres Feld: Einschalten ist unabhaengig von der Helligkeit erlaubt.
- Sensor `on`: Dunkelheit ist erfuellt.
- Sensor `off`, `unknown` oder `unavailable`: Es wird nicht automatisch eingeschaltet.

Dasselbe Prinzip gilt fuer Sperr-Entitaet und Sperrzeit. Ein bereits eingeschaltetes Licht kann trotz aktiver Sperre nach Ende der Aktivitaet ausgeschaltet werden.

## Periodische Sicherheitspruefung

Der Blueprint prueft alle 10 Minuten, ob die gesteuerten Entitaeten wegen ausbleibender Aktivitaet ausgeschaltet werden duerfen. Dabei gilt dieselbe Nachleuchtzeit wie beim regulaeren Ausschalten. Die Sicherheitspruefung kann die konfigurierte Nachleuchtzeit daher nicht verkuerzen.

Diese Pruefung erfasst insbesondere Leuchten, die manuell eingeschaltet wurden, ohne dass danach ein passendes Ende-von-Aktivitaet-Ereignis ausgeloest wurde.

## Hinweise zur Bereichszuordnung

Alle relevanten Bewegungs- und Belegungssensoren muessen in Home Assistant demselben Bereich zugeordnet sein, der in der Blueprint-Automation ausgewaehlt wurde. Wird ein Sensor spaeter diesem Bereich hinzugefuegt oder daraus entfernt, folgt die Automation automatisch der neuen Bereichszuordnung.

Auch Sensoren, die nicht fuer diese konkrete Leuchte gedacht sind, beeinflussen die Automation, wenn sie im ausgewaehlten Bereich die Geraeteklasse `motion` oder `occupancy` besitzen.

## Aenderungen in Version 1.1.0

- Bewegungs- und Belegungssensoren werden immer gemeinsam ausgewertet.
- Die Nachleuchtzeit beginnt erst nach vollstaendiger Inaktivitaet beider Sensorgruppen.
- Die Sicherheitspruefung verwendet ebenfalls die volle Nachleuchtzeit.
- Der wirkungslose Eingang `use_occupancy` wurde entfernt.
- Der separate Eingang `safety_no_motion` wurde entfernt.

Bei einer bestehenden Automation aus Version 1.0.1 muessen gespeicherte Werte fuer `use_occupancy` und `safety_no_motion` vor oder waehrend der Aktualisierung entfernt werden, falls Home Assistant sie nicht automatisch bereinigt.

## Installation

Datei in das Blueprint-Verzeichnis kopieren, zum Beispiel:

```text
config/blueprints/automation/elbrondor/AutomaticLight.yaml
```

Danach in Home Assistant die Automationen/Blueprints neu laden oder Home Assistant neu starten. Anschliessend unter **Einstellungen -> Automationen & Szenen -> Blueprints** aus dem Blueprint eine eigene Automation pro Raum bzw. Lichtgruppe erzeugen.

[Zurueck zur Uebersicht](README.md)
