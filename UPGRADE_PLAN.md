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

## Target-stack decisions (need lab sign-off before work starts)

Four decisions gate everything else:

### 1. Unity version target

| Option | Pros | Cons |
|---|---|---|
| **Unity 2022.3 LTS** | Mature (2+ years in-field), full asset-store support, MRTK 3 officially targets it, AR Foundation 5 shipped on it. LTS support through 2025 + 2 years of security updates after. | Shorter remaining shelf life. Two upgrades-from-now. |
| **Unity 6 LTS** (released mid-2024) | Longer support runway, newer render pipeline, better ARM/Apple Silicon support. | Newer ecosystem — some third-party packages haven't caught up. MRTK 3 support is catching up but not as battle-tested. |
| Unity 6.1 / 6.2 non-LTS | Latest features | No LTS guarantees. Avoid. |

**Recommendation: Unity 2022.3 LTS for the first wave of upgrades.** Lowers risk for projects that need to Just Build. Revisit Unity 6 for any project that needs >2 years of forward runway.

### 2. HoloLens-era toolkit replacement

HoloToolkit is 10+ years dead. MRTK v2 is in maintenance. Options for HoloLens-family work:

- **MRTK 3** (on OpenXR): Microsoft's current toolkit, cross-platform OpenXR support (HoloLens 2, Quest 3, Vive, etc.). Most code-compatible upgrade path from MRTK 2.
- **Raw OpenXR + lightweight utilities**: skip MRTK entirely, use Unity's XR Interaction Toolkit directly. Thinner, future-proof, but more custom integration work.

**Recommendation: MRTK 3.** The existing projects are already organized around MRTK-style interaction patterns; MRTK 3 lets us keep the conceptual model.

### 3. Azure Spatial Anchors replacement

ASA is gone. For shared-experience / cloud-anchor features:

| Option | Fit for lab use case |
|---|---|
| **Niantic Lightship VPS** | Commercial, strong localization, has a free tier for research. Our most direct replacement for "persistent anchored content in the real world." |
| **Google Cloud Anchors (via AR Foundation)** | Lighter-weight, session-scoped shared anchors. Free. Mobile-only (no HoloLens). |
| **Self-hosted anchor system** | Full control, but substantial engineering. Probably not worth it. |
| **Drop multi-user features** | Reduces GeoXplorer to single-user exploration. Acceptable if the multi-user use case was never heavily used. |

**Recommendation: start with "drop multi-user"** to unblock the single-user upgrade path for GeoXplorer mobile. Add cloud anchors back later via Google Cloud Anchors (AR Foundation) if the use case justifies it. Niantic VPS is overkill unless there's a specific outdoor-AR project.

### 4. HoloLens hardware future

Open question for the lab, not something this plan resolves:

- Continue targeting HoloLens 2 (supported through 2027, existing lab hardware)?
- Add Quest 3 / Quest Pro targets (cheaper, consumer-grade, strong Unity support)?
- Add Apple Vision Pro (visionOS, separate SDK entirely)?
- Pivot primary XR work to mobile AR (broadest reach, no headset cost)?

Recommend making this call before committing to the full HoloLens MRTK 3 port.

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

## Per-project triage

Effort tiers: **S** (1–3 days), **M** (1–2 weeks), **L** (3–6 weeks), **XL** (rewrite / 6+ weeks).

### Priority 1 — store-relevant or active-use candidates

