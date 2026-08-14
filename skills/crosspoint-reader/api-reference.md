# CrossPoint web server API

This reference describes the upstream API retrieved 2026-07-22. The server is available only in **File Transfer** or **Calibre Wireless** mode and has no authentication.

| Service | Address |
| --- | --- |
| HTTP and WebDAV | `http://<host>:80` |
| WebSocket upload | `ws://<host>:81/` |
| UDP discovery | `<host>:8134` |

Set `BASE_URL=http://crosspoint.local`. Use the IP displayed on the reader when mDNS fails.

## HTTP pages

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/` | Home and status page |
| `GET` | `/files` | File manager |
| `GET` | `/settings` | Settings, saved Wi-Fi, and OPDS management |
| `GET` | `/fonts` | SD-card font manager |
| `GET` | `/js/jszip.min.js` | File manager asset |

## Status

### `GET /api/status`

```sh
curl --fail-with-body -sS "$BASE_URL/api/status"
```

Response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| `version` | string | Firmware version |
| `ip` | string | Device IP |
| `mode` | string | `STA` for joined Wi-Fi or `AP` for hotspot |
| `rssi` | number | Signal in dBm, or `0` in AP mode |
| `freeHeap` | number | Free heap bytes |
| `uptime` | number | Seconds since boot |
| `device` | string | `X3` or `X4` |
| `serial` | string | Hardware serial, or `Not found` |

## Files

| Method and path | Input | Behavior |
| --- | --- | --- |
| `GET /api/files` | Query `path`, default `/` | Returns `{name,size,isDirectory,isEpub}[]`. Dotfiles follow `showHiddenFiles`; `System Volume Information` and `XTCache` stay hidden. |
| `GET /download` | Required query `path` | Returns EPUB as `application/epub+zip` and other files as `application/octet-stream`. Intended protected paths are rejected, but validation differs by firmware and operation. Never use it for internal paths. |
| `POST /upload` | Query `path`, default `/`; multipart field `file` | Writes a 4 KB buffered upload and clears EPUB cache for the path. Collision behavior varies by firmware. Documented builds overwrite; current upstream rejects a same-name file. |
| `POST /mkdir` | Form `name`; form `path`, default `/` | Creates one directory. |
| `POST /rename` | Form `path`, `name` | Renames one file. `name` cannot be a path. Clears old EPUB cache. |
| `POST /move` | Form `path`, `dest` | Moves one file into an existing directory. Clears old EPUB cache. |
| `POST /delete` | Form `path` or JSON-array string `paths` | Deletes files or empty directories. Protected paths and non-empty directories are rejected. Clears deleted EPUB caches. |

Use the safe examples in [SKILL.md](SKILL.md). Success text for upload is `File uploaded successfully: <filename>`. Current collision rejection is `File already exists: <filename>`.

The firmware intends to protect dotfiles, `System Volume Information`, and `XTCache`, but HTTP path validation differs by version and operation. A server accepting an internal path does not make the operation safe. Never use generic file endpoints on `/.crosspoint`, `/.fonts`, `/.dictionaries`, or other internal and hidden paths unless a documented workflow requires that exact path.

## Settings

### `GET /api/settings`

Returns a streamed JSON array. Every item includes `key`, `name`, `category`, `type`, and `value`.

**Security warning:** Current firmware returns the KOReader Sync password in the `koPassword` setting. Do not print, log, or retain the full response. Redact that item before display:

```sh
curl --fail-with-body -sS "$BASE_URL/api/settings" | jq 'map(if .key == "koPassword" then .value = "<redacted>" else . end)'
```

Treat `koUsername`, `koPassword`, and `koServerUrl` as credential settings. Ask before changing them and send secrets through secure input rather than shell history.

| Type | Extra fields |
| --- | --- |
| `toggle` | `value` is `0` or `1` |
| `enum` | `options` |
| `value` | `min`, `max`, `step` |
| `string` | No extra fields |

The font-family setting includes installed SD-card fonts. Read this endpoint before changing a key so the current value and constraints are known.

### `POST /api/settings`

Accepts a partial JSON object and returns `Applied N setting(s)`.

```sh
curl --fail-with-body -sS -X POST -H 'Content-Type: application/json' --data-binary '{"fontSize":2,"showHiddenFiles":1}' "$BASE_URL/api/settings"
```

Ask before broad updates or any KOReader credential change. Fetch settings again through the redacting command to verify the exact keys.

## Fonts

### `GET /api/fonts`

Returns `{"maxFamilies":128,"families":[...]}`. Each family has `name`, numeric `sizes`, and `files` with `name` and `size`.

### `POST /api/fonts/upload`

Accepts multipart `family` and one `.cpfont` `file`. The server validates the family name, filename, and `CPFONT` magic bytes.

List `/api/fonts` and compare the exact family and filename before uploading. Current firmware can overwrite that font file directly, and a failed invalid replacement can remove the previous copy. Ask before a collision and retain the known-good local font or an SD-card backup because there is no font-download endpoint.

```sh
curl --fail-with-body -sS -X POST -F 'family=Literata' -F 'file=@Literata_12.cpfont' "$BASE_URL/api/fonts/upload"
```

Success is `{"ok":true}`. Fetch `/api/fonts` afterward.

### `POST /api/fonts/delete`

Deletes a whole family. Ask first.

```sh
curl --fail-with-body -sS -X POST -H 'Content-Type: application/json' --data-binary '{"family":"Literata"}' "$BASE_URL/api/fonts/delete"
```

Success is `{"ok":true}`.

## OPDS servers

### `GET /api/opds`

Returns entries with `index`, `name`, `url`, `username`, and `hasPassword`. Passwords are never returned.

### `POST /api/opds`

Adds an entry or updates one when `index` is present. Fields are `name`, `url`, optional `username`, optional `password`, and optional `index`. Omitting `password` during an update preserves the saved password.

```sh
curl --fail-with-body -sS -X POST -H 'Content-Type: application/json' --data-binary '{"name":"My Catalog","url":"http://calibre.local:8080/opds","username":"reader"}' "$BASE_URL/api/opds"
```

Do not put a real password in shell history. Build JSON through a secure secret-input mechanism and pipe it to `curl --data-binary @-`. Ask before adding or changing OPDS credentials. In an indexed update, omitting `password` preserves it, while `"password":""` clears it. Current firmware stores up to eight OPDS servers.

### `POST /api/opds/delete`

Accepts `{"index":0}`. Ask first, then fetch `/api/opds` to verify.

## Saved Wi-Fi

### `GET /api/wifi`

Returns entries with `index`, `ssid`, `hasPassword`, and `isLastConnected`. Passwords are never returned.

### `POST /api/wifi`

Adds an entry or updates one when `index` is present. Fields are `ssid`, optional `password`, and optional `index`. During an indexed update, omitting `password` preserves it and `"password":""` clears it. Without `index`, a matching SSID updates the existing entry, but an omitted password becomes empty and clears the saved password. Current firmware stores up to eight networks.

Do not put a real password in shell history or output. Build JSON through a secure secret-input mechanism and pipe it to `curl --fail-with-body -sS -X POST -H 'Content-Type: application/json' --data-binary @- "$BASE_URL/api/wifi"`.

A Wi-Fi update may end the current connection. Verify physical access or a fallback path and perform it after other work.

### `POST /api/wifi/delete`

Accepts `{"index":0}`. Ask first. Deleting the active or last usable network can prevent later station-mode access.

## WebSocket upload

Connect to `ws://<host>:81/`.

