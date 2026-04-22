# Fossett Lab XR — Unity Upgrade Plan

Working plan for bringing the 17 `fossettlab/xr-*` app repos from their archived 2017–2021 Unity versions into a modern, buildable, store-publishable stack. Companion to `PLAN.md` (the migration plan) and `INVENTORY.md`.

## Why upgrade at all

Several forces put ratcheting pressure on these projects:

| Pressure | Current state on our projects | Deadline-ish |
|---|---|---|
| Google Play `targetSdkVersion` minimum | All mobile repos target SDK 29 (Android 10). Google requires ≥34 (Android 14) for new submissions; 35 from Aug 2025. | Blocking any store update today. |
| Apple App Store iOS SDK minimum | Projects built against iOS SDKs circa 2021. Apple requires iOS 17 SDK+ for new submissions. | Blocking any store update today. |
| Unity LTS support windows | Unity 2019 LTS ended 2022-04. Unity 2017/2018 ended long before. Security + build-toolchain updates stopped. | Already out of support. |
| MRTK v2 → MRTK v3 transition | All HoloLens projects use HoloToolkit (deprecated) or MRTK v2 (maintenance-only). | Medium-term friction; current code will keep working but new features land only on v3. |
| **Azure Spatial Anchors retired 2024-05** | `xr-geoxplorer`, `xr-geoxplorer-mobile`, `xr-geoxplorer-se` all use ASA for shared-experience features. | **Service no longer operational.** Any multi-user shared-experience feature is already broken in production. |
| HoloLens hardware EOL | Microsoft discontinued HoloLens 2 production in 2024; official support through 2027. No successor announced. | Long-term: strategic decision about which HMD platform the lab targets. |
| iOS / Android / Windows UX platform drift | 5+ years of platform evolution (notch-aware layouts, scoped storage, ARCore/ARKit feature expansion, Windows on ARM). | Cosmetic / ergonomic, not blocking, but compounds over time. |

The ASA retirement is the biggest **active** problem — any GeoXplorer feature that relied on shared-experience anchors is already non-functional regardless of whether we upgrade Unity.

## Target-stack decisions (lab direction set 2026-04-21)

### 1. Unity version — Unity 6 LTS

Decision: **Unity 6 LTS** (not Unity 2022.3).

Rationale: now that the lab has committed to Quest 3 as the sole HMD (see §4 below), Meta's XR SDK roadmap points at Unity 6. Meta XR SDK v65+ adds Quest-specific features (depth API, MR Utility Kit, scene understanding v2) that target Unity 6 as their primary platform; Unity 2022.3 support on those features lags or is frozen. For mobile AR, Unity 6 is a clean AR Foundation 6 landing spot. The ecosystem-maturity argument that favored 2022.3 a year ago is thinner now.

Caveat: any third-party package critical to a specific project needs a Unity-6 compatibility check before we start the upgrade for that project.

### 2. XR toolkit — Meta XR SDK (All-in-One) + OpenXR

Decision: **Meta XR All-in-One SDK** on OpenXR. Drop MRTK 3 from the plan entirely.

Rationale: the earlier MRTK 3 recommendation assumed we were keeping HoloLens. With HoloLens dropped and Quest 3 as the sole HMD, MRTK 3 is a cross-platform abstraction we don't need. Going Meta-native gets us:

