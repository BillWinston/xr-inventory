<!-- Round 2 smoke test by BillWinston, 2026-05-04 -->
# xr-inventory

Planning and inventory documents for the Fossett Lab `xr-*` project
portfolio. Tracks the 2026-04 migration of the lab's Unity/HoloLens/
mobile-AR apps from the `geospatial_data` NAS share into the new `dev`
share, and the subsequent push to GitHub under the `fossettlab` org.

## Contents

- [`INVENTORY.md`](./INVENTORY.md) — structural catalog of all projects
  that lived under `/mnt/nas/dev/fossett_xr_apps/` at migration time.
  Unity versions, platforms, status, best-guess purpose per project.
- [`PLAN.md`](./PLAN.md) — migration plan and execution log: waves of
  pushes, visibility decisions, outstanding questions.

## Related

- GitHub org: <https://github.com/fossettlab>
- NAS source of truth: `/mnt/nas/dev/fossett_xr_apps/` on the Bradley
  Lab Synology (pliny mounts it at that path; Mac mounts `smb://<nas>/dev`).
<!-- Smoke test PR by BillWinston, 2026-05-04 -->
