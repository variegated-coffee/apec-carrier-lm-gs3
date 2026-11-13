# APEC carrier (LM GS3) IO assignments

How the carrier uses the APEC SoM's pins, revision EV3. Generated from the EV3 schematic netlist; net names match the schematic. SoM pin numbers are castellation numbers. See the SoM's [ENGINEERING-NOTES.md](https://github.com/variegated-coffee/apec-som/blob/main/ENGINEERING-NOTES.md) and [IO.md](https://github.com/variegated-coffee/apec-som/blob/main/IO.md) for what the SoM uses internally.

## RP2350

| GPIO | SoM pin | Net | Function |
| ---- | ------- | --- | -------- |
| GP1 | 23 | EYESPI_BL | Display backlight (J13 pin 18, J17 pin 2). PWM0B. |
| GP2 | 22 | EYESPI_SCK | SPI0 SCK: display (J13, J17, through series resistors) and microSD (J3) |
| GP3 | 21 | EYESPI_MOSI | SPI0 TX |
| GP4 | 20 | EYESPI_MISO | SPI0 RX |
| GP5 | 19 | 24V_STEAM_OUT | 24 V output CN13, through U16 channel B and Q11. PWM2B. |
| GP6 | 18 | EYESPI_DC | Display data/command |
| GP7 | 17 | EYESPI_RES | Display reset |
| GP12 | 16 | EYESPI_DISP_CS | Display chip select |
| GP13 | 15 | EYESPI_SD_CS | Display module's SD card chip select |
| GP14 | 14 | EYESPI_MEM_CS | Display module's memory chip select |
| GP15 | 13 | EYESPI_TS_CS | Touch screen chip select |
| GP16 | 12 | QWIIC_SDA | I2C0 SDA |
| GP17 | 11 | QWIIC_SCL | I2C0 SCL |
| GP18 | 10 | EYESPI_GP1 | Display GPIO 1 |
| GP19 | 9 | EYESPI_GP2 | Display GPIO 2 |
| GP20 | 8 | MP_PADDLE_SW | Paddle switch input (J9 pin 1) |
| GP21 | 7 | EYESPI_INT | Display interrupt |
| GP22 | 6 | EYESPI_BUSY_TE | Display busy / tearing effect |
| GP23 | 73 | FLOW_METER_DATA | Flow meter pulses (J9 pin 3), pulled up to FLOW_METER_VCC by R57. PWM3B, so a PWM slice can count pulses. |
| GP24 | 72 | TEA_SOLENOID_OUT | Tea solenoid relay U11, through Q24 |
| GP25 | 71 | FG_TACHO_OUT_5V | Gear pump tacho (CN9 pin 8), direct and 5 V. PWM4B, so a PWM slice can measure frequency. |
| GP26 | 70 | RELAY_GROUP_SOLENOID | Group solenoid relay U9, through Q22 |
| GP33 | 65 | RELAY_FILL_SOLENOID | Fill solenoid relay U10, through Q25 |
| GP34 | 64 | PUMP_OUT | Rotary pump relay K1, through Q23 |
| GP35 | 63 | FG_DIRECTION_3V3 | Gear pump direction, buffered to 5 V by U4 (CN9 pin 4) |
| GP36 | 62 | SERVICE_BOILER_EN | Service boiler SSR output CN11, through Q26 |
| GP37 | 61 | BREW_BOILER_EN | Brew boiler SSR output CN10, through Q27 |
| GP40 | 32 | AUX1 | Spare, on J21 pin 2 |
| GP41 | 31 | SD_CS | microSD chip select |
| GP42 | 30 | SD_DET | microSD card detect |
| GP43 | 29 | 24V_TEA_SOLENOID | 24 V output CN14, through U16 channel A and Q12. PWM9B. |
| GP44 | 28 | FG_SPEED_IN_3V3 | Gear pump speed, buffered to 5 V by U3 (CN9 pin 7). PWM10A. |
| GP45 | 27 | NAU7802_1_DRDY_3V3 | NAU7802 data ready, level shifted by Q31 |
| GP46 | 26 | AIN_POT_2_2V5 | Analog input from J7 pin 2 (ADC6). R67 = 0R, R68 not fitted. |
| GP47 | 25 | AIN_POT_1_2V5 | Analog input from J9 pin 2 (ADC7), through a R28 10K / R29 15K divider |

Every RP2350 GPIO the SoM exposes is used.

### Buses

| Bus | Pins | Devices |
| --- | ---- | ------- |
| SPI0 | GP2/GP3/GP4 | EyeSPI display (J13, J17), microSD (J3, chip select GP41). The display module's own chip selects are on GP12–GP15. |
| I2C0 | GP16 (SDA), GP17 (SCL) | DS3231 RTC (U13) at 0x68, Qwiic connectors J15 and J16, I2C header J4, display connectors J13 and J17. Pull-ups R13/R15 (10K) on the carrier; add no others. |
| I2C1 | GPIO38/39 (SoM internal, castellations 34/33, INT_SDA/INT_SCL) | NAU7802 (U2) at 0x2A, through the Q30 level shifter, and the SoM's FDC1004 at 0x50 |
| SPI1 | SoM internal | Not used on the carrier |
| UART1 | SoM internal | RP2350–ESP32-C6 link |

### Notes

* GP23 and GP25 see 5 V signals: FLOW_METER_DATA is pulled up to FLOW_METER_VCC (+5 V with JP2 in its default position), and FG_TACHO_OUT_5V comes straight from the gear pump. The RP2350's fault-tolerant GPIOs accept up to 5.5 V only while IOVDD is at 3.3 V.
* GP46 and GP47 are ADC pins, limited to IOVDD + 0.5 V. Before setting J7's supply to +5V with JP5, change R67 and fit R68 as a divider.
* The 24 V outputs are isolated: U16 (ADuM3221) drives Q11 and Q12 from the 24 V side.

## ADS124S08 (on the SoM)

| Input | Net | Function |
| ----- | --- | -------- |
| AIN0 | ADC_AIN0 | RTD 1, J11 pin 4 (excitation). JP3 (bridged by default) also joins it to J11 pin 3. |
| AIN1 | ADC_AIN1 | RTD 1, J11 pin 3 through R6 |
| AIN2 | ADC_AIN2 | RTD 1, J11 pin 2 through R7 |
| AIN3 | ADC_AIN3 | RTD 1, J11 pin 2 through JP4 (open by default) |
| AIN4 | ADC_AIN4 | Pressure transducer on J19, through R17 (33K / 47 nF filter) |
| AIN5 | ADC_AIN5 | Pressure transducer on J10, through R10 (33K / 47 nF filter) |
| AIN6 / REFP1 | ADC_REFP1 | +5VA through R8 (0R), C51 to GNDA: ratiometric reference for the pressure inputs |
| AIN7 / REFN1 | GNDA | Pressure reference ground |
| REFP0 / REFN0 | ADC_REFP0, ADC_REFN0 | Across R5 (2K2, 0.01 %), the RTD reference resistor, low side. REFP0 through R9 (10K), REFN0 through R34 (0R). The top of R5 (REF_R) is pin 1 of J11 and J8; JP6 and JP8 (bridged by default) also join it to pin 2. |
| AIN8 | ADC_AIN8 | RTD 2, J8 pin 4 (excitation). JP7 (bridged by default) also joins it to J8 pin 3. |
| AIN9 | ADC_AIN9 | RTD 2, J8 pin 3 through R18 |
| AIN10 | ADC_AIN10 | RTD 2, J8 pin 2 through R20 |
| AIN11 | ADC_AIN11 | RTD 2, J8 pin 2 through JP9 (open by default) |

The GS3 firmware reads the brew pressure on AIN4 and the steam pressure on AIN5. [FW_NEXT.md](FW_NEXT.md) has the reference-buffer settings EV3 needs.

## FDC1004 (on the SoM)

| Input | Net | Function |
| ----- | --- | -------- |
| CIN3 | CIN3 | Boiler level probe (J9 pin 5, BOILER_CIN), through C24 (68 pF) |
| CIN4 | CIN4 | Tank level probe (J9 pin 4, TANK_CIN), through C23 (68 pF) |
| CIN1, CIN2 | — | Not connected |
| SHLD1, SHLD2 | FDC_SHLD1, FDC_SHLD2 | Tied together by JP12 (bridged by default) |

With JP12 bridged, use single-ended measurements only; see FW_NEXT.md, item 5.

## ESP32-C6 (on the SoM)

| Signal | SoM pin | Net | Function |
| ------ | ------- | --- | -------- |
| GPIO0 | 1 | ESP32_GP0 | J6 pin 3, level shifted by Q29 (LM IoT module UART) |
| GPIO1 | 2 | ESP32_GP1 | J6 pin 2, level shifted by Q29 |
| U0RXD | 3 | ESP32_U0RX | Femtoprobe (U6) UART |
| U0TXD | 4 | ESP32_U0TX | Femtoprobe (U6) UART |
| GPIO15 (JTAG select strap) | 102 | — | JP10: open by default; bridge to GND or +3V3 to set the strap |
| GPIO9 (boot mode) | 103 | ESP32_GPIO9 | SW6 to GND (hold while resetting to enter download mode) |
| EN | 104 | ESP32_EN | SW5 to GND (reset) |
| JTAG (TDO, TCK, TDI, TMS) | 98–101 | — | Not connected |

## USB, debug and reset

| Signal | SoM pin | Connected to |
| ------ | ------- | ------------ |
| USB upstream D+/D− | 88/89 | USB-C J14, through FL1 (ECLAMP8052P) |
| USB_VBUS | 87 | J14 VBUS, through FB1 (also feeds the U7 boost converter) |
| USB port 4 D+/D− | 94/95 | Femtoprobe U6 |
| USB port 3 | 91/92 | Not connected |
| RP2350 SWCLK/SWDIO | 68/67 | J25 (SWD) and U6 (Femtoprobe), through R46/R47 (100R) |
| RP2350 RUN | 66 | SW4 to GND (reset) |
| RP2350 BOOTSEL | 69 | SW3 to GND |
