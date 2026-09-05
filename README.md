# CompanyNews

Digital signage for Ignition 8.3. Put news, safety counters and shift information on the TVs around
your plant, and manage it from inside Perspective.

Made by [Parsley Automation](https://www.parsleyautomation.com).

![A slide playing, with the corner clock and the department ticker along the bottom](docs/images/slideshow.png)

## What it does

Upload an image, pick which departments see it, set how long it stays up. That's the basic loop.

Beyond that:

- **Emergency slides.** Mark a slide as an emergency and give it a tag path. When that tag goes true,
  every screen switches to it — no button to press and nobody who has to be watching. When it clears,
  the normal rotation starts again. If the tag goes bad quality that counts as *not* firing, so a PLC
  dropping off the network can't put an evacuation notice on your walls.
- **Views as slides.** Point a slide at a Perspective view you built and it goes in the rotation.
  Charts, tables, live tag values, your own styling.
- **Ticker.** A line along the bottom, per department. You can put live tag values in the text:
  `Line 3 running at {[default]Line3/Rate} units/hr`.
- **Clock.** Corner clock, date and time, a few format options.

Screens on the same department stay in step. Slide position comes from the clock rather than a
counter in each browser, so a TV that reboots rejoins where the others already are instead of
starting the loop again on its own.

Images are stored on the gateway filesystem, outside the Ignition data directory. That keeps them
out of your `.gwbk` backups, which is the main reason this exists rather than storing slides in a
database as base64.

## What you need

Ignition 8.3 or later, with Perspective. Screens can be anything with a modern browser — a smart TV,
a Chromebox, a Pi.

No database, no internet connection, no account anywhere. Licensing is checked on your own gateway,
so it works on an air-gapped network.

## Installing

Download the `.modl` from [Releases](../../releases), then **Config → Modules → Install or Upgrade a
Module** in the gateway web interface. Ignition 8.3 needs a gateway restart before the module runs.

After that you drop two or more components onto Perspective views. One is the editor, so you can give
HR (say) a single URL where they manage slides. Then you set up a view for each department, with that
component's `screenKey` set to the department name from the editor.

![The News Editor: slide list, duration and transition, scheduling, and the departments that can see each slide](docs/images/editor.png)

Full setup — emergency trigger tags, ticker tag values, backups, redundancy and a troubleshooting
table — is in [docs/INSTALL.md](docs/INSTALL.md).

## Licensing

Without a key the gateway plays the first **2 slides of your library, in total** — not 2 per
department — and then a Parsley Automation slide. If those first 2 slides all belong to one
department, the other departments will only see the notice slide. That's what the limit means.

Nothing gets deleted and nothing stops serving. Every held slide starts playing again the moment a
key goes in.

To license a gateway, go to **Config → Services → CompanyNews**, copy the Gateway ID, and send it to
us. You paste the key back into the same page.

Keys are checked offline. The module doesn't call home and doesn't send us anything. If a key
expires it drops back to the 2-slide behaviour above — it won't blank a screen.

## About the emergency slides

Worth being direct about this one.

The emergency slide is a fast way to get something onto a screen. It is not a fire alarm and not an
emergency notification system. It isn't listed or certified to UL 864, UL 2572 or NFPA 72, and it
depends on the gateway, the network, the PLC and the browser on each TV — any of which can fail
without telling you.

Whatever you use for emergency notification should be a system built and maintained for that job.
Treat anything CompanyNews shows as extra information on top of it.

Full terms are in [LICENSE.txt](LICENSE.txt), section 0.

## Known limits

- Slides and images aren't replicated between redundant gateways. Edit on the active node and copy
  the media folder and slide library across yourself.
- A `.gwbk` doesn't contain your slides or images. Use the export on the CompanyNews config page.
- Each screen showing a **view slide** runs its own copy of that view. Tag bindings mostly share;
  named queries and database bindings don't — twenty screens means twenty queries per refresh.
  Prefer tag bindings, and test with the number of screens you'll actually run.

## Support

[Support@parsleyautomation.com](mailto:Support@parsleyautomation.com)

Include your Gateway ID from **Config → Services → CompanyNews** and the module version from
**Config → Modules**.

---

[Changelog](CHANGELOG.md) · [License](LICENSE.txt)
