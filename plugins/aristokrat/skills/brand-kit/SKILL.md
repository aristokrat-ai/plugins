---
name: brand-kit
description: Turn a brand guidelines PDF of any size into an Aristokrat brand kit (colors, fonts, logos, images and written rules) on this machine, show it to the marketer, and push it to the company's brand library with the managed aristokrat CLI.
---

Use this when a marketer gives you brand guidelines (a PDF brand book, often tens of MB of slides) and wants Aristokrat's pages to use the brand. Aristokrat cannot take the PDF itself. You read it here and write a small brand kit folder; `aristokrat brand push` uploads the kit's files and writes the brand library. Pages built afterwards use the kit's colors, fonts, logos and images, and the Brain learns the written rules.

Use the plugin's managed CLI exactly as the `aristokrat` skill says: resolve `../../scripts/aristokrat` from this skill directory to an absolute path and quote it on every invocation (Claude Code can also use `${CLAUDE_PLUGIN_ROOT}/scripts/aristokrat`). Sign in the way that skill describes. Never inspect credentials or tokens.

## Never read the whole PDF at once

A brand book is mostly full-page images. Opening it whole wastes the context and can fail. Work from text and small page renders in a scratch folder outside the kit (for example `brand-work/`), and delete it when done. Never put the PDF or page renders in the kit.

The tools come from poppler. If `pdfinfo` is missing, install it (`brew install poppler` on macOS, `apt-get install poppler-utils` on Debian or Ubuntu) after telling the marketer what you are installing.

1. `pdfinfo guide.pdf`: page count and page size.
2. `pdftotext -layout guide.pdf brand-work/text.txt`: all the text, pages split by form feeds. Read it page by page and note each page's number and heading. This is where hex codes, the type hierarchy (sizes, weights, line height, letter spacing), logo rules and voice live.
3. `pdftoppm -r 30 -png guide.pdf brand-work/p`: one small image per page (about 800 px wide for a 1920 pt slide). Look at them in batches of six to twelve, joined into one sheet if you can (`magick a.png b.png c.png -resize 400x +append row.png`, then `-append` the rows). Find the pages that show the logo, its misuse, the palette, typography and example layouts. Render a single page larger (`-r 72 -f N -l N`) only when you need detail.
4. `pdfinfo -url guide.pdf`: the links, including font download links. Check each link belongs to this brand's fonts before trusting it.

## Read colors from what is painted

Collect every hex code the text gives, grouped as the guide groups them (primary, secondary, text). Then check each label against its swatch: sample the swatch's pixels from a render (`magick page.png -crop WxH+X+Y -format %c histogram:info:` and take the dominant color). A label whose swatch is painted another color, or a code with no swatch at all, is a leftover from a template: keep the painted color, drop the leftover, and tell the marketer which ones you dropped and why.

Give roles only where the guide supports them, at most once each in the whole kit:

