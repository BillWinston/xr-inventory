# Fossett Lab Development Apps — Inventory

First-pass structural catalog of what originally lived at `/Volumes/geospatial_data/Development_Apps/` on the NAS, captured 2026-04-20. Based on directory structure, `ProjectSettings/ProjectVersion.txt`, README files, git metadata, and platform-config files only — no source code was read. After the 2026-04-20 migration, the content lives at `/mnt/nas/dev/fossett_xr_apps/`.

## Top-level situation

- The whole `Development_Apps/` tree sits inside **one monolithic git repo**. Last commit is 2017-06-07 "first commit" — it's a build-artifact cleanup, not real history. Not worth preserving.
- A handful of subdirectories have their own real sub-repos (see below).
- Unity versions span 5.4.0f1-HTP (HoloLens 1 tech-preview era) → 2017.4 → 2018.3 → 2019.2/2019.4. No Unity 2020+.
- Four Windows `.lnk` shortcuts and a `.vs/` cache sit at the top level — ignorable on Mac.

## GeoXplorer family

Two parallel lineages plus a shared asset-bundle project.

### HoloLens / Mixed Reality lineage

| Directory | Unity | Own git? | Notes |
|---|---|---|---|
| `GeoExplorer` | 2017.4.2f2 | no | Oldest. UWP. 2018 csprojs. |
| `GeoXplorer` | 2017.4.21f1 | **yes** — last commit 2018-09-19 "hirise update" | Has `README.md` (Fossett Lab / HoloLens / Agisoft photogrammetry / WashU 2018). Contains `TestBuilds082021/` and an Android keystore — mobile port experiments started here. |
| `GeoXplorer 2` | 2019.2.1f1 | no | Full MRTK2 + Photon networking rewrite. Assemblies last touched Oct 2023. |
| `GeoXplorerSE (1)` | 2019.2.1f1 | no | "SE" likely = Shared Experience. Same Unity as v2 but with multi-user sharing stack. Oct 2023. |

### Mobile lineage

| Directory | Unity | Own git? | Notes |
|---|---|---|---|
| `geoxplorer_mobile` | 2017.4.21f1 | no | First Android/iOS port (Jul 2021). Same Unity as original GeoXplorer. |
| `geoxplorer_mobile__working` | 2019.4.8f1 | no | Upgraded LTS, "working" label (Jul 2021). Supersedes `geoxplorer_mobile`. |
| `GeoXplorerM (1)` | 2019.4.8f1 | no | **Unified mobile + HoloLens.** `README.txt` (Aug 2020) describes a single project targeting iOS/Android/HoloLens via MRTK 2.2 + Azure Spatial Anchors + Photon + Azure AssetBundles. Assemblies touched Oct 2023. Appears to be the **most recently active** GeoXplorer variant. |

### Shared

- `GeoXAssetBundles` (2017.4.21f1) — dedicated AssetBundle build project that feeds the other variants.

### Best guess at lineage

1. `GeoExplorer` — 2018 HoloLens proof-of-concept.
2. `GeoXplorer` — HoloLens with early mobile experiments (has git history).
3. Branches in 2019 into:
   - `GeoXplorer 2` / `GeoXplorerSE` — HoloLens + Photon networking.
   - `geoxplorer_mobile` / `__working` — pure mobile port.
4. `GeoXplorerM` (2020+) — unified HoloLens + mobile codebase; last activity Oct 2023.

## Everything else, grouped

### HoloLens / UWP teaching + research apps

