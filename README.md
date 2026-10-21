# CGS MW Assets — migration staging

This is the **visual asset-only** addon for our Arma Reforger mech project. The asset migration is staged on this feature branch so the currently published addons stay unchanged until Workbench tests pass.

## Enfusion project identity

- Project ID: `CGSMWAssets`
- Addon GUID: `6AAA31A473996E48`
- Dependencies: Arma Reforger (`58D0FB3206B6F859`) only
- No dependency on MW Gameplay or Mech Core (avoids a dependency cycle)

## What I migrated

- All **408 files** in `Common/Models`, including models (`.xob`, `.txo`, `.fbx`), textures, materials, source images and their `.meta` files
- **316.012 MiB** of model resources
- **146 original `.meta` files**, with resource GUIDs and paths intact
- Source: `Sylem2k/Reforger-Warfare--MechWarriorProject-main`, pinned source commit `bebb93f3f55f947fc74756b2995eee88ff714390`
- Verification: every copied file's Git blob SHA matched its source file, with zero missing or extra paths

No runtime `Scripts/Game` files or scripted vehicle/weapon prefabs have been moved into this addon.

## Required companion MW Gameplay branch

`Sylem2k/Reforger-Warfare--MechWarriorProject-main` on `feature/external-mw-assets-20261020`.

That branch:
- Keeps all **19 `.c` MW gameplay scripts**, mech frames, AI enemy prefabs, weapons/modules, Mech Bay, loadouts, catalogs and Game Master configuration.
- Deletes the original 408 `Common/Models` files only after confirming that all 408 now exist here byte-for-byte.
- Adds `6AAA31A473996E48` to the MW Gameplay `addon.gproj` dependencies, retaining Mech Core (`6A8C4B3A46C28E1B`) and Arma Reforger (`58D0FB3206B6F859`).

## Test sequence

1. **Keep both production `main` branches unchanged.** Back up your local addons first.
2. In separate local project checkouts, switch Assets to `feature/assets-migration-20261020` and MW Gameplay to `feature/external-mw-assets-20261020`.
3. Launch Workbench with **Mech Core CGS, CGS MW Assets, and MW Gameplay**. Rebuild/re-scan resource databases if Workbench requests it; do not copy the old MW `resourceDatabase.rdb` into Assets.
4. Verify that the Mech Bay mesh and Cricket, Warden, Judiciar and Gravemarch chassis show the correct models and materials.
5. Verify all attached arm/back modules and weapon meshes; compile scripts and check logs for **missing GUID**, **missing resource**, or **duplicate resource** errors.
6. Test Game Master placement and Mech Lab in local play. Then test on a dedicated multiplayer server with the **same three-addon versions** on both server and clients.
7. After all tests pass, merge/publish the **Assets repository first**, then merge/publish the companion **MW Gameplay repository**. Update server dependencies together.

**Important:** Do NOT load this new Assets addon together with the older MW Gameplay build that still contains `Common/Models`; that would expose the same resource IDs in two addons at once. Use the paired migration branches.

## Future size optimization

The file split reduces the MW Gameplay package size but does **not** automatically reduce the total content that clients must load. We identified around 85 MiB in byte-identical duplicated texture/source files. Removing those should be a separate, GUID-aware cleanup after the split compiles and runs successfully.

The one-time workflow `.github/workflows/stage-mw-models.yml` performed this migration. It is scoped to the migration branch and does not touch the source repository.
