# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

For the full embedded systems development guide (hardware constraints, coding patterns, ESP32-C3 pitfalls, memory management), see [.skills/SKILL.md](.skills/SKILL.md).

## Project Overview

CrossPoint Reader is open-source e-reader firmware for the Xteink X4 (ESP32-C3, single-core RISC-V @ 160MHz, ~380KB RAM, no PSRAM). Built with C++20 on Arduino/ESP-IDF via PlatformIO. No exceptions, no RTTI.

## Build Commands

```bash
pio run                        # Build (default dev environment)
pio run -e gh_release          # Build production release
pio run -t upload              # Build and flash to device
pio run -t clean               # Clean build artifacts
```

## Code Quality (run before PRs)

```bash
./bin/clang-format-fix         # Format code (requires clang-format 21+)
pio check --fail-on-defect low --fail-on-defect medium --fail-on-defect high  # Static analysis
pio run                        # Verify build succeeds
```

## Monitoring

```bash
python3 scripts/debugging_monitor.py   # Enhanced serial monitor (recommended)
pio device monitor                     # Standard monitor
```

## Architecture

**Layered design** (top to bottom):

- **Activities** (`src/activities/`) — Android-style UI screens with lifecycle: `onEnter()` → `loop()` → `onExit()`. Heap-allocated, deleted on exit. All memory allocated in `onEnter()` must be freed in `onExit()`.
- **Core Services** — Singletons: `SETTINGS`, `APP_STATE`, `GUI`, `Storage`, `I18N`
- **Libraries** (`lib/`) — Epub parser, GfxRenderer, EpdFont, I18n, Serialization
- **HAL** (`lib/hal/`) — Always use `HalDisplay`/`HalGPIO`/`HalStorage`, never raw SDK classes
- **SDK** (`open-x4-sdk/`) — Low-level hardware drivers (git submodule)

**Key entry point**: `src/main.cpp` — hardware init, font loading, activity manager setup.

## I18n (Translations)

Source files: `lib/I18n/translations/<language>.yaml` (YAML, UTF-8)

```bash
# Regenerate after editing YAML files
python scripts/gen_i18n.py lib/I18n/translations lib/I18n/
```

Usage in code: `tr(STR_LOADING)` macro — all user-facing text must use this. Logging (`LOG_DBG`/`LOG_ERR`) can be hardcoded.

**Commit**: source YAML + `I18nKeys.h` + `I18nStrings.h`. Do NOT commit `I18nStrings.cpp`.

## Generated Files — Do Not Edit

- `src/network/html/*.generated.h` — from `data/html/` via `scripts/build_html.py`
- `lib/I18n/I18nKeys.h`, `I18nStrings.h`, `I18nStrings.cpp` — from YAML via `scripts/gen_i18n.py`
- All regenerated automatically during `pio run` (pre-build scripts)

## Key Conventions

- **Formatting**: 2-space indent, 120-col limit, clang-format 21+ (see `.clang-format`)
- **Naming**: PascalCase classes, camelCase methods/vars, UPPER_SNAKE_CASE constants
- **Headers**: `#pragma once`
- **Strings**: No `std::string`/Arduino `String` in hot paths. Use `std::string_view` for read-only, `char[]` + `snprintf` for construction.
- **Memory**: 380KB RAM ceiling. `malloc` must check for nullptr. Free immediately after use. Justify heap allocations.
- **Logging**: Use `LOG_INF`/`LOG_DBG`/`LOG_ERR` from `Logging.h`, never raw Serial.
- **Orientation**: Never hardcode 800/480. Use `renderer.getScreenWidth()`/`getScreenHeight()`.
- **Buttons**: Use `MappedInputManager::Button::*` enums, never raw `HalGPIO::BTN_*`.

## CI/CD

GitHub Actions on PRs: clang-format check, cppcheck, build validation. PR titles must be semantic (`feat:`, `fix:`, `refactor:`, etc.).

## Build Environments

| Environment     | LOG_LEVEL | Use Case          |
| --------------- | --------- | ----------------- |
| `default`       | 2 (DEBUG) | Development       |
| `gh_release`    | 0 (ERROR) | Production        |
| `gh_release_rc` | 1 (INFO)  | Release candidate |
| `slim`          | —         | No serial logging |
