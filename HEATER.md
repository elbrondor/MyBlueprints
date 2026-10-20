# HeaterAutomation

`HeaterAutomation.yaml` ist ein Home-Assistant-Blueprint zur saisonalen und zeitabhaengigen Steuerung eines Raumthermostats.

**Aktuelle Blueprint-Version: 0.1.1**

## Funktionen

- Drei woechentliche Zeitplaene fuer Komfort, Normal und Nacht
- Je Wochentag frei konfigurierbare Zeitfenster
- Basis-Solltemperatur mit Komfort- und Nacht-Offset
- Zuschlag bei Anwesenheit
- Feste Solltemperaturen fuer Abwesenheit und offene Fenster oder Tueren
- Saisonale Heizfreigabe anhand der erwarteten Tagestiefsttemperatur
- Saisonale Abschaltung anhand der aktuellen Thermostattemperatur
- Manueller Heizmodus wird vor jeder Sollwertuebernahme erneut gesetzt
- Regelmaessige Neubewertung und Reaktion auf alle beteiligten Entitaeten

## Temperaturberechnung

Im Normal-Zeitplan und ausserhalb aller Zeitplaene gilt die Basis-Solltemperatur. Im Komfort-Zeitplan wird der Komforttemperatur-Offset addiert, im Nacht-Zeitplan der Nachttemperatur-Offset. Bei aktiver Anwesenheit kommt der Anwesenheitsbonus hinzu.

Bei sich ueberschneidenden Zeitplaenen gilt die Reihenfolge Nacht vor Komfort vor Normal.

Eine aktive Abwesenheitsentitaet ersetzt den berechneten Wert durch die Abwesenheitstemperatur. Ein offenes Fenster oder eine offene Tuer hat Vorrang vor Abwesenheit und setzt die Offen-Temperatur.

## Heizfreigabe

Die Automation ruft die taegliche Vorhersage mit `weather.get_forecasts` ab und liest `templow` des aktuellen Tages. Geheizt wird nur, wenn dieser Wert unter der zur Jahreszeit gehoerenden Heiz-Einschaltschwelle liegt.

Liegt die aktuelle Temperatur des Thermostats ueber der saisonalen Heiz-Ausschaltschwelle, wird der HVAC-Modus auf `off` gesetzt. Dasselbe gilt bei ungueltigen Wetter-, Saison- oder Thermostatdaten.

Nach jeder freigegebenen Neubewertung setzt die Automation den HVAC-Modus zuerst auf `heat` und uebertraegt danach den berechneten Sollwert. Dadurch bleibt der Thermostat auch nach Zeitplan-, Anwesenheits-, Fenster-, Abwesenheits-, Wetter- und Saisonwechseln im manuellen Modus.

## Standardwerte

| Einstellung | Standard |
|---|---:|
| Solltemperatur | 19,0 °C |
| Komforttemperatur-Offset | +1,5 °C |
| Nachttemperatur-Offset | -2,0 °C |
| Anwesenheitsbonus | +1,0 °C |
| Abwesenheitstemperatur | 16,0 °C |
| Offen-Temperatur | 5,0 °C |
| Heiz-Einschaltschwelle Winter | 15,0 °C |
| Heiz-Einschaltschwelle Fruehling | 12,0 °C |
| Heiz-Einschaltschwelle Sommer | 8,0 °C |
| Heiz-Einschaltschwelle Herbst | 12,0 °C |
| Heiz-Ausschaltschwelle Winter | 22,0 °C |
| Heiz-Ausschaltschwelle Fruehling | 21,0 °C |
| Heiz-Ausschaltschwelle Sommer | 20,0 °C |
| Heiz-Ausschaltschwelle Herbst | 21,0 °C |

## Installation

Datei in das Blueprint-Verzeichnis kopieren, zum Beispiel:

```text
config/blueprints/automation/elbrondor/HeaterAutomation.yaml
```

Danach den Blueprint neu laden und eine Automation daraus erstellen. Die ausgewaehlte Climate-Entitaet muss die HVAC-Modi `heat` und `off` sowie `climate.set_temperature` unterstuetzen.

[Zurueck zur Uebersicht](README.md)
