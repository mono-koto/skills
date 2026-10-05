---
name: crosspoint-reader
description: Use when connecting to or managing content on a CrossPoint Reader e-ink device, including Xteink X3 or X4 file transfer, uploads, downloads, folders, settings, fonts, OPDS, Wi-Fi, WebDAV, WebSocket, or connection troubleshooting.
---

# CrossPoint Reader

Manage a CrossPoint Reader through its unauthenticated local web server. Use it only on a trusted private network or the device's temporary hotspot.

This file covers routine work after the server is running. Read [setup-and-troubleshooting.md](setup-and-troubleshooting.md) when the device is not reachable. Read [storage-layout.md](storage-layout.md) before choosing a destination. Read [api-reference.md](api-reference.md) for settings, fonts, OPDS, Wi-Fi, WebSocket, WebDAV, UDP discovery, or uncommon endpoints.

## Connect

The server runs only while the device remains in **File Transfer** or **Calibre Wireless** mode.

```sh
BASE_URL=http://crosspoint.local
curl --fail-with-body -sS --max-time 5 "$BASE_URL/api/status"
```

Use the IP shown on the device if mDNS fails. Keep `http://`. A successful status response identifies the firmware version, IP, mode, signal, and X3 or X4 hardware.

## Safety

Read current state before changing it. Preview the exact target and ask the user before:

- replacing an existing file
- deleting files, folders, font families, Wi-Fi entries, or OPDS entries
- changing Wi-Fi, OPDS, or KOReader Sync credentials, or broad groups of settings

Never print a password or place it directly in shell history. Collect secrets through an appropriate secure input method. Update Wi-Fi last because it may disconnect the device.

The API has no atomic replacement, checksum, backup, or rollback operation. Do not claim that it does. Download a backup first when the user needs recovery.

## Routine file operations

These examples require curl 7.87 or newer. Use `--data-urlencode` for query or form values. For multipart uploads, `--url-query` safely adds the destination without changing the POST body.

```sh
# List a directory
curl --fail-with-body -sS --get --data-urlencode 'path=/Books' "$BASE_URL/api/files"

# Download to the server-provided filename
curl --fail-with-body -sS --get --data-urlencode 'path=/Books/My Book.epub' -OJ "$BASE_URL/download"

# Create a directory
curl --fail-with-body -sS -X POST --data-urlencode 'name=Science Fiction' --data-urlencode 'path=/Books' "$BASE_URL/mkdir"

# Rename one file. name is a filename, not a path.
curl --fail-with-body -sS -X POST --data-urlencode 'path=/Books/old.epub' --data-urlencode 'name=new.epub' "$BASE_URL/rename"

# Move one file into an existing directory
curl --fail-with-body -sS -X POST --data-urlencode 'path=/Books/new.epub' --data-urlencode 'dest=/Read' "$BASE_URL/move"
```

Before uploading, list the destination and compare the exact basename. Upload collision behavior differs by firmware. Documented builds overwrite, while current upstream firmware rejects the upload. Never rely on either behavior.

```sh
LOCAL='/Users/me/Books/My Book.epub'
DEST='/Books/Science Fiction'
NAME=$(basename "$LOCAL")
LISTING=$(curl --fail-with-body -sS --get --data-urlencode "path=$DEST" "$BASE_URL/api/files")
printf '%s\n' "$LISTING" | jq --arg name "$NAME" '.[] | select(.name == $name)'
# If this prints an item, stop and ask before replacing it.
# --url-query requires curl 7.87 or newer.
curl --fail-with-body -sS --url-query "path=$DEST" -F "file=@$LOCAL" "$BASE_URL/upload"
```

For an approved replacement, download a local backup first. Choose an unused remote backup name and rename the old file before uploading on every firmware. Upload and verify the new file, then ask before deleting the backup. If upload fails, restore the remote backup while the original name is unused.

Confirm deletion before sending either form:

```sh
# One item
curl --fail-with-body -sS -X POST --data-urlencode 'path=/Books/old.epub' "$BASE_URL/delete"

# Several items. paths is a JSON array encoded as one form value.
curl --fail-with-body -sS -X POST --data-urlencode 'paths=["/Books/old.epub","/OldFolder"]' "$BASE_URL/delete"
```

Only empty folders can be deleted. Rename and move accept files only.

For older curl, use the browser file manager or URL-encode the destination with a standard URL library before appending `?path=`. Replace `--fail-with-body` only if that curl lacks it, and inspect both the HTTP status and response body. Do not hand-replace spaces alone because paths may contain other reserved or non-ASCII characters.

## Common mistakes

- Using HTTPS. The device serves HTTP.
- Leaving File Transfer mode during a request. This stops the server.
- Treating `freeHeap` as SD-card capacity. It reports RAM, and the API has no free-storage endpoint.
- Uploading `.ttf` or `.otf` as a reader font. CrossPoint requires compiled `.cpfont` files.
- Editing `/.crosspoint` as ordinary content. It contains caches, state, and credentials.
- Printing the full settings response. Current firmware exposes the KOReader Sync password as `koPassword`; redact it before display.

## Verify every change

Do not treat a successful HTTP response as the only check. List the affected directory again and confirm the expected name, size, or absence. Refetch settings, fonts, OPDS, or Wi-Fi lists after changing them. Redact `koPassword` from settings output. Report the device response and verification result separately.

## Current documentation

CrossPoint firmware changes quickly. These references match upstream documentation retrieved 2026-07-22. If the device behaves differently, its firmware is newer, an operation is undocumented, or the user asks about an unknown capability, check the current [web server endpoint documentation](https://crosspointreader.com/docs#webserver-endpoints) and [upstream repository documentation](https://github.com/crosspoint-reader/crosspoint-reader) before guessing.
