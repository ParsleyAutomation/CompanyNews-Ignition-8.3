# CompanyNews — Installation and Setup

Digital signage for Ignition 8.3. Upload images, play them full-screen on department TVs, and
interrupt them with a tag-driven emergency notice.

This guide takes you from a downloaded `.modl` to a working screen on a wall. It assumes you can
sign in to the Ignition Gateway and open the Designer. It does not assume any Java or module
development.

**Before you start, read [`LICENSE.txt`](../LICENSE.txt) — in particular section 0. The emergency
slide feature is not a life-safety system and must not be relied on as one.**

---

## 1. Requirements

| | |
|---|---|
| Ignition | 8.3.0 or later |
| Modules | Perspective (required) |
| Screens | Any device with a modern browser — smart TV browser, Chromebox, Raspberry Pi, thin client |
| Disk | Free space on the gateway for your images, outside the Ignition data directory |

Nothing else. No database, no internet connection, no cloud account.

---

## 2. Install the module

1. In the Gateway web interface, go to **Config → Modules**.
2. Scroll to the bottom and choose **Install or Upgrade a Module**.
3. Select `CompanyNews-<version>.modl` and install it.
4. Accept the module certificate if prompted. CompanyNews is signed by Parsley Automation.

**Restart the gateway.** Ignition 8.3 requires it for any module install or upgrade; the module
will not run until you do.

To confirm it is running, open:

```
http://<your-gateway>:8088/data/pa-companynews/status
```

