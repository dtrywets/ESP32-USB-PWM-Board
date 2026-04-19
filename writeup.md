ESP32 PWM-/Tacho-Board (4-Pin-Lüfter, USB-Serial)
Dokumentation für ein kleines Board: ESP32 per USB (Serial Monitor / eigenes Tool), PWM und Tacho (RPM) für einen 4-Pin-PC-Lüfter (+12V, GND, Tach, PWM).

1. Ziel und Prinzip
USB am ESP32 → Kommunikation (z. B. Baud 115200) für Soll-PWM und Auslesen der Drehzahl.
Ein Kanal (leicht erweiterbar auf mehr GPIOs).
Am Lüfter ist der PWM-Eingang typisch als Open-Drain/Open-Collector von der „Motherboard-Seite“ gedacht: High über Pull-up (oft im Lüfter), Low durch Ziehen nach GND.
Der ESP32 soll die Leitung nicht mit 3,3 V Push-Pull direkt treiben, wenn der High-Pegel >3,3 V sein kann → Low-Side-Schalter mit N-MOSFET: Drain an Lüfter-PWM, Source an GND.
2. MOSFET (Standard: IRLZ44N)
Teil	Typ	Bemerkung
Q1
IRLZ44NPBF
Logic-Level N-Kanal, TO-220, Serie „Z“, für Ansteuerung aus 3,3 V-GPIO geeignet. Viel Stromreserve; der PWM-Pin des Lüfters ist kein Motor-Hauptstrom, aber der FET ist robust und gut dokumentiert.
Gate-Beschaltung

R1: ca. 100 Ω zwischen GPIO und Gate (dämpft Ringing, schont ESP-Ausgang).
R2: ca. 100 kΩ Gate → GND (Pulldown, definiert AUS beim Boot).
(Bei sehr hoher PWM-Frequenz und Problemen mit Flanken ggf. Treiber diskutieren; für typ. ~25 kHz PC-Lüfter meist nicht nötig.)

Alternativen (wenn du später SMD/kleiner willst): IRLML6344, AO3400A — hier ist IRLZ44N die feste Empfehlung.

3. Tacho (RPM)
Typisch Open-Collector am Lüfter: Impulse nach GND.
Am ESP32: R3 = 10 kΩ Pull-up auf 3,3 V, Zwischenpunkt an GPIO (mit Interrupt für Flanken / Pulszähler).
Wichtig: Mit dem Oszilloskop prüfen, ob auf der Tach-Leitung kein dauerhaftes 12 V auf den ESP-Pin kommt. Manche Aufbauten erwarten Motherboard-Pull-up auf höhere Spannung → dann Spannungsteiler, Z-Diode/Clamp oder Optokoppler statt direktem 3,3 V-PU.

4. Stromversorgung und Masse
+12 V für den Lüfter (Netzteil dimensionieren, ggf. Sicherung/TVS nach Bedarf).
ESP32 z. B. über USB (DevKit) oder eigenen 3,3 V-Regler auf dem Board.
Gemeinsame Masse: USB-GND, 12 V-GND, Lüfter-GND, MOSFet-Source niederimpedant verbinden (Stern/Massefläche).
5. Lüfter-Pinbelegung (typisch 4-Pin)
Pin	Funktion
1
+12V
2
GND
3
Tach
4
PWM
Stecker/Keying je nach Gehäuse prüfen → Datenblatt Lüfter.

6. Blockschaltbild (ASCII)
        USB                    +12V (Lüfter)
         │                          │
    ┌────┴────┐                     │
    │  ESP32  │                     │
    │ 3V3 GND │                     │
    └─┬──┬──┬─┘                     │
      │  │  │                       │
      │  │  └── GPIO_PWM ──[100R]── Gate Q1 (IRLZ44N)
      │  │              └── 100k ── GND
      │  │                     │
      │  │              Drain ──┼── Lüfter PWM (Pin 4)
      │  │              Source ─┴── GND
      │  │
      │  └── GPIO_TACH ──[10k]── 3V3
      │              └── Lüfter Tach (Pin 3)
      │
      └────────────── GND ──────── Lüfter GND (Pin 2)
Lüfter Pin 1: +12V
7. Serial / Firmware (Vorschlag)
Baud: z. B. 115200
Beispiel-Befehle: PWM 0 … PWM 100 (Duty in %), RPM oder ? für letzte Messung.
PWM-Ausgang: LEDC o. ä., Frequenz typ. ~25 kHz (PC-Lüfter üblich).
Tacho: 2 Impulse/Umdrehung ist bei vielen Lüftern üblich — im Code als Konstante, ggf. pro Modell anpassen.
8. Abschnitt für KiCad / Fertigung
KiCad liest kein Markdown direkt als Schaltplan. Praktisch:

8.1 BOM als CSV (importierbar / weiterverarbeitbar)
Reference,Value,Footprint,LCSC,Manufacturer,Part Number,Description
U1,ESP32-DevKitC-32E,MODULE_ESP32_DEVKITC,,Espressif,ESP32-DevKitC,MCU + USB-Serial
Q1,IRLZ44NPBF,TO-220-3_Vertical,,Infineon,IRLZ44NPBF,Logic N-MOSFET PWM low-side
R1,100,R_0603_1608Metric,,,100R,Gate series resistor
R2,100k,R_0603_1608Metric,,,100k,Gate pulldown
R3,10k,R_0603_1608Metric,,,10k,Tach pull-up to 3V3
J1,FAN_4PIN,PinHeader_1x04_P2.54mm,,,Fan 4-pin,4-pin fan connector
C1,100n,C_0603_1608Metric,,,100n,Decoupling 3V3 optional
C2,10u,C_0805_2012Metric,,,10u,Optional bulk 12V input
Footprints an deine KiCad-Bibliothek anpassen (z. B. Package_TO_SOT_THT:TO-220-3_Vertical für Q1).

8.2 Netzliste / Signale (Referenz fürs Zeichnen in Eeschema)
Netname	Beschreibung
+3V3
3,3 V (R3 Pull-up, optional C1)
GND
ESP-GND, Q1 Source, Lüfter GND, R2
+12V
Lüfter Pin 1
PWM_DRV
GPIO → R1 → Gate Q1
FAN_PWM
Q1 Drain ↔ Lüfter Pin 4
FAN_TACH
GPIO ↔ Mitte R3 ↔ Lüfter Pin 3
8.3 Hinweis „KiCad importieren“
Eine vollständige .kicad_sch per Copy-Paste aus dem Chat ist fehleranfällig (UUIDs, Symbole). Sinnvoll: neues Projekt in KiCad anlegen und diese Netze + BOM abarbeiten. Wer Text-basiert arbeiten will: SKiDL oder eigene Skripte — die CSV + Tabelle oben sind der stabile Bezug.

9. Checkliste Inbetriebnahme

 Tach-Leitung vermessen: kein gefährlicher Pegel für 3,3 V-GPIO.

 PWM-Frequenz und Duty im sinnvollen Bereich für den Lüfter.

 gemeinsame Masse 12 V / Logik / USB.

 Optional: Sicherung / TVS auf 12 V, Kondensator nahe Lüfteranschluss. 
