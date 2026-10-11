# Echo Automation

`EchoAutomation.yaml` ist ein Home-Assistant-Blueprint zur automatischen Radiowiedergabe auf einem Amazon Echo. Bewegung oder Belegung startet die Wiedergabe, anhaltende Inaktivitaet beendet sie. Unerwuenschte Radioprogramme werden erkannt und durch einen alternativen Sprachbefehl ersetzt.

**Aktuelle Blueprint-Version: 1.3.0**

## Voraussetzungen

- Home Assistant 2026.7.0 oder neuer
- Alexa-Media-Player-Integration mit einem erreichbaren `media_player` fuer den Echo
- Mindestens ein Bewegungs- oder Belegungssensor im ausgewaehlten Bereich
- Ein Sensor, dessen Zustand den Namen des aktuell laufenden Radioprogramms enthaelt
- Ein Script fuer die Sprachausgabe, das die Variable `message` verarbeitet
- Korrekte Bereichszuordnung der Bewegungs- und Belegungssensoren

Der Blueprint verwendet die zweckbezogenen Home-Assistant-Trigger und -Bedingungen fuer `motion`, `occupancy` und `media_player`. Sensoren ohne passende Geraeteklasse werden nicht als Aktivitaetssensoren erkannt.

## Funktionen

- Start der Radiowiedergabe bei Bewegung oder Belegung
- Gemeinsame Auswertung aller Bewegungs- und Belegungssensoren eines Bereichs
- Konfigurierbarer Sprachbefehl fuer die erste Wiedergabe
- Konfigurierbarer alternativer Sprachbefehl fuer unerwuenschte Programme
- Sofortige Auswahl der Alternative, wenn bereits beim Start ein unerwuenschtes Programm laeuft
- Wechsel zur Alternative, wenn das Radioprogramm waehrend der Wiedergabe umschaltet
- Ansage des aktuell erkannten Programms vor dem Start bzw. Wechsel
- Automatischer Stopp nach anhaltender Inaktivitaet
- Periodische Sicherheitspruefung alle 5 Minuten
- Beruecksichtigung eines uebergeordneten Echo-Gruppenplayers
- Zeitfenster und Sperr-Entitaeten fuer den automatischen Start
- Globale Erreichbarkeitspruefung des ausgewaehlten Echo
- Einstellbare Einschaltlautstaerke beim ersten automatischen Start des Tages
- Pruefung, ob Spotify bereits auf einem anderen Echo laeuft
- Konfigurierbarer Ausweichbefehl bei belegtem Spotify
- Erneute Startpruefung, wenn der konfigurierte Echo wieder online kommt

## Erreichbarkeitspruefung

Die erste Aktion jedes Automationslaufs prueft den Zustand des ausgewaehlten Echo mit `has_value()`. Fehlt die Entitaet oder besitzt der `media_player` den Zustand `unavailable` oder `unknown`, wird der Lauf sofort beendet. In diesem Fall werden weder Ansagen noch Wiedergabe- oder Stoppbefehle ausgefuehrt.

Die Pruefung gilt fuer alle Ausloeser und damit fuer Start, Stopp, Sicherheitspruefung und Programmwechsel.

## Ablauf

### Wiedergabe starten

Bewegung oder Belegung im ausgewaehlten Bereich startet den Einschaltzweig. Der Einschaltzweig wird ebenfalls erneut ausgewertet, wenn der konfigurierte Echo vom Zustand `unavailable` oder `unknown` zurueckkehrt. So kann eine waehrend eines Verbindungsverlusts erkannte Aktivitaet die Wiedergabe nach der Rueckkehr des Echo noch starten.

Die Wiedergabe startet nur, wenn alle folgenden Bedingungen erfuellt sind:

1. Der Echo ist erreichbar.
2. Im ausgewaehlten Bereich wird aktuell Bewegung oder Belegung erkannt.
3. Alle konfigurierten Sperr-Entitaeten besitzen den Zustand `off`.
4. Die aktuelle Uhrzeit liegt innerhalb des konfigurierten Zeitfensters.
5. Der uebergeordnete Gruppenplayer spielt seit mindestens der konfigurierten Ruhezeit nicht.
6. Der ausgewaehlte Echo spielt aktuell nicht.

Ist das aktuelle Radioprogramm in der Liste der unerwuenschten Programme enthalten, wird direkt der alternative Befehl ausgefuehrt. Andernfalls wird der Radio-an-Befehl verwendet.

Vor Ansage und Wiedergabe prueft der Blueprint den konfigurierten Lautstaerke-Zeitstempel. Wurde die Einschaltlautstaerke am aktuellen Kalendertag noch nicht gesetzt, stellt er den Echo auf den Wert des Lautstaerke-Schiebers und aktualisiert danach den Zeitstempel. Alle weiteren automatischen Starts am selben Tag behalten die dann aktuelle Lautstaerke bei. Ohne ausgewaehlten Zeitstempel-Helfer ist diese Funktion deaktiviert.

