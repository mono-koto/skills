# SD card layout and device UI

CrossPoint has no required `/Books` library root. The **Browse Files** screen follows the SD card's folder tree. Supported reading files can live at the root or in user-created folders. Use `/Books` only as a clear convention when the user has not chosen another layout.

## User content

| Content | Recommended path | Device behavior |
| --- | --- | --- |
| Books | `/Books/<category-or-author>/` or another user folder | `.epub`, `.xtc`, `.xtch`, and `.txt` files open from **Browse Files**. An opened book can appear in **Recent Books** with title and author metadata. |
| Viewable images | Any user folder | `.bmp` files open in the image viewer. |
| Screenshots | `/screenshots/` | Device screenshots are written here, sometimes in book-specific subfolders. They remain ordinary browsable and downloadable BMP files. |
| Single sleep image | `/sleep.bmp` | **Custom** sleep mode uses this first when it is a valid BMP. |
| Sleep image collection | `/.sleep/*.bmp` preferred, or `/sleep/*.bmp` | Used when `/sleep.bmp` is absent or invalid. CrossPoint chooses a random BMP. `/.sleep` takes priority over `/sleep`. |
| Font families | `/.fonts/<Family>/*.cpfont` preferred, or `/fonts/<Family>/*.cpfont` | Families appear in **Settings > Reader > Font Family**. Both roots are scanned. If a family exists in both, the hidden copy wins. |
| Dictionaries | `/dictionaries/<Name>/` or `/.dictionaries/<Name>/` | A valid dictionary appears in **Settings > Reader > Dictionary**. The hidden and visible roots work the same way. |
| Firmware update | `/firmware.bin` | Used only for a deliberate firmware update or recovery workflow. Do not upload it as ordinary content. |

For Calibre Wireless, the CrossPoint plugin creates one author folder under its configured upload path, which defaults to `/`. For example, an upload path of `/mybooks` produces `/mybooks/<Author>/<Book>.epub`.

## Format details

### Fonts

The font API accepts compiled `.cpfont` files, not `.ttf` or `.otf` files. Group one family by name and size:

```text
/.fonts/Literata/
├── Literata_12.cpfont
├── Literata_14.cpfont
├── Literata_16.cpfont
└── Literata_18.cpfont
```

Use `/api/fonts/upload` for font installation. It validates the family, filename, and `CPFONT` magic bytes and places the file in the managed font root. List exact filenames first. Current firmware can overwrite a matching font directly, so ask before a collision and retain a known-good local copy.

### Dictionaries

CrossPoint supports one StarDict dictionary per folder:

```text
/dictionaries/webster/
├── webster.idx
├── webster.dict.dz
└── webster.ifo
```

The uncompressed `.idx` and either `.dict` or `.dict.dz` are required. `.ifo` is optional. `.idx.gz`, `.syn`, and 64-bit index offsets are not supported. The reader may create a replaceable `.qidx` index beside the `.idx` after first use.

### Sleep images

Use uncompressed 24-bit BMP when possible. Match the screen size for best results:

- X4: 480 × 800
- X3: 528 × 792

Set **Settings > Sleep Screen** to **Custom** or **Cover + Custom**. Uploading the image alone does not select that mode.

## Internal and generated paths

Do not treat these as user content:

```text
/.crosspoint/
├── epub_<hash>/       # generated metadata, covers, images, and section caches
├── bookmarks/         # bookmark JSON
├── settings.json
├── state.json
├── recent.json
├── wifi.json          # saved Wi-Fi credentials
├── opds.json          # saved OPDS servers and credentials
├── koreader.json      # KOReader sync configuration, when used
├── sleep_frame.bin
└── dict.tmp
```

This list may grow as firmware adds features. Wi-Fi, OPDS, and KOReader Sync passwords stored here are obfuscated, not securely encrypted. Treat a backup of `/.crosspoint` as sensitive.

Deleting `/.crosspoint` resets generated metadata and forces regeneration. It can also remove settings, state, recent history, bookmarks, saved Wi-Fi and OPDS entries, and sync configuration. Never remove or edit it without explicit, informed confirmation and a sensitive backup.

Book changes made through the firmware or web API clear or re-key related EPUB caches. Manual SD card changes may leave stale caches.

`XTCache` and `System Volume Information` are intended to stay hidden and protected. Leave them alone. Other dotfiles are omitted from `/api/files` unless `showHiddenFiles` is enabled. HTTP validation differs by firmware and operation, so a request succeeding against an internal path does not make it safe or supported.

## Organizing a library

Prefer folders that match how the user browses, such as author, series, genre, or read status. Confirm an existing layout before creating a new one. The device browser shows folders directly, so unnecessary nesting adds navigation steps.
