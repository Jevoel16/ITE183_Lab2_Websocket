# ITE183 Lab 2 — WebSocket Server

A simple HTTP + WebSocket chat server written in Bonezegei (`.bzg`) for ITE183 Lab 2. The server serves a browser test client and broadcasts every message it receives to all connected clients.

## Features

- Serves `public/index.html` at `/` and `public/404.html` for any other path
- Accepts WebSocket connections on the same port (`8000`)
- Sends a welcome message to each client when it connects
- Broadcasts each incoming message to every connected client
- Logs connects, messages, and disconnects to the console

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

Then open <http://localhost:8000> in your browser. Open it in several tabs to see messages broadcast between clients.

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
