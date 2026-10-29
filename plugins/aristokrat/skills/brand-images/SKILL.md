---
name: brand-images
description: Add a few images to a company's Aristokrat brand gallery, or remove some, with the managed aristokrat CLI's brand add and brand remove, without pushing a whole brand kit.
---

Use this when a marketer wants one or a few images added to or dropped from the brand gallery: "add our new office photo", "take the old team picture out". Pages built afterwards can use the added images, and stop using the removed ones. For a whole brand guide, or to change which logo, color or font holds a role, use the `brand-kit` skill and push a kit instead.

Use the plugin's managed CLI exactly as the `aristokrat` skill says: resolve `../../scripts/aristokrat` from this skill directory to an absolute path and quote it on every invocation (Claude Code can also use `${CLAUDE_PLUGIN_ROOT}/scripts/aristokrat`). Sign in the way that skill describes. Never inspect credentials or tokens. Add `--company <company>` when the account acts in more than one company, and `--json` to every command.

## Add

```sh
"<launcher>" brand add office.jpg --alt "The team at a long table in the Haifa office" --tags photo,people,office --company <company> --json
"<launcher>" brand add a.webp b.webp c.png --company <company> --json
```

The library takes PNG, JPEG, GIF, WebP or AVIF images. Keep each under 5 MB and about 2000 px on its long side. Write `--alt` and `--tags` the way the `brand-kit` skill's Images section describes: the alt says plainly what the image shows, and the tags start with the medium. `--alt` takes one image, so add images with different alt texts one run at a time. `--tags` applies to every image in the run. Leave both out and a vision model writes them after the add.

Each image in the answer is `added`, `already_there` or `refused`. `already_there` means the gallery already holds that exact image: it stays as it was, with its own alt and tags, and the ones you sent are not used. A `refused` image carries the reason a person reads (an SVG cannot be added, a file is not an image); tell the marketer that reason. The other images in the run are still added, and the command exits 3. Running the same add again changes nothing.

## Remove

```sh
"<launcher>" brand remove old-team.jpg --company <company> --json
"<launcher>" brand remove <sha256>.png --company <company> --json
```

Confirm with the marketer which images go before removing any. Name each image by the local file it was added from, or by its content name (`<sha256>.<ext>`, the `file` that `brand add` and `brand push` answered). An SVG goes by its content name only, since the server cleans it. The command refuses an argument that is neither a raster image file nor a content name before removing anything.

Each image in the answer is `removed`, `not_found` or `refused`. A `refused` image holds a role, and its reason says what replaces it (a logo pages use is replaced by pushing a brand kit with a new logo through the `brand-kit` skill). Tell the marketer that reason instead of retrying. Removing an image again is `removed` again and changes nothing.
