# SimNation Developer Handoff - FreeSO Test Changes

This document summarizes the FreeSO test changes in five feature buckets for SimNation owners/developers:

1. Mystic Tree Changes
2. Skill Lock Changes
3. Lot Size Changes
4. Buy Mode Changes
5. Item Wear Changes

The work was tested in an isolated FreeSO checkout at:

`/Users/trevor/Desktop/test-freeso`

That path is only a test reference. File paths below are written relative to a FreeSO/SimNation source tree wherever possible.

Important scope note:

- The five feature sections are the portable changes to review.
- Local FreeSO accounts, local database edits, and local launch scripts were only for testing.
- Do not copy local test credentials, local database data, or local machine-specific scripts into production.

## 1. Mystic Tree Changes

Added and corrected the SimNation mystic tree variants so they can be safely distributed and tested in FreeSO/SimNation content.

### Assets Included

The handoff folder contains these corrected `.iff` files:

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

Suggested content location:

- `Content/Objects/SimNationMysticTrees/`

The exact folder can differ in SimNation, but both client and server content resolution need to see the same object files.

### Catalog Entries

Catalog entries used in the FreeSO test build:

```xml
<!-- SimNation Mystic Tree variants. -->
<P g="481A1094" s="29" p="0" n="Purple Mystic Tree" t="simnation, mystic, tree, rare, purple" />
<P g="483A2160" s="29" p="0" n="Rainbow Mystic Tree" t="simnation, mystic, tree, rare, rainbow" />
<P g="781A9488" s="29" p="0" n="Dark Blue Mystic Tree" t="simnation, mystic, tree, rare, dark blue" />
<P g="481A2120" s="29" p="0" n="Orange Mystic Tree" t="simnation, mystic, tree, rare, orange" />
<P g="481A6340" s="29" p="0" n="Blue Mystic Tree" t="simnation, mystic, tree, rare, blue" />
<P g="481A6EEB" s="29" p="0" n="Yellow Mystic Tree" t="simnation, mystic, tree, rare, yellow" />
<P g="481A9854" s="29" p="0" n="Red Mystic Tree" t="simnation, mystic, tree, rare, red" />
<P g="481A7AEE" s="29" p="0" n="Silver Mystic Tree" t="simnation, mystic, tree, rare, silver" />
<P g="481A4864" s="29" p="0" n="Green Mystic Tree" t="simnation, mystic, tree, rare, green" />
<P g="481A4817" s="29" p="0" n="Black Mystic Tree" t="simnation, mystic, tree, rare, black" />
```

FreeSO test file updated:

- `TSOClient/FSO.Content.TSO/Content/Objects/catalog_downloads.xml`

### Sprite And Placement Fixes

Problem:

- The added tree recolors looked uneven at zoom levels 2 and 3.
- `DGRP` offsets mostly matched the natural mystic tree.
- The real mismatch was in the recolored `SPR2` frame canvases/origins.

Fix:

- Recanvased affected recolored `SPR2` frames to match the natural mystic tree's `mystictree.spf` frame bounds.
- Preserved each recolor's pixels, palettes, and depth data.
- Affected sprite chunks were `4000` through `5003`.

Additional `DGRP` cleanup:

- Red, dark blue, and purple trees had leftover placement differences.
- These drawing groups were normalized to the natural/good layout:
  - `103`
  - `104`
  - `203`
  - `204`
  - `403`

Validation:

- Corrected test copies had `0` remaining `SPR2` canvas mismatches against the natural mystic tree reference.
- Red, dark blue, and purple had `0` remaining `DGRP` mismatches for the affected placement chunks.
- Mesh cache geometry already matched the natural tree payload.
- No `.fsom` or mesh cache edits were required.

## 2. Skill Lock Changes

Changed skill locks so each lock protects through `.99` past its whole skill point.

If SimNation already has this behavior, treat this section as a comparison/reference rather than a required port.

### Behavior

Old behavior:

- `1` lock protected to `1.00`.
- `2` locks protected to `2.00`.

New behavior:

- `1` lock protects to `1.99`.
- `2` locks protect to `2.99`.
- `19` locks protect to `19.99`.
- `20` locks protect to `20.99`.

### Files Changed

- `TSOClient/tso.simantics/NetPlay/Model/Commands/VMNetSkillLockCmd.cs`
- `TSOClient/FSO.Server/Servers/Lot/Domain/LotContainer.cs`

### Implementation Notes

Add threshold conversion helpers in `VMNetSkillLockCmd.cs`:

```csharp
public static short LockThresholdForLevel(int level)
{
    if (level <= 0) return 0;
    return (short)(level * 100 + 99);
}

public static int LockLevelForThreshold(int threshold)
{
    return Math.Max(0, threshold / 100);
}
```

