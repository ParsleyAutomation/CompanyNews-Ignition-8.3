# Changelog

All notable changes to CompanyNews. Versions match what Config → Modules shows.

## 1.0.0.75 — 2026-09-07

Rebrand to Parsley Automation. Functionally identical to 1.0.0.71 — no feature or behaviour changes.

### Breaking

The module identifier and both component identifiers changed, because the company name is part of
each one:

| Was | Now |
|---|---|
| `com.centralvalleyignition.companynews.CompanyNews` | `com.parsleyautomation.companynews.CompanyNews` |
| `cvi.display.newsslideshow` | `pa.display.newsslideshow` |
| `cvi.input.newseditor` | `pa.input.newseditor` |
| `/data/cvi-companynews/` | `/data/pa-companynews/` |

Ignition treats this as a different module, so upgrading is not in place:

1. Export your setup from the old version first — the slide library and images are not carried over
   by anything else.
2. Uninstall the old module in **Config → Modules**, then install this one.
3. Re-place both components on your views. A view still referencing `cvi.display.newsslideshow`
   renders nothing, because that component no longer exists.
4. Import your setup, then re-enter your license key under **Config → Services → CompanyNews**.

Keys issued for 1.0.0.x remain valid; they are bound to the gateway, not to the module identifier.

### Changed

- The unlicensed notice slide now reads **Parsley Automation** and points at
  `www.parsleyautomation.com`.
- The Perspective palette category is now **Parsley Automation**.
- The bundled end user license agreement names Parsley Automation.

## 1.0.0 — 2026-08-19

First public release.

### Features

- **Slides** — images with per-slide duration, transition and department scoping, plus optional
  start/end scheduling with daily, weekly or monthly recurrence.
- **Emergency slides** — held out of the normal rotation and shown on every screen the moment their
  trigger tag goes true. The gateway subscribes to the tag itself: no script, no button, nobody who
  has to notice. Bad tag quality counts as *not* firing, so a PLC dropping off the network cannot
  evacuate a building. Lowest priority wins when several fire at once, so every screen picks the
  same one. Rotation restarts when the emergency clears.
- **View slides** — a slide renders a Perspective view you built, by path. Only the slide currently
  on screen is mounted, so a rotation of many view slides still costs one live view per screen.
- **Ticker** — scrolling bar along the bottom, department-scoped, with `{[provider]Path/To/Tag}`
  value templates and its own scheduling. A tag that cannot be read shows `--` rather than a gap.
- **Clock** — optional corner clock, date and time, with format options.
- **Multi-screen sync** — screens sharing a department show the same slide at the same moment. Slide
  position is derived from the wall clock rather than a per-browser counter, so a TV that reboots
  rejoins the rotation in step instead of restarting the loop.
- **Backup and restore** — export the whole setup, images included, as one archive; import restores
  it onto any gateway. An Ignition `.gwbk` does **not** contain your slides or images, because media
  lives outside the data directory by design.
- **Offline licensing** — signed, gateway-bound keys validated on the gateway itself. No activation
  server, no phone-home, no telemetry; works on air-gapped networks. Unlicensed gateways play the
  first 2 slides of the library **in total** — not 2 per department — plus a Parsley Automation
  slide. Nothing is deleted, and an expired key never blanks a screen.
- **Redundancy aware** — the backup half of a redundant pair refuses content edits, because nothing
  is replicated between nodes and an edit made there would be lost at the next failover. Reads and
  tag subscriptions continue on both, so a failover stays seamless and an alarm during the failover
  window still reaches the screens.

### Storage

Media is stored on the gateway filesystem outside the Ignition data directory, under keyed-hash
names, and served with immutable cache headers — so uploads never bloat a `.gwbk` and are not
round-tripped through a database as base64.

### Known limitations

- Slides and media are **not replicated between redundant gateways**. Edit on the active node and
  copy the media root and slide library to the backup yourself.
- A `.gwbk` does not contain slides or images. Use the built-in export.
- Each screen showing a **view slide** runs its own view instance. Tag bindings largely coalesce at
  the tag provider; named-query and database bindings do not. Prefer tag bindings and test with the
  screen count you actually deploy.