Soll wegen eines unerwuenschten Programms zu Spotify gewechselt werden, prueft der Blueprint zuvor alle konfigurierten Spotify-Player. Spielt mindestens einer dieser Player und verweist dessen `entity_picture` auf Spotifys CDN `scdn.co`, gilt Spotify als belegt. Statt des Spotify-Befehls wird dann der konfigurierbare Ausweichbefehl ausgefuehrt. Der Standard dafuer ist `Spiel Bayern 2`.

### Laufendes Programm wechseln

Der Programmwechsel wird geprueft, wenn sich der Programmsensor aendert oder der Echo die Wiedergabe beginnt. Ein Wechsel erfolgt nur, wenn:

- der Echo gerade spielt,
- das Attribut `media_title` dem konfigurierten Radiotitel entspricht und
- der Zustand des Programmsensors in der Liste der unerwuenschten Programme enthalten ist.

Damit wird andere Musik auf dem Echo nicht allein wegen eines unerwuenschten Radioprogramms unterbrochen.

### Wiedergabe stoppen

Das Ausschalten wird nach Ende von Bewegung oder Belegung sowie alle 5 Minuten als Sicherheitspruefung ausgewertet. Der Echo wird nur gestoppt, wenn:

- der Echo erreichbar ist und gerade spielt,
- der uebergeordnete Gruppenplayer nicht spielt und
- alle Bewegungs- und Belegungssensoren im Bereich mindestens fuer die Bestaetigungsdauer inaktiv sind.

Der regulaere Ende-von-Aktivitaet-Trigger wartet zusaetzlich die konfigurierte Triggerdauer ab. Mit den Standardwerten wird der Trigger nach 5 Minuten ausgeloest und bestaetigt dabei mindestens 3 Minuten gemeinsame Inaktivitaet.

## Eingaben

### Bereich und Echo

| Einstellung | Standard | Beschreibung |
|---|---|---|
| Bereich fuer Bewegung und Belegung | `kuche` | Bereich, dessen Aktivitaetssensoren ausgewertet werden |
| Amazon Echo | `media_player.echo_kuche` | Zu steuernder Echo |
| Uebergeordneter Echo-Gruppenplayer | `media_player.uberall` | Verhindert Einzelsteuerung, solange die Gruppe spielt |
| Aktuelles Radioprogramm | `sensor.jetzt_in_bayern_1` | Zustand enthaelt den Namen des laufenden Beitrags |
| Ansage-Script | `script.notify_alexa_kuche` | Erhaelt den Ansagetext als Variable `message` |

### Wiedergabe

| Einstellung | Standard | Beschreibung |
|---|---|---|
| Radio-an-Befehl | `Spiel Bayern eins` | Sprachbefehl fuer die erste Radio-Wiedergabe |
| Alternativer Musikbefehl | `Spiele Musik, die mir gefällt, von Spotify in zufälliger Reihenfolge` | Befehl bei einem unerwuenschten Programm |
| Alternative bei bereits laufendem Spotify | `Spiel Bayern 2` | Ausweichbefehl, wenn Spotify schon auf einem ausgewaehlten Player laeuft |
| Player fuer die Spotify-Pruefung | leer | Echo-Player, die auf aktive Spotify-Wiedergabe geprueft werden |
| Titel des laufenden Radiosenders | `BR1 - Oberbayern` | Erwarteter Wert des Echo-Attributs `media_title` |
| Unerwuenschte Programme | siehe unten | Exakte Zustaende des Programmsensors, die einen Wechsel ausloesen |
| Einschaltlautstaerke | 30 % | Lautstaerke beim ersten automatischen Start des Kalendertags |
| Zeitstempel fuer die Einschaltlautstaerke | leer | `input_datetime` mit Datum und Uhrzeit; aktiviert die einmalige taegliche Einstellung |

Die Standardliste enthaelt:

- `Bayern 1 Heute im Stadion`
- `Evangelische Morgenfeier`
- `Katholische Morgenfeier`

Die Programmnamen werden exakt verglichen. Abweichende Schreibweisen des Sensors muessen als zusaetzliche Listeneintraege aufgenommen werden.

### Einschaltbedingungen

| Einstellung | Standard | Beschreibung |
|---|---|---|
| Sperr-Entitaeten | `input_boolean.abwesend_urlaub`, `input_boolean.gastemodus`, `input_boolean.home_office` | Alle Eintraege muessen `off` sein |
| Aktiv ab | `05:00:00` | Beginn des erlaubten Startzeitraums |
| Aktiv bis | `20:30:00` | Ende des erlaubten Startzeitraums |
| Mindest-Ruhezeit des Gruppenplayers | 10 min | So lange darf der Gruppenplayer nicht gespielt haben |

Die Einschaltbedingungen gelten nur fuer den automatischen Start. Ein spaeterer Programmwechsel und das automatische Stoppen bleiben ausserhalb dieses Zeitfensters aktiv.

### Ausschaltverhalten