You should get a small block of JSON. `mediaWritable` must be `true`; if it is `false`, see
[Where images are stored](#7-where-images-are-stored).

---

## 3. Add the two components to a project

CompanyNews ships two Perspective components. In the Designer they appear in the component palette
under **Parsley Automation**.

| Component | Where it goes |
|---|---|
| **News Slideshow** (`pa.display.newsslideshow`) | A view that TVs open. One per screen layout. |
| **News Editor** (`pa.input.newseditor`) | A view your staff open to manage content. |

### The player view

1. Create a new Perspective view, for example `Signage/Screen`.
2. Set its layout to **Coordinate** or **Flex**, filling the page.
3. Drag **News Slideshow** onto it and size it to fill the view.
4. Select the component and set the **`screenKey`** property.

`screenKey` is what makes a screen show one department's slides. Use `All` for a screen that shows
everything. Otherwise use a short key with no spaces — `Shop`, `Warehouse`, `Lobby`, `Line3`.

### The editor view

1. Create another view, for example `Signage/Manage`.
2. Drag **News Editor** onto it and size it generously — it is a full admin UI, not a widget.

> **Restrict this view.** Anyone who can open the editor view can change what appears on every
> screen. Put it behind an Ignition security level, a role check, or an identity provider, exactly
> as you would any other admin screen. See [Access and security](#8-access-and-security).

---

## 4. Point a TV at a screen

A Perspective client URL uses the **page path**, not the view path, so first give your player view a
page. In the Designer, open **Perspective → Page Configuration**, add a page such as `/screen`, and
point it at the view you made in section 3.

Then open this in the TV's browser, full screen:

```
http://<your-gateway>:8088/data/perspective/client/<ProjectName>/screen
```

> Using the view path here (`.../News/Signage/Screen`) gives **"View Not Found — No view configured
> for this page"** even though the view exists. That message means the page mapping is missing, not
> the view.

Screens sharing a `screenKey` stay in step with each other. Slide position is derived from the wall
clock rather than counted per browser, so a TV that reboots rejoins the rotation where everyone else
is instead of restarting the loop.

**Tips for a wall display**

- Use the browser's kiosk or full-screen mode so no address bar shows.
- Disable the device's screen saver and sleep timer.
- If the TV has an "auto power on" setting, enable it, so a power blip brings the screen back.

---

## 5. Add content

Everything below happens in the editor view.

### Departments

The department list is yours to define. Click **+ Manage** next to the department chips to add,
rename, or remove departments. A department's key is what you put in a player's `screenKey`.

### Slides

**Slides** tab → **+ Upload** → choose a JPEG, PNG, GIF, or WebP up to 15 MB.

For each slide you can set:

- **Duration** — how long it stays on screen.
- **Transition** — how it arrives. **Preview** plays it once so you can see it.
- **Departments** — which screens show this slide. `All` means everywhere.
- **Enabled** — off hides it without deleting it.
- **Schedule** — optional start and end date/time, with daily, weekly, or monthly recurrence. Outside
  its window a slide is simply skipped.

Deleting a slide removes the image from the gateway's disk too, provided no other slide uses it.

### View slides

**Views** tab → **+ View** → type the path of a Perspective view you built, for example
`Dashboards/Line3`.

This puts your own view into the rotation — charts, tables, gauges, whatever the Designer can build.

> **Cost warning.** Every other slide type is effectively free per screen: an image is a cached
> file, and a ticker tag is one gateway subscription shared by every screen. A view slide is
> different. Each screen showing it runs its own copy of the view, with its own bindings. Tag
> bindings mostly coalesce; **named-query and database bindings do not** — twenty screens on a view
> with a SQL binding is twenty queries per refresh. Prefer tag bindings, and test with the number of
> screens you actually plan to run.

Only the slide currently on screen is loaded, so a rotation with ten view slides still runs at most
one view at a time per screen.

### Ticker

**Ticker** tab. A scrolling bar along the bottom of every screen in the chosen departments.

Messages can include live tag values using braces:

```
Line 3 running at {[default]Line3/Rate} units/hr — {[default]Line3/Pct}% of target
```

A tag that cannot be read shows `--` rather than leaving a gap, so a broken path is visible instead
of silently blank. The tab shows the live value of every tag you reference.

Ticker messages support the same scheduling as slides, plus recurrence. A one-off message whose end
time passes is removed automatically; a recurring one stays and reappears on its next window.

### Clock

**Clock** tab. An optional corner clock, configured once for all screens. Date, time, or both, with
format options.

---

## 6. Emergency slides

An emergency slide is held **out** of the normal rotation and takes over every screen the moment its
trigger tag becomes true.

1. On the **Slides** tab, add or select a slide and mark it as an emergency slide.
2. Set its **trigger tag** — a gateway tag path, for example `[default]Alarms/FireAlarm`.
3. Set a **priority** if you have more than one. The lowest number wins, so every screen picks the
   same slide when several fire at once.

The gateway subscribes to the tag itself. There is no button to press and nobody who has to notice —
that is the point.

**How it behaves**

- A tag with **bad quality counts as NOT firing.** A PLC dropping off the network must not evacuate
  your building.
- When the emergency clears, the rotation **restarts at the first slide**.
- Screens check for emergencies every few seconds, separately from the slower content poll, so the
  delay is seconds rather than the length of a rotation.
- The editor shows the live state of every trigger tag, including a path that resolves to nothing —
  the failure that otherwise looks perfectly configured.

**Read section 0 of the EULA.** This is a display convenience, not a fire alarm.

---

## 7. Where images are stored

Images are stored on the gateway **filesystem, outside the Ignition data directory**, and served
with cache headers. This is deliberate: storing them in the database as base64 makes every gateway
backup enormous.

The consequence matters:

> **An Ignition gateway backup (`.gwbk`) does NOT contain your slides or images.**
> Restoring a `.gwbk` onto a fresh gateway gives you an empty signage system.

**Back up separately.** Copy both of these on whatever schedule suits you:

| What | Where |
|---|---|
| Slide library | `<Ignition data dir>/modules/companynews/slides.json` |
| Images | The media root — shown at `/data/pa-companynews/status` as `mediaRoot` |

To change where images live, use the media root setting in the editor. It takes effect at the next
gateway restart, and existing images are **not** moved — point it at a directory you have already
populated, or one you are happy to start empty.

If `mediaWritable` is `false` in the status output, the gateway cannot write to that directory.
Check filesystem permissions for the account the Ignition service runs as.

---

## 8. Access and security

The module's HTTP routes are reachable **without gateway authentication**. This is by design: a
Perspective session has no gateway login session for a route to check, so requiring one would break
the editor for everyone.

**Access control is your job, in two places:**

1. **The editor view.** Gate it with Ignition project security. Anyone who can open it controls
   every screen.
2. **The network.** The routes under `/data/pa-companynews/` are reachable by anything that can
   reach your gateway's web port. If your gateway is exposed beyond your plant network, restrict it
   with your firewall or reverse proxy.

Image URLs use long unguessable names, so an image cannot be found by guessing — but it is not
secret. Do not put anything on a slide you would not put on a wall.

---

## 9. Licensing

Licensing lives on the **gateway**, not in the editor: go to **Config → Services → CompanyNews** in
the Gateway web interface. That page requires a gateway login, so whoever manages slides cannot
change the license or download the whole library.

**Without a license key** CompanyNews plays the first **2 slides of your library in total** — not 2
per department — followed by a Parsley Automation notice slide. A department whose slides are
not among those first 2 shows only that notice. Nothing is deleted and nothing stops working: every
slide you have configured starts playing again the moment a key is entered.

**To license a gateway:**

1. Open **Config → Services → CompanyNews**.
2. Copy the **Gateway ID** shown there.
3. Send it to Parsley Automation with your order.
4. Paste the key you receive into the License field and click **Activate**.

It takes effect immediately — no restart.

**How licensing works**

- Keys are validated **entirely offline**. The module never contacts Parsley Automation or any other server, and
  sends no telemetry. It works on air-gapped networks.
- A key is bound to one gateway installation. Moving to a different gateway needs a new key.
- A key may carry an expiry date. When it expires the module drops back to unlicensed behaviour
  above — **screens keep playing**. An expired key never blanks a wall.
- If a key is rejected, the page shows exactly why: expired, bound to a different gateway, wrong
  product, or malformed.

### Backing up your signage

The **Backup** section on that same page is how you capture what a gateway backup does not — see
[Where images are stored](#7-where-images-are-stored).

- **Export** downloads one zip containing the slide library and every image it references.
- **Import and replace** restores such a zip, replacing the current library.

An export restores onto **any** gateway, not just the one it came from: image names are derived from
a per-gateway secret, so import re-stores each image under the new gateway's naming and rewrites
every slide to match.

## 10. Troubleshooting

| Symptom | Cause and fix |
|---|---|
| Screen is blank | Check the view's `screenKey` matches a department that has enabled slides. A slide outside its schedule window is skipped. |
| "View Not Found" on the TV | The URL uses a view path instead of a page path. Add a page in Perspective > Page Configuration and use that. See section 4. |
| Only 2 slides play, then a notice slide | The gateway is unlicensed. The cap is 2 slides **in total**, not per department, so some departments may show only the notice slide. See [Licensing](#9-licensing). |
| No License tab in the editor | Correct — licensing moved to **Config → Services → CompanyNews**, behind a gateway login. |
| Upload fails | Check `mediaWritable` at `/data/pa-companynews/status`. Files over 15 MB are refused, as are SVGs. |
| Emergency slide never fires | The editor's trigger status line says why. The usual causes are a tag path that resolves to nothing, or a tag whose quality is bad — bad quality deliberately counts as not firing. |
| Ticker shows `--` | That tag could not be read. The Ticker tab shows the live value and quality of every tag you reference. |
| Screens show different slides | They are on different `screenKey`s, or one screen's clock is far off. Screens derive position from the wall clock. |
| A view slide is blank | The view path is wrong, or the view failed to load. Check the path in the Designer. |
| "This is the backup gateway" when saving | You are editing the backup half of a redundant pair. Slides are not replicated between nodes — edit on the active gateway. |
| Editor shows "changed by someone else" | Another admin saved while you were editing. Reload and re-apply your change; this exists so their work is not silently erased. |

---

## 11. Redundant gateways

CompanyNews supports redundant pairs with one restriction: **content is not replicated between
nodes.** Each gateway keeps its own slide library and its own images.

- Make all edits on the **active** gateway. The backup refuses content edits and says so.
- The backup keeps serving playlists and images from its own copy, so a failover is seamless for
  the screens.
- Both nodes watch the emergency trigger tags, so an alarm during a failover still reaches screens.
- **After changing content, copy the slide library and media root to the backup yourself**, or the
  backup will serve whatever it last had when it takes over.
- Each node needs its own license key, since a key is bound to one gateway installation.

---

## Support

Parsley Automation — Support@parsleyautomation.com

Include your Gateway ID (License tab) and the module version (Config → Modules).
