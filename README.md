# office2md

Read documents from the command line like `cat`, printing their contents as
Markdown (or plain text). A thin wrapper around **pandoc**, **LibreOffice** and
**pdftotext**.

```console
$ office2md report.docx
# Quarterly Report

...
```

## Usage

```
office2md FILE [FILE ...]
```

- Multiple files are concatenated to stdout (like `cat`).
- Output goes to stdout, so it pipes and redirects naturally:
  ```sh
  office2md notes.doc | less
  office2md slides.pptx > slides.md
  ```
- Exit status is non-zero if any file failed.

## Supported formats

| Route | Formats |
|-------|---------|
| **pandoc** (direct) | docx, odt, epub, pptx, xlsx, rtf, html, csv, tsv, md, rst, org, latex, mediawiki, asciidoc, textile, fb2, ipynb, typst, docbook, json, ... |
| **LibreOffice → pandoc** | doc, ppt, xls, dot, pps, xlt, wpd, wps, abw, sxw/sxi/sxc, odp, ods, key, numbers, pages, dbf, ... |
| **pdftohtml → pandoc** (else **pdftotext**) | pdf |
| **verbatim** | txt, text, log, and any file `file(1)` detects as text |

Files whose extension is unknown are sniffed with `file(1)`: text is printed
verbatim, recognised office documents go through LibreOffice, and anything else
(images, archives, random binaries) is reported as unsupported instead of being
dumped as garbage.

## Images

Images embedded in a document are **extracted to a temporary folder** (by
default under `/tmp`, e.g. `/tmp/XXX/`), and the Markdown
links in the output point to those real files, so they can be opened:

```console
$ office2md report.docx
...
![A red square](/tmp/qwz/media/rId9.png)
...
```

The folder is only kept when the document actually contains images; image-free
documents leave nothing behind. Set `DOC2TXT_MEDIADIR` to choose a different
base directory. (These are temp files — `/tmp` is cleared on reboot; copy any
images you want to keep.)

## How it works

- `.docx`/`.odt`/`.pptx`/`.xlsx`/… are read by pandoc directly.
- Legacy/binary office files are converted by LibreOffice to the matching modern
  format (`doc→docx`, `ppt→pptx`, `xls→xlsx`) and then handed to pandoc.
  The conversion uses a private, temporary LibreOffice profile, so it works even
  while you have LibreOffice open.
- `.pdf` is handled by `pdftohtml` when available — its HTML keeps bold/italic
  and page structure, which pandoc renders to Markdown. When `pdftohtml` is not
  installed it falls back to `pdftotext -layout` (plain text). LibreOffice and
  pandoc cannot read PDFs reliably, so they are not used for PDF.

## Requirements

- `pandoc`
- `libreoffice` / `soffice` — for legacy office formats
- `poppler-utils` (`pdftohtml` preferred, else `pdftotext`) — for PDF
- `file` — for content sniffing of unknown extensions

## Configuration

| Variable | Default | Meaning |
|----------|---------|---------|
| `DOC2TXT_FORMAT` | `gfm` | pandoc output format (e.g. `markdown`, `plain`, `commonmark`) |
| `DOC2TXT_SOFFICE` | autodetected | path to the `libreoffice`/`soffice` binary |
| `DOC2TXT_MEDIADIR` | `$TMPDIR` or `/tmp` | base directory for extracted images |

```sh
DOC2TXT_FORMAT=plain office2md report.docx   # plain text instead of Markdown
```

## Install

```sh
chmod +x office2md
ln -s "$PWD/office2md" ~/.local/bin/office2md   # ensure ~/.local/bin is on PATH
```
