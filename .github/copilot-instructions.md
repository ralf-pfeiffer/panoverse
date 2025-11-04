### Repo at-a-glance

- This is a small static frontend that displays a 360° panorama using Pannellum (v2.x) and a set of image hotspots.
- Single-page app: `index.html` is the app entrypoint. Panoramas live in `pano/` and supporting images are under `images/fulls/` and `images/thumbs/`.

### Big picture / architecture

- No build system — plain HTML + CDN-hosted JS/CSS. The app initializes `pannellum.viewer()` directly in `index.html`.
- Hotspots are defined inline in the Pannellum config in `index.html` under the `hotSpots` array. Each hotspot uses a custom `createTooltipFunc` to inject thumbnails from `images/thumbs/` and a `clickHandlerFunc` to call `showPopup(...)` which renders a simple modal.
- Image assets: thumbnails (small preview) vs fulls (high-resolution image shown in popup). Example files: `images/thumbs/DP-HAWK2A - rect.jpg`, `images/fulls/goat-rock/IMG20241007114333.jpg` and `pano/IMG20250415074116.jpg`.

### What an AI agent should know to edit code safely

- Single-file surface area: most interactive behavior lives in `index.html`. Edits will usually be to the Pannellum config object or the small helper `showPopup()`.
- Keep DOM/CSS changes minimal and inline-styled styles in `index.html` consistent with existing conventions (no external CSS files used).
- Pannellum usage examples in this file show these key patterns:
  - Hotspot sizing: created via `createTooltipFunc` by inserting an `<img>` and setting `hotSpotDiv.style.width`/`height` after `img.onload`.
  - Force redraw after image load by tweaking pitch: `viewer.setPitch(viewer.getPitch() + 0.0001)` then reversing it — leave this unless you test and confirm an alternative works across browsers.
  - Hotspot classes: use `pnlm-hotspot-base pnlm-hotspot pnlm-info pnlm-pointer custom-hotspot` when creating custom hotspots so pannellum styles still apply.

### Common edits and examples

- To add a hotspot: add a new object to the `hotSpots` array in `index.html`. Minimal example (follow existing pattern):

  - set `pitch` and `yaw` (numbers)
  - copy the `cssClass` string from existing hotspots
  - implement `createTooltipFunc` to set the thumbnail HTML and sizing on `img.onload`
  - implement `clickHandlerFunc` to call `showPopup(title, imgPath, caption, description)`

- To change the panorama image, edit the `panorama` path in the viewer config (e.g. `pano/IMG20250415074116.jpg`).

### Developer workflows (how to run/test)

- This is a static site — open `index.html` in a browser. For correct file:// image loading and to test CORS/asset paths reliably, run a local static file server. Example (macOS / zsh):

```bash
# from repo root
python3 -m http.server 8000
# then open http://localhost:8000/index.html
```

- Debugging tips:
  - Use the browser devtools console. `index.html` already logs diagnostic messages for thumbnail sizing and click events.
  - If thumbnails don't resize correctly, inspect `naturalWidth`/`naturalHeight` on the image and follow the existing pattern of computing ratio then setting div width/height.

### Project-specific conventions & gotchas

- No JS bundler, so avoid adding modules or import syntax unless you also add a build step and document it.
- Keep CSS small and inline in `index.html`. The existing style block contains layout and `.custom-hotspot` rules—extend it there.
- Filenames may contain spaces and hyphens (e.g., `DP-HAWK2A - rect.jpg`). When generating new asset paths, preserve actual filenames; prefer hyphens or underscores but match filesystem.
- The small viewer pitch tweak hack (0.0001) is deliberate and used to force hotspot redraws. Remove only after verifying cross-browser behavior.

### Integration points & external dependencies

- Pannellum (JS & CSS) are loaded from CDN: `https://cdn.jsdelivr.net/npm/pannellum@2.5.6/...`. Lock edits to versions when upgrading and test hotspots thoroughly after any upgrade.
- No server-side components or APIs are present in this repo.

### Quick references (files to open first)

- `index.html` — app entry, viewer config, hotspots, popup helper
- `pano/` — panorama images (example: `IMG20250415074116.jpg`)
- `images/thumbs/` and `images/fulls/` — thumbnails and full images used by hotspots

If a section is unclear or you'd like me to add a short examples section with exact copy-paste hotspot snippets for new hotspots, tell me which variant you prefer (thumbnail-only, thumbnail+caption, or remote image path) and I'll update the file.