| Directory | Unity | Platform | Purpose (best guess) |
|---|---|---|---|
| `Fossett Lab Demo` | 2017.4.21 | HoloLens (HoloToolkit) | Crystal + protein models. Has `ITC_TestBuild/`. |
| `CrystalViewer` | 2017.1.2 | HoloLens (HoloToolkit) | Has `README.md` + `GettingStarted.md`. "Part of HoloToolkit." |
| `MeltComplex` | 5.4.0f1-HTP | HoloLens 1 (HoloToolkit) | Magma phase-diagram teaching. 75 MB `subsample_mesh.obj`. |
| `3DPhaseDiagrams` | 2017.4.2 | Desktop | Petrology phase-diagram interactive; Blender models + `CrystalSlicer`/`FoldGenerator` scenes. |
| `MineralHandSamples` | 2019.2.1 | HoloLens/UWP + AssetBundles | ~30 FBX mineral specimens (Actinolite, Apatite, Calcite on quartz, etc.). |
| `LROAssetBundles` | 2019.2.1 | HoloLens/UWP + AssetBundles | Apollo 11/12/14/15/16 landing sites + Mono Lake imagery. Lunar exploration viewer. |
| `RoverTraverse` | 2017.4.2 | HoloLens (HoloToolkit) | Rover path viewer with `SiteSphere.prefab`. |
| `DemoForDCO` | 2017.4.21 | HoloLens + WebGL | **Own git** — last commit 2019-06-06 "UI improvements 8". Deep Carbon Observatory demo. Unusual `WebGLSupport/` folder. |
| `IntermediateTriggering` | 2019.2.1 | HoloLens (MRTK + Photon) | Networked MR app, purpose unclear from names. |

### Azure Spatial Anchors scaffolding (SDK samples, not lab apps)

| Directory | Notes |
|---|---|
| `ASADemo` | Working ASA wrapper, Unity 2019.2.1. |
| `azure-spatial-anchors-samples` | **Own git** — Microsoft's official sample repo, Feb 2019 snapshot. |
| `azure-spatial-anchors-samples-master` | Zip download of same repo, Feb 2020. Adds Xamarin samples. |

### Volcano / sonification

| Directory | Unity | Notes |
|---|---|---|
| `KilaueaSono` | 2017.4.21 | Kilauea DEM + scripts; name suggests sonification. Desktop/mobile. |

### More geoscience / Earth-data apps (added after initial catalog)

| Directory | Unity | Platform | Purpose (best guess) |
|---|---|---|---|
| `SeismicityViewer` | 5.5.2f1 | HoloLens 1 / UWP | Earthquake / seismicity visualization. Legacy Unity 5.5. |
| `VirtualEarth` | 2017.4.21f1 | HoloLens / UWP | Has `AssetBundles/` + `AssetBundles_old/` + `External/ReadMeImages/` + `CalibrationData.txt`. Likely a Microsoft Bing VirtualEarth / globe integration. |
| `VirtualEarth2` | 2017.4.21f1 | HoloLens / UWP | Newer version with `DataFile/` and `packages.config`. |
| `VolcanoViewer` | 2017.4.21f1 | Desktop | Has `VideoCaptureLib_Log.txt`, `packages.config` (legacy .NET). No UWP folder — likely desktop. Volcano visualization tool. |

### Third-party / vendor code

| Directory | Notes |
|---|---|
| `Unity3DTiles-master` | GitHub zip (Unity 2019.2.1f1) of the [3D Tiles](https://github.com/Unity3DTiles) Unity implementation. Real `README.md` + `Docs/` (algorithm reference, schema docs). Vendor code — pointer, not a copy. |

### Sandboxes, test projects, cruft

| Directory | Unity | Notes |
|---|---|---|
| `OculusGOTest` | 2017.4.2 | Oculus Go test, compiled `.apk` present. |
| `Sandbox` / `Sandbox_NGA` | mixed | UWP scratch project; assemblies touched Aug 2023. |
| `Bundles` | 5.4.0f1-HTP | Minimal, probably dead. |
| `Demo` | 5.5.2f1 | Generic template, minimal content. |
| `New Unity Startup Project` | 2018.3.0f2 | Untouched default Unity template. |

## Observations for next steps

1. **GeoXplorer** is the only substantial project with meaningful git history. `DemoForDCO` has a minor one. Everything else is effectively a frozen source tree.
2. Roughly **10–15 serious apps, 5–8 disposable** (sandboxes, default template, duplicated MS sample, etc.).
3. **Unity `Library/`, `obj/`, `bin/`, `App/` are regenerable caches.** Most of the multi-GB per-project sizes are these. A clean archive is <10% of current disk use.
4. **Likely-duplicate pairs to resolve:**
   - `geoxplorer_mobile` ↔ `geoxplorer_mobile__working` — keep the working one.
   - `azure-spatial-anchors-samples` ↔ `...-master` — vendor code; keep a pointer, not a copy.
5. No Unity 2020+. Any resuscitation will require a Unity upgrade and probably an MRTK 2 → MRTK 3 or OpenXR migration for the HoloLens apps.
