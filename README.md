# The health ministry’s first 180 days, in its own words. One word missing: measles.

The Daily Star. Design & development: Mir Rownak.

Goes live at **https://campaign.thedailystar.net/health-promises-and-measles/** (October 2026). Preview: Vercel, from the GitHub repo `mir-17101088/health-promises-and-measles` (`main`).

This folder is the finished website: plain static files, no server code, no calls to outside services (fonts and scripts are self-hosted). Every path inside it is relative, so it works at the root of a domain (a Vercel preview) and inside a `/health-promises-and-measles/` folder on the live server.

Don't edit these files by hand. They are written by `build.py` from the project folder (`dist/`).

---

## 1. Put it live

1. Copy **everything in this folder** (except `.git`) into a folder named `health-promises-and-measles` at the web root of `campaign.thedailystar.net`, so that `index.html` is served at https://campaign.thedailystar.net/health-promises-and-measles/.
   - `vercel.json` and this file aren't needed on the live server. They're harmless if copied.
2. **Keep the trailing slash.** `.../health-promises-and-measles` (no slash) must redirect to `.../health-promises-and-measles/`. nginx and Apache do this for real folders by default.
3. If the site sits behind Cloudflare, purge the cache for `/health-promises-and-measles/` after each upload.

Make sure these types are served (older servers may not know them): `.avif` as `image/avif`, `.webp` as `image/webp`, `.woff2` as `font/woff2`. Serve HTML, JS, CSS, SVG and XML with gzip or Brotli: the page is measured with compression on.

### Caching

The build gives the JavaScript bundle and the vendor files a content hash in the file name (`app.3fa9c1d2.js`), so they never change under the same name. The CSS is inlined in `index.html`, so there is no CSS file. Fonts, images and media keep their names and are cached for a week.

| Files | Header |
|---|---|
| hashed files in `js/` and `vendor/` | `public, max-age=31536000, immutable` |
| `fonts/ img/ media/` (names not hashed) | `public, max-age=604800` |
| `index.html` and everything else | `public, max-age=0, must-revalidate` (or `no-cache`) |

**nginx:**

```nginx
location /health-promises-and-measles/ {
    index index.html;
    add_header Cache-Control "public, max-age=0, must-revalidate";
}
location ~ ^/health-promises-and-measles/(fonts|img|media)/ {
    add_header Cache-Control "public, max-age=604800";
}
location ~ ^/health-promises-and-measles/(js|vendor)/.*[.-][0-9a-f]{8,}\.[a-z0-9]+$ {
    add_header Cache-Control "public, max-age=31536000, immutable";
}
```

**Apache** (a `.htaccess` inside `health-promises-and-measles/`, if `AllowOverride` permits it):

```apache
DirectoryIndex index.html
AddType image/avif .avif
AddType image/webp .webp
AddType font/woff2 .woff2
<IfModule mod_headers.c>
  Header set X-Content-Type-Options "nosniff"
  <FilesMatch "\.html$">
    Header set Cache-Control "public, max-age=0, must-revalidate"
  </FilesMatch>
  <FilesMatch "\.(woff2|avif|webp|jpg|png|svg|mp4)$">
    Header set Cache-Control "public, max-age=604800"
  </FilesMatch>
  <FilesMatch "[.-][0-9a-f]{8,}\.js$">
    Header set Cache-Control "public, max-age=31536000, immutable"
  </FilesMatch>
</IfModule>
```

---

## 2. Search engines and share cards

Already in the page:
- Title, description, author, robots directives (large image previews allowed) and the canonical URL.
- Open Graph and X (Twitter) cards with `img/share.jpg`, a 1200 x 630 crop of the hero photo.
- `NewsArticle` structured data: headline, description, author, contributor (Mir Rownak), publisher (The Daily Star), publish date, and the hero photo in 16:9, 4:3 and 1:1 crops (`img/share-16x9.jpg`, `share-4x3.jpg`, `share-1x1.jpg`).
- `sitemap.xml` and `robots.txt`.

**One step on the live server:** crawlers read `robots.txt` only at the root of a domain. If `https://campaign.thedailystar.net/robots.txt` exists, add the line below (don't replace the file); if it doesn't, create it from this folder's `robots.txt`:

```
Sitemap: https://campaign.thedailystar.net/health-promises-and-measles/sitemap.xml
```

**After it's live:**
1. Google Search Console: submit the sitemap and request indexing of the page.
2. Check the article data at https://search.google.com/test/rich-results.
3. Refresh the share card at https://developers.facebook.com/tools/debug/ ("Scrape Again"). Share cards only show the photo once the page is live at its address.

A Vercel preview (`*.vercel.app`) sends `X-Robots-Tag: noindex`, so it never competes with the live page in search. The header is added only for hosts ending in `.vercel.app`; a custom domain on Vercel is indexable.

---

## Folder contents

| Path | What |
|---|---|
| `index.html` | The whole story, every paragraph in the HTML (readable with JavaScript off), with all of its CSS inlined |
| `js/` | Minified, hashed. Scroll steps, charts and the story's own scenes (the CSS is inside `index.html`) |
| `vendor/` | GSAP + ScrollTrigger in one file (loaded after the page has painted; skipped under reduced motion) |
| `fonts/` | Newsreader and IBM Plex (WOFF2, cut to the characters the page uses; OFL licences included) |
| `img/` | Photos (AVIF, WebP, JPEG at several sizes), share images, favicon |
| `robots.txt`, `sitemap.xml` | For search engines (see section 2) |
| `vercel.json` | Vercel preview settings only (caching, noindex on `*.vercel.app`) |
