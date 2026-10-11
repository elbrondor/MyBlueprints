# Echo Automation

`echo_automation.yaml` ist ein Home-Assistant-Blueprint zur automatischen Radiowiedergabe auf einem Amazon Echo. Bewegung oder Belegung startet die Wiedergabe, anhaltende Inaktivitaet beendet sie. Unerwuenschte Radioprogramme werden erkannt und durch einen alternativen Sprachbefehl ersetzt.

**Aktuelle Blueprint-Version: 1.8.0**

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
- Einstellbare Nachlaufzeit von 1 bis 60 Minuten
- Separate Sperr-Entitaeten fuer das automatische Ausschalten
- Einstellbare Blindzeit von 1 bis 600 Sekunden zwischen Automationslaeufen
- Positionspruefung bei einem vom Echo gemeldeten `playing`-Status
- Atomare Automationslaeufe; neue Ausloeser werden waehrend eines Laufs verworfen

## Erreichbarkeitspruefung

Nach der Blindzeitpruefung prueft jeder Automationslauf den Zustand des ausgewaehlten Echo mit `has_value()`. Fehlt die Entitaet oder besitzt der `media_player` den Zustand `unavailable` oder `unknown`, wird der Lauf sofort beendet. In diesem Fall werden weder Ansagen noch Wiedergabe- oder Stoppbefehle ausgefuehrt.

Die Pruefung gilt fuer alle Ausloeser und damit fuer Start, Stopp, Sicherheitspruefung und Programmwechsel.

Vor der Erreichbarkeitspruefung vergleicht der Blueprint den vorherigen
Laufzeitpunkt der Automation mit der konfigurierten Blindzeit. Ist der Abstand
kleiner, endet der neue Lauf sofort, ohne eine Aktion auszufuehren. Die
Automation bleibt dabei nicht aktiv und wartet nicht mit einer Verzoegerung.

Meldet der Echo danach den Zustand `playing`, aktualisiert der Blueprint die
Entitaet und speichert die aktuelle Wiedergabeposition. Nach zehn Sekunden wird
die Entitaet erneut aktualisiert. Nur wenn sich die Position geaendert hat, gilt die
Wiedergabe als tatsaechlich laufend. Fehlt das Positionsattribut, wird der
gemeldete `playing`-Status aus Sicherheitsgruenden beibehalten. Bei allen
anderen Playerzustaenden findet diese zehnsekündige Pruefung nicht statt.

Der Blueprint verwendet den Automationsmodus `single`. Ein laufender Durchgang
wird daher nicht durch einen neuen Ausloeser unterbrochen oder neu gestartet.
Neue Ausloeser werden waehrenddessen ohne Meldung verworfen. Home Assistant
bietet fuer Automationen keine globale maximale Laufzeit. Der Blueprint besitzt
jedoch keine unbegrenzten Warteaktionen; seine einzige feste Wartezeit ist die
zehnsekündige Positionspruefung.

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
- alle konfigurierten Ausschalt-Sperr-Entitaeten `off` sind und
- alle Bewegungs- und Belegungssensoren im Bereich mindestens fuer die Bestaetigungsdauer inaktiv sind.

Der regulaere Ende-von-Aktivitaet-Trigger wartet zusaetzlich die am Schieberegler eingestellte Nachlaufzeit ab. Sie kann in ganzen Minuten von 1 bis 60 eingestellt werden. Mit den Standardwerten wird der Trigger nach 5 Minuten ausgeloest und bestaetigt dabei mindestens 3 Minuten gemeinsame Inaktivitaet.

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
| Blindzeit zwischen Automationslaeufen | 60 s | Mindestabstand zwischen Laeufen; Schieberegler von 1 bis 600 Sekunden |

Die Blindzeit gilt fuer jeden Automationslauf. Die uebrigen Einschaltbedingungen
gelten nur fuer den automatischen Start. Ein spaeterer Programmwechsel und das
automatische Stoppen bleiben ausserhalb des Zeitfensters aktiv.

### Ausschaltverhalten

| Einstellung | Standard | Beschreibung |
|---|---|---|
| Ausschalt-Sperr-Entitaeten | leer | Alle Eintraege muessen `off` sein, damit der Echo automatisch gestoppt wird |
| Nachlaufzeit | 5 min | Schieberegler fuer die Wartezeit nach dem Ende eines Aktivitaetssignals; einstellbar von 1 bis 60 Minuten |
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
| Blindzeit | 60 s |

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

## Beispiel Werkstatt

Die installierte Automation **Echo Werkstatt** verwendet folgende raumspezifische Werte:

| Einstellung | Wert |
|---|---|
| Bereich | `werkstatt` |
| Echo | `media_player.echo_werkstatt` |
| Ansage-Script | `script.notify_alexa_werkstatt` |
| Aktiv ab / bis | `05:00:00` / `21:30:00` |
| Mindest-Ruhezeit des Gruppenplayers | 0 s |
| Nachlaufzeit | 8 min |
| Bestaetigte Inaktivitaet | 3 min |
| Einschaltlautstaerke | 30 % |
| Lautstaerke-Zeitstempel | `input_datetime.echo_werkstatt_lautstaerke_gesetzt` |
| Spotify-Ausweichbefehl | `Spiel Bayern 2` |
| Spotify-Pruefung | alle vorhandenen Echo-Player |

