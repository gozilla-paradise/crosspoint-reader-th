# CLAUDE.md - Project Guide for AI Assistants

## Project Overview

CrossPoint Reader TH is a Thai language fork of CrossPoint Reader, firmware for the Xteink X4 e-paper reader. It adds Thai text rendering, word segmentation, and glyph shaping capabilities.

**Target Hardware:** Xteink X4 (ESP32-C3, ~380KB usable RAM)
**Build System:** PlatformIO
**Language:** C++ (Arduino framework)

## Build & Flash Commands

```bash
# Build firmware
pio run

# Build and flash to device
pio run --target upload

# Clean build
pio run --target clean
```

## Project Architecture

### Source Structure

```
src/                      # Main application code
  main.cpp                # Entry point
  activities/             # UI screens/activities (MVC-like pattern)
    reader/               # EPUB, TXT, XTC reader activities
    home/                 # Home screen, library browser
    settings/             # Settings screens
    network/              # WiFi, web server activities
  network/                # HTTP, OTA, web server
  util/                   # String utilities, URL handling

lib/                      # Libraries (internal)
  Epub/                   # EPUB parsing and rendering (CRITICAL)
    Epub/                 # Core EPUB implementation
      ParsedText.cpp      # Text layout, line extraction, spacing
      EpubParser.cpp      # HTML parsing, word tokenization
      GfxRenderer.cpp     # Text rendering to display
  ThaiShaper/             # Thai text processing (THIS FORK)
    ThaiCharacter.cpp/h   # Thai Unicode classification
    ThaiWordBreak.cpp/h   # Cluster-based word segmentation
    ThaiClusterBuilder.*  # Glyph shaping, combining marks
  EpdFont/                # Font handling, glyph rendering
    builtinFonts/         # NotoSansThai font files
  GfxRenderer/            # Display rendering abstraction
  Utf8/                   # UTF-8 string handling

open-x4-sdk/              # Hardware abstraction (submodule)
  libs/display/           # E-ink display driver
```

### Key Files for Thai Support

| File | Purpose |
|------|---------|
| `lib/Epub/Epub/ParsedText.cpp` | Text layout, Thai spacing logic, line extraction |
| `lib/ThaiShaper/ThaiWordBreak.cpp` | Thai cluster-based word segmentation |
| `lib/ThaiShaper/ThaiClusterBuilder.cpp` | Thai glyph shaping, combining marks |
| `lib/ThaiShaper/ThaiCharacter.cpp` | Thai Unicode character classification |
| `lib/Epub/Epub/GfxRenderer.cpp` | `drawThaiText()` for Thai rendering |

### Thai Text Processing Pipeline

1. **HTML Parsing** (`EpubParser.cpp`): Tokenizes text on whitespace, calls `addWord()`
2. **Word Segmentation** (`ParsedText.cpp` → `ThaiWordBreak`): Splits Thai words at cluster boundaries
3. **Spacing Logic** (`ParsedText.cpp`): Suppresses artificial spaces between Thai words (HTML artifact)
4. **Glyph Shaping** (`ThaiClusterBuilder`): Positions combining marks (vowels, tones)
5. **Rendering** (`GfxRenderer.cpp`): Draws Thai text with proper glyph offsets

### Thai Spacing Rules (ParsedText.cpp)

- **Thai-to-Thai**: No space between consecutive Thai words (HTML whitespace is an artifact)
- **Thai punctuation** (ๆ, ฯ): Preserve space after these marks (they are word boundaries)
- **Mixed text**: Preserve original spacing for Thai-Latin transitions
- **wordNoSpaceBefore flag**: Set by word segmenter for continuation clusters

## Memory Constraints

The ESP32-C3 has only ~380KB usable RAM. The codebase uses:
- Aggressive SD card caching (`.crosspoint/` directory)
- Static buffers instead of dynamic allocation in Thai segmentation
- Streaming/chunked processing for EPUB content

## Common Tasks

### Debugging Thai Text Issues

1. Check `ParsedText.cpp` for spacing logic in `extractLine()` and `addWord()`
2. Verify `ThaiWordBreak.cpp` cluster boundary detection
3. Check `containsThai()` function for Thai detection accuracy
4. Examine `ThaiClusterBuilder` for glyph positioning issues

### Adding New Thai Features

1. Thai character classification: `ThaiCharacter.cpp`
2. Word breaking rules: `ThaiWordBreak.cpp`
3. Rendering changes: `GfxRenderer.cpp` → `drawThaiText()`
4. Layout/spacing: `ParsedText.cpp`

## Upstream Sync

This is a fork of https://github.com/daveallie/crosspoint-reader

```bash
# Add upstream remote (if not present)
git remote add crosspoint-reader https://github.com/crosspoint-reader/crosspoint-reader.git

# Fetch and merge upstream changes
git fetch crosspoint-reader
git merge crosspoint-reader/master
```

## Testing

No automated test suite. Testing is done on physical Xteink X4 hardware with Thai EPUB files.

Test cases for Thai:
- Normal Thai text: Should flow without spaces
- Thai + numbers: Spacing preserved
- Thai punctuation (ๆ, ฯ): Space after punctuation
- Mixed Thai-English: Proper spacing at transitions