Use `LockLevelForThreshold(...)` when calculating how many locks are already spent:

```csharp
otherLocked += LockLevelForThreshold(
    caller.GetPersonData((VMPersonDataVariable)((int)VMPersonDataVariable.SkillLockBase + i))
);
```

Use `LockThresholdForLevel(...)` when storing the selected lock threshold:

```csharp
caller.SetPersonData(
    (VMPersonDataVariable)((int)VMPersonDataVariable.SkillLockBase + SkillID),
    LockThresholdForLevel(LockLevel)
);
```

On avatar load in `LotContainer.cs`, convert DB whole lock counts to runtime `.99` thresholds:

```csharp
state.SkillLockBody = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_body);
state.SkillLockCharisma = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_charisma);
state.SkillLockCooking = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_cooking);
state.SkillLockCreativity = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_creativity);
state.SkillLockLogic = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_logic);
state.SkillLockMechanical = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_mechanical);
```

On avatar save in `LotContainer.cs`, convert runtime thresholds back to DB whole lock counts:

```csharp
state.lock_body = (ushort)VMNetSkillLockCmd.LockLevelForThreshold(avatar.SkillLockBody);
state.lock_charisma = (ushort)VMNetSkillLockCmd.LockLevelForThreshold(avatar.SkillLockCharisma);
state.lock_cooking = (ushort)VMNetSkillLockCmd.LockLevelForThreshold(avatar.SkillLockCooking);
state.lock_creativity = (ushort)VMNetSkillLockCmd.LockLevelForThreshold(avatar.SkillLockCreativity);
state.lock_logic = (ushort)VMNetSkillLockCmd.LockLevelForThreshold(avatar.SkillLockLogic);
state.lock_mechanical = (ushort)VMNetSkillLockCmd.LockLevelForThreshold(avatar.SkillLockMechanical);
```

Database note:

- No schema change is required.
- The database can keep storing whole lock counts.
- Runtime converts whole counts to `.99` thresholds on load.
- Runtime converts thresholds back to whole counts on save.

### Verification

- A skill at `20.99` with `20` locks should not decay below `20.99`.
- A skill at `20.99` with `0` locks should remain eligible for normal decay.
- Lock point spending should still count whole lock levels, not `.99` thresholds.

## 3. Lot Size Changes

Added a 12th property upgrade tier that covers the full lot map.

If SimNation already has full-map property upgrades, treat this section as a comparison/reference rather than a required port.

### Files Changed

- `TSOClient/tso.simantics/Model/VMBuildableAreaInfo.cs`
- `TSOClient/tso.simantics/VMContext.cs`
- `TSOClient/tso.client/UI/Panels/UIHouseMode.cs`

### Implementation Notes

In `VMBuildableAreaInfo.cs`, add one final tier:

```csharp
BuildableSizes: add 77
BuildableAreaPrices: add 83000
ObjectLimitPerPerson: add 325
```

In `VMContext.GetTSOBuildableArea`, clamp the tier index and make the final tier return the whole map:

```csharp
lotSize = Math.Min(lotSize, VMBuildableAreaInfo.BuildableSizes.Length - 1);
var fullMapSize = lotSize == VMBuildableAreaInfo.BuildableSizes.Length - 1;

if (fullMapSize)
{
    return new Rectangle(0, 0, w, h);
}
```

In `UIHouseMode.cs`, scale the lot upgrade preview against the largest relevant size so the full-map tier fits:

```csharp
float previewMaxSize = Math.Max(64f, Math.Max(oldSize, newSize));
float sizePct = newSize / previewMaxSize;
```

### Behavior

- The property upgrade UI gains a 12th tier.
- The final tier should make the entire lot square buildable.
- The preview UI should not overflow when showing the final tier.

### Verification

- Upgrade UI shows tier 12.
- Final tier covers the full square.
- Preview remains visually centered and scaled.
- Existing lower tiers still behave normally.

## 4. Buy Mode Changes

Added/adjusted buy-mode behavior for staff/admin rare catalog access and guest read-only inspection.

If SimNation already has a staff creative catalog, compare before porting the staff/admin catalog part. The guest inspection part is separate and can be ported independently.

### 4A. Staff/Admin Rare Catalog Access

Enabled staff/admin users to access rare catalog category `29` from buy mode.

File changed:

- `TSOClient/tso.client/UI/Panels/UIBuyMode.cs`

Implementation:

```csharp
private bool HasStaffCatalogAccess()
{
    var permissions = (LotController?.ActiveEntity?.TSOState as VMTSOAvatarState)?.Permissions;
    return GameFacade.EnableMod || (permissions.HasValue && permissions.Value >= VMTSOAvatarPermissions.Admin);
}
```

