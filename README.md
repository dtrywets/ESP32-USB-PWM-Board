# ESP32 PWM- / Tacho-Board (4-Pin-Lüfter)

Kurzbeschreibung, Schaltung und Inbetriebnahme: siehe [writeup.md](writeup.md). Stückliste: [hardware/bom.csv](hardware/bom.csv).

## Firmware (PlatformIO)

Voraussetzung: [PlatformIO](https://platformio.org/) (IDE-Plugin oder CLI).

```bash
cd ESP32-USB-PWM-Board
pio run -t upload
pio device monitor
```

Standard: **115200 Baud**. GPIO **25** = PWM (nach MOSFET-Gate), **26** = Tach (Interrupt, FALLING) — anpassbar in `include/config.h`.

### Serielle Befehle

| Befehl        | Wirkung                          |
|---------------|----------------------------------|
| `PWM <0-100>` | Soll-Duty in Prozent             |
| `RPM`         | zuletzt geschätzte Drehzahl      |
| `?`           | wie `RPM`                        |
| `STATUS`      | PWM-% und RPM                    |
| `HELP`        | Hilfe                            |

PWM ca. **25 kHz**, Tach: **2 Impulse/Umdrehung** (Konstante `PULSES_PER_REVOLUTION` in `config.h`).

## KiCad

Es gibt absichtlich **keine** `.kicad_sch`-Datei im Repo (stabiler Bezug: BOM + Netztabelle in `writeup.md`). Schaltplan in KiCad neu anlegen und Netze aus Abschnitt 8.2 verdrahten.

## Inbetriebnahme

Checkliste: [writeup.md Abschnitt 9](writeup.md).
