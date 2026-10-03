# Hana Pink — FFXIV photo archive

This is the site I built for Pink (Hana Pink, Gilgamesh / Aether) to show off her GPose
photography. It runs on Carrd, the photos live in Cloudinary, and the whole thing is
bilingual (English / 简体中文).

![The landing screen](preview.png)

Keeping my notes here so future me remembers how any of this works.

---

## The basic idea

Pink shouldn't have to touch code to add a photo, and I shouldn't have to re-upload
anything every time she shoots. So:

**She uploads a photo to Cloudinary and gives it a tag. The site reads the tags and builds
the galleries itself.**

The site asks Cloudinary "give me everything tagged `soft-dreamy`",
Cloudinary hands back a JSON list, and the gallery gets built from that list in the browser.
No database, no backend, no re-publishing the Carrd page.

Tags that matter right now:

| Group | Tags |
|---|---|
| Featured (banner + landing screen) | `featured` |
| Portraits 人像写真 | `portraits` |
| Atmosphere 氛围场景 | `soft-dreamy` `dynamic-mood` |
| Glamour 幻化穿搭 | `light-palette` `dark-palette` |
| Together 双人及多人 | `couples` `groups` |

Groups with more than one set also get an automatic "All" entry that merges them, so
visitors can see all of Atmosphere in one click without me tagging anything extra.

Pink's own instructions are in `docs/cloudinary-guide-zh.md` (Chinese). My longer notes on
the Cloudinary side are in `docs/cloudinary-guide.md`, and the Carrd/design side is in
`docs/carrd-design-guide.md`.

---

## What's in each file

The site is one Carrd page made of 14 Embed elements. They have to go in this order,
because the later scripts use what the earlier ones set up.

| File | What it does | Goes in Carrd as |
|---|---|---|
| `01-css-1-base.html` | Fonts, colours, the paper-on-a-table look, the margin ornaments | Hidden → Head |
| `02-css-2-header.html` | Header bar, viewfinder styling, the landing screen, the facts strip | Hidden → Head |
| `03-css-3-gallery.html` | Section headings, the teleport list, the photo grid | Hidden → Head |
| `04-css-4-plate.html` | Her character profile, the links section, the footer | Hidden → Head |
| `05-css-5-layers.html` | Photo viewer, teleport bar, chat log, toast, intro card | Hidden → Head |
| `06-markup-1-page.html` | The actual page content | Inline |
| `07-markup-2-layers.html` | The overlay bits; the script moves these to the end of the page | Inline, right after 06 |
| `08-js-1-settings.html` | **The file I actually edit.** Cloud name, categories, links, her profile | Hidden → Body End |
| `09-js-2-text.html` | Every word on screen, English and Chinese side by side | Hidden → Body End |
| `10-js-3-core.html` | Shared helpers, the Cloudinary fetching, the Eorzea clock | Hidden → Body End |
| `11-js-4-gallery.html` | Language switching, the profile, the teleport list, landing frames | Hidden → Body End |
| `12-js-5-zone.html` | Category view, the photo grid, the photo viewer | Hidden → Body End |
| `13-js-6-chat.html` | Chat log and slash commands, GPose mode, keyboard shortcuts, nav | Hidden → Body End |
| `14-js-7-effects.html` | Custom cursor, sparkle trail, ambient sound, startup | Hidden → Body End |

Part 08 makes one global object, `window.PM`, and every file after it hangs its piece off
that. Nothing else goes into the global scope, so it can't clash with whatever Carrd loads.

`hana-pink-site.html` is the same site as a single file. That's what I open locally to try
something quickly — then I move the change into whichever part it belongs to. The live site
runs on the 14 parts, not on that file.

To check the split version the way Carrd runs it:

```sh
sh tools/build-preview.sh > preview.html && open preview.html
```

---

## Some fun stuff

- The landing screen is a camera viewfinder: REC dot, a timecode counting up, thirds grid,
  focus brackets, and a shutter button that flips to the next featured photo.
