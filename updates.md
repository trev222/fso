# SimNation Developer Handoff - FreeSO Test Changes

This document is intended for SimNation owners/developers who want to review and port the changes tested in the local FreeSO sandbox.

The work was tested in an isolated FreeSO checkout at:

`/Users/trevor/Desktop/test-freeso`

That local path is only a reference. Paths below are written relative to a FreeSO/SimNation source tree wherever possible.

Important scope note:

- The portable changes are the code/content changes listed under "Portable Changes".
- The local FreeSO database edits, test accounts, local run scripts, and macOS launch helpers were used only for testing.
- Do not copy the local test database, local credentials, or local launch scripts into a production SimNation setup unless you intentionally want the same sandbox behavior.

## Recommended Integration Order

1. Review the portable code changes in this document.
2. Copy the corrected mystic tree `.iff` assets from this folder into the target content tree.
3. Add or verify the catalog entries for the mystic tree variants.
4. Port the C# changes into the matching SimNation source files.
5. Rebuild the client and server.
6. Test on a non-production shard before deploying.
7. Treat all database/account/skill-value edits as test-only unless the SimNation team explicitly wants them on a dev shard.

## Portable Changes

## 1. Corrected SimNation Mystic Tree Assets

Added corrected SimNation mystic tree variants for testing in FreeSO.

Files included in this handoff folder:

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

Suggested install location:

- `Content/Objects/SimNationMysticTrees/`

The exact folder can differ in SimNation, but the client and server must both be able to resolve the objects.

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

Source file updated in the FreeSO test build:

- `TSOClient/FSO.Content.TSO/Content/Objects/catalog_downloads.xml`

Asset fix details:

- The new tree recolors had uneven placement in zoom 2/3.
- The `DGRP` offsets mostly matched the natural mystic tree.
- The actual issue was in the recolored `SPR2` frames for sprite chunks `4000` through `5003`.
- Those recolored frames had different canvases/origins from the natural tree.
- The fix recanvased the recolored `SPR2` frames to match the original natural mystic tree's `mystictree.spf` frame bounds.
- Recolored pixels, palettes, and depth data were preserved.

Additional drawing group cleanup:

- Red, dark blue, and purple trees had leftover placement differences.
- These `DGRP` chunks were normalized to the natural/good layout:
  - `103`
  - `104`
  - `203`
  - `204`
  - `403`

Validation performed:

- Corrected test copies had `0` remaining `SPR2` canvas mismatches against the natural mystic tree reference.
- Red, dark blue, and purple had `0` remaining `DGRP` mismatches for the affected placement chunks.
- Mesh cache geometry already matched the natural tree payload.
- No `.fsom` or mesh cache edits were required.

## 2. Staff/Admin Rare Catalog Access

Enabled staff/admin users to access rare catalog category `29` from buy mode.

Source file changed:

- `TSOClient/tso.client/UI/Panels/UIBuyMode.cs`

Implementation details:

- Added `using FSO.SimAntics.Model.TSOPlatform;`.
- Added a helper equivalent to:

```csharp
private bool HasStaffCatalogAccess()
{
    var permissions = (LotController?.ActiveEntity?.TSOState as VMTSOAvatarState)?.Permissions;
    return GameFacade.EnableMod || (permissions.HasValue && permissions.Value >= VMTSOAvatarPermissions.Admin);
}
```

- In `Update(UpdateState state)`, remapped the Misc category when Shift is held by staff/admin:

```csharp
CategoryMap[MiscButton] = (state.ShiftDown && HasStaffCatalogAccess()) ? 29 : 18;
```

Behavior:

- Staff/admin users can hold Shift while using Misc to access category `29`.
- Normal users stay on the normal Misc category `18`.

## 3. 12th Full-Map Lot Size Upgrade

Added a 12th property upgrade tier that covers the full lot map.

Source files changed:

- `TSOClient/tso.simantics/Model/VMBuildableAreaInfo.cs`
- `TSOClient/tso.simantics/VMContext.cs`
- `TSOClient/tso.client/UI/Panels/UIHouseMode.cs`