In `Update(UpdateState state)`:

```csharp
CategoryMap[MiscButton] = (state.ShiftDown && HasStaffCatalogAccess()) ? 29 : 18;
```

Behavior:

- Staff/admin users can hold Shift while using Misc to access category `29`.
- Normal users stay on normal Misc category `18`.

### 4B. Guest Read-Only Object Inspection

Allowed non-roommate, non-admin guests to inspect lot objects in buy mode without modifying them.

Files changed:

- `TSOClient/tso.client/UI/Panels/UIObjectHolder.cs`
- `TSOClient/tso.client/UI/Panels/UIQueryPanel.cs`

Guest behavior:

- Guests can enter buy mode.
- Guests can click an object and view object info.
- Guests can see object name, description, wear, and owner information.
- Guests cannot move, sell, inventory, upgrade, buy, cancel sale, edit price, or otherwise modify the object.

Unaffected users:

- Owners
- Roommates
- Build-buy roommates
- Admins

Implementation in `UIObjectHolder.cs`:

```csharp
private bool IsReadOnlyGuest()
{
    var permissions = (ParentControl.ActiveEntity?.TSOState as VMTSOAvatarState)?.Permissions ?? VMTSOAvatarPermissions.Visitor;
    return !Roommate && permissions < VMTSOAvatarPermissions.Admin;
}
```

```csharp
private void ShowReadOnlyObjectInfo(VMEntity obj, UpdateState state)
{
    if (Holding != null)
    {
        OnDelete?.Invoke(Holding, state);
        ClearSelected();
    }

    var queryPanel = ParentControl.QueryPanel;
    queryPanel.InInventory = 0;
    queryPanel.Mode = 0;
    queryPanel.Active = true;
    queryPanel.SetInfo(vm, obj.MultitileGroup.BaseObject, true);
    queryPanel.Tab = 1;
}
```

In the object click path:

- If `IsReadOnlyGuest()` is true, call `ShowReadOnlyObjectInfo(...)`.
- Otherwise keep the normal ghost-copy/move-object flow.

Implementation in `UIQueryPanel.cs`:

```csharp
private bool IsReadOnlyObjectInfo()
{
    var permissions = (LotParent.ActiveEntity?.TSOState as VMTSOAvatarState)?.Permissions;
    return !Roommate && permissions.HasValue && permissions.Value < VMTSOAvatarPermissions.Admin;
}
```

Disable these controls when `IsReadOnlyObjectInfo()` is true:

- Sell back
- Inventory/send back
- Async sale
- Async cancel sale
- Async edit price
- Async buy
- Upgrade button
- Upgrade list editing

### 4C. Guest Inspection Cursor

Added a help/question cursor for the read-only guest inspection mode.

Files changed:

- `TSOClient/tso.common/Rendering/Framework/CursorManager.cs`
- `TSOClient/tso.client/UI/Panels/UIObjectHolder.cs`

Implementation:

```csharp
CursorType.Help
```

```csharp
{ CursorType.Help, "help.cur" }
```

In `UIObjectHolder.Update`:

```csharp
CursorType cur = IsReadOnlyGuest() ? CursorType.Help : CursorType.SimsMove;
```

Asset used:

- `TSOClient/uigraphics/shared/cursors/help.cur`

### Verification

- Staff/admin users can access rare category `29` if this part is ported.
- Normal users cannot access the staff-only rare catalog path.
- Guests can inspect object details in buy mode.
- Guests cannot move, sell, inventory, upgrade, or edit objects.
- Owners, roommates, build-buy roommates, and admins still have normal buy-mode controls.
- Read-only guest inspection uses the help/question cursor.

## 5. Item Wear Changes

Changed wear behavior so objects with an initial price of `$0` or `$1` always remain at `0%` wear.

### Behavior

- Objects with `InitialPrice <= 1` are treated as no-wear items.
- These objects should not wear down.
- These objects should not break because of wear.
- This applies to all `$0` and `$1` objects, not only rare items.
- Normal-priced objects keep normal wear behavior.

### Files Changed

- `TSOClient/tso.simantics/Model/TSOPlatform/VMTSOObjectState.cs`
- `TSOClient/tso.simantics/Entities/VMMultitileGroup.cs`
- `TSOClient/tso.simantics/VMContext.cs`
- `TSOClient/tso.simantics/NetPlay/Model/Commands/VMNetBuyObjectCmd.cs`
- `TSOClient/tso.simantics/NetPlay/Model/Commands/VMNetPlaceInventoryCmd.cs`
- `TSOClient/tso.simantics/NetPlay/Model/Commands/VMNetUpgradeCmd.cs`
- `TSOClient/tso.simantics/Primitives/VMGenericTSOCall.cs`

