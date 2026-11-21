# Earth GLO-90 heights

Copernicus GLO-90 land heights for the whole globe, converted for the Vintage Story mod **Earth**. Earth downloads
them once into `VintagestoryData/ModData/earth/glo90`, and every world and every later version of the mod reuses them.

- 26,475 one-degree squares (every square of land GLO-90 covers), 8.61 GB.
- Heights in whole metres on GLO-90's own 3″ grid (about 90 m), compressed without loss.
- The same data Earth gets by converting the original 70.8 GB of GeoTIFFs itself, which Earth keeps as its fallback.

## Files

The data is in the release [`egl-v1`](../../releases/tag/egl-v1): five parts of about 1.72 GB each, under GitHub's
2 GB limit per file, and a manifest.

| File | Bytes | Squares | From | To | SHA-256 |
|---|---|---|---|---|---|
| `glo90-egl1-part1of5.zip` | 1,723,127,338 | 3,534 | N00E006 | N27E025 | `51342809c7ac20c726aff0c977acb2515076ef873fe6ec8fd24f3a9eefc59c9e` |
| `glo90-egl1-part2of5.zip` | 1,723,306,569 | 2,806 | N27E026 | N42W088 | `1686334a96ce4049cb070fa297049077dd2e6f174daa1a55ba589e5c82da8172` |
| `glo90-egl1-part3of5.zip` | 1,723,155,645 | 3,796 | N42W089 | N59E136 | `67f36a2ba7bf891ffe4443a85de2666550ec9394a999c785402d5322dff5ef7b` |
| `glo90-egl1-part4of5.zip` | 1,723,302,957 | 6,537 | N59E137 | S11W069 | `7cd3c1d0b028f83fa5b12c31e0adb493e87df04e7829d4efb9e01793e3f4303f` |
| `glo90-egl1-part5of5.zip` | 1,724,390,980 | 9,802 | S11W070 | S90W180 | `e6f0d00876da0f4eabbbe961548479362e913974b6b85a06577a8470e83302e0` |
| `manifest.json` | 2,597,215 | | | | in this repository too |

Each part is an ordinary zip with no compression (the tiles are already compressed), so any unzip tool opens it. Each
holds `ATTRIBUTION.txt`, `eula_F.pdf` and its tiles, one file per square named like `N60E006.egl` (south-west corner).

### Version

`egl1` and the tag `egl-v1` name the tile format, not a mod version. Earth versions that read format 1 use these files
as they are, so the data is downloaded once. A new tile format would get a new tag and new files; this release stays.

### manifest.json

```json
{ "format": "egl", "formatVersion": 1, "converterVersion": 1, "tileCount": 26475, "tileBytes": 8614175289,
  "parts": [ { "name": "glo90-egl1-part1of5.zip", "bytes": 1723127338, "sha256": "…", "tiles": 3534,
               "first": "N00E006", "last": "N27E025" }, … ],
  "tiles": { "N00E006": [1, 102632, 74707, "31b31cac…"], … } }
```

For each square: the part (1–5), the byte offset of the tile's data inside that part, its length, and its SHA-256. A
client can fetch a single square with an HTTP range request on the part instead of the whole part.

## Tile format (EGLO, version 1)

Little-endian. `"EGLO"`, format (byte, 1), source (byte, 1 = GLO-90), converter (uint16, 1 = rounded to whole metres),
LatS and LonW (int16, the square's south-west corner), Cols and Rows (uint16), Min and Max (int16, metres), the source
GeoTIFF's size (int64), its ETag (byte length + ASCII), a checksum of the heights (uint64, FNV-1a over the int16
heights), the payload's length (int32) and the payload.

Rows run north to south (1,200 per square); columns per square follow GLO-90's latitude bands (1,200 below 50°, then
800, 600, 400, 240, 120). Height (r, c) is the point at latitude LatS + 1 − r / Rows and longitude LonW + c / Cols, as in
the source (PixelIsPoint). The payload is each height minus the median edge prediction from its west, north and
north-west neighbours (0 outside the square), zigzag-coded to 16 bits, all low bytes then all high bytes, compressed
with Brotli (quality 9, window 22).

## Source and changes

Copernicus DEM GLO-90 (COP-DEM-GLO-90-F), from the public `copernicus-dem-90m` bucket. Heights are rounded to whole
metres; the source's no-data value, where present, is stored as 0. Nothing else is changed: the grid, its columns per
latitude band and its coverage are the source's. Squares without land (open sea) have no file, as in the source.

## Water files

The same release holds the water files Earth's GLO-90 worlds read beside the heights, baked on GLO-90's water body mask
and heights. Each version is a new file; older ones stay for the worlds made with them.

| File | Bytes | What it is | Worlds |
|---|---|---|---|
| `lakes-glo90-1.bin` to `-4.bin` | 56.8-58.1 MB each | lake outlines and levels | made with earlier Earth versions and test builds |
| `lakes-glo90-5.bin` | 57,108,482 | lake outlines and levels on GLO-90's own grid (Earth decision 0092) | new worlds from Earth 2.2.0 |
| `rivers-outline-glo90-1.bin` to `-4.bin` | 30.3-35.8 MB each | the river water of the mask, along the courses | made with earlier Earth versions and test builds |
| `rivers-outline-glo90-5.bin` | 35,947,613 | river outlines on GLO-90's own grid | new worlds from Earth 2.2.0 |
| `rivers-flow-glo90-1.bin` | 93,803,882 | which river water flows, which way and how fast | made with earlier Earth versions and test builds |
| `rivers-flow-glo90-2.bin` | 93,894,092 | the same on GLO-90's own grid | new worlds from Earth 2.2.0 |
| `dams-glo90-1.bin` to `-3.bin` | 4.8-6.2 MB each | the heights under modern reservoirs, dams taken down (no dams in 1318) | made with earlier Earth versions and test builds |
| `dams-glo90-4.bin` | 6,390,099 | the same on GLO-90's own grid | new worlds from Earth 2.2.0 |
| `streams-glo90-1.bin` | 64,553,844 | the streams from 1 to 20 m³/s on GLO-90's valley floors (Earth decision 0061) | new worlds at 1:25 and finer from Earth 2.2.0 |

Earth checks each against the size and SHA-256 it was built with. They are adapted from Copernicus GLO-90 and from
HydroRIVERS and HydroLAKES (HydroSHEDS licence), Global Dam Watch (CC BY 4.0) and, for `streams-glo90-1.bin`, GIRES
(CC BY-SA 4.0): [ATTRIBUTION.txt](ATTRIBUTION.txt) has the notices. **`streams-glo90-1.bin` is distributed under CC
BY-SA 4.0**, as GIRES requires.

## Licence and notices

The heights are under the Copernicus WorldDEM™-90 licence: see [LICENSE.md](LICENSE.md) and [eula_F.pdf](eula_F.pdf). The
water files are under the licences of their sources as well ([ATTRIBUTION.txt](ATTRIBUTION.txt)).

Produced using Copernicus WorldDEM™-90 © DLR e.V. 2010-2014 and © Airbus Defence and Space GmbH 2014-2018 provided under
COPERNICUS by the European Union and ESA; all rights reserved. The organisations in charge of the Copernicus programme by
law or by delegation do not incur any liability for any use of the Copernicus WorldDEM™-90.

This repository, Earth and Temporal Poets Society are not endorsed by the Copernicus programme, the European Union, ESA,
DLR or Airbus.
