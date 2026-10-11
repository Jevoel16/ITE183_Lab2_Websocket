# ITE183 Lab 2 — WebSocket Server

A Google Meet–style video call with in-call text chat, built on an HTTP + WebSocket server written in Bonezegei (`.bzg`) for ITE183 Lab 2.

Video and audio travel peer-to-peer between browsers using **WebRTC**. The Bonezegei WebSocket server is the **signaling and chat relay**: it relays every message to all clients, and each browser ignores messages that aren't addressed to it.

## Features

- Join screen with camera preview and name entry
- Multi-person video grid (mesh WebRTC) with name labels and muted-mic indicators
- Mic / camera toggles, screen sharing, and a Leave button
- In-call chat side panel with an unread badge and pop-up notifications
- Serves `public/index.html` at `/` and `public/404.html` for any other path
- Accepts WebSocket connections on the same port (`8000`)
- Logs connects, messages, and disconnects to the console

## How It Works

| Server → browser | Meaning |
| --- | --- |
| `__id__:<clientId>` | Sent on connect so the browser knows its own ID |
| `__left__:<clientId>` | Broadcast on disconnect so others remove that video tile |

Browsers send JSON messages (`join`, `offer`, `answer`, `ice`, `media`, `chat`, `leave`) that the server relays unchanged with `sendAll`. When someone joins, each existing participant sends them a WebRTC offer; they answer, exchange ICE candidates, and the media then flows directly between browsers.

The native `http.dll` crashes on WebSocket frames somewhere between 8 KB and 16 KB, and a WebRTC offer is often around 9 KB. To stay under that limit, the browser splits any message over 3,000 characters into `chunk` messages and puts them back together when they arrive.

## Project Structure

```
ITE183_Lab2_Websocket/
├── app.bzg               # Server entry point (HTTP + WebSocket handlers)
├── package.json          # Project metadata and the "start" script used by `bzg run`
├── libwinpthread-1.dll   # MinGW threading runtime required by http.dll
├── lib/
│   ├── http.bzg          # Loads the native http module
│   ├── http.json         # API description of the http module
│   └── http/
│       └── http.dll      # Native HTTP/WebSocket module
└── public/
    ├── index.html        # Browser WebSocket test client
    └── 404.html          # Not-found page
```

## Requirements

- Windows (64-bit)
- Bonezegei installed, with `bonezegei` and `bzg` on your PATH (default location: `C:\Program Files\Bonezegei`)
- A modern web browser

## Running the Server

From the project root, run:

```bash
bzg run
```

or, equivalently:

```bash
bonezegei app.bzg
```

You should see:

```
HTTP server listening on port 8000 (High Concurrency Thread-Pool Active)
```

Then open <http://localhost:8000> in your browser, enter your name, and click **Join now**. Open it in a second window (or a different browser) and join again to start a call between the two.

> **Camera access:** browsers only allow the camera/microphone on `localhost` or HTTPS. Other devices on your LAN can open `http://<your-ip>:8000` and use chat, but their camera will be blocked over plain HTTP.

> **Note:** Run the server from the project root so that `lib/`, `public/`, and `libwinpthread-1.dll` are found.

## Server API (`http` module)

| Member | Description |
| --- | --- |
| `http(port)` | Creates a server on the given port |
| `server.get = function (path, data)` | HTTP GET callback |
| `server.post = function (path, data)` | HTTP POST callback |
| `server.sendFile(filename)` | Sends a file to the HTTP client |
| `server.sendResponse(message, type, code)` | Sends a custom HTTP response |
| `server.onConnect = function (clientId)` | Called when a WebSocket client connects |
| `server.onData = function (message, clientId)` | Called when a WebSocket client sends data |
| `server.onDisconnect = function (clientId)` | Called when a WebSocket client disconnects |
| `server.send(clientId, message)` | Sends a message to one client |
| `server.sendAll(message)` | Sends a message to all clients |
| `server.run()` | Starts the server loop (blocking) |

## Troubleshooting

### `Failed to load library: lib/http/http.dll`

```
Failed to load library: lib/http/http.dll
Undefined variable http
Error at Line 8: Not a function in simple call context near 'server' (Token IDENT)
```

`http.dll` was built with MinGW and depends on `libwinpthread-1.dll` (the MinGW POSIX-threads runtime). If Windows can't find that file, the module fails to load and the later errors follow from that.

Fix it in one of these ways:

- Keep `libwinpthread-1.dll` in the project root (included in this repo) and run from the project root, **or**
- Add a folder that contains it to your PATH, e.g. `C:\Program Files\Git\mingw64\bin` if Git for Windows is installed.

### Port 8000 already in use

Stop the other process using port 8000, or change the port in `app.bzg` (`http(8000)`) and the `wsUrl` in `public/index.html` to match.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).