Implementation details:

In `VMBuildableAreaInfo.cs`:

- Added a final buildable size of `77`.
- Added final upgrade price `83000`.
- Added final object limit `325`.

Values used:

```csharp
BuildableSizes: add 77
BuildableAreaPrices: add 83000
ObjectLimitPerPerson: add 325
```

In `VMContext.GetTSOBuildableArea`:

- Clamp the lot size index to the valid array range.
- Treat the final size tier as a full-map lot.
- Return the full architecture rectangle for that final tier:

```csharp
lotSize = Math.Min(lotSize, VMBuildableAreaInfo.BuildableSizes.Length - 1);
var fullMapSize = lotSize == VMBuildableAreaInfo.BuildableSizes.Length - 1;

if (fullMapSize)
{
    return new Rectangle(0, 0, w, h);
}
```

In `UIHouseMode.cs`:

- Updated the upgrade preview scaling so the new size does not overflow the preview.

```csharp
float previewMaxSize = Math.Max(64f, Math.Max(oldSize, newSize));
float sizePct = newSize / previewMaxSize;
```

Behavior:

- The property upgrade UI gains a 12th tier.
- The final tier should cover the entire map square.

## 4. Skill Locks Now Protect `.99` Past The Whole Skill Point

Changed skill locks so a lock count protects through `.99` of that level.

Old behavior:

- `1` lock protected to `1.00`.
- `2` locks protected to `2.00`.

New behavior:

- `1` lock protects to `1.99`.
- `2` locks protect to `2.99`.
- `19` locks protect to `19.99`.
- `20` locks protect to `20.99`.

Source files changed:

- `TSOClient/tso.simantics/NetPlay/Model/Commands/VMNetSkillLockCmd.cs`
- `TSOClient/FSO.Server/Servers/Lot/Domain/LotContainer.cs`

Implementation details:

Add helpers in `VMNetSkillLockCmd.cs`:

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

Update lock budget calculation:

```csharp
otherLocked += LockLevelForThreshold(
    caller.GetPersonData((VMPersonDataVariable)((int)VMPersonDataVariable.SkillLockBase + i))
);
```

Update selected lock storage:

```csharp
caller.SetPersonData(
    (VMPersonDataVariable)((int)VMPersonDataVariable.SkillLockBase + SkillID),
    LockThresholdForLevel(LockLevel)
);
```

Update `LotContainer.cs` avatar load:

```csharp
state.SkillLockBody = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_body);
state.SkillLockCharisma = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_charisma);
state.SkillLockCooking = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_cooking);
state.SkillLockCreativity = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_creativity);
state.SkillLockLogic = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_logic);
state.SkillLockMechanical = VMNetSkillLockCmd.LockThresholdForLevel(avatar.lock_mechanical);
```

Update `LotContainer.cs` avatar save:

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
- The database can continue storing whole lock counts.
- Runtime converts whole lock counts to `.99` thresholds on load.
- Runtime converts thresholds back to whole lock counts on save.

## 5. Guest Read-Only Buy Mode Inspection

Allowed non-roommate, non-admin guests to inspect lot objects in buy mode without being able to move or modify them.

Goal:

- Guests can enter buy mode.
- Guests can click an object and view object info.
- Guests can see the object name, description, wear, and owner info through the normal query/info panel.
- Guests cannot move, sell, inventory, upgrade, buy, cancel sale, edit price, or otherwise modify the object.
- Owners, roommates, build-buy roommates, and admins keep normal behavior.

Source files changed:

- `TSOClient/tso.client/UI/Panels/UIObjectHolder.cs`
- `TSOClient/tso.client/UI/Panels/UIQueryPanel.cs`

Implementation details in `UIObjectHolder.cs`:

- Added a read-only guest check:

```csharp
private bool IsReadOnlyGuest()
{
    var permissions = (ParentControl.ActiveEntity?.TSOState as VMTSOAvatarState)?.Permissions ?? VMTSOAvatarPermissions.Visitor;
    return !Roommate && permissions < VMTSOAvatarPermissions.Admin;
}
```

