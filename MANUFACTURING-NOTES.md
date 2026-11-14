# APEC carrier (LM GS3) manufacturing notes

How to fabricate, assemble and bring up the carrier, revision R1A. R1A hasn't been built yet; EV2 has. Values below come from the R1A design files (KiCad 10). The production files are set up for JLCPCB, but any fab and assembler with the same capabilities will do.

For pinouts and configuration, see [ENGINEERING-NOTES.md](ENGINEERING-NOTES.md).

## PCB fabrication

| Parameter | Value |
| --------- | ----- |
| Layers | 6 |
| Thickness | 1.6 mm |
| Material | FR-4 |
| Copper weight | 35 µm (1 oz) on all layers |
| Surface finish | Not specified in the design; choose ENIG or lead-free HASL |
| Vias | Through-hole only |
| Default net class | 0.2 mm track, 0.2 mm clearance, 0.6 mm via with 0.3 mm drill |
| Smallest allowed by the rules | 0.09 mm (3.5 mil) track and space |
| Impedance control | Not specified in the design |
| Milled slots | **Yes.** The board outline includes internal isolation slots in the mains and 24 V areas; check that the fab mills them. |

**Stackup** (from the board file, top to bottom):

| Layer | Use | Thickness |
| ----- | --- | --------- |
| F.Cu | Components, signals | 35 µm |
| Prepreg | | 0.1 mm |
| In1.Cu | Ground | 35 µm |
| Core | | 0.535 mm |
| In2.Cu | Power | 35 µm |
| Prepreg | | 0.1 mm |
| In3.Cu | Signals (including the USB pair) | 35 µm |
| Core | | 0.535 mm |
| In4.Cu | Ground | 35 µm |
| Prepreg | | 0.1 mm |
| B.Cu | Signals | 35 µm |

**Mains.** The mains tracks are 2.5 mm wide on 35 µm copper, and mains nets keep 2.4 mm from other copper. Don't let the fab reduce either.

## Assembly

* **Assembly service parts:** the SMD parts carry LCSC part numbers in the schematic. The production files are generated with the Fabrication Toolkit plugin for KiCad, which writes them to `production/` (Gerber zip, BOM, placement file, IPC netlist). Its settings are in `fabrication-toolkit-options.json`. The plugin takes LCSC numbers from the footprints on the PCB, so sync them from the schematic first (see the checklist below).
* **Fitted, but excluded from the BOM**, so source and fit these yourself:
  * U1 (APEC SoM), U5 (AC/DC module), U6 (Femtoprobe), U9–U11 and K1 (relays), Q11 and Q12 (TO-220 MOSFETs), F2 and F3 (fuse clips), RV1–RV5 (varistors)
  * C1, C2, C4, C53, C54, C56, C58, C63
  * J2–J13, J18, J19, J21 (connectors)
* **Not fitted (DNP):**
  * C31, C32, C35, C36, C37, C39, C42, C43, C44, C45, C46, C47, C48, C57: spare footprints, with no value chosen, in case a build needs more bulk capacitance
  * C8, R65, R68
  * CN9–CN14: fit the gear pump, SSR, RTC battery and 24 V output connectors that your build uses
* **Through-hole parts** are on the top side: the Phoenix terminal blocks J2, J5 and J9, the relays, U5, Q11/Q12, the fuse clips, the varistors, the IDC header J13 and the pin headers.
* **The APEC SoM** (U1) mounts on the `APEC_SoM_SMD` footprint: reflow or hand-solder its castellations. The footprint has keepout areas under the module's exposed test pads, thermal vias and bottom-side traces; keep exposed copper out of them.
* **Rotations:** check the placement preview for rotation errors before confirming the order.

## Before ordering

1. **Sync part numbers:** in the PCB editor, run Tools → Update PCB from Schematic with "Update footprint fields" enabled, so the LCSC numbers on the footprints match the schematic.
2. **DRC:** refill all zones and run DRC. Review every error; the design should have no unconnected items. Warnings that a footprint's library isn't found are expected for the embedded-only parts listed in [THIRD_PARTY.md](THIRD_PARTY.md).
3. **Outputs:** generate the production files and check the isolation slots in a Gerber viewer before uploading.

## Bring-up

**Do the first power-up from USB, with no mains connected.** The board runs from USB alone: U7 boosts USB VBUS to about 11.5 V on VIN.

1. **USB power.** Connect J14 to a USB port (a powered hub if the port trips on the inrush). Check the rail LEDs: on USB power alone, only USB_VBUS and +3V3 light. +24V and 12V_ISO stay off until J5 is fed, and 12V_ACDC until mains is connected. There's no VIN LED, so measure VIN (about 11.5 V), +3V3, +5V and +5VA.
2. **Program the SoM.** Hold SW3 (BOOTSEL) while pressing SW4 (reset) to enter the RP2350's USB bootloader. SW6 (ESP32-C6 GPIO9) and SW5 (ESP32-C6 reset) put the ESP32-C6 in download mode the same way. SWD is on J25, and on the Femtoprobe (U6) if fitted.
3. **Outputs.** With firmware running, the indicator LEDs show each relay, SSR and 24 V output being switched.
4. **24 V section.** Feed J5 from a current-limited 24 V bench supply (pin 2 +24V, pin 1 return). Check the +24V and 12V_ISO LEDs and the CN13/CN14 outputs with a dummy load.
5. **Mains.** Fit the correct fuse (see the fuse table in ENGINEERING-NOTES.md). Only connect mains with the board in a closed, earthed enclosure. That can be in the machine with all wiring done, or on the bench, preferably through an isolation transformer. Always use a GFCI-protected (RCD) outlet. Never touch the board while it's connected to mains.
