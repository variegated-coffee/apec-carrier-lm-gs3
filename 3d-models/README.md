# Vendor 3D models

The PCB references these STEP models as `${KIPRJMOD}/3d-models/<file>`. Their vendors' terms don't allow redistribution, so they aren't in the repository. The design is complete without them; they're only needed for 3D views, renders and enclosure work.

To get full renders, download each model from its manufacturer (or the service named) and save it here under the file name in the first column. Rename the download if needed. Everything in this directory except this file is ignored by git.

| File | Part | Used by | Where to get it |
| ---- | ---- | ------- | --------------- |
| `1757129.stp`, `1758128.stp` | Phoenix Contact 1757129 and 1758128 | J2 (mains, 13 positions) | Phoenix Contact product pages, or TraceParts |
| `1757077.stp`, `1758076.stp` | Phoenix Contact 1757077 and 1758076 | J9 (low-voltage inputs, 8 positions) | Phoenix Contact product pages, or TraceParts |
| `1757019.stp`, `1758018.stp` | Phoenix Contact 1757019 and 1758018 | J5 (24 V input, 2 positions) | Phoenix Contact product pages, or TraceParts |
| `280370-1.stp` | TE Connectivity 280370-1 | CN10–CN14, J8, J11 | TE Connectivity product page, or TraceParts |
| `280371-2.stp` | TE Connectivity 280371-2 | J6, J10, J19 | TE Connectivity product page, or TraceParts |
| `3519.STEP` | Keystone 3519 fuse clip | F2, F3 | Keystone Electronics product page |
| `ALDP112.stp` | Panasonic ALDP112 relay | U9, U10, U11 | Panasonic Industry product page |
| `ece10us12.stp` | XP Power ECE10US12 AC/DC module | U5 | XP Power product page |
| `IPL1-104-01-L-D-K.stp` | Samtec IPL1-104-01-L-D-K | CN9 | Samtec product page |
| `BM03B-SRSS-TB.STEP` | JST BM03B-SRSS-TB | J25 (SWD) | JST product page, or SnapMagic |
| `BM04B-SRSS-TB_LF__SN_.step` | JST BM04B-SRSS-TB | J15, J16 (Qwiic) | JST product page, or SnapMagic |

J8 and J11 use the `280370-1` model although their footprint is `TE_280371-2`, and J12, J18, J17 and FL1 have no 3D model. U1 (the APEC SoM) has no model either: `APEC_SoM.step` from [variegated-coffee/kicad-footprints](https://github.com/variegated-coffee/kicad-footprints) isn't aligned to the `APEC_SoM_SMD` footprint yet.
