# Fossett Lab XR — GitHub Migration Plan

Plan for pushing the contents of `/mnt/nas/dev/fossett_xr_apps/` to the `fossettlab` GitHub org. Working document — update as decisions are made.

Current state: 31 project directories, ~80 GB, post-migration and post-Unity-cache-cleanup. INVENTORY.md in this repo catalogs each.

## Decisions made (2026-04-21)

- **One repo per project**, under `fossettlab` GitHub org.
- **`xr-` prefix** on all repo names, topics for filtering, profile README at `fossettlab/.github/profile/README.md` for the landing page.
- **Archive dir approach:** projects we're not pushing get moved to `/mnt/nas/dev/fossett_xr_apps/_archive/` as a holding area. Not deleted. Decide-later.
- **Fossett Lab Demo was repurposed in 2023** for a museum-artifact viewer (Buddhist sculptures, Shang-dynasty bronzes, Met Museum objects). Keep, rename repo to `xr-museum-viewer`.
- **WashU has GitHub Teams (100% education discount).** Unlimited private repos available. Initial push can be private; flip public per-repo when each is ready.
- **Large-repo strategy undecided.** LFS budget and hybrid-hosting approach (source on GitHub, bulk assets on NAS/S3) to be revisited when we hit the first big repo.

## Pending decisions / open questions

1. **Apple Developer account status.** (OPEN — not blocking push.) Need to determine whether the Fossett Lab account still exists and still claims `com.FossettLab.GeoXplorer`; also verify store listings are gone. Decision: proceed with pushes in the meantime; store-relevant repos stay private until this resolves.
2. **Museum-artifact work on `Fossett Lab Demo`.** RESOLVED — A.B. knows who did the 2023 work; OK to rename the repo to `xr-museum-viewer` and proceed.
3. **LFS budget / hybrid hosting.** RESOLVED — bulk assets stay on NAS for now. Do not enable LFS. The three oversized repos (`xr-mineral-hand-samples`, `xr-virtual-earth`, `xr-geoxplorer-assets`) are deferred until we decide whether they're worth the engineering work to split source-from-assets. Revisit LFS later if needed.
4. **Initial repo visibility.** RESOLVED — use the per-repo defaults recorded in the tables below. GeoXplorer family + museum viewer = **private**. Everything else = **public**. Private repos flip to public once the Apple Developer account situation is resolved (question 1).

## Repos to create — definitive list

### Active (push with full source)

Visibility column: **private** for anything the Apple account question might touch; **public** for standalone science/teaching tools. Flip private → public after question 1 resolves.

| Local directory | Repo name | Size | Visibility | Wave | Notes |
|---|---|---|---|---|---|
| `GeoXplorerM (1)` | `xr-geoxplorer` | 361 MB | private | 2 | Canonical head: unified HoloLens-2 + iOS/Android. Real source activity through 2021. |
| `geoxplorer_mobile__working` | `xr-geoxplorer-mobile` | 84 MB | private | 2 | **Store-relevant.** iOS bundle `com.FossettLab.GeoXplorer`, version 10. Deep-link scheme `geoxdl://geox`. |
| `GeoXplorerSE (1)` | `xr-geoxplorer-se` | 339 MB | private | 2 | Sibling "shared experience" variant, HL2 + Photon + Azure Spatial Anchors. |
| `GeoXAssetBundles` | `xr-geoxplorer-assets` | 48 GB raw | private | **3 (deferred)** | Too big for current strategy. Bulk assets stay on NAS. Revisit. |
| `CrystalViewer` | `xr-crystalviewer` | 718 MB | public | 2 | HoloLens crystal-structure viewer. Has README + GettingStarted.md. |
| `MineralHandSamples` | `xr-mineral-hand-samples` | 5.3 GB | public | **3 (deferred)** | ~30 FBX mineral specimens. Bulk assets stay on NAS. Revisit. |
| `LROAssetBundles` | `xr-lro-asset-bundles` | 456 MB | public | 2 | Lunar Reconnaissance Orbiter — Apollo 11/12/14/15/16 landing sites + Mono Lake. |
| `RoverTraverse` | `xr-rover-traverse` | 274 MB | public | 2 | Rover path viewer. HoloToolkit, `SiteSphere.prefab`. |
| `VolcanoViewer` | `xr-volcano-viewer` | 1.2 GB | public | 2 | Volcano visualization. Has `VideoCaptureLib`. |
| `SeismicityViewer` | `xr-seismicity-viewer` | 237 MB | public | 2 | Earthquake/seismicity viewer. Unity 5.5, old but targeted. |
| `KilaueaSono` | `xr-kilauea-sono` | 18 MB | public | **1 (PoC)** | Kilauea DEM sonification. Smallest project. |
| `3DPhaseDiagrams` | `xr-3d-phase-diagrams` | 461 MB | public | **1 (PoC)** | Petrology phase-diagram interactive. |
| `IntermediateTriggering` | `xr-intermediate-triggering` | 199 MB | public | 2 | HoloLens networked app, MRTK + Photon. Purpose unclear. |
| `VirtualEarth` | `xr-virtual-earth` | 4.2 GB | public | **3 (deferred)** | Bulk assets stay on NAS. Revisit. |
| `VirtualEarth2` | `xr-virtual-earth-2` | 776 MB | public | 2 | Newer iteration of `VirtualEarth`. |
| `MeltComplex` | `xr-meltcomplex` | 141 MB | public | 2 | Unity 5.4-HTP, very old. Push but mark `archived` — unlikely to resuscitate without full rewrite. |
| `Fossett Lab Demo` | `xr-museum-viewer` | 972 MB | private | 2 | **Renamed.** 2023 museum-artifact viewer (Buddhist sculptures, Met Museum objects). Private until provenance confirmed publishable. |

