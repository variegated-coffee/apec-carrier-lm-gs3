# Changelog

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