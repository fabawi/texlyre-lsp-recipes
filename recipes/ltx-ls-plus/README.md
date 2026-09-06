# LTeX LS Plus (LanguageTool)

LTeX LS Plus is a grammar and spelling checker for LaTeX, Markdown, BibTeX, and
other prose-heavy formats. It is based on LanguageTool and works well for
academic writing.

TeXlyre connects to LTeX LS Plus via WebSocket using `lsp-ws-proxy`.

---

## 1. Install the server

Download and extract LTeX LS Plus from the releases page.

Example for Linux x64:

```bash
VERSION="18.7.0"
ARCHIVE="ltex-ls-plus-${VERSION}-linux-x64.tar.gz"

curl -L -o "$ARCHIVE" \
  "https://github.com/ltex-plus/ltex-ls-plus/releases/download/$VERSION/$ARCHIVE"

tar -xzf "$ARCHIVE"
````

This creates a directory such as:

```bash
ltex-ls-plus-18.7.0
```

LTeX LS Plus requires Java. If Java is not already installed, install a recent
JRE, such as Temurin JRE 21.

---

## 2. Install the WebSocket proxy

TeXlyre requires a WebSocket transport. `lsp-ws-proxy` bridges WebSocket to stdio.

Install `lsp-ws-proxy`:

```bash
cargo install lsp-ws-proxy --locked
```

---

## 3. Run the server

Start the proxy and launch LTeX LS Plus over stdio:

```bash
lsp-ws-proxy -l 127.0.0.1:7020 -- ./ltex-ls-plus-18.7.0/bin/ltex-ls-plus
```

If you use a manually downloaded JRE, set `JAVA_HOME` first:

```bash
export JAVA_HOME="/path/to/jre"
lsp-ws-proxy -l 127.0.0.1:7020 -- ./ltex-ls-plus-18.7.0/bin/ltex-ls-plus
```

* The proxy listens on `ws://localhost:7020`
* Keep this process running while using TeXlyre

---

## 4. TeXlyre configuration

You can use **more than one LSP at the same time**. To do this, simply **add
another config block below** in the **same list**.

Keep everything inside the square brackets `[` `]` and separate each `{` `}`
block with a comma.

Paste the following JSON into <SettingsPath path="External Tools > Generic LSP > LSP Recipes" />.

```json
[
  {
    "id": "ltex-ls",
    "name": "LTeX LS Plus",
    "enabled": true,
    "fileExtensions": ["tex", "latex", "typ", "typst", "bib", "md", "markdown"],
    "languageIdMap": {
      "tex": "latex",
      "latex": "latex",
      "typ": "typst",
      "typst": "typst",
      "bib": "bibtex",
      "md": "markdown",
      "markdown": "markdown"
    },
    "transportConfig": {
      "type": "websocket",
      "url": "ws://localhost:7020",
      "contentLength": false
    },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[],\"capabilities\":{\"workspace\":{\"configuration\":true},\"window\":{\"workDoneProgress\":false}},\"initializationOptions\":{\"ltex\":{\"language\":\"en-US\",\"additionalRules\":{\"motherTongue\":\"\"},\"dictionary\":{},\"disabledRules\":{},\"enabledRules\":{}}}}"
  }
]
```

---

## Debugging

To check that the server starts correctly, run it directly first:

```bash
./ltex-ls-plus-18.7.0/bin/ltex-ls-plus
```

To confirm the WebSocket proxy is listening:

```bash
lsp-ws-proxy -l 127.0.0.1:7020 -- ./ltex-ls-plus-18.7.0/bin/ltex-ls-plus
```

Common things to check:

* Make sure the path to `ltex-ls-plus` is correct.
* Make sure Java is installed and available on `PATH`, or set `JAVA_HOME`.
* If you change the proxy port, update `transportConfig.url`.
* If diagnostics do not appear, restart both the proxy and TeXlyre.