### Archived, preserve existing git history

| Local directory | Repo name | Visibility | Wave | Has own .git? | Notes |
|---|---|---|---|---|---|
| `GeoXplorer` | `xr-geoxplorer-v1` | private | 4 | yes — last commit 2018-09-19 "hirise update" | HoloLens-1 era canonical. Preserve sub-`.git` through push. |
| `DemoForDCO` | `xr-dco-demo` | public | 4 | yes — last commit 2019-06-06 "UI improvements 8" | Deep Carbon Observatory demo. |
| `GeoExplorer` | `xr-geoexplorer-original` | private | 4 | no | Predecessor to `GeoXplorer`. One isolated edit 2025-01-14 on `EarthView.unity`; rest frozen 2018. Push as reference-only. |

### Moved to `_archive/` (not pushed; decide later)

Moved to `/mnt/nas/dev/fossett_xr_apps/_archive/` on NAS:

| Local directory | Reason |
|---|---|
| `GeoXplorer 2` | Abandoned prototype. 16 .cs files. Features absorbed into `GeoXplorerM`. |
| `geoxplorer_mobile` | Strictly superseded by `__working` (same bundle ID, older Unity 2017.4 vs 2019.4). |
| `ASADemo` | Azure Spatial Anchors SDK demo, not a Fossett project. |
| `azure-spatial-anchors-samples` | Microsoft's official sample repo. Link to upstream instead of vendoring. Has its own git (vendor history). |
| `azure-spatial-anchors-samples-master` | Duplicate zip download of above. |
| `Unity3DTiles-master` | Third-party open-source library. Link upstream instead of vendoring. |
| `Bundles` | Unity 5.4-HTP. Minimal content. Probably dead. |
| `Demo` | Generic Unity template. 5.5.2. No Fossett content. |
| `New Unity Startup Project` | Untouched default Unity template. |
| `OculusGOTest` | Oculus Go platform is dead. Contains `.apk` but not store-relevant. |
| `Sandbox` | Build artifacts only (no real Unity project source). |

## Per-repo topic scheme

All repos:
- `unity`, `xr`, `fossettlab`

Plus platform (one of):
- `hololens`, `mobile-ar`, `multi-platform`, `desktop`

Plus domain (where applicable):
- `geoscience`, `mineralogy`, `volcanology`, `seismology`, `planetary-science`, `education`, `art-history` (for `xr-museum-viewer`)

Plus status:
- `active`, `archived`, `reference`

Plus Unity era (for quick upgrade-triage filtering):
- `unity-2019`, `unity-2017`, `unity-5`

## Org-level landing page

Create `fossettlab/.github` repo with `profile/README.md`:

