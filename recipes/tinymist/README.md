# Tinymist (Typst Language Service)

Tinymist is the language server behind the official Typst VS Code extension. It
provides an integrated language service for `.typ` files, including diagnostics,
autocompletion, hover information, go-to-definition, document symbols, and
formatting as you edit.

TeXlyre connects to Tinymist via WebSocket using lsp-ws-proxy. Tinymist speaks
LSP over stdio, so the proxy bridges the WebSocket directly to the server's
standard input and output with no extra hops.

---

## 1. Install the server

Tinymist is distributed as a single `tinymist` binary. You can build it from
source with [Cargo](https://www.rust-lang.org/tools/install) (requires the Rust
toolchain):

```bash
cargo install --git https://github.com/Myriad-Dreamin/tinymist --locked tinymist-cli
```

Prebuilt binaries are also available from the project's
[GitHub Releases](https://github.com/Myriad-Dreamin/tinymist/releases) if you
prefer not to compile.

---

## 2. Install the WebSocket proxy

TeXlyre requires a WebSocket transport. lsp-ws-proxy bridges WebSocket to stdio.

Install lsp-ws-proxy (distributed via Cargo, requires the Rust toolchain):

```bash
cargo install lsp-ws-proxy --locked
```

---

## 3. Run the server

Start the proxy and let it spawn the language server over stdio:

```bash
lsp-ws-proxy -l 127.0.0.1:7030 -- tinymist lsp
```

- The proxy listens on ws://localhost:7030
- Tinymist runs as a child process and communicates over stdio
- Keep the process running while using TeXlyre

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
    "id": "tinymist",
    "name": "Tinymist (Typst)",
    "enabled": true,
    "fileExtensions": ["typ", "typst"],
    "languageIdMap": {
      "typ": "typst",
      "typst": "typst"
    },
    "transportConfig": {
      "type": "websocket",
      "url": "ws://localhost:7030",
      "contentLength": false
    },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[],\"capabilities\":{\"workspace\":{\"configuration\":false}},\"initializationOptions\":{\"semanticTokens\":\"disable\",\"systemFonts\":false}}"
  }
]
```

---

## Notes

- If you change the proxy port, update `transportConfig.url` accordingly.
- The first build compiles Tinymist from source and can take several minutes. To
  pin a version, replace the `cargo install --git` target with a tagged ref or
  use a published release binary.
- Tinymist computes diagnostics against a project root; for best results open a
  workspace folder rather than a single isolated file.
- Tinymist also provides preview and tracing servers; this recipe runs only the
  language server (`tinymist lsp`).