- There's a chat log bottom-left with slash commands. `/help`, `/tp glamour`, `/gpose`,
  `/time`, plus emotes like `/wave` and `/meowdy`.
- `/gpose` (or H in the photo viewer) hides the whole interface so only the photo is left.
- Ambient sound is generated in the browser, no audio file. It's off until you press ♪.
- `?zone=soft-dreamy` deep-links to a set, and the back button works properly.
- Everything respects reduced-motion settings.

---

## Stuff that bit me

Keeping this list so I don't rediscover any of it the hard way.

**Carrd embeds cap at 16,384 characters.** That's why the site is 14 files instead of one.
Each JS part has to be a complete, valid script on its own, which is why they share state
through `window.PM` instead of being one big function.

**Carrd's page animation breaks fixed positioning.** It puts a CSS transform on a wrapper,
and a transform makes `position: fixed` resolve against that wrapper instead of the window.
The cursor trail ended up drifting by exactly however far the page was scrolled, which
looked like a bug that got worse every reload (because the browser restores your scroll
position). The trail now measures itself and corrects, but **the page animation needs to
stay off** or the viewer and chat log will drift too.

**Cloudinary's tag list is switched off by default.** Settings → Security → Restricted media
types → uncheck "Resource list". Until you do that, every single category comes back empty
and it looks like the site is broken. First thing to check if the galleries are blank.

**One unquoted URL took the whole settings file down.** I pasted a YouTube link into
`musicUrl` without quotes; that's a syntax error, so the browser skipped all of part 08 and
the categories, links and her profile all vanished at once. If a big chunk of the site
disappears, it's almost always a typo in part 08. Also: YouTube links can't be used as audio
at all, it needs a direct `.mp3`.

**Cloudinary free plan rejects big images.** Max 10 MB and 25 megapixels. 8K GPose shots are
33 MP, so they fail, and most 4K PNGs are over 10 MB. Export JPEG at 90–95, max 5K on the
long edge.

**A CSS reset that was too strong.** My `button { border: 0 }` reset was beating the actual
component styles, so every bordered button lost its outline. Fixed by wrapping the resets in
`:where()` so they have zero specificity. If a border or padding mysteriously doesn't apply,
check the resets in part 01 first.

**The justified photo grid grew rows.** Flexbox stretches the last row's items to fill the
width, so with a big base row height the rows ballooned to 400px tall. Dropping `--rh` to
210 keeps the stretch under control.

**GPose mode tapped through.** Tapping to exit GPose was also clicking whatever was
underneath, so you'd exit and immediately open a photo. It now swallows that one click.

**Splitting the code into parts cost me one bug.** The frame-number code needed a helper
that the zone part didn't import, and because that error happened inside a `.catch()` it
showed up as "these photos couldn't load" instead of a console error. If I move code between
parts again: check the alias block at the top of the file first.

**Noise texture looked great and read terribly.** I had a grain overlay on everything at one
point; grey text on grain is rough to read. The texture is now only the dot grid on the
ground outside the print, where no text sits.

---

## If something looks broken

1. All categories empty → "Resource list" restricted in Cloudinary, or `cloudName` is wrong
   in part 08.
2. One category empty → tag typo. They're case-sensitive: `soft-dreamy`, not `Soft-Dreamy`.
3. Big chunks of the site missing → syntax error in part 08 or 09; open the console.
4. New photo not showing → the tag list is cached for about 60 seconds. Wait, refresh.
5. Overlays drifting as you scroll → Carrd page animation is back on.
6. Site looks empty in the Carrd editor → that's normal, embeds don't run in the editor.
   Check the published page.

Quick sanity check on the Cloudinary side: open
`https://res.cloudinary.com/pa5desou/image/list/featured.json` in a browser. JSON means it
works, an error page means the problem is there and not in the site.

---

## Credits

FINAL FANTASY XIV © SQUARE ENIX CO., LTD. Screenshots used under Square Enix's material
usage licence. All photographs belong to Pink.

Fonts are Cormorant Garamond, Chakra Petch, Sixtyfour and Noto Sans/Serif SC from Google
Fonts.