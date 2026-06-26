# FreeSO / SimNation Custom Assets

Packaged from `/Users/trevor/Desktop/test-freeso` on 2026-06-26.

## What is included

- 4 tuned `mah201_punxz` mohawk head recolors: green, yellow, purple, white.
- 4 `mah517_tsobandana` band-color head variants.
- 4 `mab978_ct-rave` outfit recolors: black, white, yellow, purple.
- Patched head and body `collections.dat` files from the local test layer.
- Downscaled, cropped FreeSO client screenshots in `screenshots/`.

## Folder layout

```text
fso/
  install_overlay/
    SimNationTestClient/Content/Avatar/...          # loose custom asset files
    The Sims Online/TSOClient/avatardata/...        # patched collections.dat files
  screenshots/
  docs/
```

## Simple install into SimNation

1. Quit SimNation / FreeSO before copying files.
2. Back up these target folders/files first:
   - `SimNationTestClient/Content/Avatar/`
   - `The Sims Online/TSOClient/avatardata/heads/collections/collections.dat`
   - `The Sims Online/TSOClient/avatardata/bodies/collections/collections.dat`
3. Copy everything inside `install_overlay/SimNationTestClient/Content/Avatar/` into the matching `SimNationTestClient/Content/Avatar/` folder.
4. Copy `install_overlay/The Sims Online/TSOClient/avatardata/heads/collections/collections.dat` into the matching heads collections folder.
5. Copy `install_overlay/The Sims Online/TSOClient/avatardata/bodies/collections/collections.dat` into the matching bodies collections folder.
6. Launch SimNation and check the male head/clothing catalog.

## If SimNation already has custom catalog edits

Do not blindly overwrite `collections.dat` if that SimNation copy already has other custom catalog changes.
Instead, merge these purchasable IDs into the matching collections:

`ea_male_heads.col`:

- `0x5a201001` - `mah201_punxz_green`
- `0x5a201101` - `mah201_punxz_yellow`
- `0x5a201201` - `mah201_punxz_purple`
- `0x5a201301` - `mah201_punxz_white`
- `0x5a518201` - `mah517_blackhair_yellow_band`
- `0x5a518401` - `mah517_blondhair_white_band`
- `0x5a518501` - `mah517_blondhair_black_band`
- `0x5a518b01` - `mah517_brownhair_purple_band`

`ea_male.col`:

- `0x5b978001` - `mab978_ct_rave_black`
- `0x5b978101` - `mab978_ct_rave_white`
- `0x5b978201` - `mab978_ct_rave_yellow`
- `0x5b978301` - `mab978_ct_rave_purple`

The loose asset files can still be copied directly; the collection merge only controls whether they appear in the catalog.

## Notes

- The loose files reference base-game meshes already present in the normal TSO/SimNation data packs.
- The collection files in this package are the patched versions from the local test layer.
- Screenshots are cropped to the FreeSO client content area and downscaled so the folder stays reasonably small.

## Manifest summary

- Loose custom files copied: 168
- Collection files copied: 2
- Preview screenshots written: 13

## Live 3D Screenshots

Cropped, downscaled FreeSO client screenshots are included in `screenshots/3d_live/`. The contact sheet is `screenshots/3d_live/all_new_assets_3d_contact_sheet.png`.
