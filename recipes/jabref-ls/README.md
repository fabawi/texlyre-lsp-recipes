# JabRef LS (BibTeX Diagnostics)

JabRef LS (jabls) is the language server behind JabRef's official VS Code
extension. It provides BibTeX and BibLaTeX integrity diagnostics for `.bib`
files, surfacing problems such as duplicate citation keys, malformed entries,
invalid years, and bad page ranges as you edit.

TeXlyre connects to JabRef LS via WebSocket using lsp-ws-proxy. Unlike most
language servers, jabls speaks LSP over a TCP socket rather than stdio, so an
extra hop with socat is required to bridge the two.

---

## 1. Install the server

JabRef LS is run directly from JabRef's `JabLsLauncher.java` script via
[JBang](https://www.jbang.dev/), which fetches and compiles it on first run.
It requires Java 21.

Install JBang:

```bash
curl -Ls https://sh.jbang.dev | bash -s - app setup
```

Trust the JabRef source so JBang can run it non-interactively, then prefetch the
launcher (the first run downloads and compiles it):

```bash
jbang trust add https://raw.githubusercontent.com/JabRef/jabref/main/.jbang/
jbang https://raw.githubusercontent.com/JabRef/jabref/main/.jbang/JabLsLauncher.java --help
```

---

## 2. Install the WebSocket proxy and bridge

TeXlyre requires a WebSocket transport. lsp-ws-proxy bridges WebSocket to stdio.
Because jabls listens on a TCP socket, socat is used to connect that stdio to
the server's port.

Install lsp-ws-proxy (distributed via Cargo, requires the Rust toolchain):

```bash
cargo install lsp-ws-proxy --locked
```

Install socat with your system package manager, for example:

```bash
sudo apt install socat        # Debian/Ubuntu
brew install socat            # macOS
```

---

## 3. Run the server

Start jabls as a TCP server on an internal port, then bridge it to TeXlyre.
Run each command in its own terminal, or background the first.

Start JabRef LS on TCP port 2087:

```bash
jbang https://raw.githubusercontent.com/JabRef/jabref/main/.jbang/JabLsLauncher.java -p 2087
```

Start the proxy, bridging the WebSocket to the TCP server via socat:

```bash
lsp-ws-proxy -l 127.0.0.1:7021 -- socat STDIO TCP:127.0.0.1:2087
```

- The proxy listens on ws://localhost:7021
- jabls listens internally on TCP port 2087
- Keep both processes running while using TeXlyre

---

## 4. TeXlyre configuration

You can use **more than one LSP at the same time**. To do this, simply **add
another config block below this one in the same list**.
Keep everything inside the square brackets `[` `]` and separate each `{` `}`
block with a comma.

Paste the following JSON into <SettingsPath path="External Tools > Generic LSP > LSP Recipes" />.

```json
[
  {
    "id": "jabref-ls",
    "name": "JabRef LS",
    "enabled": true,
    "fileExtensions": ["bib", "bibtex"],
    "languageIdMap": {
      "bib": "bibtex",
      "bibtex": "bibtex"
    },
    "transportConfig": {
      "type": "websocket",
      "url": "ws://localhost:7021",
      "contentLength": false
    },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[]}"
  }
]
```

---

## Notes

- If you change the proxy port, update `transportConfig.url` accordingly. The
  internal TCP port (2087) is independent and only needs to match between the
  jabls and socat commands.
- Diagnostics are read-only integrity checks; the server does not modify your
  `.bib` files.
- jabls is built primarily for JabRef's VS Code extension and tracks the
  development branch; running from `main` uses the latest server.