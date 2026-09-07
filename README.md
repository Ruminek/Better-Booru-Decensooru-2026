[README.md](https://github.com/user-attachments/files/31900960/README.md)

# better_better_booru — 2025 lineage revival (BBB + decensooru)

A single-file userscript for **Danbooru** (`*://*.donmai.us/*`), continuing the
**better_better_booru** lineage (otani → Jawertae → Hfaify's 2025 fork) and
merging in a **revived decensooru** (friendlyanon's hidden-content hash dump).

One install, one file: the complete BBB feature set plus hidden/gold-gated
media restoration.

> **Disclaimer:** I wrote none of the code and none of this, I dont even know how to code. So this is a complete AI desc, I dont even know what its saying lmfao. I take no responsibility if this fucks something up in your PC :3

## What this is

- **BBB core (v8.3.7)** — the full fork lineage: settings menu, custom tag/status
  borders, blacklist manager (per-entry toggles, smart view), endless pages,
  quick search, hotkeys, post resizing, tag sidebar tools, thumbnail info,
  track-new, page counter, and everything else BBB provides.
- **Decensooru revival** — restores thumbnails and post-page media for hidden
  / gold-gated posts using friendlyanon's **hash dump** (`post id → md5.ext`),
  fetched from the decensooru repo (rawgit is dead; served via
  `raw.githubusercontent.com`), stored in GM storage and refreshed every 8 h.

## Install

1. Install [Tampermonkey](https://www.tampermonkey.net/) (or Violentmonkey).
2. Open `bbb-2025-decensor-fixed.user.js` in this repo → **Raw** → Tampermonkey
   should offer to install it. (Or copy the file into a new userscript.)
3. Remove any older copy of BBB or the 2025 fork first — two copies fight over
   the same settings storage.
4. Open any `https://danbooru.donmai.us/posts?tags=...` page. The **BBB
   Settings** menu appears in the top nav. First run downloads the ~3.5 MB
   hash dump once (`[bbbDecensor] DB ready` in the console).

## What was fixed vs. BBB 8.3.6 (Hfaify 2025)

The 2025 fork only worked intermittently. Root causes found and fixed:

| Issue | Fix |
|---|---|
| BBB core never booted on modern Danbooru — waited on `/assets/application-*`, which no longer exists (now `/packs/js/application-*.js` modules) | `runBBBScript` listens on both selectors, polls for `Danbooru` every 250 ms up to 30 s |
| `isLoggedIn()` always returned `true` (queried a meta tag that no longer exists) | reads the authoritative `<body data-current-user-id>` attribute (`"null"` = logged out) |
| Account settings read dead meta tags (`blacklisted-tags`, `always-resize-images`, `default-image-size`, tooltips flag) | meta read with fallback to body `data-current-user-*` attributes |
| `getPaginator()` ignored its target argument → endless/pagination broke | restored target-scoped lookup |
| Grid-fill fallback raced BBB and Danbooru's own renderer, double-fetched `/posts.json`, and stamped every card with a giant base64 placeholder | single-flight, re-checks the container before writing, runs *after* BBB, uses real preview URLs from the JSON API |
| Dead `hide_ai_content` option | (left as-is in the option list; no code consumed it) |

## Decensooru revival details

- **DB**: batches `0..36` from
  `https://raw.githubusercontent.com/friendlyanon/decensooru/master/batches/N`
  (line format `postid:md5.ext`; ~3.5 MB total), merged into GM storage
  (`bbb_decensor_db`), refreshed every 8 h. `bbbDecensor.updateDB()` forces a
  refresh; `bbbDecensor.status()` reports entry count.
- **Listing pages**: any `data:image` placeholder thumbnail whose post id is in
  the DB is swapped for the real 180×180 CDN thumb
  (`https://cdn.donmai.us/180x180/<md5[0:2]>/<md5[2:4]>/<md5>.<ext>`).
- **Post pages**: gold-gated posts (modern Danbooru renders them as a bare
  `<p>` inside `section.image-container`) are replaced with full-resolution
  media — `https://cdn.donmai.us/original/...` for images, `<video>` for
  mp4/webm, the ugoira webm for zip posts — plus a "Save this image/video"
  link.
- **pixiv fallback**: posts carrying a pixiv id but missing from the DB use
  `https://pixiv.cat/<pixivId>[-pageIndex].<ext>`.
- **Not in database**: posts absent from the DB (and not pixiv) show the
  original decensooru red "Not in database" tile, on both listing and post pages.

## Known limits (from friendlyanon/decensooru#7)

- Media tagged `banned_artist` / `status:banned` is **hard-deleted** from
  Danbooru — the CDN URLs 404 and those posts stay hidden. Unrecoverable from
  Danbooru itself.
- Danbooru's API is Cloudflare-gated; this script deliberately avoids the API
  (uses the offline hash dump + CDN URLs) and works around CORS via
  `GM_xmlhttpRequest`.
- A post can only be restored if its id is in the hash dump **or** it is
  pixiv-sourced.

## AI usage disclosure

This project was developed with heavy AI assistance:

- **Models**: DeepSeek (opencode-go/deepseek-v4-flash and -pro) and others via
  the [OpenCode](https://opencode.ai) CLI.
- **Process**: the AI agent (a) audited the 2025 fork against the live Danbooru
  DOM, (b) diagnosed the intermittent-boot/race regressions, (c) implemented
  the fixes and the decensooru revival module, and (d) validated the file with
  a syntax build (`bun build`, exit 0).
- All changes are localized and documented in this README; no credentials or
  private data are included. The AI was used as a coding assistant — the
  maintainer reviewed and tested the result.

## Credits & attribution

- **better_better_booru** — otani, modified by Jawertae, fixed by Hfaify.
  Lineage: https://greasyfork.org/scripts/3575-better-better-booru and
  https://github.com/hfaify/better-better-booru-2025
- **decensooru** — friendlyanon: https://github.com/friendlyanon/decensooru
  (the `hash` branch hash dump and `batches/` database; WTFPL license).

## License

The original BBB lineage does not ship an explicit license header. This repo is
provided as a community continuation; respect the original authors' work. The
decensooru portions are WTFPL (per the upstream README/license).

## Testing checklist

- [ ] BBB Settings menu appears in the top nav (boot fix)
- [ ] Grid shows real previews on `/posts?tags=...`
- [ ] Endless scroll and pagination advance correctly
- [ ] Hidden-post thumbs restore when present in the DB
- [ ] Gold-gated post page shows full-res image / video
- [ ] Not-in-DB posts show the red "Not in database" tiledecensooru hash dump + cdn.donmai.us.
