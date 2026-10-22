**Document version: 8** (2026-10-21)

# Chronobar: how to make a skin

Chronobar is a client time tracker shown as a small borderless window that stays on top of other windows. Its look is defined by a **skin**: a regular HTML page with styles. Shape and style are up to you: anything you can build with HTML and CSS inside the given size. The window is transparent, so a strip, a rounded plate, a circle or any other shape is just a transparent background plus images.

**Terms:** **client counter** is the client time counter (`.timer-btn`). **Timer** is the countdown (`.countdown-btn`).

## 1. What a skin consists of

A skin is a folder (or a zip with the same files, optionally inside one root folder) in the `Skins/` folder next to the exe.

```
MySkin/
├── skin.json     required
├── skin.html     required
├── skin.css      styles (practically required)
└── elements/     images and fonts, any subfolders
```

`skin.json`:

```json
{
  "name": "My Skin",
  "version": "1.0",
  "author": "Name",
  "description": "Short description",
  "window": { "width": 400, "height": 16 }
}
```

`window` is the window size in CSS pixels: width 16 to 2000, height 8 to 500. Without it the size is 400x16. **The skin sets the window size.**

## 2. Required elements

The app finds elements by **CSS class** (the first match is used, so use each class once). Everything else (markup, background, decoration) is up to you.

| Class | What the app does |
|---|---|
| `.drag-zone` | Drags the window with the left mouse button (the window snaps to screen edges). Make it a visible "handle". |
| `.combo` | A click opens the client list under the window. |
| `.combo-text` | The app writes the **client name** here (`textContent`). The text can be long: use `overflow:hidden; text-overflow:ellipsis; white-space:nowrap`. |
| `.timer-btn` | **Client counter.** Click = start/pause. The app writes the **time** here and adds the `.running` class while the counter runs. |
| `.settings-btn` | A click opens the menu (Reset Time, Skins, Scale, Reload clients, About, Exit and so on). |
| `.countdown-btn` | **Timer (Pro).** Left click opens a 5/10/15/20/30 min menu and starts the countdown, right click restarts from the last value. The app writes the **text** here (while counting `14:59` (MM:SS), and plain seconds `45` when less than a minute is left; when idle, including after zero, the last selected value `15 min`, which is kept between launches; `--` until a value is chosen). It also sets the CSS variable `--progress` (1 to 0, the remainder to zero) and, at zero, adds the `.alarm` class for about 1.3 s (the skin draws the flashes). While the countdown runs (until zero) the button has the `.running` class: the skin may animate the progress bar (in Default a 12x16 tile `elements/countdown_tile.png` at 10% opacity crawls left along it, the speed is set by `--cd-tile-period`). The same `.running` enables a frame layer on `.countdown-btn::after` (above the bar, transparent, lines at the top and bottom): configure it with `--cd-bar-color`, `--cd-frame-color`, `--cd-frame-width`, `--cd-frame-bg` or the rule `.countdown-btn.running::after`; at zero `.running` is removed and the same `::after` draws the `.alarm` flash. Recommended width: half of `.timer-btn`. In Free the button is hidden (`display:none`) and the window shrinks by its width. |
| `.ram-cell` | **RAM indicator (Pro).** It is shown or hidden with the general menu item `Show RAM`; the rest (view, interval, levels) the user changes in the cell's own menu (left or right click on it). Two views. **Numbers:** the app writes **text** (`63%`; `--%` before the first sample) and adds `.warn` or `.crit` to the cell by the warning and danger levels (75% and 90% by default). **Graph:** the app clears the cell, adds the `.graph` class and creates 10 `<span class="ram-bar">` elements inside (the newest on the right), no text. For each bar it writes `--h` (0 to 1, the load), and on the cell itself `--warn` and `--danger` (0 to 1, the levels). The skin draws a bar **at the full height of the cell**, colors it in zones with a `linear-gradient` driven by `--warn` and `--danger`, and clips it at the top: `clip-path: inset(calc((1 - var(--h)) * 100%) 0 0 0)`; this way the zones (gray, yellow, red) always sit at their levels. Colors are best kept in variables (in Default `--ram-ok`, `--ram-warn`, `--ram-danger`). Every 5th sample (counted since the history was cleared, not by position) gets the `.tick` class: the skin makes that bar slightly transparent (in Default `--ram-tick-opacity: 0.9`); it moves left with the bars and shows motion when the load does not change. The cell background is solid and is set by the skin. In hidden mode (and always in Free) the app hides the cell (`display:none`) and shrinks the window by its width. Width: the text `100%` plus about 10% on each side (32 px in Default). Place it before `.settings-btn`. |

Important:
- The app overwrites the content of `.timer-btn` and `.combo-text`, so **there must be no nested elements of your own inside them**, only text. Make decoration with backgrounds or `::before` / `::after`.
- The time format is `MM SS` (for example `05 07`) and `H:MM SS` (for example `1:05 07`) from one hour on. Leave room for at least `999:59 59`; a monospaced font is better. The initial text in the HTML (`00:00 00`) is replaced.
- The app gives the skin nothing else (only the client name, the counter time and the "running" state).
- If one of the seven classes is missing, the matching function is simply absent.
- For the window to shrink correctly without `.ram-cell` and `.countdown-btn` (in Free), the root wrapper of the skin (`.window`) must be the only direct child of `body` and as wide as the whole window: the app sets a new width on it, on `html` and on `body`. The rest of the layout must survive those cells disappearing (flex, `.combo { flex:1 }`).
- Draw the timer progress with `::before` and `width: calc(var(--progress, 0) * 100%)`, and the flashes with `.countdown-btn.alarm::after` and `@keyframes`.
- The context menu opens on right click only on `.settings-btn`; on other elements right click does nothing (except `.countdown-btn` and `.ram-cell` in Pro).

