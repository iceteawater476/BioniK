# Bionic Shelf

A phone-friendly **PDF & EPUB reader** with **bionic reading**, **dark mode**, and a few things to make reading feel like less of a chore — reading streaks, badges, and a "continue reading" shelf.

**Live app:** https://claude.ai/artifact/Fc9z6t9Z97idiWre8w1owk

Share that link with anyone — it opens straight into the app, works on phone or desktop, and needs no sign-up or install to use.

## Hosting it yourself on GitHub

The whole app is one file: **`bionic-shelf.html`**. To make it a real, shareable link:

1. Create a repo (or use an existing one) and add `bionic-shelf.html` to it.
2. Go to **Settings → Pages**.
3. Under **Source**, pick the branch and folder the file is in (e.g. `main` / `/root`), and save.
4. GitHub gives you a URL like `https://yourname.github.io/your-repo/bionic-shelf.html` — that's the link to share.

Just committing the file to a repo **without** turning on Pages will only show people the raw source code, not the running app — GitHub Pages is what actually serves it as a live page. It can take a minute or two after enabling Pages for the link to go live.

The file is fully self-contained (the PDF/EPUB engine is bundled directly inside the HTML — see "Why it kept loading" below), so it also works if you just double-click it and open it locally in a browser, no server required.

## Why it kept loading, and what changed

The stuck spinner was a real bug, not just the in-chat preview: the PDF engine (pdf.js) needs a small background "worker" script, and it was loading that from a public CDN at runtime. Browsers are strict about where a worker script can come from, and that setup could silently fail — especially when the page was opened as a local `file://` page — leaving it stuck on the loading screen forever.

**Fixed by removing the external dependency entirely.** pdf.js, its worker, and JSZip (used for EPUB) are now bundled straight into the HTML file itself, and the worker is spun up from that embedded copy rather than fetched from anywhere. There's nothing left to block, whether you're using the link above, hosting it on GitHub Pages, or opening the downloaded file directly. On top of that, opening a book still has the 45-second timeout and Cancel button from before, as a safety net.

One knock-on effect: the file is now about 2 MB (it carries its own PDF/EPUB engine), instead of ~100 KB. That's still a perfectly normal size for a single HTML file and won't affect how it runs.

---

## What it does

- **Add a PDF or EPUB** — pick a file, and it's parsed right in the browser and saved to your device so it's still on your shelf next time you open the app.
- **Bionic reading** — bolds the leading letters of each word so your eyes can skip ahead. Toggle it on/off and tune the intensity in Settings or right from the reader.
- **Themes** — Light, Sepia, and Dark, switchable any time.
- **Real page-turning** — tap the left/right edge of the screen or swipe to turn pages; tap the middle to hide the UI for distraction-free reading.
- **Made for fun, not just function** — a "continue reading" card, a daily reading streak, a stats page (pages turned, words read, books finished), and unlockable badges with a little confetti when you earn one.

## Installing it as an app

Because it's a single hosted page, it installs as a normal **web app (PWA)** — no app store needed.

**On Android (Chrome):**
1. Open the link above.
2. Tap the **⋮** menu → **Install app** (or **Add to Home screen**).
3. It'll launch full-screen from your home screen, with its own icon and name.

**On iPhone/iPad (Safari):**
1. Open the link above in **Safari** (not Chrome — iOS only allows this from Safari).
2. Tap the **Share** icon → **Add to Home Screen**.
3. It opens full-screen, without Safari's address bar, like a regular app.

**On desktop (Chrome/Edge):**
1. Open the link above.
2. Click the **install icon** in the address bar (or the **⋮** menu → **Install Bionic Shelf**).
3. It opens in its own window, separate from your browser tabs.

You can also just keep it as a bookmark or browser tab — everything works the same either way. Installing just makes it feel more like a native app (its own icon, no browser chrome).

## Good to know before you share it

- **Everything stays on-device.** There's no server or account — books and reading data are stored in that browser, on that device, using the browser's own storage (IndexedDB for books, localStorage for settings/stats). Nothing is uploaded anywhere.
- **Storage is per browser, per device.** If someone reads on their phone and later opens the link on a laptop, their shelf won't follow them — each install has its own local library.
- **Clearing browser data wipes the shelf.** So does uninstalling the app or clearing site data for it.
- **PDFs need selectable text.** Scanned/image-only PDFs won't have text to extract, so they can't be read in bionic or reflowed mode.
- **Layout is reflowed, not pixel-perfect.** Rather than rendering a fixed PDF page image, the app extracts the text and reflows it into clean, paginated columns — this is actually what makes bionic bolding and adjustable font size possible, but it means original PDF formatting (multi-column layouts, tables, images) isn't preserved.
- **No offline caching for the very first load.** The page needs an internet connection the first time (mainly for the Google Fonts stylesheet — the PDF/EPUB engine itself is bundled in, no CDN needed for that). Once loaded, all reading and library features work fully offline for that session.
- **There's a safety net if opening a book ever seems stuck:** a 45-second timeout and a **Cancel** button on the loading screen — it'll never spin forever.

## Tech under the hood

- Single self-contained HTML/CSS/JS page — no backend, and no CDN dependency for the reading engine (see above).
- [pdf.js](https://mozilla.github.io/pdf.js/) (Apache-2.0) for PDF text extraction — bundled inline, including its worker.
- [JSZip](https://stuk.github.io/jszip/) (MIT/GPL) for unzipping EPUB archives — bundled inline.
- [Lora & Manrope](https://fonts.google.com) (SIL Open Font License) for the reading and UI typefaces — loaded from Google Fonts (the one remaining external request, purely cosmetic).
- Browser `IndexedDB` for book storage, `localStorage` for settings and stats.
- Pagination is done with a CSS multi-column trick (no PDF/EPUB viewer library needed for page layout).

## Feedback

If something looks off on your device or a particular book doesn't parse well, note the file type and roughly how it looked — that's the easiest way to track down formatting edge cases in the text-extraction heuristics.