### Implementation Notes

Core helper added in `VMTSOObjectState.cs`:

```csharp
public static bool HasNoWearValue(VMMultitileGroup group)
{
    return group != null && group.InitialPrice <= 1;
}

public static void ApplyNoWear(VMMultitileGroup group)
{
    if (group == null) return;

    foreach (var obj in group.Objects)
    {
        var state = obj.TSOState as VMTSOObjectState;
        if (state == null) continue;

        state.Wear = 0;
        state.QtrDaysSinceLastRepair = 0;
        (obj as VMGameObject)?.DisableParticle(256);
    }
}

public static void ApplyNoWearIfFree(VMMultitileGroup group)
{
    if (HasNoWearValue(group)) ApplyNoWear(group);
}
```

Quarter-day wear behavior:

- Donated objects still force no wear.
- `$0` and `$1` objects now force no wear.
- Normal objects keep normal wear behavior.

Call sites added:

- `VMMultitileGroup.Load(...)`
- `VMContext.CreateObjectInstance(...)`
- `VMNetBuyObjectCmd`
- `VMNetPlaceInventoryCmd`
- `VMNetUpgradeCmd`
- `VMGenericTSOCall`

Repair behavior:

- Normal objects still use the usual repair minimum wear behavior.
- `$0` and `$1` objects are forced back to `0` wear instead of being repaired to the normal minimum.

### Verification

- `$0` and `$1` objects load with `0%` wear.
- `$0` and `$1` objects stay at `0%` wear after quarter-day processing.
- `$0` and `$1` objects do not show broken wear particles.
- Normal-priced objects still wear normally.

## Local Test Notes

These notes explain how the features were tested. They are not required production setup steps.

### Isolated FreeSO Sandbox

Local sandbox path:

- `/Users/trevor/Desktop/test-freeso`

Local database:

- MariaDB on `127.0.0.1:13306`
- Database name: `fso`

Local helper scripts:

- `run-freeso-mysql.command`
- `stop-freeso-mysql.command`
- `run-freeso-server.command`
- `stop-freeso-server.command`
- `run-freeso.command`
- `init-freeso-db.command`

### Test Accounts And Avatars

Local test accounts:

- `admin2`
- `guest`

Local test avatar values:

- `admin` / `trevor` was given `5,000,000` money for testing.
- `admin` / `trevor` was given all six skills at `20.99`.
- `admin` / `trevor` kept Body locked at `20` locks for skill-lock testing.
- `guest` / `guest` was given all six skills at `20.99`.
- `guest` / `guest` had no skill locks set.

Database field values used for `20.99`:

```sql
skill_body = 2099
skill_charisma = 2099
skill_cooking = 2099
skill_creativity = 2099
skill_logic = 2099
skill_mechanical = 2099
```

### Optional macOS/Mono Development Support

These changes were useful for running the FreeSO test client on macOS/Mono. They may not be needed for SimNation production.

Files touched:

- `TSOClient/FSO.Windows/FSO.Windows.csproj`
- `TSOClient/FSO.Windows/UITTSContext.cs`
- `TSOClient/tso.client/FSO.Client.csproj`
- `TSOClient/tso.client/Utils/MonogameLinker.cs`

Summary:

- Added `NO_SYSTEM_SPEECH` on non-Windows builds.
- Made `System.Speech` a Windows-only reference.
- Guarded `UITTSContext.Speak(...)` so non-Windows builds return early.
- Added macOS SDL/OpenAL dylibs to client output content.
- Adjusted MonoGame linking so macOS uses the WindowsGL MonoGame path used by the local test setup.

Recommendation:

- Treat this as development support only.
- SimNation should port it only if their developers need the same macOS/Mono local client path.

## Final Review Checklist

1. Mystic tree variants appear in the intended catalog category.
2. Mystic tree variants align correctly at all zoom levels and rotations.
3. Skill locks protect `.99` past the whole lock count.
4. Full-map lot tier works if the Lot Size Changes are ported.
5. Guests can inspect object details in buy mode.
6. Guests cannot modify objects through read-only inspection.
7. Owners, roommates, build-buy roommates, and admins still have normal buy-mode controls.
8. Read-only guest inspection uses the help/question cursor.
9. `$0` and `$1` objects load with `0%` wear.
10. `$0` and `$1` objects do not gain wear over quarter-day processing.
11. Normal-priced objects still wear normally.

## Files In This Handoff Folder

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
- `updates.md`
