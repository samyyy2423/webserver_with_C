# C Web Server

A small HTTP server written from scratch in C: it opens a TCP socket, parses the request line, looks the
path up in a binary-search-tree router, and serves HTML templates and a stylesheet. Built to learn
sockets, HTTP and routing without a framework.

## What's inside

| File | Role |
|------|------|
| `src/HTTP_Server.c`, `include/HTTP_Server.h` | Creates the TCP socket, binds the port, listens |
| `src/main.c` | Accept loop: reads the request, parses the first line, routes, sends the response |
| `src/Routes.c`, `include/Routes.h` | Binary search tree mapping paths (`/`, `/about`) to templates |
| `src/Response.c`, `include/Response.h` | Reads a file into memory for the response body |
| `templates/` | `index.html`, `about.html`, `404.html` |
| `static/` | `index.css`, served under `/static/` |

## Build and run

```bash
make            # builds ./server.o
./server.o      # listens on port 6969
```

Then open <http://127.0.0.1:6969/> or:

```bash
curl -v http://127.0.0.1:6969/
curl -v http://127.0.0.1:6969/about
```

Unknown paths fall back to `templates/404.html`.

## Known limitations / next steps

- Send `Content-Length` and `Content-Type` headers.
- Check return values of `socket()`, `bind()`, `listen()`, `accept()`, `read()`, `send()`.
- Parse the request line strictly (CRLF, method, path) instead of `strtok`.
- Generalise static-file serving and block `..` path traversal.
- Replace `strcat` with `snprintf` to rule out buffer overflows.
