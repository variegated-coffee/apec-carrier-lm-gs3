# Changelog

## EV2 to EV3

* Added power rail indicator LEDs (D19-D23) with series resistors (R77-R81), one per rail: +24V, 12V_ISO, 12V_ACDC, USB_VBUS and 3V3
* Named the AC/DC brick output net 12V_ACDC (previously unnamed), and assigned it the 12V-1A net class, same as 12V_ISO
* Added isolation slots to the board outline
  * A diagonal slot in the 24V section, next to J5 and Q11
  * An L-shaped slot alongside the U10/U11 mechanical relays
* Minor rerouting around the new LEDs, and of RP2350_SWCLK_POST_R

## EV1 to EV2

* Corrected pins on mechanical relay FETs (gate and drain swapped)
* Added gate series resistors for mechanical relay FETs
* Slight rerouting to move signal traces further away from the mechanical relays
* Moved fuse holders slightly closer together to ensure a better fuse fit
* Used slightly larger pads for U5, U9-11
* Added J6, a UART connector to the ESP32-C6, pin compatible with the LM IoT module
  * Level shifted, with signalling voltage selectable by solder jumper on the back side, 5V is default, 3V3 is selectable
  * If you really want to, and have a bodge wire, you could even shift it to 12V.
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