1. Send text `START:<filename>:<size>:<path>`.
2. Wait for `READY`.
3. Send binary chunks.
4. Receive `PROGRESS:<received>:<total>` every 64 KB or at completion.
5. Receive `DONE`, or `ERROR:<message>`.

Avoid colons in filename and path because the control message uses colons as separators. Only one upload can run at a time. An incomplete upload is deleted on disconnect or error.

For normal transfers, prefer HTTP multipart. For a fast upload, install the Python `websockets` package and adapt this client:

```python
import asyncio
import pathlib
import sys

import websockets


async def upload(host: str, source: str, destination: str) -> None:
    path = pathlib.Path(source)
    size = path.stat().st_size
    async with websockets.connect(f"ws://{host}:81/") as socket:
        await socket.send(f"START:{path.name}:{size}:{destination}")
        reply = await socket.recv()
        if reply != "READY":
            raise RuntimeError(reply)
        with path.open("rb") as file:
            while chunk := file.read(16 * 1024):
                await socket.send(chunk)
        while True:
            reply = await socket.recv()
            print(reply)
            if reply == "DONE":
                return
            if reply.startswith("ERROR:"):
                raise RuntimeError(reply)


asyncio.run(upload(sys.argv[1], sys.argv[2], sys.argv[3]))
```

Run it as `python upload.py crosspoint.local local.epub /Books` after checking for a same-name destination.

| Error | Meaning |
| --- | --- |
| `ERROR:Upload already in progress` | Another upload has not finished. |
| `ERROR:Invalid START format` | Control message or size is invalid. |
| `ERROR:Failed to create file` | Destination file could not be opened. |
| `ERROR:No upload in progress` | Binary data arrived before a valid START. |
| `ERROR:Upload overflow` | More bytes arrived than declared. |
| `ERROR:Write failed - disk full?` | SD write failed. |
| `ERROR:File already exists: <filename>` | Current firmware rejected a collision. |

## WebDAV

The HTTP server handles `OPTIONS`, `GET`, `HEAD`, `PUT`, `DELETE`, `PROPFIND`, `MKCOL`, `MOVE`, `COPY`, `LOCK`, and `UNLOCK`.

```sh
# Inspect one level
curl --fail-with-body -sS -X PROPFIND -H 'Depth: 1' "$BASE_URL/Books/"

# Create a collection
curl --fail-with-body -sS -X MKCOL "$BASE_URL/Books/NewFolder/"
```

WebDAV paths must be URL-encoded. Use a WebDAV client when paths contain spaces or non-ASCII characters. `PUT`, `DELETE`, `MOVE`, and `COPY` can overwrite or remove data, so inspect and confirm targets first.

`PUT` writes to a temporary `.davtmp` file and then renames it. Protected paths are rejected. `LOCK` and `UNLOCK` exist for client compatibility only. They do not provide persistent Class 2 locks or lock discovery.

## UDP discovery and network modes

Send the text `hello` to UDP port `8134`. The server replies `crosspoint (on <hostname>);81`. See [setup-and-troubleshooting.md](setup-and-troubleshooting.md) for a netcat example.

In station mode, the reader joins 2.4 GHz Wi-Fi and advertises `crosspoint.local` when mDNS works. In access point mode, it creates the open `CrossPoint-Reader` hotspot and commonly uses `192.168.4.1`. Calibre Wireless uses station mode and the same HTTP and WebSocket servers.

## Live sources

When firmware behavior differs, check:

- [CrossPoint endpoint docs](https://crosspointreader.com/docs#webserver-endpoints)
- [Upstream endpoint source](https://github.com/crosspoint-reader/crosspoint-reader/blob/master/docs/webserver-endpoints.md)
- [Web server guide](https://github.com/crosspoint-reader/crosspoint-reader/blob/master/docs/webserver.md)
