# APEC carrier (LM GS3) engineering notes

What you need to know to wire, configure and write firmware for the carrier, revision R1A. GPIO and ADC channel assignments are in [IO.md](IO.md). Net names match the schematic.

## Power

| Rail | Source | Feeds |
| ---- | ------ | ----- |
| VIN (about 12 V) | U5 (XP Power ECE10US12, 12 V 10 W AC/DC) through D7 (U5's output, before D7, is 12V_ACDC), or U7 (AP3012 boost from USB VBUS, 11.5 V) through D2 | SoM VIN (pins 79–80), relay coils, SSR outputs CN10/CN11, J13 pin 19, JP11 |
| +3V3, +5V, +5VA | APEC SoM regulators | Logic, sensors (+5VA powers the pressure transducers and the ADC reference for them) |
| +24V | External, J5 pin 2 (return GNDPWR on pin 1). Nothing on the board generates it. | Gear pump connector CN9, 24 V outputs CN13/CN14, U15 |
| 12V_ISO | U15 (L78L12) from +24V | 24 V side of the isolated gate driver U16 |
| NAU7802_1_AVDD | The NAU7802's internal LDO | Load cell excitation (J12, J18 pin 1) |

* **The board runs from USB alone.** U7 boosts USB VBUS to 11.5 V on VIN, so the SoM and the ADCs work normally on the bench. With mains connected, U5 supplies VIN through D7 instead.
* **USB inrush.** VBUS reaches VIN's capacitance (about 70 µF) through U7's inductor and diode, so the plug-in inrush exceeds USB's 10 µF limit. Most ports tolerate it; if one trips its current limit, use a powered hub. USB is for bench programming and debugging only.
* The SoM's +6V rail isn't used.
* Rail indicator LEDs D19–D23 show +24V, 12V_ISO, 12V_ACDC, USB_VBUS and +3V3.

### Ground domains

| Ground | Use | Link to GND |
| ------ | --- | ----------- |
| GND | Main ground, and the return of U5 | — |
| GND1 | Load cells (NAU7802) | Joined to GND only at the net tie NT1 |
| GNDA | ADC ground (RTDs, pressure transducers, +5VA) | Only at SoM pin 49 (GNDA), which the SoM ties to its ground plane. Nothing else on the carrier may join GNDA to GND: a second link would form a ground loop through the SoM. |
| GNDPWR | 24 V return, from J5 pin 1 | Isolated by U16. R2 (100 MΩ) is the only link, so GNDPWR can't float far from GND. |

GNDA follows the "separate GNDA area" layout described in the SoM's [ENGINEERING-NOTES.md](https://github.com/variegated-coffee/apec-som/blob/main/ENGINEERING-NOTES.md). The pressure transducers, the +5VA decoupling, the RTD reference resistor R5 (with REFN0 through R34) and the ADC filter capacitors all return to GNDA. AIN7 (SoM pin 48), REFN1 for the pressure inputs, is tied to GNDA next to pin 49.

On the gear pump connector CN9, keep the FG signal ground (pin 3, GND) and the motor return (pin 5, GNDPWR) separate, as the FG304 datasheet requires.

## Mains

**Mains voltage is present on the relays, J2, the fuse, U5 and the varistors whenever the machine is plugged in.** PH, PH_UNFUSED, N, ROTARY_PUMP, GROUP_SOLENOID, FILL_SOLENOID and TEA_SOLENOID are all hazardous. Treat N as live: with an unpolarised plug it can carry line voltage.

### J2 mains terminals

| Pin | Net | Function |
| --- | --- | -------- |
| 1 | PH_UNFUSED | Line in, to the fuse |
| 3, 4 | N | Neutral |
| 5 | ROTARY_PUMP | Stock rotary pump, switched by K1 |
| 6 | GROUP_SOLENOID | Group solenoid, switched by U9 |
| 10 | FILL_SOLENOID | Boiler fill solenoid, switched by U10 |
| 11 | TEA_SOLENOID | Tea (hot water) solenoid, switched by U11 |
| 2, 7, 8, 9, 12, 13 | — | Not connected on this board |

### Fuse

F2 and F3 are the two clips of one 5×20 mm fuse, between PH_UNFUSED and PH. It carries U5, the three solenoids and, if fitted, the stock rotary pump, and protects the PCB. Use a time-lag (T), ceramic high-breaking (H), 250 VAC fuse.

| Supply | Pump | Suggested fuse | Approximate running current |
| ------ | ---- | -------------- | --------------------------- |
| 230 VAC | Stock rotary pump | T2.5A | 1.6 A |
| 230 VAC | Gear pump fed from J5, or no pump | T1A | 0.4 A |
| 110 VAC | Stock rotary pump | T4A | 3.1 A |
| 110 VAC | Gear pump fed from J5, or no pump | T1.6A | 0.8 A |

Running currents are estimates. Check your motor's nameplate and keep at least 25 % margin. The maximum is T5A at either voltage: a T5A may hold 7.5 A for an hour, and the 2.5 mm, 35 µm mains tracks then rise about 30 K. Keep I²t well below 600 A²s.

### Switching

* U9, U10 and U11 (Panasonic ALDP112) are mechanical relays, not SSRs, despite the U prefix. K1 (Schrack RT1, 16 A) switches the rotary pump. All four switch PH to their output, and their 12 V coils run from VIN, driven by Q22–Q25 with flyback diodes D8–D10 and D12.
* RV1 (varistor) sits across the mains input after the fuse; RV2–RV5 sit across each switched output to N.
* Pull-down resistors keep every relay and SSR output off while the SoM is starting.
* K1's normally-closed contact (pin 12) isn't connected, but it carries line voltage while the relay is off. The design rules treat it as mains.

### Clearances and layout

* Net classes 230VAC-16A and 230VAC-5A keep 2.4 mm between mains nets and all other copper, with 2.5 mm tracks. The rules are in `apec-carrier-lm-gs3.kicad_dru`; later rules take precedence, so keep the mains rules last.
* Isolation slots in the board outline separate the mains area: one alongside the U10/U11 relays, and one in the 24 V section next to J5 and Q11. Don't route low-voltage copper across them.
* N, PH and PH_UNFUSED run about 1.5–1.7 mm from the right-hand board edge. In the GS3 that edge faces a plastic enclosure. If you mount the board elsewhere, keep conductive parts away from that edge.

## Outputs

| Connector | Output | Driven by | Notes |
| --------- | ------ | --------- | ----- |
| CN10 | Brew boiler SSR control | Q27 (BREW_BOILER_EN) | Pin 2 is VIN, pin 1 is switched to GND |
| CN11 | Service boiler SSR control | Q26 (SERVICE_BOILER_EN) | **Reverse polarity compared to CN10:** pin 1 is VIN, pin 2 is switched to GND |
| CN13 | 24 V output (steam, proportional solenoid) | Q11 (IRFZ44N), U16 channel B, 24V_STEAM_OUT | Pin 2 is +24V, pin 1 is switched to GNDPWR. D13 flyback. PWM-capable. |
| CN14 | 24 V output (tea, proportional solenoid) | Q12 (IRFZ44N), U16 channel A, 24V_TEA_SOLENOID | As CN13; D11 flyback |
| CN9 | Gear pump (FG304) | U3, U4 (74LV1T125 buffers to 5 V) | Pin 1 +24V, 3 GND, 4 direction, 5 GNDPWR, 7 speed (PWM), 8 tacho (to GP25, 5 V, no pull-up on the board). Pins 2 and 6 not connected. |

Each output has an indicator LED.

## Inputs

### RTDs (J11, J8)

Two RTD inputs on the SoM's ADS124S08, following TI's ratiometric reference designs. Both are set up for **2-wire PT1000** by default.

| Pin | J11 (RTD 1) | J8 (RTD 2) |
| --- | ----------- | ---------- |
| 1 | REF_R (top of the 2K2 reference resistor R5) | REF_R |
| 2 | Sense, to AIN2 through R7; JP6 joins it to REF_R; JP4 to AIN3 | Sense, to AIN10 through R20; JP8 joins it to REF_R; JP9 to AIN11 |
| 3 | Sense, to AIN1 through R6; JP3 joins it to AIN0 | Sense, to AIN9 through R18; JP7 joins it to AIN8 |
| 4 | AIN0 (excitation) | AIN8 (excitation) |

* For 3-wire or 4-wire RTDs, cut and solder the jumpers as described in TI SBAA275A, and change the filter parts. For PT100, change the filter resistors and capacitors.
* R1A values: R_Ref (R5) = 2K2; inputs 2 × 4K99 into 47 nF + 2 × 4.7 nF (about 323 Hz); reference REFP0 through R9 = 10K into C10 + C26 = 51.7 nF (about 308 Hz), REFN0 straight to R5 (R34 = 0R). **This needs the REFP0 buffer enabled in firmware**; with it on, R9 adds no meaningful error.
* TI's alternative values: 4-wire (SBAA336b) or 2-wire (SBAA329b) RTD: R_Ref = 1.62K, R = 3.16K, C = 100 nF. 3-wire with two IDACs (SBAA330c): R_Ref = 820R, R = 4.02K, C = 100 nF.

### Pressure transducers (J19, J10)

Pin 1 +5VA, pin 2 signal, pin 3 GNDA, for 0.5–4.5 V ratiometric transducers. Each signal has a 33K / 47 nF RC filter (about 100 Hz) with the capacitor on the ADC side, so the sensor never sees it; the 33K also limits fault current if the signal wire is shorted. J19 goes to AIN4, J10 to AIN5. The reference is +5VA on REFP1 through R8 (0R), with C51 (100 nF) at the SoM pins; REFP1 sits at AVDD, so its buffer must be off.

### Load cells (J12, J18)

Two channels of an NAU7802 (U2), pin 1 E+ (NAU7802_1_AVDD), pin 2 E− (GND1), pin 3 A+, pin 4 A−. The NAU7802 runs from +5V and sits on I2C1 at 0x2A through the Q30 level shifter; its data-ready goes to GP45 through Q31. Its ground, GND1, joins GND only at NT1.

### J9 low-voltage inputs

| Pin | Net | Function |
| --- | --- | -------- |
| 1 | MP_PADDLE_SW | Paddle switch, to GP20. No pull resistor on the board; see the E9 note below. |
| 2 | AIN_POT_1 | Analog input, through a 10K / 15K divider to GP47 |
| 3 | FLOW_METER_DATA | Flow meter pulses, to GP23, pulled up by R57 (10K) to FLOW_METER_VCC |
| 4 | TANK_CIN | Tank level probe, to FDC1004 CIN4 through C23 |
| 5 | BOILER_CIN | Boiler level probe, to FDC1004 CIN3 through C24 |
| 6 | FLOW_METER_VCC | Flow meter supply, +5V by default (JP2) |
| 7 | GND | |
| 8 | TANK_REF | Tank reference electrode |

**Paddle switch and RP2350 erratum E9.** MP_PADDLE_SW has no pull resistor on the board, and it isn't known whether the GS3 MP's paddle switch needs a pull-up or a pull-down. The RP2350's internal pull-up works on every stepping, but on the A2 stepping, erratum E9 stops the internal pull-down from holding the pin low. When building for a GS3 MP, either make sure the SoM has an A4 RP2350, confirm that the switch needs a pull-up (which seems likely), or be prepared to bodge in an external pull-down of 8.2K or less.

### Capacitive level sensing

* C23 and C24 are 68 pF DC-blocking capacitors, chosen by experimentation. They use a hand-solderable package, placed where they're easy to swap.
* JP12 (bridged) ties the FDC1004's SHLD1 and SHLD2, which single-ended measurements with CAPDAC need (SHLD2 floats in that mode). Open it only for differential use; with it bridged, never configure a differential measurement.

## Expansion and debug

* **I2C0** (GP16/GP17): DS3231 RTC (U13, 0x68) with a backup battery on CN12, Qwiic connectors J15/J16, header J4 (GND, 3V3, SDA, SCL) and the display connectors. R13/R15 (10K) are the bus pull-ups; they're discrete, rather than the MCU's, to avoid problems at reset. Connected devices need no extra pull-ups.
* **Display:** J13 and J17 carry SPI0, I2C0, the display control signals and +3V3. J17 is an 18-pin EyeSPI FFC connector. J13 is a 2×10 IDC header with the same signals, which this project calls "EyeDC-SPI" (EyeSPI over IDC); it also carries +5V (pin 20) and VIN (pin 19). SPI0 also serves the microSD slot J3, which runs in 1-bit SPI mode with its own chip select (GP41) and card detect (GP42). Using an SD card is optional.
* **J6** (UART to the ESP32-C6, pin compatible with the LM IoT module): pin 1 GND, 2 ESP32 GPIO1, 3 ESP32 GPIO0, 4 signalling level, level shifted by Q29. See JP11 and JP1 below for the level.
* **J7** (3-pin): supply (JP5), analog input to GP46, GND. **J21** (3-pin): +3V3, GP40, GND.
* **Debug:** J25 (JST SH, SWCLK/GND/SWDIO) and the Femtoprobe footprint U6, which also connects to the SoM's USB hub port 4 and the ESP32-C6's UART0. Buttons: SW3 RP2350 BOOTSEL, SW4 RP2350 reset, SW5 ESP32-C6 reset (EN), SW6 ESP32-C6 GPIO9 (download mode).
* **USB-C** (J14): to the SoM's upstream hub port through FL1 (ECLAMP8052P) with D24 on VBUS. R69 (1M) and C52 (100 V) tie the shield to GND. C67 and C68 carry the USB pair's return current where it changes layer; they aren't decoupling capacitors.

## Solder jumpers

| Jumper | Default | Function |
| ------ | ------- | -------- |
| JP1 | Open | Selects +5V or +3V3 for JP11 pad 3 |
| JP2 | Pads 1–2 (+5V) | FLOW_METER_VCC: +5V or +3V3 |
| JP3, JP6 | Bridged | RTD 1 (J11) 2-wire |
| JP4 | Open | RTD 1: J11 pin 2 to AIN3 |
| JP5 | Pads 1–2 (+3V3) | J7 pin 1 supply: +3V3 or +5V. Before choosing +5V, change R67 and fit R68 as a divider, because GP46 is an ADC pin limited to IOVDD + 0.5 V. |
| JP7, JP8 | Bridged | RTD 2 (J8) 2-wire |
| JP9 | Open | RTD 2: J8 pin 2 to AIN11 |
| JP10 | Open | ESP32-C6 GPIO15 (JTAG source strap): bridge to GND or +3V3 to set it |
| JP11 | Pads 1–2 (VIN) | J6 signalling level: VIN, or JP1's selection (pads 2–3) |
| JP12 | Bridged | FDC1004 SHLD1–SHLD2 tie |

**Check JP11 before connecting an LM IoT module to J6.** By default, JP11 pads 1–2 are bridged, which puts J6's signals at VIN (about 12 V), and JP1 is open. For 5 V signalling, cut JP11 between pads 1 and 2, bridge pads 2 and 3, and bridge JP1 to +5V.

## Environment

The APEC SoM's USB hub is rated for 0–70 °C, which limits the whole controller. Mount the board away from the boilers and other heat sources.
