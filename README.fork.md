# dvda-author-neo

> **Fork notice.** This is a fork of [fabnicol/dvda-author](https://github.com/fabnicol/dvda-author)
> (GPL-3.0, © Dave Chapman, Lee and Tim Feldkamp, Fabrice Nicol and contributors).
> It is **not** endorsed by nor affiliated with the original author. All upstream
> credits and the full licence text are preserved; every change relative to
> upstream master is a separate, labelled commit.

## Why a fork

Upstream has a queue of open reports and does not build on current toolchains
(GCC 14+/15, libavcodec 60+/libavutil 60+). This fork starts from upstream
master, fixes what is currently blocking, and is intended to **feed every fix
back as a pull request** rather than diverge.

## What is in here

| Area | Change |
|---|---|
| 24-bit / 6-channel LPCM | channel-group permutation corrected — fixed broadband white noise on 5.1 24-bit discs |
| AMG header | `sectoroffset` sized for its sentinel write — fixed `AUDIO_TS.IFO`/`.BUP` corruption that broke foobar2000 detection |
| Build | `mlp.c` ported to libavcodec/libavutil 7.x–8.x; `close_handles` platform guard; `--merge` interleave pointer types; missing prototype |
| Runtime | `snprintf` stack overflow on the VIDEO_TS path; `fprintf(NULL)` on every `-l`/`-L` run |
| Hygiene | `.gitignore` covering autogen/configure/libtool output; test audio never committed |

## Building

```bash
./autogen
./configure --enable-core-build
make -j$(nproc)
sudo make install      # install is only needed for dvda-author.conf to be found
```

Verified with Ubuntu 26.04, gcc 15.2, libavcodec 62 / libavutil 60,
libFLAC 1.5.0. Two notes that cost me time:

- `sudo make install` partially fails on `install-data-local` (it wants
  `wkhtmltopdf`/`man2html` for the docs). The binary and the config are enough:
  ```bash
  sudo install -m755 src/dvda-author-dev /usr/local/bin/
  sudo install -Dm644 dvda-author.conf \
       /usr/local/share/applications/dvda-author-dev/dvda-author.conf
  ```
  Without that config file the program dies with a fortify abort, not a
  readable error.
- A parallel `make -j16` can corrupt `libfixwav.a` (`bad reloc symbol index`).
  If that happens, `make -j1` after removing the archive is enough.
- `--mkisofs` still needs Jörg Schilling's patched `cdrtools`; the distro
  `genisoimage` is not a substitute.

## Known open problems

- **FLAC input is silently absent** when `configure` misses libFLAC: the build
  then rejects every `.flac` file — including 2-channel ones — with a
  misleading `Null audio characteristics`. Check `config.h` for `#undef FLAC`
  before blaming your 5.1 files. (Upstream #12.)
- **Non-ASCII paths on Windows**: output paths are narrowed to the ANSI codepage,
  which silently drops the top-menu VOB, and filenames containing U+301C are
  rejected as "not a file". Needs `wmain`/`_wfopen`. (Upstream #16.)
- **24-bit multichannel MLP** is still refused by `mlp.c` (upstream behaviour).
- `S[0][5]` (16-bit 6-channel) still uses a channel grouping that disagrees with
  cga 12; it does not produce noise but probably reorders channels.

## Licence

GPL-3.0, unchanged. See [COPYING](COPYING). If you distribute binaries you must
also convey the corresponding source, mark your modifications, and keep every
copyright notice. **Third-party components matter too**: the Windows bundle
ships Qt6 (LGPL-3.0), ImageMagick, cdrtools/mkisofs, ffmpeg and sox, each with
its own redistribution obligations — Qt in particular must stay replaceable
(dynamic linking satisfies that).