| Repo | Unity | Platform | Blockers | Effort | Verdict |
|---|---|---|---|---|---|
| `xr-geoxplorer-mobile` | 2019.4 | iOS + Android (AR Foundation) | Android SDK 29 → 34+, iOS SDK update, AR Foundation 4 → 6, Apple Dev account still active, bundle ID intact. App is currently in App Store. | **M** | **UPGRADE.** Highest-value single project in the portfolio. Reclaims the store listing. |
| `xr-geoxplorer` | 2019.4 | HoloLens + mobile (MRTK 2 + Photon + ASA) | MRTK 2 → 3, ASA removal, Unity 2019.4 → 2022.3, Photon version check. | **L** | **UPGRADE** or **REWRITE**, lab decision. Rewriting lets us drop baggage from the attempted HoloLens / mobile unification if we're willing to go single-platform first. |
| `xr-geoxplorer-se` | 2019.2 | HoloLens (MRTK 2 + Photon + ASA) | Same as `xr-geoxplorer`. Unclear whether "shared experience" is differentiated from `xr-geoxplorer` or just a variant of it. | **L** | **CONSOLIDATE** into `xr-geoxplorer` if the SE variant no longer has a distinct purpose. Otherwise same L upgrade path. |

### Priority 2 — teaching/research tools with clear domain value

| Repo | Unity | Platform | Effort | Verdict |
|---|---|---|---|---|
| `xr-kilauea-sono` | 2017.4 | Desktop | **S** | **UPGRADE** if used in teaching. No toolkit dependencies; mostly a Unity version bump. |
| `xr-3d-phase-diagrams` | 2017.4 | Desktop | **S–M** | **UPGRADE** if used in teaching. Desktop-only, some scripting to review. |
| `xr-crystalviewer` | 2017.1 | HoloLens (HoloToolkit) | **XL** | **REWRITE** in Unity 2022.3 + MRTK 3. HoloToolkit is too far gone; easier to rewrite than lift-and-shift. |
| `xr-mineral-hand-samples` | 2019.2 | HoloLens/UWP + AssetBundles | **L** | **UPGRADE** if mineralogy-teaching use is current. Content (~30 FBX specimens) is valuable. Deferred from migration Wave 3 — need LFS or hybrid hosting. |
| `xr-volcano-viewer` | 2017.4 | HoloLens (HoloToolkit) | **XL** | **REWRITE** or **RETIRE.** HoloToolkit origin, limited clear use case. |
| `xr-rover-traverse` | 2017.4 | HoloLens (HoloToolkit) | **L–XL** | **UPGRADE** if rover-traverse research is active; otherwise **ARCHIVE.** |
| `xr-lro-asset-bundles` | 2019.2 | HoloLens/UWP + AssetBundles | **M–L** | **UPGRADE** if lunar-exploration use is current. Asset-bundle pipeline adds complexity. |

### Priority 3 — specialized or unclear use

| Repo | Unity | Platform | Effort | Verdict |
|---|---|---|---|---|
| `xr-intermediate-triggering` | 2019.2 | HoloLens (MRTK 2 + Photon) | **L** | **INVESTIGATE then decide.** Purpose not documented; may be scratch work. Probably retire. |
| `xr-virtual-earth-2` | 2017.4 | HoloLens (HoloToolkit) | **XL** | **REWRITE** or **RETIRE.** Bing/VirtualEarth integration likely needs a current replacement (Cesium? Google Maps Platform?). |
| `xr-dco-demo` | 2017.4 | HoloLens + WebGL | **L–XL** | **UPGRADE** if DCO connection is still relevant; else **ARCHIVE.** WebGL adds its own upgrade burden. |
| `xr-museum-viewer` | 2017.4 | HoloLens (HoloToolkit) | **M–L** | Depends on who's using the museum content. If active, **REWRITE** as a mobile-AR viewer (artifacts are inherently portable-viewer content). If inactive, **ARCHIVE.** |

### Priority 4 — retire / archive as-is

| Repo | Verdict | Rationale |
|---|---|---|
| `xr-geoxplorer-v1` | **ARCHIVE** (no upgrade) | Historical reference. `xr-geoxplorer` is the successor. Git history preserved. |
| `xr-geoexplorer-original` | **ARCHIVE** (no upgrade) | Predecessor to `xr-geoxplorer-v1`. Reference only. |
| `xr-meltcomplex` | **RETIRE** or rewrite from scratch | Unity 5.4-HTP era, 2017 source. Full ground-up rebuild is cheaper than upgrade. |
| `xr-seismicity-viewer` | **RETIRE** or rewrite from scratch | Unity 5.5, 2017 source. Same reasoning. |