- Added an object-info path:

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

- In the object click path, route read-only guests to `ShowReadOnlyObjectInfo(...)` instead of creating a ghost copy and starting the move-object flow.

Implementation details in `UIQueryPanel.cs`:

- Added a read-only object-info check:

```csharp
private bool IsReadOnlyObjectInfo()
{
    var permissions = (LotParent.ActiveEntity?.TSOState as VMTSOAvatarState)?.Permissions;
    return !Roommate && permissions.HasValue && permissions.Value < VMTSOAvatarPermissions.Admin;
}
```

- Disabled object modification controls when `IsReadOnlyObjectInfo()` is true:
  - Sell back
  - Inventory/send back
  - Async sale
  - Async cancel sale
  - Async edit price
  - Async buy
  - Upgrade button
  - Upgrade list editing

Behavior:

- Guest inspection is read-only.
- Admins and roommates are unaffected.

## 6. Help Cursor For Guest Inspection

Added a question/help cursor for the read-only guest inspection mode.

Source files changed:

- `TSOClient/tso.common/Rendering/Framework/CursorManager.cs`
- `TSOClient/tso.client/UI/Panels/UIObjectHolder.cs`

Implementation details:

- Added `CursorType.Help`.
- Mapped it to the existing `help.cur` cursor asset:

```csharp
{ CursorType.Help, "help.cur" }
```

- In `UIObjectHolder.Update`, use the help cursor for read-only guests:

```csharp
CursorType cur = IsReadOnlyGuest() ? CursorType.Help : CursorType.SimsMove;
```

Asset used:

- `TSOClient/uigraphics/shared/cursors/help.cur`

## 7. Zero Wear For `$0` And `$1` Items

Changed wear behavior so objects with an initial price of `$0` or `$1` always remain at `0%` wear.

Goal:

- Rare items often cost `$0` or `$1`.
- These items should not wear down or break.
- The rule applies to all objects with `InitialPrice <= 1`, not only rare items.

Source files changed:

- `TSOClient/tso.simantics/Model/TSOPlatform/VMTSOObjectState.cs`
- `TSOClient/tso.simantics/Entities/VMMultitileGroup.cs`
- `TSOClient/tso.simantics/VMContext.cs`
- `TSOClient/tso.simantics/NetPlay/Model/Commands/VMNetBuyObjectCmd.cs`
- `TSOClient/tso.simantics/NetPlay/Model/Commands/VMNetPlaceInventoryCmd.cs`
- `TSOClient/tso.simantics/NetPlay/Model/Commands/VMNetUpgradeCmd.cs`
- `TSOClient/tso.simantics/Primitives/VMGenericTSOCall.cs`

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

- On lot/object load:
  - `VMMultitileGroup.Load(...)`
- On object creation:
  - `VMContext.CreateObjectInstance(...)`
- After object purchase price is assigned:
  - `VMNetBuyObjectCmd`
- After placing from inventory:
  - `VMNetPlaceInventoryCmd`
- After upgrade price adjustments:
  - `VMNetUpgradeCmd`
- During repair behavior:
  - `VMGenericTSOCall`

Repair behavior note:

- Normal objects still use the usual repair minimum wear behavior.
- `$0` and `$1` objects are forced back to `0` wear instead of being repaired to the normal minimum.

## 8. Optional macOS/Mono Development Compatibility

These changes were useful for running the FreeSO test client on macOS/Mono. They may not be needed for SimNation production.

Source files changed:

- `TSOClient/FSO.Windows/FSO.Windows.csproj`
- `TSOClient/FSO.Windows/UITTSContext.cs`
- `TSOClient/tso.client/FSO.Client.csproj`
- `TSOClient/tso.client/Utils/MonogameLinker.cs`

What changed:

