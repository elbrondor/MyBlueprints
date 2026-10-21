# Heizungssteuerung: Blueprint-Inputs

Blueprint-Name: `HeaterAutomation`

## Wetter

| Name | Typ | Vorgabe |
|---|---|---|
| Wettervorhersage | Entitaet | `weather` |

## Solltemperaturen

| Name | Typ | Standard | Minimum | Maximum | Schritt | Einheit |
|---|---|---:|---:|---:|---:|---|
| Solltemperatur | Zahl | 19,0 | 5 | 30 | 0,5 | °C |

## Zeitplaene

| Name | Typ | Vorgabe |
|---|---|---|
| Komfort-Zeitplan | Entitaet | `schedule` |
| Normal-Zeitplan | Entitaet | `schedule` |
| Nacht-Zeitplan | Entitaet | `schedule` |

## Thermostat und Anwesenheit

| Name | Typ | Vorgabe |
|---|---|---|
| Thermostat | Entitaet | `climate` |
| Anwesenheitssensoren | Entitaeten | `binary_sensor` mit Geraeteklasse `motion` oder `occupancy` |

| Name | Typ | Standard | Minimum | Maximum | Schritt | Einheit |
|---|---|---:|---:|---:|---:|---|
| Anwesenheitsbonus | Zahl | 1,0 | 0 | 5 | 0,5 | °C |
| Komforttemperatur-Offset | Zahl | 1,5 | -5 | 5 | 0,5 | °C |
| Nachttemperatur-Offset | Zahl | -2,0 | -5 | 5 | 0,5 | °C |

## Abwesenheit

| Name | Typ | Vorgabe |
|---|---|---|
| Abwesenheitsentitaet | Entitaet | `input_boolean` oder `binary_sensor` |

| Name | Typ | Standard | Minimum | Maximum | Schritt | Einheit |
|---|---|---:|---:|---:|---:|---|
| Abwesenheitstemperatur | Zahl | 16,0 | 5 | 30 | 0,5 | °C |

Ist die Abwesenheitsentitaet `on`, ersetzt die Abwesenheitstemperatur pauschal die aus Zeitplan, Solltemperatur, Anwesenheitsbonus und Temperatur-Offsets berechnete Solltemperatur. Die saisonale Heizfreigabe und die Heiz-Ausschaltschwelle haben weiterhin Vorrang.

## Offene Fenster und Tueren

| Name | Typ | Vorgabe |
|---|---|---|
| Fenster-/Tuersensoren | Entitaeten | `binary_sensor` mit Geraeteklasse `window`, `door` oder `opening` |

| Name | Typ | Standard | Minimum | Maximum | Schritt | Einheit |
|---|---|---:|---:|---:|---:|---|
| Offen-Temperatur | Zahl | 5,0 | 5 | 30 | 0,5 | °C |

Meldet mindestens ein Fenster- oder Tuersensor den Zustand `on`, ersetzt die Offen-Temperatur die aus Zeitplan, Solltemperatur, Anwesenheit und Abwesenheit berechnete Solltemperatur. Die saisonale Heizfreigabe und die Heiz-Ausschaltschwelle haben weiterhin Vorrang.

## Saisonale Heizgrenzen

| Name | Typ | Vorgabe |
|---|---|---|
| Jahreszeit | Entitaet | `sensor.season` |

| Name | Typ | Standard | Minimum | Maximum | Schritt | Einheit |
|---|---|---:|---:|---:|---:|---|
| Heiz-Einschaltschwelle Winter | Zahl | 15,0 | -20 | 30 | 0,5 | °C |
| Heiz-Einschaltschwelle Fruehling | Zahl | 12,0 | -20 | 30 | 0,5 | °C |
| Heiz-Einschaltschwelle Sommer | Zahl | 8,0 | -20 | 30 | 0,5 | °C |
| Heiz-Einschaltschwelle Herbst | Zahl | 12,0 | -20 | 30 | 0,5 | °C |

Geheizt werden darf nur, wenn die fuer den aktuellen Tag erwartete Tiefsttemperatur unter der zur aktuellen Jahreszeit gehoerenden Heiz-Einschaltschwelle liegt.

| Name | Typ | Standard | Minimum | Maximum | Schritt | Einheit |
|---|---|---:|---:|---:|---:|---|
| Heiz-Ausschaltschwelle Winter | Zahl | 22,0 | 5 | 30 | 0,5 | °C |
| Heiz-Ausschaltschwelle Fruehling | Zahl | 21,0 | 5 | 30 | 0,5 | °C |
| Heiz-Ausschaltschwelle Sommer | Zahl | 20,0 | 5 | 30 | 0,5 | °C |
| Heiz-Ausschaltschwelle Herbst | Zahl | 21,0 | 5 | 30 | 0,5 | °C |

Liegt die vom Thermostat gemeldete aktuelle Temperatur ueber der zur aktuellen Jahreszeit gehoerenden Heiz-Ausschaltschwelle, muss die Heizung abgeschaltet werden.

## Vorausschauende Heizsperre

| Name | Typ | Standard |
|---|---|---|
| Vormittag - Beginn | Zeit | 04:00 |
| Vormittag - Ende | Zeit | 12:00 |
| Nachmittag - Beginn | Zeit | 12:00 |
| Nachmittag - Ende | Zeit | 17:00 |

| Name | Typ | Standard | Minimum | Maximum | Schritt | Einheit |
|---|---|---:|---:|---:|---:|---|
| Tagesmittel-Schwelle Winter - Vormittag | Zahl | 17,0 | -20 | 30 | 0,5 | °C |
| Tagesmittel-Schwelle Winter - Nachmittag | Zahl | 18,0 | -20 | 30 | 0,5 | °C |
| Tagesmittel-Schwelle Fruehling - Vormittag | Zahl | 14,0 | -20 | 30 | 0,5 | °C |
| Tagesmittel-Schwelle Fruehling - Nachmittag | Zahl | 17,0 | -20 | 30 | 0,5 | °C |
| Tagesmittel-Schwelle Sommer - Vormittag | Zahl | 14,0 | -20 | 30 | 0,5 | °C |
| Tagesmittel-Schwelle Sommer - Nachmittag | Zahl | 14,0 | -20 | 30 | 0,5 | °C |
| Tagesmittel-Schwelle Herbst - Vormittag | Zahl | 15,0 | -20 | 30 | 0,5 | °C |
| Tagesmittel-Schwelle Herbst - Nachmittag | Zahl | 16,0 | -20 | 30 | 0,5 | °C |

Das erwartete Tagesmittel wird aus `(temperature + templow) / 2` der taeglichen Wettervorhersage berechnet. Liegt es waehrend eines aktiven Zeitfensters ueber der passenden saisonalen Schwelle, wird der HVAC-Modus ausgeschaltet. Die vorausschauende Heizsperre hat Vorrang vor Zeitplaenen, Anwesenheit, Abwesenheit und offenen Fenstern. Die Zeitfenster duerfen Luecken haben, sollten sich aber nicht ueberschneiden; bei einer Ueberschneidung gilt das Vormittagsfenster.
