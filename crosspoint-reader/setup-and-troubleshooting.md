# Setup and troubleshooting

Use this guide when the CrossPoint server is not already reachable. The server has no authentication and stops when the user exits **File Transfer** or **Calibre Wireless** mode.

## Start the server

From the Home screen, open **File Transfer** and choose one mode:

| Mode | Connection |
| --- | --- |
| **Join Network** | The reader joins an existing 2.4 GHz Wi-Fi network. The client must be on the same network. |
| **Calibre Wireless** | The reader joins Wi-Fi and starts the same server with Calibre upload progress. |
| **Create Hotspot** | The reader creates the open `CrossPoint-Reader` network. Connect the client to it. |

Keep the reader on the server screen. It shows the direct IP URL and usually `http://crosspoint.local/`. In hotspot mode, the fallback IP is typically `192.168.4.1`, but prefer the URL shown on screen.

## Diagnose in order

1. Confirm the reader still shows File Transfer or Calibre Wireless.
2. Confirm the client is on the same joined network or connected to `CrossPoint-Reader`.
3. Use `http://`, not HTTPS.
4. Try status through mDNS:

```sh
curl -v --max-time 5 http://crosspoint.local/api/status
```

5. If the host does not resolve, use the IP shown on the reader:

```sh
BASE_URL=http://192.168.1.102
curl -v --max-time 5 "$BASE_URL/api/status"
```

6. If neither works, disable VPN, proxy, iCloud Private Relay, or cellular fallback temporarily. Guest Wi-Fi and some managed networks use client isolation. Switch to **Create Hotspot** when isolation cannot be changed.
7. Exit and re-enter File Transfer, reconnect the client, then retry.

A JSON response from `/api/status` proves both network reachability and the CrossPoint HTTP server. The home page alone is a weaker test.

## Interpret common failures

| Symptom | Likely cause | Next action |
| --- | --- | --- |
| `Could not resolve host` | mDNS failure | Use the displayed IP. |
| Connection timeout | Wrong network, wrong IP, isolation, VPN, or weak signal | Check both devices, use hotspot mode, or move closer. |
| Connection refused | Device reachable but server not listening | Re-enter File Transfer or Calibre Wireless and keep that screen open. |
| HTTP error response | Server reached; request or firmware behavior differs | Read the response and current endpoint docs. |
| Upload stalls or fails | Weak Wi-Fi, full SD card, invalid filename, or WebSocket trouble | Check status RSSI, inspect SD-card capacity on the device or another computer, try a small HTTP upload, then retry. |

Station mode status returns `"mode":"STA"` and RSSI in dBm. Hotspot mode returns `"mode":"AP"` and RSSI `0`. `freeHeap` is free RAM, not SD-card capacity. The documented API has no free-storage endpoint.

## UDP discovery

When the displayed IP is unavailable, send `hello` to UDP port `8134` on the local network. The response is `crosspoint (on <hostname>);81`, where `81` is the WebSocket port.

Netcat flags differ by platform. A common command is:

```sh
printf 'hello' | nc -u -w 2 255.255.255.255 8134
```

Use the subnet broadcast address if global broadcast is blocked. UDP discovery identifies the service but does not replace the HTTP status check.

## Security and cleanup

- HTTP uses port 80, WebSocket uses 81, and discovery uses UDP 8134.
- The server and hotspot have no authentication.
- Anyone with network access can manage files and settings while the server runs.
- Exit File Transfer when finished to stop the server and conserve battery.

## Firmware drift

CrossPoint is under active development. If these steps differ from the device, note the firmware version from `/api/status` and check the current [web server guide](https://github.com/crosspoint-reader/crosspoint-reader/blob/master/docs/webserver.md), [endpoint reference](https://crosspointreader.com/docs#webserver-endpoints), and [troubleshooting guide](https://github.com/crosspoint-reader/crosspoint-reader/blob/master/docs/troubleshooting.md).