- Added `NO_SYSTEM_SPEECH` on non-Windows builds.
- Made `System.Speech` a Windows-only reference.
- Guarded `UITTSContext.Speak(...)` so non-Windows builds return early.
- Added macOS SDL/OpenAL dylibs to client output content.
- Adjusted MonoGame linking so macOS uses the WindowsGL MonoGame path used by the local test setup.

Recommendation:

- Treat this as dev-environment support, not a required gameplay feature.
- SimNation should only port this if their developers need the same macOS/Mono local client path.

## Local Test-Only Changes

The following changes were made only to test the features in an isolated FreeSO environment. They are documented for transparency, but they should not be treated as production SimNation install steps.

## 9. Isolated FreeSO Sandbox

Set up a local FreeSO sandbox so the work could be tested without modifying SimNation.

Local sandbox path:

- `/Users/trevor/Desktop/test-freeso`

Local database:

- MariaDB running on `127.0.0.1:13306`
- Database name: `fso`

Local helper scripts:

- `run-freeso-mysql.command`
- `stop-freeso-mysql.command`
- `run-freeso-server.command`
- `stop-freeso-server.command`
- `run-freeso.command`
- `init-freeso-db.command`

Production note:

- These scripts are local-machine convenience scripts.
- They are not required for SimNation production.

## 10. Local Test Accounts

Created test accounts in the local FreeSO database.

Accounts:

- `admin2`
- `guest`

Purpose:

- `admin2` was used for two-admin/two-client testing.
- `guest` was used to test non-roommate, non-admin buy-mode inspection behavior.

Production note:

- These were local database accounts only.
- Do not create these accounts in production unless wanted for a test shard.

## 11. Local Test Character Money And Skills

Updated local test character values to verify gameplay behavior.

Local admin avatar:

- Account/avatar: `admin` / `trevor`
- Money was set to `5,000,000`.
- All six skills were later set to `20.99`.
- Body remained locked at `20` locks for skill-lock testing.

Local guest avatar:

- Account/avatar: `guest` / `guest`
- All six skills were set to `20.99`.
- No skill locks were set.

Database field values used for `20.99`:

```sql
skill_body = 2099
skill_charisma = 2099
skill_cooking = 2099
skill_creativity = 2099
skill_logic = 2099
skill_mechanical = 2099
```

Production note:

- These are test data edits only.
- They are not required for the code/content changes.

## 12. Local Build And Restart Commands

Commands used in the FreeSO test checkout.

Client build:

```sh
msbuild TSOClient/FSO.Windows/FSO.Windows.csproj /p:Configuration=Release /p:Platform=AnyCPU /v:minimal
```

Server build:

```sh
MSBuildSDKsPath=<repo>/.dotnet-2.2/sdk/2.2.207/Sdks msbuild TSOClient/FSO.Server.Core/FSO.Server.Core.csproj /p:Configuration=Release /p:Platform=AnyCPU /p:OutputPath=<repo>/freeso-server/app-msbuild/ /v:minimal
```

Production note:

- SimNation developers should use their normal build pipeline.
- The commands above are useful only as a reference for the FreeSO test environment.

## Verification Checklist For SimNation Developers

Use this checklist on a dev shard after porting.

1. Mystic tree variants appear in the intended catalog category.
2. Mystic tree variants align correctly at all zoom levels and rotations.
3. Staff/admin users can access rare category `29`.
4. Normal users cannot access staff-only rare catalog behavior.
5. The property upgrade UI shows the 12th lot-size tier.
6. The 12th lot-size tier makes the whole map square buildable.
7. Skill locks protect `.99` past the whole lock count.
8. A `20`-lock skill at `20.99` does not decay below `20.99`.
9. Guests can open buy mode and inspect object details.
10. Guests cannot move, sell, inventory, upgrade, or edit objects.
11. Owners, roommates, build-buy roommates, and admins still have normal buy-mode controls.
12. Read-only guest inspection uses the help/question cursor.
13. `$0` and `$1` objects load with `0%` wear.
14. `$0` and `$1` objects do not gain wear over quarter-day processing.
15. Normal-priced objects still wear normally.

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
