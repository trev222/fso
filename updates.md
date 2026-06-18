# SimNation Mystic Tree Asset Updates

This folder contains the fixed SimNation mystic tree `.iff` assets.

## What Was Fixed

- Recanvased the affected `SPR2` frames so the recolored trees use the same frame bounds as the original natural mystic tree.
- This fixes the uneven/off look that appeared in the second and third zoomed-in states.
- Preserved the recolored pixels, palettes, and depth data while restoring the expected sprite canvas placement.
- Normalized remaining `DGRP` placement differences for the red, dark blue, and purple trees.
- The second pass fixed one remaining angle where those three colors were slightly out of position.

## Specific DGRP Cleanup

The red, dark blue, and purple trees had leftover placement differences in these drawing groups:

- `103`
- `104`
- `203`
- `204`
- `403`

Those chunks were normalized to the good/natural layout used by the other corrected recolors.

## Files Included

- `SN_RainbowMysticTree.iff`
- `SimNation_BlackMysticTree.iff`
- `SimNation_BlueMysticTree.iff`
- `SimNation_DarkBlueMysticTree.iff`
- `SimNation_GreenMysticTree.iff`
- `SimNation_OrangeMysticTree.iff`
- `SimNation_PurpleMysticTree.iff`
- `SimNation_RedMysticTree.iff`
- `SimNation_SilverMysticTree.iff`
- `SimNation_YellowMysticTree.iff`

## Integration Notes

- Replace the existing SimNation mystic tree `.iff` files with these files, preserving filenames.
- No code changes are required.
- No `.fsom` or mesh cache files are required.
- Restart the client or refresh redistributed/downloaded object assets after replacing the files.

## Validation

- The corrected test-freeso copies had zero remaining SPR2 canvas mismatches against the natural mystic tree reference.
- The red, dark blue, and purple trees had zero remaining DGRP mismatches for the affected placement chunks.
- Mesh cache geometry matched the natural tree payload, so mesh cache edits were not needed.