## 3. Button states

Use ordinary `:hover` and `:active`. The app tracks these states itself:
- `:active` ("pressed") applies only to the **left** mouse button;
- hover is cleared when the cursor leaves the window or a menu opens;
- `.timer-btn` has an extra state, `.running`: `.timer-btn.running`, `.timer-btn.running:hover`, `.timer-btn.running:active`;
- states can be set for **all** elements, including `.drag-zone`, `.combo`, `.settings-btn` and decoration;
- do not use `:focus`, `:focus-within`, `:visited`.

A typical button set: `normal`, `hover`, `pressed`, and for the counter three more for `running`. Switch images with `background-image` or `border-image-source`.

## 4. What is allowed and what is not

**Allowed:** any CSS (flex, grid, gradients, `border-radius`, shadows, `clip-path`, `transform`, `@keyframes` animations and `transition`), system fonts (Tahoma, Segoe UI, Consolas...), custom fonts via `@font-face` with a relative `url()`.

**Allowed tags:** `div span button img p b i em strong small label section article header footer nav ul ol li br hr table thead tbody tr td th`.
**Allowed attributes:** `class`, `id`, `title`; for `img` also `src`, `alt`, `width`, `height`.

**Not allowed (stripped or blocked):**
- JavaScript of any kind, `on*` attributes, `<script>`;
- the `style=` attribute (all styling goes in `skin.css`);
- `<svg>` in the markup, `<a>`, `<iframe>`, `<input>`, `<form>`, `<link>`, `<meta>`, `<base>`;
- `@import` and **any external address** (`http://...`): there is no network. Put everything you need into the skin files.

**Resources:** use relative paths (`url('elements/a.png')` or `<img src="elements/a.png">`, no `..`). Formats: png, jpg, gif, webp, svg (as an image only, not inline), woff2, woff, ttf, otf. One file up to 8 MB, the whole skin up to 24 MB, a zip up to 32 MB and up to 1000 files.

## 5. Size, transparency, scale

- The window is exactly the size set in `skin.json`. In CSS you must set:
  ```css
  html, body { margin:0; width:400px; height:16px; overflow:hidden;
               background:transparent; user-select:none; }
  ```
  (use your own `width` and `height`).
- Transparent areas are see-through but still belong to the window: clicks in them do not reach the desktop.
- In Pro the user can switch the scale to 100% / 150% / 200%. The window and its content are enlarged as a whole, the CSS sizes of the skin do not change. For a crisp result at larger sizes prefer **SVG** or images at 2x resolution with `background-size` in CSS pixels. At 200% a "crisp pixels" mode (no smoothing) is enabled.

## 6. Minimal skeleton

`skin.html`:

```html
<!DOCTYPE html>
<html><head><meta charset="UTF-8"></head>
<body>
  <div class="window">
    <div class="drag-zone"></div>
    <div class="combo"><span class="combo-text">Client</span></div>
    <button class="timer-btn">00 00</button>
    <button class="settings-btn"></button>
  </div>
</body></html>
```

`skin.css` (the essence):

```css
.window { width:400px; height:16px; display:flex; align-items:center; background:#222; }
.drag-zone { width:12px; height:16px; cursor:grab; background:#555; }
.drag-zone:hover { background:#777; }
.drag-zone:active { background:#999; }
.combo { flex:1; height:16px; cursor:pointer; padding:0 8px; display:flex; align-items:center; }
.combo:hover { background:#333; }
.combo-text { color:#fff; font:10px Tahoma; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
.timer-btn { width:110px; height:16px; border:0; cursor:pointer; color:#fff; font:bold 13px Consolas; background:#333; }
.timer-btn:hover { background:#444; }
.timer-btn.running { background:#1d5; }
.settings-btn { width:18px; height:16px; border:0; cursor:pointer; background:#555; }
.settings-btn:hover { background:#777; }
```

## 7. Checklist before you ship

1. `skin.json`, `skin.html` and `skin.css` exist, and each of the seven classes appears once.
2. `width` and `height` in `skin.json` match the CSS (`html, body`, `.window`).
3. Inside `.timer-btn` and `.combo-text` there is only text.
4. No scripts, `style=`, external addresses or forbidden tags.
5. Every button has `:hover` and `:active`, and the counter also has `.running`.
6. A long client name is cut with an ellipsis, and the time fits in `.timer-btn` (check `999:59 59`).
7. All image paths are relative and the files exist.
8. The skin looks right in Free too (without `.countdown-btn` and `.ram-cell`, the window narrower by their width).

## 8. Free and Pro

- Until **November 1, 2026, 00:00 UTC** the beta runs: Pro is unlocked for everyone. After that date the app works as Free.
- **Free** shows only `.drag-zone`, `.combo` (+ `.combo-text`), `.timer-btn` (client counter) and `.settings-btn`. The timer (`.countdown-btn`) and the RAM cell (`.ram-cell`) are hidden with `display:none`, and the window shrinks by their total width.
- In Free only the `Default` skin is available (a `Default` folder or `default_skin`), at 100% scale and up to 3 clients. Skins by other authors work in Pro.
- Any cells and features added later will belong to Pro: in Free the app hides them in the same way.