- Passthrough MR APIs that just work (HoloLens-style world-anchored content is done differently on Quest 3's video passthrough; Meta's SDK handles the differences).
- Scene understanding (Meta MR Utility Kit) for room geometry, which replaces HoloLens spatial-mapping workflows in a Quest-idiomatic way.
- Hand tracking v2.3+ directly supported without a translation layer.
- Building Blocks (Meta's prefab-based template system) as starting points for common patterns.

Trade: if we ever need another HMD (Vision Pro, PSVR2, next-gen HTC) we'd pay an integration cost. Acceptable given the Quest-only commitment.

### 3. Shared-experience / Azure Spatial Anchors — drop by default, investigate before replacing

Decision: **drop multi-user features for the first pass of every upgrade.** The ASA service is retired so the existing code is already non-functional, and the GeoXplorer apps work as single-user experiences.

**Open investigation (added today):** what was the multi-user feature actually used for? We have three codepaths that used it — `xr-geoxplorer`, `xr-geoxplorer-mobile`, `xr-geoxplorer-se` ("SE" = shared experience). Before we decide whether to rebuild shared-experience, someone should spend ~half a day on:
- Reading the README and commit messages in the `xr-geoxplorer-se` repo (preserved git history) to see what the original author intended.
- Searching for any published paper, poster, or talk that referenced GeoXplorer's multi-user capability.
- Talking to the departing dev's mentor or a lab member who saw it in use.
- If no one remembers it being used: leave dropped.
- If it was core to a teaching moment or research demo: plan for a rebuild.

Replacement options if we decide shared-experience matters:
- **Meta Shared Spatial Anchors** (best fit for Quest 3). Local-network co-location, free, part of the Meta SDK. Direct replacement for the co-located classroom-demo use case.
- **Photon Fusion** or **Unity Netcode for GameObjects** for networked state sync. Both are alive and well. Photon PUN (what the old code used) is legacy and should be replaced by Fusion on any rewrite.
- **Google Cloud Anchors via AR Foundation** — only if we also need cross-device anchor sharing on mobile AR, and only if the outdoor-AR use case justifies it.

### 4. HMD target — Quest 3 only

Decision: **Quest 3 only for headset-based XR.** HoloLens support dropped entirely.

Implications:
- All HoloLens-only projects need to decide: port to Quest 3 (if the use case benefits from a headset), port to mobile AR (if phone/tablet is fine), port to desktop (if XR isn't essential), or retire.
- The remaining-private GeoXplorer-family repos (`xr-geoxplorer-v1`, `xr-geoxplorer-se`, `xr-geoexplorer-original`) lose their "might revive on HoloLens" rationale. They're reference-only going forward; the useful work moves into Quest 3 ports.
- Mobile AR remains important — not everyone has a Quest 3 — but it's a separate deliverable, not a fallback for the HMD version.

### 5. Per-project target-platform decision

Every project now needs a platform target: **Quest 3**, **Mobile AR (iOS + Android)**, **Desktop**, or **Retire**. Triage table below.

## Triage framework

Per-project decision tree:

```
Is there a current user (teaching / research / store listing)?
├── Yes → Is the project's content valuable enough to re-create from scratch?
│   ├── Yes → REWRITE in current Unity (avoids Unity-upgrade pain)
│   └── No  → UPGRADE in place (preserve content, pay upgrade cost)
└── No  → Is the project's content of historical or reference value?
    ├── Yes → ARCHIVE as-is (no upgrade; preserve for reference only)
    └── No  → RETIRE (delete / consolidate, keep INVENTORY entry)
```

## Per-project triage (Quest 3 / Mobile AR / Desktop)

Effort tiers: **S** (1–3 days), **M** (1–2 weeks), **L** (3–6 weeks), **XL** (rewrite / 6+ weeks).

Every project gets a platform decision plus an action verdict. Platform choices:
- **Quest 3** — headset-immersive use cases that benefit from room-scale + stereo depth + passthrough MR.
- **Mobile AR** — phone/tablet reach via AR Foundation (iOS + Android). Best for teaching where everyone in the room has a device.
- **Desktop** — flat-screen Unity when XR isn't essential.
- **Retire** — not worth rebuilding.

### Priority 1 — store-relevant and high-value

| Repo | Current | Target platform | Effort | Verdict |
|---|---|---|---|---|
| `xr-geoxplorer-mobile` | Unity 2019.4, AR Foundation, iOS+Android. In App Store as `com.FossettLab.GeoXplorer` | **Mobile AR** (unchanged) | **M** | **UPGRADE** to Unity 6 + AR Foundation 6 + Android SDK 34+ + iOS 17 SDK. Highest single-project ROI — reclaims the live store listing. Strip any ASA code (multi-user). |
| `xr-geoxplorer` | Unity 2019.4, MRTK 2 + Photon + ASA, HoloLens-primary | **Quest 3** (new) + **Mobile AR** (keep via `xr-geoxplorer-mobile`) | **L** | **PORT to Quest 3.** Unity 6 + Meta XR SDK rewrite of the interaction layer. Reuse scenes, data loaders, geoscience content. Drop multi-user unless the investigation (below) shows we need it. |
| `xr-geoxplorer-se` | Unity 2019.2, MRTK 2 + Photon + ASA — "shared experience" HoloLens variant | — | **—** | **CONSOLIDATE** — fold any unique content into `xr-geoxplorer`, then treat as archive. Its reason-for-being was the ASA+Photon co-located demo, which we're dropping (possibly adding back via Meta Shared Spatial Anchors if the investigation shows value). |

### Priority 2 — teaching / research tools with clear domain value

Platform choice matters here: Quest 3 is elegant but limited to whoever is wearing the headset. Mobile AR reaches every phone in the classroom. Desktop works in a browser-less context. Default to mobile AR for teaching reach unless there's a specific depth/stereo argument.

| Repo | Current | Recommended platform | Effort | Verdict |
|---|---|---|---|---|
| `xr-kilauea-sono` | Unity 2017.4, desktop | **Desktop** (unchanged) | **S** | **UPGRADE** Unity version only. No XR component; classroom-friendly as a desktop app. |
| `xr-3d-phase-diagrams` | Unity 2017.4, desktop | **Desktop** primary + optional **Mobile AR** companion | **S–M** | **UPGRADE** for desktop. Mobile-AR version would be a nice-to-have but desktop is sufficient for teaching. |
| `xr-crystalviewer` | Unity 2017.1, HoloToolkit HoloLens | **Mobile AR** (change) | **XL** | **REWRITE** as a mobile-AR crystal viewer. Crystals are inherently hand-held-scale — phone AR is a better fit than a Quest 3 session, and reaches every student. |
| `xr-mineral-hand-samples` | Unity 2019.2, HoloLens/UWP | **Mobile AR** (change) | **L** | **PORT** to mobile AR. 30 FBX specimens, already asset-bundled. Same argument as `xr-crystalviewer` — specimen-scale content fits phone-held form factor. Resolves the "Wave 3 deferred" question if we drop compiled bundles and rebuild. |
| `xr-lro-asset-bundles` | Unity 2019.2, HoloLens/UWP | **Quest 3** (change) + **Desktop** companion | **L** | **PORT** to Quest 3 if planetary-science revival is active — Apollo landing sites benefit from headset-immersive scale. Mobile version would feel cramped; desktop is viable as fallback. |
| `xr-rover-traverse` | Unity 2017.4, HoloToolkit | **Quest 3** (change) | **L–XL** | **PORT** if active, **ARCHIVE** if not. Rover path at scale is well-suited to Quest 3 passthrough + room-scale. |
| `xr-volcano-viewer` | Unity 2017.4, HoloToolkit | **Quest 3** or **Desktop** | **XL** | **REWRITE** or **RETIRE.** `VideoCaptureLib` dependency is defunct. Depends on whether a specific volcano-teaching or research use case still needs it. |

### Priority 3 — specialized or unclear use

| Repo | Current | Recommended platform | Effort | Verdict |
|---|---|---|---|---|
| `xr-intermediate-triggering` | Unity 2019.2, MRTK 2 + Photon HoloLens | — | **—** | **RETIRE** unless someone can explain what it was. Name suggests experimental scratch work; no README; no clear domain content. |
| `xr-virtual-earth-2` | Unity 2017.4, HoloToolkit HoloLens | **Quest 3** or **Retire** | **XL** | **REWRITE** from scratch in Unity 6 + Meta XR SDK + [Cesium for Unity](https://cesium.com/platform/cesium-for-unity/) (which replaces the old Bing VirtualEarth dependency cleanly). Only justified if globe-scale geospatial viewing is a lab-active use case. |
| `xr-dco-demo` | Unity 2017.4, HoloLens + WebGL. Original git 2019 | **Desktop** or **Retire** | **L** | Depends on whether DCO demo is still something the lab shows. If yes: upgrade desktop-only Unity. Drop HoloLens + WebGL targets. If not: **ARCHIVE.** |
| `xr-museum-viewer` | Unity 2017.4, HoloToolkit | **Mobile AR** or **Desktop** | **M–L** | Depends on who's using the 2023 museum content. If active, **REWRITE** as mobile AR (museum artifacts are inherently shareable-on-phone content). If inactive, **ARCHIVE.** |

### Priority 4 — archive or retire (no upgrade)

| Repo | Verdict | Rationale |
|---|---|---|
| `xr-geoxplorer-v1` | **ARCHIVE** | Historical HoloLens reference. Superseded by Quest 3 port. |
| `xr-geoexplorer-original` | **ARCHIVE** | Predecessor. Reference only. |
| `xr-meltcomplex` | **RETIRE** | Unity 5.4-HTP HoloLens. Rewrite from scratch if ever needed. |
| `xr-seismicity-viewer` | **RETIRE** | Unity 5.5 HoloLens. Same reasoning. |

### Deferred-hosting (Wave 3 of migration, not upgrade)

| Repo | Why deferred |
|---|---|
| `xr-geoxplorer-assets` | 48 GB compiled asset bundles. Hosting strategy decision independent of Unity upgrade. If we're moving to Quest 3 via Meta XR SDK, the old HoloLens-targeted AssetBundles likely need regeneration anyway. |
| `xr-virtual-earth` | 4.2 GB. Obsoleted by `xr-virtual-earth-2` decision — if we retire/rewrite v2, v1 also retires. |

## Common migration work (reusable across projects)

Each upgrade is project-specific, but shared foundations avoid redoing setup per repo:

1. **`fossettlab/xr-geoxplorer-mobile`** as the reference mobile-AR upgrade (Phase A). Its recipe — Unity 6 + AR Foundation 6 + Android SDK 34 + iOS 17 + deep-link scheme — carries directly into `xr-crystalviewer`, `xr-mineral-hand-samples`, `xr-museum-viewer` when those go mobile-AR.
2. **`fossettlab/xr-geoxplorer`** as the reference Quest 3 port (Phase B). Its recipe — Unity 6 + Meta XR All-in-One SDK + OpenXR + Building Blocks + (optional) Meta Shared Spatial Anchors — carries into the other Quest 3 ports.
3. **Document both recipes** in `fossettlab/xr-inventory` as they're worked out. Future projects start from a known-good template instead of re-learning the stack.
4. **"Fossett XR template" starter projects** (optional but recommended once the two reference upgrades are done): one for Unity 6 + Meta XR + Quest 3, one for Unity 6 + AR Foundation 6 + mobile. A new lab project starts from the template, not from scratch.
5. **Shared `.gitattributes` + `.gitignore`** across all Unity repos, with Git-LFS patterns for large binary assets (FBX, PNG, TIF, WAV, Blend) agreed once and applied uniformly.

## Sequencing proposal

**Phase A — Unblock the App Store app.**
- `xr-geoxplorer-mobile` — Unity 2019.4 → **Unity 6**, AR Foundation 4 → 6, Android SDK 29 → 34+, iOS deployment target → 17+. Strip any ASA code (there may not be any in this mobile-only repo, verify first). Republish to App Store + Google Play. Keeps the live store listing.
- Effort: ~2–4 weeks of Unity-capable dev time. Good first project for a CS undergrad learning Unity + AR Foundation; needs occasional senior review on store-submission details.
- Deliverable: one signed `.aab` in Play Console internal testing, one `.ipa` in TestFlight. Then public release once validated.

**Phase B — Quest 3 port of GeoXplorer.**
- `xr-geoxplorer` — new Unity 6 project built on **Meta XR All-in-One SDK**. Port the scenes, data loaders, and geoscience content from the existing Unity 2019.4 project; rewrite the interaction layer against Meta's Building Blocks and hand-tracking APIs. Drop all ASA / Photon / MRTK dependencies. Leave a `MULTIUSER.md` note in the repo so a future rebuild knows to start with Meta Shared Spatial Anchors if multi-user comes back.
- Effort: 4–8 weeks. More senior dev effort than Phase A; Meta XR SDK has a learning curve on MR-specific patterns (passthrough, scene understanding, hand pose semantics).
- Deliverable: a Quest 3 build sideloadable via SideQuest, feature-complete vs the Unity 2019.4 HoloLens single-user mode.
- Parallel investigation: the multi-user-use-case investigation described in §3 of target-stack decisions. If findings say multi-user matters, add a `MULTIUSER-plan.md` to this repo and budget ~3-4 additional weeks for the Meta Shared Spatial Anchors integration before public release.

**Phase C — Teaching-tool quick wins.**
- `xr-kilauea-sono` — desktop Unity 2017.4 → Unity 6 upgrade. S-effort. Classroom-friendly deliverable.
- `xr-3d-phase-diagrams` — same. S–M effort.
- Good parallel work for the undergrad while Phase B is in flight (different enough that they won't step on each other).

**Phase D — Mobile AR ports.**
- `xr-crystalviewer` — new Unity 6 mobile AR rewrite. Crystals are phone-scale content; classroom reach beats Quest 3 lock-in.
- `xr-mineral-hand-samples` — same pattern. ~30 FBX specimens imported into the new mobile-AR template.
- `xr-museum-viewer` — same, if provenance check confirms it's Fossett-Lab-appropriate content.
- Effort per project after the Phase A recipe exists: M each.

**Phase E — Quest 3 content ports (as needed).**
- `xr-lro-asset-bundles` — Quest 3 port if lunar-exploration revival is active.
- `xr-rover-traverse` — Quest 3 port if rover research is active.
- `xr-volcano-viewer` — Quest 3 or desktop, if needed.
- Each is a "pick up when there's a use case" candidate, not something we schedule speculatively.

**Phase F — Retirements and decisions-by-absence.**
- `xr-meltcomplex`, `xr-seismicity-viewer`, `xr-intermediate-triggering` — retire unless someone steps up.
- `xr-virtual-earth`, `xr-virtual-earth-2` — decide based on whether globe-scale geospatial viewing is still active lab work.
- `xr-dco-demo` — decide based on DCO engagement.
- `xr-geoxplorer-se`, `xr-geoxplorer-v1`, `xr-geoexplorer-original` — remain as archive-only reference repos.

## Tooling & process

- **Unity licensing:** free Unity Personal works for educational lab use under the current revenue/team-size thresholds. Unity Student is an alternative route for the undergrad. Pro licenses not needed.
- **Quest 3 dev hardware:** the lab needs at least one Quest 3 for the dev to use. If there isn't one already, budget ~$500. Sideloading via [SideQuest](https://sidequestvr.com) or Meta's own "Developer Hub" for testing builds.
- **Apple Developer Program membership** ($99/yr) required for App Store republish. Status confirmed active as of 2026-04.
- **Google Play Console account** ($25 one-time, presumably already exists from the original publish).
- **CI builds:** GitHub Actions + [GameCI](https://game.ci/) can do headless Unity builds per push. Worth adding once Phase A completes (store builds benefit most from CI consistency). Skip for archived projects.
- **Branch strategy per repo:** `main` = current (archived) state. Each upgrade goes on a branch like `upgrade/unity-6-meta-xr` or `port/quest3`. Squash-merge only when the build passes and someone has smoke-tested on-device.
- **Testing:** minimal formal testing. Per-project smoke-test checklist (does it launch? hand tracking work? core interaction flow work? no obvious visual glitches in passthrough?). Add TestFlight beta + Google Play Internal Testing for `xr-geoxplorer-mobile` specifically before public release.
- **Undergrad-friendly onboarding:** Phase A is a good first project (mobile AR, tractable scope, clear deliverable, existing store listing to validate against). Budget 2–4 weeks of onboarding before productive output — Unity basics + AR Foundation + store-submission pipeline. Pair-program the first-time submission to TestFlight / Play Internal Testing.

## Open decisions for the lab

Most of the big directional decisions are now made:

- ✅ Unity 6 LTS
- ✅ Meta XR SDK (Quest 3 only, no HoloLens, no MRTK 3)
- ✅ Drop multi-user by default; investigate historical use before deciding whether to rebuild on Meta Shared Spatial Anchors
- ✅ Phase A (`xr-geoxplorer-mobile` republish) is the first concrete deliverable
- ✅ Primary developer: CS undergrad (interested, part-time, learning curve)

Remaining decisions, each of which can be made as that phase nears:

1. **Multi-user investigation outcome.** Someone needs to figure out what the ASA + Photon "shared experience" actually did in practice. If we can't find anyone who remembers it being used, default to dropping for good. (Half-day's work; see §3 of target-stack decisions.)
2. **Teaching-tool priority order.** Of `xr-crystalviewer`, `xr-mineral-hand-samples`, `xr-museum-viewer`, which does the lab actually want first? Depends on the teaching calendar.
3. **Which Priority-3 projects get revived vs retired.** `xr-virtual-earth-2`, `xr-dco-demo`, `xr-rover-traverse`, `xr-lro-asset-bundles`, `xr-volcano-viewer` each need a "is anyone using this?" yes/no before they get scheduled.
4. **Store-app republish timing.** Phase A is ready to start as soon as the undergrad is onboarded. No reason to block it on later phases.
5. **Mentorship capacity.** Who reviews the undergrad's PRs? Who pairs on the first store submission? Lab-level decision.
