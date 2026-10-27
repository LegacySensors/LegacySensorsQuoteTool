# Sensor Layout Planner (offline iPad web app)

Mark rooms on floor plans and evacuation maps, auto-lay out smoke/vape sensors, and export counts.
The program and every project live on the iPad. After the first install, no internet connection is needed.

## What's in this folder

| File | Purpose |
|---|---|
| `index.html` | The whole app (UI, drawing, sensor math, on-device storage) |
| `sw.js` | Offline cache. Keeps a copy of the app on the iPad |
| `manifest.webmanifest` | Name, icon and full-screen settings for the Home Screen install |
| `icons/` | App icons |

## Why it has to be hosted once

iPadOS only lets a web app install for offline use when it comes from an **HTTPS website**.
Opening `index.html` from the Files app will not work. The website is only used to deliver the
files and updates. Projects are never sent to it.

Any static HTTPS host works. Two easy ones:

**GitHub Pages**
1. Create a repository (for example `sensor-layout-planner`) and upload the contents of this folder to it, keeping `icons/` as a folder.
2. In the repository: Settings → Pages → Source: "Deploy from a branch" → branch `main`, folder `/ (root)` → Save.
3. After a minute the site is live at `https://<your-account>.github.io/sensor-layout-planner/`.
   Note: on a free GitHub plan the repository must be public for Pages. The code becomes public; project data does not, since it never leaves the iPad.

**Cloudflare Pages or Netlify:** create a site and upload this folder as-is. Both give an HTTPS address.

## Installing on the iPad

1. Open the site address in **Safari**. Other browsers on iPad cannot install to the Home Screen.
2. Tap **Share → Add to Home Screen → Add**.
3. Open the planner **from the new Home Screen icon** while still online, and let it sit a few seconds.
   That first launch stores the app and the PDF reader for offline use.
4. To check: turn on Airplane Mode and open it again.

Always use the Home Screen icon. iPadOS keeps the Home Screen app's data separate from Safari's,
so projects made in a Safari tab will not appear in the installed app.

## Backups

Projects → **Back up** creates one `.json` file containing the project, its rooms, sensors and plan images.
The share sheet opens, so you can choose Save to Files, AirDrop or Mail.
**Restore a project from a backup file** brings it back. If the project already exists, it is restored as a copy.

Back up anything you care about. iPadOS can clear website data if the device runs critically low on storage,
and deleting the Home Screen icon deletes its data.

## Updating the app

1. Edit the files.
2. In `sw.js`, change `VERSION` (for example `slp-2026-10-16-1` → `slp-2026-10-20-1`). **Required**, or iPads keep the old copy.
3. Upload to the host.
4. The next time the iPad opens the app while online, an "A new version is ready" bar appears. Tap **Update now**.

## Client report (PDF)

Floor panel (tap empty space on the plan) → **Client report (PDF)**. Each floor gets:
- **Page 1:** the plan with every sensor as a plain red dot, class-colored rooms, and a legend (company logo at the top). The legend is placed in empty space on the plan when there is room, otherwise beside it.
- **Page 2:** totals, rooms, floor area, plan scale, the by-class table, and **Meeting Notes**. Long notes continue on extra pages.

**Report, all floors** puts every floor in one PDF. **Plan page (PNG)** saves page 1 as an image.
Meeting notes are typed in the floor panel, one item per line, and saved with that floor.
Company name and logo are set once in Settings → Report branding, and are kept on the iPad for all projects.

## How sensors are counted

- Grid spacing = √(coverage sq ft). Cig/THC and Vape each have their own coverage, set in Settings.
- Each room gets `ROUNDUP(L ÷ spacing) × ROUNDUP(W ÷ spacing)` sensors, using the vape spacing if the room's class is a Vape class.
- Priority (P) sensors are added on top. Dragging an auto sensor switches that room to hand placement (M).
- Default coverage values (1,000 / 500 sq ft) are placeholders, not a sensor spec.

## Backup file format (for a future native app)

```
{
  "format": "sensor-layout-planner-backup", "version": 1,
  "project": { id, schema: 1, name, siteType,
               settings: { cigCov, vapeCov },               // sq ft per sensor
               classes: [ { id, name, mode: "vape"|"general"|"none", color } ] },
  "floors": [ { id, name, order, imgW, imgH,
                pxPerFt,                                    // null until scale is set
                scaleLine: { a:{x,y}, b:{x,y}, ft } | null,
                rooms:   [ { id, name, cls, x, y, w, h,      // plan-image pixels
                             dim: { l, w } | null,          // feet, typed by hand; overrides scale
                             auto: true|false } ],
                sensors: [ { id, x, y, roomId|null, note, detached } ],
                notesTitle, notes,                          // meeting notes; one item per line
                image: { type, base64 } } ]
}
```
A Swift version can read this file directly, so projects carry over.

## Known limits

- **Detect** fills the open area around your tap, out to the nearest dark lines. It works on clean line drawings and struggles with angled photos, busy colored evacuation maps, or heavy hatching. Use Wall darkness / Gap seal, or draw the rectangle by hand.
- **PDF import** needs the PDF reader cached on the first online launch. If a PDF won't load, screenshot the plan and import the image.
