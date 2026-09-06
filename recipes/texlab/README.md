# TexLab (LaTeX & BibTeX)

TexLab is a language server for LaTeX and BibTeX. It provides diagnostics,
completion, hover information, go-to-definition, references, document symbols,
and formatting for `.tex` and `.bib` files. Unlike prose-focused servers such as
Harper LS or LTeX LS Plus, TexLab handles LaTeX structure including commands,
environments, labels, citations, and the project files.

TeXlyre connects to TexLab via WebSocket using lsp-ws-proxy. TexLab communicates LSP
over stdio, so the proxy bridges the WebSocket directly to the server's standard
input and output.

---

## 1. Install the server

TexLab includes prebuilt binaries for Linux, macOS, and Windows.

Example for Linux x64:

```bash
VERSION="v5.26.0"
ARCHIVE="texlab-x86_64-linux.tar.gz"

curl -L -o "$ARCHIVE" \
  "https://github.com/latex-lsp/texlab/releases/download/$VERSION/$ARCHIVE"

tar -xzf "$ARCHIVE"
```

This extracts a single `texlab` binary. Move it somewhere on your `PATH`:

```bash
chmod +x texlab
mv texlab ~/.local/bin/
```

Alternatively, build from the tagged source with
[Cargo](https://www.rust-lang.org/tools/install) (requires the Rust toolchain):

```bash
cargo install --git https://github.com/latex-lsp/texlab --tag v5.26.0 --locked texlab
```

Do not install TexLab from crates.io; the project no longer publishes server
releases to the registry, so that version will be outdated.

---

## 2. Install the WebSocket proxy

TeXlyre requires a WebSocket transport. lsp-ws-proxy bridges WebSocket to stdio.

Install lsp-ws-proxy (distributed via Cargo, requires the Rust toolchain):

```bash
cargo install lsp-ws-proxy --locked
```

---

## 3. Run the server

Start the proxy and launch TexLab over stdio:

```bash
lsp-ws-proxy -l 127.0.0.1:7031 -- texlab
```

- The proxy listens on ws://localhost:7031
- TexLab runs as a child process and communicates over stdio
- Keep this process running while using TeXlyre

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
    "id": "texlab",
    "name": "TexLab (LaTeX)",
    "enabled": true,
    "fileExtensions": ["tex", "latex", "bib", "bibtex"],
    "languageIdMap": {
      "tex": "latex",
      "latex": "latex",
      "bib": "bibtex",
      "bibtex": "bibtex"
    },
    "transportConfig": {
      "type": "websocket",
      "url": "ws://localhost:7031",
      "contentLength": false
    },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[],\"capabilities\":{\"workspace\":{\"configuration\":true}},\"initializationOptions\":{}}"
  }
]
```

TexLab pairs well with a prose checker: running it alongside LTeX LS Plus gives
structural LaTeX diagnostics from TexLab and grammar diagnostics from LTeX on
the same file.

---

## Notes

- If you change the proxy port, update `transportConfig.url` accordingly.
- TexLab indexes the packages a project uses to build its completion data, so
  the first index of a large project can take a moment. Completions improve once
  indexing finishes.
- Build-related features such as forward and inverse search, or `chktex`
  diagnostics, depend on a local TeX distribution and external tools, and are
  not supported through TeXlyre.
- TexLab resolves a project's dependencies on its own and does not need
  `%!TEX root` magic comments.

---

## Debugging

To check that the server starts correctly, run it directly first:

```bash
texlab --version
```

Common things to check:

- Make sure `texlab` is on your `PATH`, or pass its absolute path to the proxy.
- If diagnostics do not appear, restart both the proxy and TeXlyre.