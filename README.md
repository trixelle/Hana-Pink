# Hana Pink — FFXIV photo archive

A personal GPose photography site for Hana Pink, built to run inside
Carrd. The photos live in Cloudinary and are tagged there; the site reads those tags and
builds the galleries by itself, so adding a photo never means touching code.

Bilingual (English / 简体中文), works on phones and desktops, and the whole thing is about
110 KB of hand-written HTML, CSS and JavaScript. No frameworks, no build step.

---

### How it fits together

Carrd can't host files, and each of its Embed elements is capped at 16,384 characters, so
the site ships as 14 embeds pasted into one Carrd page. They're numbered in the order they
need to appear:

```
01–05  CSS          Hidden → Head
06–07  markup       Inline, where the site should appear
08–14  JavaScript   Hidden → Body End
```

Part 08 creates a single global, `window.PM`, and every later part hangs its piece off it:
settings, wording, core helpers, the gallery, the photo viewer, the chat log, and the
startup code. Nothing else touches the global scope, so it can't collide with Carrd's own
scripts. Load order matters for 08–14, and the only thing enforcing it is the order of the
elements in Carrd's panel, so keep the numbering.

| File | What's in it |
|---|---|
| `01-css-1-base.html` | Google Fonts link, design tokens, paper and ground treatment, margin ornaments |
| `02-css-2-header.html` | header, viewfinder motif, landing screen, facts strip |
| `03-css-3-gallery.html` | section headings, teleport list, category view, photo grid |
| `04-css-4-plate.html` | character profile, linkshell, footer |
| `05-css-5-layers.html` | photo viewer, teleport bar, chat log, toast, intro card |
| `06-markup-1-page.html` | the page itself |
| `07-markup-2-layers.html` | overlay markup, moved to the end of `<body>` on load |
| `08-js-1-settings.html` | Cloudinary account, categories, links, character profile |
| `09-js-2-text.html` | every string on screen, English and Chinese |
| `10-js-3-core.html` | helpers, Cloudinary loading, Eorzea clock |
| `11-js-4-gallery.html` | language switching, profile rendering, teleport list, landing frames |
| `12-js-5-zone.html` | category view, justified grid, photo viewer |
| `13-js-6-chat.html` | chat log and slash commands, GPose mode, keyboard, navigation |
| `14-js-7-effects.html` | custom cursor, sparkle trail, ambience, startup |

`hana-pink-site.html` in the root is the same site in one file. It's the quickest way to try
a change: open it in a browser, get it right, then move the change into the matching part.
The live site runs on the 14 parts, not on that file.

### Where the photos come from

Cloudinary exposes a JSON list of everything carrying a given tag:

```
https://res.cloudinary.com/<cloud>/image/list/<tag>.json
```

That's one request per category, made the first time it's needed and then kept in memory.
Each entry includes the image's dimensions, so tiles reserve the right space before the
image arrives and nothing jumps around while loading. Thumbnails are requested at three
fixed widths (480/960/1600) through `srcset`, and the viewer asks for 960, 1600 or 2400
depending on the screen. Cloudinary generates and caches each size once, which keeps the
free tier's transformation count predictable.

Nothing polls. The only things on a timer are the Eorzea clock and the REC counter, and both
are arithmetic, not network calls.

Two quirks of that endpoint: the list is cached for roughly 60 seconds, and it caps out at
1,000 photos per tag.

### Other bits worth knowing

- **Infinite scroll** is batched rendering, not paging. The full list arrives in one
  request, and an IntersectionObserver adds 18 more tiles as you near the bottom.
- **The grid** is a flexbox justified layout: each tile's `flex-grow` is its aspect ratio,
  with a very greedy `::after` filler so the last row doesn't stretch. No JS layout pass.
- **Deep links** work. `?zone=close-up` opens that category, and the back button behaves.
- **Ambience** is generated with the Web Audio API: a high open chord plus occasional bell
  phrases. There's no audio file unless you set `musicUrl`, and nothing loads or plays until
  someone presses the ♪ button.
- **Reduced motion** is respected. The intro card, teleport animation, sparkle trail and
  parallax all switch off.

### Settings reference

```js
cloudName     // Cloudinary account name; empty shows grey placeholder images
thumbWidths   // widths offered to the browser for grid thumbnails
fullWidths    // widths used by the photo viewer
batchSize     // tiles added per scroll step
defaultLang   // 'en' or 'zh' for first-time visitors
heroImage     // pin one image as the first landing frame (optional)
profileImage  // portrait for the character plate
cursor        // custom cursor images, around 32×32
cursorTrail   // sparkles following the pointer on desktop
musicUrl      // direct link to an audio FILE; a YouTube page will not work
castTime      // length of the teleport cast bar, in ms
showIntro     // title card, once per visit
```

`REGIONS` is the category tree, `LINKS` the contact rows, `PROFILE` the character sheet.
Everything is written as `{ en: '…', zh: '…' }` pairs so both languages sit side by side.

---

## Credits

FINAL FANTASY XIV © SQUARE ENIX CO., LTD. Screenshots are used under Square Enix's material
usage licence. The photographs are Hana Pink's.

Fonts are Cormorant Garamond, Chakra Petch, Sixtyfour and Noto Sans/Serif SC, all from
Google Fonts under the SIL Open Font License.