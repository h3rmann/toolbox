# toolbox

Small command-line tools I built to automate my own workflow: getting knowledge out of PDFs and videos and into plain Markdown, where I (and the AI assistants I work with) can search, read and reuse it.

| Command | What it does |
| :--- | :--- |
| [`pdf2md`](pdf2md) | Converts a PDF into a Markdown file next to it. Uses [docling](https://github.com/docling-project/docling) when installed; otherwise falls back to `pdftotext`, with OCR for scanned PDFs. |
| [`yt2md`](yt2md) | Turns a YouTube video's captions into a Markdown transcript: YAML metadata, chapter list, and timestamped paragraphs of about a minute each. |

Both are single-file Bash scripts with a small embedded Python step. macOS and Linux.

## Install

```bash
git clone https://github.com/h3rmann/toolbox.git ~/toolbox
echo 'export PATH="$HOME/toolbox:$PATH"' >> ~/.zshrc   # or ~/.bashrc
```

Open a new terminal, and the commands are available.

**zsh users:** YouTube URLs contain `?`, which zsh treats as a filename wildcard. Either quote the URL, or add this once to `~/.zshrc`:

```bash
alias yt2md='noglob yt2md'
```

## pdf2md

```bash
pdf2md paper.pdf          # writes paper.md next to it
PDF2MD_ENGINE=basic pdf2md paper.pdf   # skip docling, use pdftotext
PDF2MD_OCR_LANGS=eng pdf2md scan.pdf   # OCR languages (default: spa+eng)
```

**How the docling route runs:** OCR is skipped when the PDF already has a text layer (OCR is the slowest stage; force it with `PDF2MD_OCR=1`), tables use docling's fast mode, images become placeholders instead of embedded base64, and all CPU cores are used. It prints a progress line every 30 s, and gives up after 30 s per page (at least 15 min; `PDF2MD_DOCLING_TIMEOUT` to change). If docling fails or times out, `pdf2md` falls back to the basic engine. On macOS docling runs on the CPU, because its layout model currently crashes on Apple's GPU backend (MPS); override with `PDF2MD_DOCLING_DEVICE=auto|cpu|mps|cuda`.

**Requirements:** `docling` (recommended, `pip install docling`) — or — `poppler` (`pdftotext`) and `python3`, plus `ocrmypdf` and `tesseract` with the language packs you need for scanned PDFs.

On macOS: `brew install poppler ocrmypdf tesseract tesseract-lang`.

**Limits of the basic engine:** it produces clean text with guessed headings (short ALL-CAPS lines become `##`), not structural Markdown. Tables and lists don't survive. For those, install docling.

## yt2md

```bash
yt2md https://youtu.be/VIDEO_ID        # the video's own language (falls back to English)
yt2md https://youtu.be/VIDEO_ID es     # force Spanish captions
```

Writes `<video title>.md` in the current folder:

```markdown
---
title: "..."
channel: "..."
url: "https://www.youtube.com/watch?v=..."
published: "2026-09-07"
duration: "21:30"
language: "en"
captions: "auto-generated"
fetched: "2026-09-28"
---

# Video title

## Chapters
- 00:00 Intro
...

## Transcript

**[00:00]** First minute of speech...
```

**Which captions it picks:** manual captions in the requested language first, then YouTube's original auto-generated track (`<lang>-orig`), and only then the machine-translated track. That order matters: YouTube rate-limits the translated track heavily (HTTP 429), while the original track downloads fine. Without a language argument it uses the video's own language. Videos with no captions at all are reported and skipped: yt2md reads captions, it does not transcribe audio.

**Requirements:** [`yt-dlp`](https://github.com/yt-dlp/yt-dlp) and `python3` (3.9+). On macOS: `brew install yt-dlp`.

**Notes:** auto-generated captions have no punctuation guarantees and mis-hear words; the transcript is kept verbatim, errors included. The tool only downloads caption files and metadata, never the video. It's meant for personal note-taking — respect creators and YouTube's terms.

## License

[MIT](LICENSE)
