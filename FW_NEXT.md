# Firmware changes needed for EV3

Hardware changes in EV3 that the firmware has to follow. Code references are to
`variegated-umbrella/variegated-rs`.

| Part | EV2 | EV3 |
|---|---|---|
| R9 (REFP0 filter) | 4K99 0.1% | 10K 1% |
| R34 (REFN0 filter) | 4K99 | 0R |
| R8 (REFP1 feed from +5VA) | 33K | 0R |
| C51 (REFP1–REFN1 bypass) | — | 100 nF |
| C9 / C12 (pressure filter caps) | connector side of R10 / R17 | ADC side of R10 / R17 |
| JP12 (FDC1004 SHLD1–SHLD2 tie) | — | bridged |

Items 1–3 differ between EV2 and EV3, so the firmware needs a way to tell the board
revisions apart (build feature, config setting or a hardware strap — not decided yet).

## 1. ADS124S08: RTD reference buffer (REF0) — required

`crates/variegated-ads124s08/src/lib.rs`, `measure_ratiometric_low_side`, lines 365–366.
Today both reference buffers are disabled (the `n_` bits are active-low, so `true` = off).

For EV3, enable the positive buffer and keep the negative one off:

```rust
config.refctrl.n_refp_buf = false; // REFP0 ≈ I_IDAC × 2k2: buffered, so R9 (10K) adds no error
config.refctrl.n_refn_buf = true;  // REFN0 is at AVSS: datasheet recommends the buffer off
```

Why: an unbuffered reference input looks like ~250 kΩ (4 µA/V typical, ±6 µA/V max). Through
R9 = 10 kΩ that is a ~4 % gain error. Buffered, the input current is ±15 nA, i.e. ~0.15 mV
across R9 — negligible, which is also why R9 can be a 1 % part.

Do not run EV3 hardware with the current firmware: with the buffer off, R9 sets that ~4 % error.

EV2: if this change also runs on EV2 hardware, its error roughly halves (REFN0 is still
unbuffered through 4K99), so EV2's correction factor in item 2 has to be re-derived — or keep
EV2 on the current behaviour.

## 2. RTD reference resistance — required (EV3)

`firmwares/variegated-gs3-firmware/src/main.rs`, lines 2812 (brew) and 2841 (steam):
`2200.0 / 1.03` → `2200.0` on EV3, once item 1 is in.

The `/ 1.03` was compensating the reference-input loading described in item 1. Verify the
readings against a known resistor (or decade box) after the change.

## 3. ADS124S08: pressure reference buffer (REF1) — required (EV3)

`crates/variegated-ads124s08/src/lib.rs`, `measure_single_ended` (line 322) — or whichever
function the pressure channels use after item 4. It currently inherits the default REFCTRL:
REFP buffer on, REFN buffer off.

On EV3, REFP1 sits at AVDD (R8 = 0R), and the datasheet says to disable the positive buffer
when REFPx is close to AVDD:

```rust
config.refctrl.n_refp_buf = true; // REFP1 ≈ AVDD: buffer off (datasheet); R8 = 0R, so no loading error
config.refctrl.n_refn_buf = true; // REFN1 = GNDA at AVSS
```

EV2: keep the buffer on. With R8 = 33K, an unbuffered REFP1 reads ~12 % low.

## 4. Pressure: measure against AIN7 instead of AINCOM — recommended (EV2 and EV3)

`firmwares/variegated-gs3-firmware/src/main.rs`, lines 2823–2825 (brew, AIN4) and
2852–2854 (steam, AIN5): `SingleEnded(Mux::AINx, ReferenceInput::Refp1Refn1, 5.0)`.

`SingleEnded` measures against AINCOM, which the SoM ties to its digital GND. AIN7 is REFN1 and
is tied to GNDA right next to the GNDA–GND join, i.e. the same ground as the sensors and the
reference. Measure AIN4–AIN7 and AIN5–AIN7 instead, e.g. with
`measure_differential(Mux::AIN4, Mux::AIN7, ReferenceInput::Refp1Refn1, PGAGain::Gain1)`
(Gain1 keeps the PGA bypassed, as today). The 0.5–4.5 V mapping and the 5.0 V full scale are
unchanged.

This needs a differential variant in `MeasurementType`
(`crates/variegated-hal/src/adc/ads124s08.rs`, line 18), or a negative-input argument on
`SingleEnded`.

Watch out: `measure_differential` (`lib.rs`, line 386) does not set a data rate, so it falls
back to the register default (20 SPS), while `measure_single_ended` sets 200 SPS. Set
`config.datarate.rate = DataRate::SPS200` (or make it a parameter) to keep the current pressure
update rate, and apply item 3's buffer settings there as well.

## 5. FDC1004: single-ended measurements only — constraint (EV3)

JP12 ties SHLD1 and SHLD2, so SHLD1 drives the carrier's shield pours and the SoM's CIN3/CIN4
guards (in single-ended mode with CAPDAC, SHLD2 floats). With JP12 bridged, never configure a
differential measurement (CHB = CINx): both shield drivers would then be active and shorted
together.

The current code is fine — `configure_single_measurement` always sets CHB = CAPDAC
(`crates/variegated-fdc1004/src/lib.rs`, line 536). Add a comment there and next to the CIN3 /
CIN4 sensor setup in the GS3 firmware, and a guard if a differential API is ever added.

## 6. RTD excitation current — optional

Both RTDs run at `IDACMagnitude::Mag1000uA` (`main.rs`, lines 2812 and 2841). In a PT1000 that
is ~1.4 mW of self-heating at brew temperature. Suggested: `Mag500uA`.

- No hardware change: the input and reference filters are linear RC networks, so their corner
  frequencies (~323 Hz inputs, ~308 Hz reference) don't depend on the excitation current.
- The reading is R_RTD / R_REF whatever the current, so no rescaling is needed. Keep
  `PGAGain::Gain1`: the ratio is already ~0.7 at 150 °C, so a higher gain would overflow.
- V_REF = I × 2.2 kΩ: 1.1 V at 500 µA. The ADS124S08 needs at least 0.5 V, so 250 µA
  (0.55 V) is the practical floor.
- Self-heating drops to ~0.35 mW at 500 µA. The worst-case error from the REFP buffer's input
  current (±15 nA through R9) grows to ~0.05 °C, which is still negligible.

## 7. Legacy HMI devices on I2C1 — required (EV3)

`firmwares/variegated-gs3-firmware/src/main.rs` puts two MCP23017s (0x20 buttons, 0x21 LCD) and
the TLC59108 (0x40) on the internal bus (`board-cfg.toml`: I2C1, GPIO38/39), next to the
FDC1004. EV3 removed the character display and button connectors (the Unimodule display on
J13 / J17 replaces them, on I2C0), so on EV3 I2C1 (`INT_SCL` / `INT_SDA`) only reaches the
FDC1004 on the SoM and the NAU7802 (0x2A, through Q30).

`btn_mcp23017.init().await.unwrap()` (line 2586) would panic at boot on EV3, since nothing
answers at 0x20. Skip the MCP23017 / TLC59108 setup on EV3.

## No firmware change needed

- C9 / C12 on the ADC side (33K / 47 nF, ~100 Hz): each pressure input has its own RC and the
  ADC input current is nA-level, so no extra settling delay is needed when switching channels.
- C51 and R8 are hardware-only, apart from item 3.