Die bestehende Entity-ID der umgebauten Automation bleibt `automation.werkstatt_echo_on_motion`; ihr sichtbarer Name lautet **Echo Werkstatt**. Die bisherige Automation `automation.echo_werkstatt_off_motion` wurde nach der Umstellung entfernt.

## Beispiel Bad

Die installierte Automation **Echo Bad** verwendet folgende raumspezifische Werte:

| Einstellung | Wert |
|---|---|
| Bereich | `bad` |
| Echo | `media_player.echo_bad` |
| Ansage-Script | `script.notify_alexa_bad` |
| Radio-an-Befehl | `spiele bayern 1` |
| Aktiv ab / bis | `05:00:00` / `20:30:00` |
| Mindest-Ruhezeit des Gruppenplayers | 0 s |
| Ausschalt-Sperr-Entitaet | `input_boolean.badezeit` |
| Nachlaufzeit | 5 min |
| Bestaetigte Inaktivitaet | 3 min |
| Einschaltlautstaerke | 10 % |
| Lautstaerke-Zeitstempel | `input_datetime.echo_bad_lautstaerke_gesetzt` |

Die bestehende Entity-ID bleibt `automation.bad_motion_echo_on`; ihr sichtbarer Name lautet **Echo Bad**. Die bisherige Automation `automation.bad_echo_off_motion` wurde nach der Umstellung entfernt. Solange `input_boolean.badezeit` aktiv ist, verhindert die Ausschalt-Sperre das automatische Stoppen.

## Beispiel Gaestezimmer 1

Die installierte Automation **Echo Gaestezimmer 1** verwendet folgende raumspezifische Werte:

| Einstellung | Wert |
|---|---|
| Bereich | `gastezimmer_1` |
| Echo | `media_player.iots_echo_pop` |
| Ansage-Script | `script.notify_alexa_gastezimmer_1` |
| Aktiv ab / bis | `05:00:00` / `20:30:00` |
| Nachlaufzeit | 5 min |
| Bestaetigte Inaktivitaet | 3 min |
| Einschaltlautstaerke | 30 % |
| Lautstaerke-Zeitstempel | `input_datetime.echo_gaestezimmer_1_lautstaerke_gesetzt` |
| Zusaetzliche Start-Sperre | `input_boolean.gastezimmer_belegt` |

Die neue Automation besitzt die Entity-ID `automation.echo_gastezimmer_1` und verwendet den Blueprint vollstaendig fuer Start, Programmwechsel und Stopp.

## Beispiel Buegelzimmer

Die installierte Automation **Echo Buegelzimmer** verwendet folgende raumspezifische Werte:

| Einstellung | Wert |
|---|---|
| Bereich | `bugelzimmer` |
| Echo | `media_player.echo_bugelzimmer` |
| Ansage-Script | `script.notify_alexa_bugelzimmer` |
| Aktiv ab / bis | `05:00:00` / `20:30:00` |
| Nachlaufzeit | 5 min |
| Bestaetigte Inaktivitaet | 3 min |
| Einschaltlautstaerke | 30 % |
| Lautstaerke-Zeitstempel | `input_datetime.echo_bugelzimmer_lautstaerke_gesetzt` |

Die neue Automation besitzt die Entity-ID `automation.echo_bugelzimmer`. Ist der Echo `unavailable` oder `unknown`, beendet der Blueprint den Lauf ohne Ansage oder Wiedergabebefehl.

## Beispiel Keller 2

Die installierte Automation **Echo Keller 2** verwendet folgende raumspezifische Werte:

| Einstellung | Wert |
|---|---|
| Bereich | `keller_2` |
| Echo | `media_player.echo_keller_2` |
| Ansage-Script | `script.notify_alexa_keller_2` |
| Aktiv ab / bis | `05:00:00` / `20:30:00` |
| Nachlaufzeit | 5 min |
| Bestaetigte Inaktivitaet | 3 min |
| Einschaltlautstaerke | 30 % |
| Lautstaerke-Zeitstempel | `input_datetime.echo_keller_2_lautstaerke_gesetzt` |

Die neue Automation besitzt die Entity-ID `automation.echo_keller_2`. Die bestehenden Automationen `automation.keller_2_alexa_ein_motion` und `automation.keller_2_alexa_aus` bleiben aktiv, weil sie ueber `switch.alexa` die Stromversorgung des Echo steuern. Der Blueprint steuert die Medienwiedergabe, sobald `media_player.echo_keller_2` erreichbar ist.

## Installation

Datei in das Blueprint-Verzeichnis kopieren, zum Beispiel:

```text
config/blueprints/automation/elbrondor/echo_automation.yaml
```

Danach in Home Assistant die Automationen und Blueprints neu laden. Anschliessend unter **Einstellungen -> Automationen & Szenen -> Blueprints** aus dem Blueprint eine Automation fuer den gewuenschten Echo erzeugen.

Bei einer Aktualisierung ueber die in `source_url` hinterlegte GitHub-Datei kann der Blueprint in Home Assistant erneut importiert werden. Bestehende Blueprint-Automationen verwenden die aktualisierte Logik nach dem Neuladen automatisch.

## Aenderungen

### Version 1.5.0

- Mehrfachauswahl fuer Ausschalt-Sperr-Entitaeten hinzugefuegt
- Automatischer Stopp erfolgt nur, wenn alle ausgewaehlten Sperren `off` sind

### Version 1.4.0

- Nachlaufzeit auf einen Minutenschieber von 1 bis 60 Minuten umgestellt

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