| Einstellung | Standard | Beschreibung |
|---|---|---|
| Inaktivitaet bis zum Ausschalt-Trigger | 5 min | Wartezeit nach dem Ende eines Aktivitaetssignals |
| Bestaetigte Inaktivitaet | 3 min | Mindestdauer, fuer die alle Sensoren inaktiv sein muessen |

## Schnittstelle des Ansage-Scripts

Das ausgewaehlte Script wird ueber `script.turn_on` gestartet. Der Blueprint uebergibt den Text in `data.variables.message`:

```yaml
sequence:
  - action: notify.alexa_media
    data:
      target: media_player.echo_kuche
      message: "{{ message }}"
      data:
        type: tts
```

Das Script darf weitere Bedingungen enthalten, beispielsweise Ruhezeiten oder eine eigene Anwesenheitspruefung. Wird die Ansage dort blockiert, laeuft der anschliessende Wiedergabebefehl des Blueprints trotzdem weiter.

## Beispiel Arbeitszimmer

Die installierte Automation **Echo Arbeitszimmer** verwendet unter anderem folgende Abweichungen von den Standardwerten:

| Einstellung | Wert |
|---|---|
| Bereich | `arbeitszimmer` |
| Echo | `media_player.echo_arbeitszimmer` |
| Ansage-Script | `script.notify_alexa_arbeitszimmer` |
| Alternativer Musikbefehl | `Spiele Musik, die mir gefällt auf Spotify, in zufälliger Reihenfolge.` |
| Zusaetzlicher Programmname | `Heute im Stadion` |
| Einschaltlautstaerke | 10 % |
| Lautstaerke-Zeitstempel | `input_datetime.echo_arbeitszimmer_lautstaerke_gesetzt` |
| Spotify-Ausweichbefehl | `Spiel Bayern 2` |
| Spotify-Pruefung | alle vorhandenen Echo-Player |

Die bestehende Entity-ID der umgebauten Automation bleibt `automation.echo_arbeitszimmer_scheiss_programm`; ihr sichtbarer Name lautet **Echo Arbeitszimmer**.

## Beispiel Küche

Die installierte Automation **Echo Küche** verwendet folgende raumspezifische Werte:

| Einstellung | Wert |
|---|---|
| Bereich | `kuche` |
| Echo | `media_player.echo_kuche` |
| Ansage-Script | `script.notify_alexa_kuche` |
| Einschaltlautstaerke | 27 % |
| Lautstaerke-Zeitstempel | `input_datetime.echo_kueche_lautstaerke_gesetzt` |
| Spotify-Ausweichbefehl | `Spiel Bayern 2` |
| Spotify-Pruefung | alle vorhandenen Echo-Player |

Die bestehende Entity-ID der umgebauten Automation bleibt `automation.alexa_kuche_scheiss_programm`; ihr sichtbarer Name lautet **Echo Küche**. Die bisherigen Automationen `automation.kuche_echo_on_motion` und `automation.kuche_echo_off_motion` wurden nach der Umstellung entfernt.

## Installation

Datei in das Blueprint-Verzeichnis kopieren, zum Beispiel:

```text
config/blueprints/automation/elbrondor/EchoAutomation.yaml
```

Danach in Home Assistant die Automationen und Blueprints neu laden. Anschliessend unter **Einstellungen -> Automationen & Szenen -> Blueprints** aus dem Blueprint eine Automation fuer den gewuenschten Echo erzeugen.

Bei einer Aktualisierung ueber die in `source_url` hinterlegte GitHub-Datei kann der Blueprint in Home Assistant erneut importiert werden. Bestehende Blueprint-Automationen verwenden die aktualisierte Logik nach dem Neuladen automatisch.

## Aenderungen

### Version 1.3.0

- Startpruefung bei Rueckkehr des konfigurierten Echo aus `unavailable` oder `unknown` hinzugefuegt
- Aktuelle Bewegung oder Belegung wird vor jedem automatischen Start explizit bestaetigt

### Version 1.2.0

- Mehrfachauswahl der auf Spotify zu pruefenden Media Player hinzugefuegt
- Aktive Spotify-Wiedergabe wird ueber das Spotify-CDN im Cover-Attribut erkannt
- Konfigurierbarer Ausweichbefehl mit Standard `Spiel Bayern 2` hinzugefuegt

### Version 1.1.0

- Schieber fuer die Einschaltlautstaerke hinzugefuegt
- Lautstaerke wird nur beim ersten automatischen Start eines Kalendertags gesetzt
- Optionaler `input_datetime` speichert den letzten Ausfuehrungszeitpunkt dauerhaft

### Version 1.0.1

- Globale Erreichbarkeitspruefung vor allen Aktionen hinzugefuegt
- Bei fehlender, `unavailable` oder `unknown` Echo-Entitaet wird der Automationslauf ohne Aktion beendet

### Version 1.0.0

- Erste Version mit Aktivitaetsstart, Inaktivitaetsstopp und automatischem Programmwechsel

[Zurueck zur Uebersicht](README.md)
