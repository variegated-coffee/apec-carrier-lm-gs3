# Third-party content

Everything in this repository is licensed under the CERN Open Hardware Licence Version 2 – Permissive (CERN-OHL-P-2.0, see [LICENSE.txt](LICENSE.txt)), except the content listed below, which keeps its original licence.

## Library files included in this repository

| Files | Used by | Source | Licence | Changes made here |
| ----- | ------- | ------ | ------- | ----------------- |
| `3519/` (`3519.kicad_sym`, `FUSE_3519.kicad_mod`) | F2, F3 (Keystone 3519 fuse clips) | SnapMagic (formerly SnapEDA), <https://www.snapeda.com/> | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) with SnapMagic's Design Exception 1.0, subject to SnapMagic's [terms of service](https://www.snapeda.com/about/terms/) | None recorded. The download's STEP model and import guide aren't included. |
| `ECLAMP8052P_TCT/` (`ECLAMP8052P.TCT.kicad_sym`, `FIL_ECLAMP8052P.TCT.kicad_mod`) | FL1 (Semtech ECLAMP8052P USB filter) | SnapMagic | As above | None recorded. Import guide not included. |
| `IPL1-104-01-L-D-K/SAMTEC_IPL1-104-01-L-D-K.kicad_mod` | CN9 (Samtec IPL1-104-01-L-D-K) | SnapMagic | As above | None recorded. The download's symbol, STEP model and import guide aren't included. |
| `TE_280370-1/TE_280370-1.kicad_mod` | CN10–CN14 (TE Connectivity 280370-1) | SnapMagic | As above | None recorded. The download's symbol and import guide aren't included. |
| `TE_280371-2/TE_280371-2.kicad_mod` | J6, J8, J10, J11, J12, J18, J19 (TE Connectivity 280371-2) | SnapMagic | As above | None recorded. The download's symbol and import guide aren't included. |
| `BM04B-SRSS-TB_LF__SN_/` (`BM04B-SRSS-TB_LF__SN_.kicad_sym`, `JST_BM04B-SRSS-TB(LF)(SN).kicad_mod`) | J15, J16 (JST BM04B-SRSS-TB Qwiic connectors) | SnapEDA, <https://www.snapeda.com/> | CC BY-SA 4.0 with Design Exception 1.0 | None recorded. The download's STEP model and import guide aren't included. |
| `APEC_Carrier.pretty/RV_Disc_D9mm_W5.4mm_P5mm_Drill0.9mm.kicad_mod` | RV1–RV5 | Derived from `RV_Disc_D9mm_W5.4mm_P5mm` in the KiCad footprint library | [CC BY-SA 4.0 with the KiCad libraries exception](https://www.kicad.org/libraries/license/) | Drill enlarged to 0.9 mm and pads to 2.0 mm, for 0.6 mm varistor leads. Same pad positions and outline. |

Each vendor directory above also contains a `LICENSE.md` with the same information. The SnapMagic directories keep the `License.txt` that came with the download.

The APEC SoM, Femtoprobe and Variegated Coffee logo libraries (`APEC_SoM.kicad_sym`, `APEC_SoM.pretty/`, `Femtoprobe.kicad_sym`, `Femtoprobe.pretty/`, `Graphics.kicad_sym`, `VariegatedCoffeeLogo.pretty/`) are copied from [variegated-coffee/kicad-footprints](https://github.com/variegated-coffee/kicad-footprints) (commit `1eca8d7`) and are CERN-OHL-P-2.0 like the rest of this repository. `APEC_SoM_SMD` here has keepout areas under the module that the published version doesn't have yet, and the `Femtoprobe SMT` 3D model paths point to `${KICAD10_3DMODEL_DIR}`.

## Content embedded in the design files

The schematic and PCB contain copies of symbols, footprints and 3D model references from these libraries. The library files themselves aren't included.

| Library | Used for | Licence |
| ------- | -------- | ------- |
| KiCad standard libraries | Most symbols and footprints | [CC BY-SA 4.0 with the KiCad libraries exception](https://www.kicad.org/libraries/license/): designs that use them, and files generated from them, aren't covered by the licence. |
| Alternate KiCad Library (AKL) by Dawid Cisło, <https://github.com/DawidCislo/Alternate-KiCad-Library> | FB1 (symbol and footprint), U2 footprint, D11 and D13 footprints | CC BY-SA 4.0 with the same exception as the KiCad libraries; no attribution is required in designs. |
| Ultra Librarian (Cadence), <https://www.ultralibrarian.com/> | U9–U11 (Panasonic ALDP112), U5 (XP Power ECE10US12), J25 (JST BM03B-SRSS-TB), J17 footprint (Molex), U2 symbol (Nuvoton NAU7802) | Ultra Librarian's terms grant no right to redistribute the library files, so only the copies embedded in the schematic and PCB are included. Download the parts from Ultra Librarian if you need to edit them as library parts. |

## 3D models

Vendor 3D models (Phoenix Contact, TE Connectivity, Keystone, Panasonic, Samtec, JST, XP Power) aren't included, because their terms don't allow redistribution. The PCB looks for them in `3d-models/`; [3d-models/README.md](3d-models/README.md) lists each file and where to download it. Without them, those parts are missing from 3D views and renders, but the design is otherwise complete. The KiCad standard 3D models are referenced through `${KICAD10_3DMODEL_DIR}`.
