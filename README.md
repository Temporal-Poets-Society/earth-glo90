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

## Licence and notices

The data is under the Copernicus WorldDEM™-90 licence: see [LICENSE.md](LICENSE.md) and [eula_F.pdf](eula_F.pdf).

Produced using Copernicus WorldDEM™-90 © DLR e.V. 2010-2014 and © Airbus Defence and Space GmbH 2014-2018 provided under
COPERNICUS by the European Union and ESA; all rights reserved. The organisations in charge of the Copernicus programme by
law or by delegation do not incur any liability for any use of the Copernicus WorldDEM™-90.

This repository, Earth and Temporal Poets Society are not endorsed by the Copernicus programme, the European Union, ESA,
DLR or Airbus.
