# C HTTP Server (simple routes + static files)

A minimal HTTP/1.1 toy web server in C that listens on a TCP port, parses the request line, routes known paths to HTML templates, and serves a CSS asset from a static directory. It demonstrates basic socket setup, a binary-search-tree route map, and file-to-response rendering.

## Project layout

- include/
  - HTTP_Server.h — server struct and init entrypoint
  - Routes.h — Route tree API (init, add, search, inorder)
  - Response.h — file rendering API (returns file contents as heap buffer)
- src/
  - HTTP_Server.c — socket(), bind(), listen() and server initialization
  - main.c — accept loop, request parsing, routing, and response sending
  - Routes.c — route tree implementation (BST)
  - Response.c — read file contents into memory for responses
- templates/
  - index.html — home page template
  - about.html — about page template
  - 404.html — not found page
- static/
  - index.css — stylesheet
- compile_commands.json — example compilation database (for tooling)

Note: Folder names reflect intent; if files are currently flat, create these directories and place files accordingly.

## Features

- Minimal HTTP request-line parsing (method and path).
- Route registry backed by a simple BST:
  - “/” -> templates/index.html
  - “/about” -> templates/about.html
- Static asset handling for a CSS file under /static/.
- Basic console logging of incoming requests.

## Build

Requires a POSIX system with a C compiler and sockets:

- Dependencies: gcc or clang, make (optional)

Quick compile (direct):

- gcc -Iinclude -Wall -Wextra -Wpedantic -O2 -g -o websrv \
  src/HTTP_Server.c src/Routes.c src/Response.c src/main.c

Using compile_commands.json (tooling):

- The provided compile_commands.json shows how individual objects are built with -Iinclude. Adjust paths if the directory structure differs locally.

## Run

- ./websrv
  - The server in main.c initializes on port 6969 by default (as coded).
- Visit:
  - http://127.0.0.1:6969/ -> templates/index.html
  - http://127.0.0.1:6969/about -> templates/about.html
  - http://127.0.0.1:6969/static/index.css -> static CSS asset

Use curl to test:

1. curl -v http://127.0.0.1:6969/
2. curl -v http://127.0.0.1:6969/about
3. curl -v http://127.0.0.1:6969/does-not-exist

## Current limitations

1. HTTP parsing is minimal (only the request line is used).
2. Content-Length and Content-Type are not fully implemented.
3. Error handling is limited; some system call results may not be checked.
4. strtok-based parsing mutates request buffers and needs careful control flow.
5. Response send length should match actual header+body length (avoid sizeof on fixed buffers).
6. File reading should null-terminate buffers and return sizes for accurate Content-Length.

These are intentional simplifications for learning; see “Improvement roadmap” for next steps.

## Routing

- Register routes in main.c via:
  - struct Route* route = initRoute("/", "index.html");
  - addRoute(route, "/about", "about.html");
- The server looks up the path (e.g., “/about”) and loads templates/<value>.
- Unknown routes fall back to templates/404.html.

Static files

- Requests containing “/static/” are served from the static directory (currently wired to static/index.css in code; generalization recommended).

## Improvement roadmap

- Robust HTTP: parse CRLF properly, validate method/path, handle headers, and compute Content-Length exactly.
- Safer string handling: prefer snprintf over strcat; trim ‘\r’ from tokens.
- Response correctness: send exact header_len + body_len; set Connection: close or implement keep-alive.
- File serving: generalize static file mapping, add MIME types, and prevent directory traversal.
- Error handling: check socket(), bind(), listen(), accept(), read(), send() return values; add SO_REUSEADDR.
- Memory and lifecycle: ensure all allocated buffers are freed and client sockets closed on all paths.
- Logging and testing: structured logs and basic test scripts with curl.

## Getting started (local dev)

1) Create folders:

- mkdir -p include src templates static

2) Place files:

- include/: HTTP_Server.h, Routes.h, Response.h
- src/: HTTP_Server.c, Routes.c, Response.c, main.c
- templates/: index.html, about.html, 404.html
- static/: index.css

3) Build:

- gcc -Iinclude -Wall -Wextra -Wpedantic -O2 -g -o websrv \
  src/HTTP_Server.c src/Routes.c src/Response.c src/main.c

4) Run and test:

- ./websrv
- curl -v http://127.0.0.1:6969/

## License

Add your chosen license (MIT recommended for simple examples). Include a LICENSE file at the repo root.

## Acknowledgments

- This project is a learning exercise in C socket programming, manual HTTP handling, and basic routing.

If a Makefile is desired, a simple one can be provided. Also happy to add a corrected, drop-in patch set once ready to accept improvements (Content-Length, safer parsing, and error checks).
