# Changelog

## EV2 to EV3

* Added power rail indicator LEDs (D19-D23) with series resistors (R77-R81), one per rail: +24V, 12V_ISO, 12V_ACDC, USB_VBUS and 3V3
* Named the AC/DC brick output net 12V_ACDC (previously unnamed), and assigned it the 12V-1A net class, same as 12V_ISO
* Added isolation slots to the board outline
  * A diagonal slot in the 24V section, next to J5 and Q11
  * An L-shaped slot alongside the U10/U11 mechanical relays
* Minor rerouting around the new LEDs, and of RP2350_SWCLK_POST_R
* Added an EyeDC-SPI connector
* Removed character display and buttons connectors (unnecessary with the Unimodule display module)
* Changed to a right-angle USB-C connector
* Reworked the ADS124S08 reference inputs, which were loaded by series resistance on unbuffered reference inputs (~4 % gain error on the RTDs, previously compensated in firmware)
  * R34 (REFN0) changed from 4K99 to 0R
  * R9 (REFP0) changed from 4K99 0.1% to 10K 1%, the same part as R19/R28/R57, keeping the reference filter close to the RTD input filters (~308 Hz vs ~323 Hz). Requires the REFP0 buffer to be enabled in firmware, which also makes R9's tolerance and tempco irrelevant
  * R8 (REFP1) changed from 33K to 0R
  * Added C51, 100 nF across REFP1/REFN1 at the SoM pins
* Moved the pressure input filter caps C9/C12 to the ADC side of R10/R17, forming a ~100 Hz RC filter at the ADC pins, and keeping the capacitance off the transducer outputs
* Updated the APEC_SoM_SMD footprint with keepouts under the SoM's exposed thermal vias, bottom-side test pads, and bottom-side USB and flash traces
* Schematic naming and documentation cleanup
  * Renamed the FDC1004 shield nets to FDC_SHLD1/FDC_SHLD2 (previously C_SHIELD_2/C_SHIELD, with the numbering swapped relative to the SoM)
  * Relabelled the +12V power symbols as VIN, the name the net already had
  * Removed a dangling wire stub on J13 pin 7
  * Updated the I2C address summary, SD card note, and RTD reference filter note, and added notes for JP12, the SD card SPI bus selection resistors, and the FG304 ground separation
* Added FW_NEXT.md, listing the firmware changes the EV3 hardware needs
* Caps near USB D+- vias to/from In3 and correct impedance matching
* Repository cleanup for the open source release (no electrical changes)
  * The APEC SoM, Femtoprobe and logo libraries are now bundled from variegated-coffee/kicad-footprints, instead of being loaded from a local checkout
  * The EyeSPI connector symbol moved to a new project library, APEC_Carrier.kicad_sym
  * Removed unused libraries, and the library files of parts whose licences don't allow redistribution (they stay embedded in the schematic and PCB)
  * Vendor 3D models are no longer in the repository; 3d-models/README.md lists where to get them
  * Added README, licence, third-party notice, and engineering, IO and manufacturing notes

## EV1 to EV2

* Corrected pins on mechanical relay FETs (gate and drain swapped)
* Added gate series resistors for mechanical relay FETs
* Slight rerouting to move signal traces further away from the mechanical relays
* Moved fuse holders slightly closer together to ensure a better fuse fit
* Used slightly larger pads for U5, U9-11
* Added J6, a UART connector to the ESP32-C6, pin compatible with the LM IoT module
  * Level shifted, with the signalling voltage set by solder jumpers on the back side: VIN (about 12 V) by default, or 5V or 3V3 with JP11 and JP1
* Added and moved various silk screen texts
* Added two on-board LEDs, connected to the TLC59108 LED driver
* Added pull-down resistors for all relays and SSRs, to ensure the pumps, heating elements etc start as off
* Added 10K pull to ground instead of direct ground connection for tank ref
* Added 100R to 5V to keypad connector, to be isomorphic with the original board
* Changed SD Card to use SPI instead of SDIO (freeing up IO)
* Added true 24V section, with FETs and gate driver to drive 24V proportional solenoids
* Changed DC-de-biasing cap to 68 pF, based on experimentation
* Added LED indicators to SSR outputs, mechanical relay outputs, and 24V outputs. Switched signal FETs to dual FETs to drive the LEDs.
* Added two additional auxilliary connectors, one of which has a (by default inoperable) voltage divider 