### Deferred (Wave 3) — decide later

| Repo | Why deferred |
|---|---|
| `xr-geoxplorer-assets` | 48 GB compiled asset bundles; needs a hosting strategy independent of the Unity upgrade question. |
| `xr-virtual-earth` | 4.2 GB. Blocked on same hosting strategy + on whether `xr-virtual-earth-2` is the head. |

## Common migration work (reusable across HoloLens projects)

Instead of treating each MRTK-based project as a separate upgrade, establish shared patterns:

1. **`fossettlab/xr-geoxplorer`** as the reference upgrade. First project to go through the full pipeline; everything learned informs the others.
2. **Document a "HoloLens → MRTK 3 recipe"** in the `xr-inventory` repo: which Unity packages to add, which to remove, how to map HoloToolkit/MRTK2 prefabs to MRTK 3 equivalents, how to handle input/gaze/hand-tracking rewiring.
3. **Shared `.gitattributes` + `.gitignore`** for all Unity repos (LFS patterns, Library/, obj/, etc.).
4. **Optional: a "Fossett XR template" Unity project** with MRTK 3 + AR Foundation + lab code-style pre-configured. Future projects start from this template, don't rebuild from scratch.

## Sequencing proposal

**Phase A: Unblock the store app (highest ROI).**
- `xr-geoxplorer-mobile` — Unity 2019.4 → 2022.3 LTS, Android SDK 29 → 34, iOS deployment target update, AR Foundation 4 → 5 or 6. Strip any ASA dependency if present. Target: republish to App Store + Play Store.
- Effort estimate: ~2–4 weeks of Unity-capable dev time.

**Phase B: Reference HoloLens upgrade.**
- `xr-geoxplorer` — full MRTK 2 → 3 migration on Unity 2022.3. Drop multi-user ASA dependency; document pattern. Possibly consolidate `xr-geoxplorer-se` into this.
- Effort estimate: 4–8 weeks.

**Phase C: Teaching-tool quick wins.**
- `xr-kilauea-sono`, `xr-3d-phase-diagrams` — desktop Unity upgrades, each S-effort. Low risk, classroom-visible results.

**Phase D: Rewrite or retire the HoloToolkit-era projects** based on lab use cases. Decisions driven by whether anyone is actively using them, not by "can we upgrade them."

**Phase E: Asset-bundle / large-repo projects** (`xr-geoxplorer-assets`, `xr-mineral-hand-samples`, `xr-virtual-earth`) — deferred, decide on hosting strategy first.

## Tooling & process

- **Unity licensing:** each developer needs a Unity Pro or Student license. Free Unity Personal works under certain revenue/team size thresholds.
- **CI builds:** GitHub Actions + [GameCI](https://game.ci/) can do headless Unity builds per push. Useful once Phase A completes; premature for archived projects.
- **Branch strategy:** `main` = last-known-good (current archived state). `upgrade/unity-2022` or similar for each project's upgrade work. Merge only when the upgrade build passes.
- **Testing:** minimal — HMD hands-on smoke tests per project is enough; no unit tests existed in the originals. If any project goes back into the App Store, add TestFlight beta + Google Play Internal Testing for sanity checks before public release.

## Open decisions for the lab

Before Phase A kicks off:

1. **Unity version target:** 2022.3 LTS or Unity 6 LTS? (Recommend: 2022.3.)
2. **HoloLens future:** keep HoloLens 2 as primary HMD, or pivot to Quest 3 / Vision Pro / mobile-first? (Influences Phase B onward.)
3. **Multi-user / shared-experience ambition:** drop entirely, defer, or commit to replacing ASA with something like Google Cloud Anchors?
4. **Who does the dev work:** you + another lab member, a PhD student hire, a contractor? Informs effort-to-calendar conversion.
5. **App Store republish priority:** Phase A as written, or defer while the broader Fossett Lab XR strategy is worked out?

None of these are urgent, but Phase A shouldn't start until at least decisions 1 and 3 are made.