- `background`: the main page background (the guide's soft neutral, the color its own pages and example layouts sit on).
- `text`: the main text color.
- `primary`: the call-to-action color, the color of the main button in the guide's example layouts. Aristokrat paints buttons with it, so it is not necessarily the color the guide calls "primary".
- `secondary`: a second button style, only if the guide shows one in a color not already used for `text` or `background`.
- `accent`: a color the guide uses sparingly for emphasis.

List the rest of the palette without a role.

## Fonts

Name each family the guide uses with its role: `font_heading` for headlines, `font_body` for body and UI text. Other families (a script accent, say) go in without a role. Include WOFF2 files under `fonts/` only when the marketer gives them to you and confirms they have the right to use them on the web. Never download a commercial font. Without files, a visitor who does not have the family sees its `fallback`, so give every font one that matches what the guide shows: a list of web-safe families ending in the generic family of the same kind, `["Georgia", "Times New Roman", "serif"]` for a serif like Teodor, `["Helvetica Neue", "Arial", "sans-serif"]` for a sans like Inter, `["cursive"]` for a script, `["Courier New", "monospace"]` for a monospace. Judge the kind from the specimen pages, not the name. Without a fallback the pages fall back to the website's stack for that family, or to sans-serif.

## Logos

Find the page that shows the logo on a light and a dark background. Extract it as vectors: `pdftocairo -svg -f N -l N guide.pdf brand-work/pN.svg`, then keep only the `<path>` elements inside the logo's box (drop `<image>` elements and clip paths), set the SVG's `viewBox` to that box, and render it (`magick -density 200 logo.svg logo.png`) to check it is the whole logo and nothing else. Keep each SVG small (tens of KB at most): round coordinates and drop default attributes.

- `logo_light`: the logo drawn for light backgrounds (dark ink).
- `logo_dark`: the logo drawn for dark backgrounds (light ink).
- `favicon`: the icon or symbol, if the guide has one.

If the PDF holds the logo only as a picture, ask the marketer for the logo files (SVG preferred, else a large PNG) instead of cutting a blurry copy out of a render.

## Images

Many brands live in their imagery as much as in their colors: illustrations, photography, patterns, textures, icons. Pages can use up to 20 kit images, so pick the ones that carry the brand.

1. Find them in the small page renders: the cover, moodboard or imagery pages, example layouts, and any section showing how images are made. Note each page number.
2. `pdfimages -list guide.pdf` lists every raster with its page, size and object ID. An `smask` row right after an `image` row is that image's transparency. The same object ID on several pages is the same picture; take it once.
3. Extract the originals of the pages you chose, one page at a time (`pdfimages -png -f N -l N guide.pdf brand-work/raw/pN`), never page screenshots. Numbering restarts on each page: `pN-003.png` is the page's fourth raster in the list.
4. Where the guide shows an image as a cut-out (a figure with no background), join it with its soft mask into a transparent image: `magick image.png mask.png -alpha off -compose copy-opacity -composite out.png`. Look at the result on a light background to check the edges.
5. Make each one at most about 2000 px on its long side and encode WebP: `magick out.png -resize '2000x2000>' -quality 82 images/name.webp` (AVIF is fine too). Keep each under 5 MB; most end up a few hundred KB.

Skip:

- mockups and screenshots of websites, apps, decks or chats, which show layouts, not reusable images;
- the steps of a process when the guide also shows the finished image, and near-duplicates (the same picture cropped or recolored);
- reference pictures the guide borrowed (old paintings, stock photos, photos of real people used as a model) and anything that is another company's: their logos, products, packaging or ads. Leave out an image with another company's mark in it even if the brand made it, and tell the marketer which ones you left out and why;
- blurred backgrounds, gradients and thin strips that only decorate a slide.

Pick at most 20, the most representative first: the order is the order builds see them. Prefer variety (people, scenes, objects, icons; cut-outs and full scenes) over many of one kind. Give each:

- `alt`: what the image shows, plainly and factually, in a sentence ("Etching-style illustration of a woman in a ruff working on a laptop"). Pages use it as the image's alt text, and page builders find images by it. Write it: you read the guide, so yours names what the brand calls things. An image you leave without one gets an alt from a vision model after the push.
- `tags`: a few short lowercase words for what kind of image it is and what is in it. Start with its medium (`photo`, `illustration`, `screenshot`, `icon`, `chart`), then what it is for when one fits (`product`, `people`, `customer-logo`, `decoration`, `ad-creative`), then what is in it (`cut-out`, `scene`, `laptop`). Builders search these words, and generated tags use the same ones. At most 8. An image you leave without tags gets them from the vision model.
- `"role": "og_image"` on one image only if the guide names a share or social image; that becomes the picture shown when a page is shared.

## Rules

Write `rules` as concise markdown in the brand's own words, with a page citation on each point ("(p.12)"): tone of voice and example headlines, logo clear space, minimum size and misuse, the typography hierarchy, color distribution and accessible pairs, imagery style, and the guide's do's and don'ts.

Give each part a `##` heading (`## Voice`, `## Logo`, `## Typography`, `## Color`, `## Imagery`). The app's Guidelines screen makes a section of each `##` heading and marks the Voice one as the section chat pages read; a `#` line is only the title.

If the guide says how its images are made (AI prompts, photography direction, illustration style), copy that into an imagery part of the rules with page citations. Copy prompts word for word, typos included, in quotes, with the tool and step each belongs to ("Midjourney, step 1 (p.34): \"…\""), so the next image can be made the same way.

Keep the whole of `rules` under 33,000 characters; the Brain reads no further. If the prompts would push it over, keep the general rules and the prompts for the most representative images.

## Keep only this brand

Brand books are often built from an agency's template and carry other companies' leftovers: another company's name in a sentence, links to another company's site, a font download link for a different family, a palette label that matches no swatch, placeholder copy. Keep only what the guide states for this brand. When the text and the page disagree, trust what the page shows and mention the disagreement to the marketer. Do not add rules the guide does not state.

## Write the kit

A folder (for example `brand-kit/<company>/`) with `brand.json` and the files it names, by paths inside the folder:

```json
{
  "title": "Acme Brand Guidelines 2026",
  "colors": [ { "value": "#ee3986", "role": "primary", "name": "Signal Pink" }, { "value": "#f9f7ee", "role": "background" } ],
  "fonts":  [ { "value": "Teodor", "role": "font_heading", "files": ["fonts/teodor-light.woff2"], "fallback": ["Georgia", "Times New Roman", "serif"] },
              { "value": "Inter", "role": "font_body", "fallback": ["Helvetica Neue", "Arial", "sans-serif"] } ],
  "logos":  [ { "file": "logos/logo-light.svg", "role": "logo_light", "alt": "Acme" } ],
  "images": [ { "file": "images/founder-at-desk.webp", "alt": "Illustration of a woman at a desk with a laptop", "tags": ["illustration", "cut-out", "person"] } ],
  "rules": "# Acme brand rules\n\n## Voice\n..."
}
```

`title` names the guide (it becomes the Brain's document source). Colors are `#rrggbb`. A font's `fallback` is at most 8 plain family names (letters, digits, spaces, dashes) and must end in a generic family (`serif`, `sans-serif`, `monospace`, `cursive`). `name` is optional and only when the guide names the color. Logo files are SVG, PNG, JPEG, WebP, AVIF or GIF, 5 MB at most; image files PNG, JPEG, WebP, AVIF or GIF, 5 MB at most; font files WOFF2.

## Show the marketer before pushing

Show the kit for a yes before anything is sent: each color with its role (and what you dropped as leftovers), the fonts with roles, fallbacks and whether files are included, the logo images you rendered with their roles, the images as one contact sheet with each file's name under it (`magick montage -label '%t' images/*.webp -background '#f5f5f5' -tile 5x -geometry 360x300+8+8 brand-work/images.png`, then show `brand-work/images.png`; if magick cannot read a font for the labels, add `-font` with the path of any `.ttf` file) with their alt texts and what you left out, and the rules. Say that pushing makes the kit's roles win over what Aristokrat read from the website, and that a later push of a new kit replaces this one. Push only after the marketer agrees.

## Push

```sh
"<launcher>" brand push brand-kit/<company> --company <company> --json
```

Push refuses a kit naming a missing file or a file outside the folder before uploading anything. The server answers 422 with each problem (`colors[0].value`, `role` given twice, a file it does not hold) and writes nothing; fix the kit and push again. Pushing the same kit again changes nothing. Report the color, font and logo roles now held (the answer leaves out image roles, so name the share image yourself if the kit gave one), the counts written and archived (colors, fonts, logos, images), and whether the rules became a Brain source. Then delete the scratch folder.

To add or remove a few images later without pushing the whole kit again, use the `brand-images` skill.
