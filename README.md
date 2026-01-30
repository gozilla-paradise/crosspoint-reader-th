# CrossPoint Reader TH

## สำหรับท่านที่ต้องการสนับสนุนการทำ Firmware ภาษาไทยนี้ต่อไป สามารถสนับสนุนเลี้ยงกาแฟได้ที่ โดยตกเฉลี่ยๆ 5$ ต่อเดือน ขอบคุณมากครับ
### สิ่งที่ได้รับคือ สามารถดาวน์โหลด Firmware Bleeding Edge ใหม่ล่าสุด รวมถึงเครื่องมือใหม่ๆ ก่อนใคร
> https://www.patreon.com/13080975/join

-------------------

Thai language fork of [CrossPoint Reader](https://github.com/daveallie/crosspoint-reader) — firmware for the **Xteink X4** e-paper display reader (unaffiliated with Xteink).
Built using **PlatformIO** and targeting the **ESP32-C3** microcontroller.

This fork adds **Thai language support** including proper Thai text rendering, word segmentation, and glyph shaping for Thai EPUB content.

![](./docs/images/cover.jpg)

## Motivation

This fork extends the excellent [CrossPoint Reader](https://github.com/daveallie/crosspoint-reader) project with Thai language support. Thai text requires special handling due to:
- Complex script with combining marks (vowels, tone marks above/below consonants)
- No spaces between words (requires cluster-based word segmentation)
- Proper glyph shaping for correct visual rendering

CrossPoint Reader TH aims to:
* Provide Thai EPUB reading capability on the **Xteink X4**.
* Properly render Thai text with correct glyph positioning.
* Handle Thai word boundaries for line wrapping.
* Maintain compatibility with upstream CrossPoint features.

This project is **not affiliated with Xteink**; it's built as a community fork.

## Features & Usage

- [x] EPUB parsing and rendering (EPUB 2 and EPUB 3)
- [ ] Image support within EPUB
- [x] Saved reading position
- [x] File explorer with file picker
  - [x] Basic EPUB picker from root directory
  - [x] Support nested folders
  - [ ] EPUB picker with cover art
- [x] Custom sleep screen
  - [x] Cover sleep screen
- [x] Wifi book upload
- [x] Wifi OTA updates
- [x] Configurable font, layout, and display options
  - [ ] User provided fonts
  - [ ] Full UTF support
- [x] Screen rotation
- [x] **Thai language support** (this fork)
  - [x] Thai text rendering with proper glyph shaping
  - [x] Thai cluster-based word segmentation
  - [x] Built-in NotoSansThai font
  - [x] Thai punctuation and spacing handling

Multi-language support: Read EPUBs in various languages, including English, Spanish, French, German, Italian, Portuguese, Russian, Ukrainian, Polish, Swedish, Norwegian, [and more](./USER_GUIDE.md#supported-languages).

See [the user guide](./USER_GUIDE.md) for instructions on operating CrossPoint. 

## Installing

### Web (latest firmware)

1. Connect your Xteink X4 to your computer via USB-C
2. Go to https://xteink.dve.al/ and click "Flash CrossPoint firmware"

To revert back to the official firmware, you can flash the latest official firmware from https://xteink.dve.al/, or swap
back to the other partition using the "Swap boot partition" button here https://xteink.dve.al/debug.

### Web (specific firmware version)

1. Connect your Xteink X4 to your computer via USB-C
2. Download the `firmware.bin` file from the release of your choice via the [releases page](https://github.com/daveallie/crosspoint-reader/releases)
3. Go to https://xteink.dve.al/ and flash the firmware file using the "OTA fast flash controls" section

To revert back to the official firmware, you can flash the latest official firmware from https://xteink.dve.al/, or swap
back to the other partition using the "Swap boot partition" button here https://xteink.dve.al/debug.

### Manual

See [Development](#development) below.

## Development

### Prerequisites

* **PlatformIO Core** (`pio`) or **VS Code + PlatformIO IDE**
* Python 3.8+
* USB-C cable for flashing the ESP32-C3
* Xteink X4

### Checking out the code

CrossPoint uses PlatformIO for building and flashing the firmware. To get started, clone the repository:

```
git clone --recursive https://github.com/daveallie/crosspoint-reader

# Or, if you've already cloned without --recursive:
git submodule update --init --recursive
```

### Flashing your device

Connect your Xteink X4 to your computer via USB-C and run the following command.

```sh
pio run --target upload
```

## Internals

CrossPoint Reader is pretty aggressive about caching data down to the SD card to minimise RAM usage. The ESP32-C3 only
has ~380KB of usable RAM, so we have to be careful. A lot of the decisions made in the design of the firmware were based
on this constraint.

### Data caching

The first time chapters of a book are loaded, they are cached to the SD card. Subsequent loads are served from the 
cache. This cache directory exists at `.crosspoint` on the SD card. The structure is as follows:


```
.crosspoint/
├── epub_12471232/       # Each EPUB is cached to a subdirectory named `epub_<hash>`
│   ├── progress.bin     # Stores reading progress (chapter, page, etc.)
│   ├── cover.bmp        # Book cover image (once generated)
│   ├── book.bin         # Book metadata (title, author, spine, table of contents, etc.)
│   └── sections/        # All chapter data is stored in the sections subdirectory
│       ├── 0.bin        # Chapter data (screen count, all text layout info, etc.)
│       ├── 1.bin        #     files are named by their index in the spine
│       └── ...
│
└── epub_189013891/
```

Deleting the `.crosspoint` directory will clear the entire cache. 

Due the way it's currently implemented, the cache is not automatically cleared when a book is deleted and moving a book
file will use a new cache directory, resetting the reading progress.

For more details on the internal file structures, see the [file formats document](./docs/file-formats.md).

## Contributing

Contributions are very welcome!

If you're looking for a way to help out, take a look at the [ideas discussion board](https://github.com/daveallie/crosspoint-reader/discussions/categories/ideas).
If there's something there you'd like to work on, leave a comment so that we can avoid duplicated effort.

### To submit a contribution:

1. Fork the repo
2. Create a branch (`feature/dithering-improvement`)
3. Make changes
4. Submit a PR

---

CrossPoint Reader is **not affiliated with Xteink or any manufacturer of the X4 hardware**.

Huge shoutout to [**diy-esp32-epub-reader** by atomic14](https://github.com/atomic14/diy-esp32-epub-reader), which was a project I took a lot of inspiration from as I
was making CrossPoint.
