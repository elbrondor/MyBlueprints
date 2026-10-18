# ClimateWindow

`ClimateWindow.yaml` ist ein Home-Assistant-Blueprint zur
feuchtegefuehrten Steuerung motorisierter Kellerfenster.

**Aktuelle Blueprint-Version: 1.0.0**

## Voraussetzungen

- Home Assistant 2026.7.0 oder neuer
- Ein motorisiertes Fenster als `cover`-Entitaet
- Sensoren fuer absolute Luftfeuchtigkeit innen und aussen in `g/m3`
- Ein Sensor fuer die relative Raumfeuchte
- Ein Sensor fuer die Aussentemperatur

## Funktionsweise

Das Fenster wird nur geoeffnet, wenn alle folgenden Bedingungen erfuellt sind:

- Die relative Raumfeuchte liegt mindestens an der Oeffnungsgrenze.
- Die absolute Luftfeuchtigkeit innen ist um den konfigurierten Mindestwert
  hoeher als aussen.
- Die Aussentemperatur liegt innerhalb des erlaubten Bereichs.
- Weder Regenschutz noch Sperr-Entitaet sind aktiv.
- Alle benoetigten Sensorwerte sind gueltig.

Das Fenster wird geschlossen, sobald mindestens eine der folgenden
Bedingungen zutrifft:

- Die relative Raumfeuchte erreicht die Schliessgrenze.
- Der Vorteil der trockeneren Aussenluft wird zu klein.
- Die Aussentemperatur verlaesst den erlaubten Bereich.
- Ein Regensensor meldet Regen.
- Eine aktivierte Sperr-Entitaet meldet `on`.
- Ein benoetigter Sensor liefert keinen gueltigen Zahlenwert.

Zwischen Oeffnungs- und Schliessgrenzen bleibt der aktuelle Fensterzustand
unveraendert. Diese Hysterese verhindert haeufige Richtungswechsel.

## Eingaben

### Fenster und Sensoren

| Einstellung | Erforderlich | Beschreibung |
|---|---|---|
| Fensteroeffner | ja | Zu steuernde `cover`-Entitaet |
| Absolute Luftfeuchtigkeit innen | ja | Raumwert in `g/m3` |
| Absolute Luftfeuchtigkeit aussen | ja | Aussenwert in `g/m3` |
| Relative Luftfeuchtigkeit innen | ja | Raumfeuchte in Prozent |
| Aussentemperatur | ja | Temperatur der Aussenluft |

### Schutz und Sperren

| Einstellung | Standard | Beschreibung |
|---|---|---|
| Regenschutz aktivieren | an | Aktiviert die Auswertung der Regensensoren |
| Regensensoren | leer | Ein oder mehrere `binary_sensor` mit `on` bei Regen |
| Sperr-Entitaet beruecksichtigen | aus | Aktiviert die Sperrfunktion |
| Sperr-Entitaet | leer | Entitaeten, die bei `on` das Fenster schliessen |

Die Sperr-Entitaet ist bewusst allgemein gehalten. Fuer den Waschkeller
koennen beispielsweise `switch.shelly63` fuer den Trockner und
`input_boolean.abwesend_urlaub` gemeinsam ausgewaehlt werden. Ist die Option
deaktiviert, werden alle ausgewaehlten Sperr-Entitaeten ignoriert.

### Lueftungsgrenzen

| Einstellung | Standard | Beschreibung |
|---|---:|---|
| Raumfeuchte zum Oeffnen | 60 % | Ab diesem Wert darf geoeffnet werden |
| Raumfeuchte zum Schliessen | 55 % | Ab diesem Wert oder darunter wird geschlossen |
| Mindestvorteil zum Oeffnen | 0,3 g/m3 | Erforderlicher Unterschied innen minus aussen |
| Restvorteil zum Schliessen | 0,1 g/m3 | Bei diesem Unterschied oder darunter wird geschlossen |
| Minimale Aussentemperatur | 5 degC | Unterhalb bleibt das Fenster geschlossen |
| Maximale Aussentemperatur | 25 degC | Oberhalb bleibt das Fenster geschlossen |

Die Schliessgrenze der Raumfeuchte sollte kleiner als die Oeffnungsgrenze
sein. Ebenso sollte der Restvorteil zum Schliessen kleiner als der
Mindestvorteil zum Oeffnen sein.

## Auswertung

Die Automation bewertet die Bedingungen:

- bei jeder Aenderung eines konfigurierten Sensors,
- bei jeder Aenderung eines Regen- oder Sperrsensors,
- beim Start von Home Assistant,
- sowie alle fuenf Minuten als Sicherheitspruefung.

## Hinweise zur Migration

Der Blueprint verwendet keinen festen Grenzwert fuer die absolute
Aussenluftfeuchtigkeit. Ein Wert wie `4,5 g/m3` passt nicht zu korrekt
berechneten Absolutfeuchtewerten und wuerde das Oeffnen unter normalen
Bedingungen fast immer blockieren.

Eine zu niedrige relative Raumfeuchte loest ebenfalls kein Oeffnen aus. Mit
trockenerer Aussenluft wuerde das den Raum nur weiter austrocknen.

Fuer die vorhandenen Automationen sind typischerweise folgende Zuordnungen
geeignet:

| Bereich | Fenster | Innen absolut | Relative Feuchte |
|---|---|---|---|
| Waschkeller | `cover.shelly28` | `sensor.absolute_luftfeuchtigkeit_waschkeller` | `sensor.bttemp04_humidity` |
| Keller 2 links | `cover.shelly50` | `sensor.absolute_luftfeuchtigkeit_keller_2` | `sensor.bttemp09_luftfeuchtigkeit` |
| Keller 2 rechts | `cover.shelly48` | `sensor.absolute_luftfeuchtigkeit_keller_2` | `sensor.bttemp09_luftfeuchtigkeit` |
| Werkstatt | `cover.shelly22` | `sensor.absolute_luftfeuchtigkeit_werkstatt` | `sensor.bttemp07_luftfeuchtigkeit` |

Als gemeinsamer Aussensensor kann
`sensor.absolute_luftfeuchtigkeit_aussen` verwendet werden.

## Installation

Datei in das Blueprint-Verzeichnis kopieren, zum Beispiel:

```text
config/blueprints/automation/elbrondor/ClimateWindow.yaml
```

Danach die Blueprints in Home Assistant neu laden und fuer jedes Fenster eine
eigene Automation aus dem Blueprint erzeugen.

[Zurueck zur Uebersicht](README.md)