```markdown
# Fossett Lab — WashU

Research tools & extended-reality applications from the Fossett Laboratory at
Washington University in St. Louis.

## GeoXplorer family
- [xr-geoxplorer](…) — unified HoloLens + iOS/Android geoscience explorer (current)
- [xr-geoxplorer-mobile](…) — mobile-only head (AR Foundation, iOS + Android)
- [xr-geoxplorer-se](…) — shared-experience variant (MRTK2 + Photon + Azure Spatial Anchors)
- [xr-geoxplorer-assets](…) — shared asset-bundle pipeline
- [xr-geoxplorer-v1](…) — 2018 HoloLens-1 baseline (archived, git history preserved)
- [xr-geoexplorer-original](…) — 2017 predecessor (reference)

## Standalone teaching / research applications
- [xr-crystalviewer](…)
- [xr-mineral-hand-samples](…)
- [xr-lro-asset-bundles](…) — lunar exploration
- [xr-rover-traverse](…)
- [xr-volcano-viewer](…)
- [xr-seismicity-viewer](…)
- [xr-kilauea-sono](…) — Kilauea sonification
- [xr-3d-phase-diagrams](…)
- [xr-virtual-earth](…), [xr-virtual-earth-2](…)
- [xr-intermediate-triggering](…)
- [xr-museum-viewer](…) — 3D museum artifacts
- [xr-dco-demo](…) — Deep Carbon Observatory demo (archived)
- [xr-meltcomplex](…) — magma phase diagrams (archived)

## Vendor dependencies (upstream links; not vendored here)
- Azure Spatial Anchors samples → github.com/Azure/azure-spatial-anchors-samples
- Unity3DTiles → github.com/Unity3DTiles/Unity3DTiles
```

## Mechanics: per-project push procedure

For each project to push:

1. `cd /mnt/nas/dev/fossett_xr_apps/<project>/`
2. If no `.git` exists, `git init` and add Unity `.gitignore` (use standard [github/gitignore/Unity.gitignore](https://github.com/github/gitignore/blob/main/Unity.gitignore)).
3. Commit: one initial `"Initial import from Fossett Lab NAS 2026-04-21"` commit. If the project already has a `.git`, skip this step.
4. Write a bespoke `README.md` per project: what it does, target platform, Unity version, build instructions if known, status (active/archived/reference).
5. `gh repo create fossettlab/xr-<name> --private --source=. --push`
6. `gh repo edit fossettlab/xr-<name> --add-topic unity --add-topic xr --add-topic fossettlab --add-topic <platform> --add-topic <status> --add-topic <unity-era> [etc]`
7. Record push status in this file (Done column below).

## Large-repo handling

Three repos exceed comfortable GitHub limits:

| Repo | Size | Strategy |
|---|---|---|
| `xr-geoxplorer-assets` | 48 GB (compiled bundles) | Push source only; `.gitignore` the AssetBundles/ folder. Store compiled bundles on NAS + optionally S3. README documents the rebuild procedure. |
| `xr-mineral-hand-samples` | 5.3 GB (FBX specimens) | Decide at push time: LFS, or split (code to GitHub, FBX specimens stay on NAS with a pull script). |
| `xr-virtual-earth` | 4.2 GB | Same as above. Note: has `AssetBundles_old/` which may be 90%+ of the bulk — prune before push if so. |

## Sequencing

**Wave 1 — DONE (2026-04-21):** `xr-kilauea-sono`, `xr-3d-phase-diagrams`. Public. PoC validated the full `git init` + `.gitignore` + `gh repo create` + topic pipeline. Mac-side CIFS was painfully slow for `git add` on 3823-file projects.

**Wave 2 — DONE (2026-04-21):** 12 repos pushed. New pipeline: pliny does the `git init` + `add` + `commit` (still CIFS-bound but avoids Mac-side tool-timeout races), then Mac does `gh repo create --push` + topics.
- Caught one large-file rejection: `xr-seismicity-viewer` contained `world.topo.bathy.200410.3x21600x10800.png` (186 MB, NASA Blue Marble). Added to `.gitignore`, amended commit, retried. File stays on NAS per "bulk assets on NAS" decision.

**Wave 3 — DEFERRED:** 3 oversized repos. `xr-mineral-hand-samples`, `xr-virtual-earth`, `xr-geoxplorer-assets`. Revisit LFS vs split-hosting when we decide to resurrect any of them.

**Wave 4 — PENDING:** 3 repos with preserved git history. `xr-geoxplorer-v1`, `xr-dco-demo`, `xr-geoexplorer-original`. Path differs — each has an existing `.git` we want to ship with history intact, plus a new `.gitignore` and README added on top. Needs care: `.gitignore` should be a fresh commit, not rebased into history, so the original `"hirise update"` and `"UI improvements 8"` commits stay visible.

## Retro / deferred

- **Unity project upgrades** — every project will need Unity 2022 LTS or Unity 6 to remain build-able. Separate project per app, not blocking initial push.
- **Apple Developer account recovery** — separate task, blocks any store republish.
- **GitHub Projects v2 board** — one board per active project, or one org-level board that tracks "which apps are buildable / store-ready / deprecated". Decide after first few repos exist.
- **CI builds** — each Unity repo could get a GitHub Action that does a headless Unity build on push. Deferred until we know which projects are worth the CI-license expense.
