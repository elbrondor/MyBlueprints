# Shutter Cover Automation

`ShutterCoverAutomation.yml` ist ein Home-Assistant-Blueprint zur automatischen Steuerung von Rollläden.

**Aktuelle Blueprint-Version: 0.3.7**

## Funktionen

- Automatisches Öffnen bei Sonnenaufgang mit konfigurierbarem Offset
- Früheste Öffnungszeit getrennt für Arbeitstage und Wochenende/Feiertage
- Automatisches Schließen bei Sonnenuntergang mit Offset
- Workday-Unterstützung
- Fenster-/Türschutz gegen Aussperren
- Lüftungsschutz bei geöffnetem oder gekipptem Fenster
- Regenschutz bei geöffnetem/gekipptem Fenster
- Manuelle bzw. externe Übersteuerung
- Temperatur-, Raumtemperatur- und bewölkungsabhängige Globalbeschattung
- Schlafzimmer-Sonderfunktion bei frühem Sonnenaufgang
- Automatische Wiederholung fehlgeschlagener Fahrbefehle

## Cover-Position

Home Assistant verwendet:

- `0 %` = vollständig geschlossen
- `100 %` = vollständig geöffnet

## Prioritäten

Die Steuerung berücksichtigt die Funktionen in folgender Reihenfolge:

1. Aktiver Fenster-/Aussperrschutz
2. Regenschutz bei geöffnetem/gekipptem Fenster
3. Sonderfall Sonnenaufgang vor frühester Öffnungszeit
4. Manuelle/externe Übersteuerung
5. Globalbeschattung
6. Normaler Tages-/Nachtbetrieb

## Fenster- und Aussperrschutz

Es können sowohl einfache Zwei-Zustands-Fensterkontakte als auch Drei-Zustands-Sensoren für Fenstergriffe verwendet werden.

Bei Drei-Zustands-Sensoren werden die Zustände `open`, `tilted` und `closed` ausgewertet. Ein vollständig geöffnetes Fenster besitzt Aussperrschutz; bei gekipptem Fenster greift der konfigurierbare Lüftungsschutz.

Die maximale Schließung bei Lüftungsschutz ist einstellbar. Beispiel: **60 % geschlossen** entspricht einer Home-Assistant-Cover-Position von **40 %**.

## Regenschutz

Der Regenschutz kann optional aktiviert werden. Ein oder mehrere binäre Regensensoren können verwendet werden:

- `on` = Regen
- `off` = trocken

Bei aktivem Regenschutz und geöffnetem bzw. gekipptem Fenster wird eine konfigurierbare Rollladenposition angefahren. Der Aussperrschutz bei einem vollständig geöffneten Drei-Zustands-Fenstergriff besitzt höhere Priorität.

## Sonnenaufgang und Öffnen

Für Arbeitstage sowie Wochenenden/Feiertage kann jeweils eine früheste Öffnungszeit festgelegt werden. Zusätzlich kann der Sonnenaufgang mit einem positiven oder negativen Offset verschoben werden.

Die früheste Öffnungszeit wird dabei nicht unterschritten.

Beispiele für den Offset:

- `-00:30:00` = 30 Minuten vor Sonnenaufgang
- `00:00:00` = genau bei Sonnenaufgang
- `00:30:00` = 30 Minuten nach Sonnenaufgang

Für Schlafzimmer kann optional festgelegt werden, dass der Rollladen bei einem frühen Sonnenaufgang bis zur frühesten Öffnungszeit geschlossen bleibt.

## Sonnenuntergang

Auch das automatische Schließen bei Sonnenuntergang unterstützt einen Offset, beispielsweise:

- `-00:30:00` = 30 Minuten vorher
- `00:00:00` = genau bei Sonnenuntergang
- `00:30:00` = 30 Minuten danach

## Manuelle Übersteuerung

Im Bereich **Übersteuerung** können lokale Rollladenschalter sowie übersteuernde Home-Assistant-Automationen eingetragen werden. Nach einer erkannten manuellen bzw. externen Bedienung pausiert die normale Rollladenautomatik für die konfigurierte Dauer.

Der Fenster-/Aussperrschutz besitzt weiterhin höhere Priorität.

## Globalbeschattung

Die optionale Globalbeschattung kann unter anderem Außentemperatur, Raumtemperatur und Bewölkung berücksichtigen. Die gewünschte Rollladenposition während der Beschattung ist konfigurierbar.

## Wiederholung von Fahrbefehlen

Fahrbefehle werden anhand der tatsächlichen `current_position` geprüft. Bei fehlgeschlagenen Fahrbefehlen versucht der Blueprint die Position erneut anzufahren:

- maximal 10 Versuche
- 3 Sekunden zwischen den Versuchen

## Installation

Datei in das Blueprint-Verzeichnis kopieren, zum Beispiel:

```text
config/blueprints/automation/elbrondor/ShutterCoverAutomation.yml
```

Danach in Home Assistant die Automationen/Blueprints neu laden oder Home Assistant neu starten. Anschließend unter **Einstellungen → Automationen & Szenen → Blueprints** aus dem Blueprint eine Automation für den jeweiligen Rollladen erzeugen.

[← Zurück zur Übersicht](README.md)
