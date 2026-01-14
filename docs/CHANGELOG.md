# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [3.8] - 2026-01-14

### Fixed
- **List marker placement bug**: Fixed issue where text preceding a list would incorrectly receive the list marker
- Added buffer split logic in CASE A to ensure only the last line receives list markers

### Technical
- Added `PatternCount` check for embedded returns in buffer before list marker application
- Properly flushes multi-line buffer content before applying list formatting

---

## [3.7] - 2026-01-14

### Fixed
- **1111px font size corruption bug**: Fixed critical FileMaker styled text concatenation issue
- Removed `$styledCR` variable that caused text corruption during concatenation
- Implemented inline styled CR at each flush point

### Changed
- Emergency flush now re-applies base styling to prevent corruption
- All flush operations use `TextSize ( TextFont ( ¶ ; "Arial" ) ; 12 )` inline

---

## [3.6] - 2026-01-14

### Fixed
- **Color not working**: Fixed `$textPart` → `$chunk` variable name error in CASE C and CASE D
- **Font change not working**: Restored full 10-font mapping (was reduced to 2 in v3.5)
- **Size by number not working**: Restored numeric/px size parsing

### Technical
- All formatting blocks now consistently use `$chunk` variable
- Added `?` character guard to prevent invalid JSON element parsing

---

## [3.5] - 2026-01-14

### Added
- **Styled CR constant**: Pre-styled carriage return to prevent default inheritance
- **Forced base style**: Arial 12 applied to all text before attribute processing

### Changed
- Simplified CASE C logic with "Flush & Split" approach
- Removed complex `ValueCount` loop that caused ghost lines

### Collaboration
- Root cause analysis by Gemini 3 Pro
- Implementation by Claude Opus 4.5

---

## [3.4] - 2026-01-13

### Added
- **Font face support**: 10 font families mapped from Quill to FileMaker
  - Arial, Times New Roman, Courier New, Georgia, Verdana
  - Tahoma, Trebuchet MS, Helvetica, Serif, Monospace
- **Font size support**: Named sizes (small, large, huge) and numeric px values

### Fixed
- Typo: `$Buffe` → `$Buffer` in D6 and Emergency Flush sections

---

## [3.3] - 2026-01-13

### Added
- **Superscript support**: `TextStyleAdd ( $text ; Superscript )`
- **Subscript support**: `TextStyleAdd ( $text ; Subscript )`
- Quill `script` attribute parsing ("super" / "sub")

---

## [3.2] - 2026-01-13

### Added
- **HTML persistence**: Saves Quill HTML to separate field for WebViewer reload
- WebViewer now maintains state across layout changes and record navigation

### Changed
- Script now accepts both `delta` and `html` in JSON payload
- Added `yourTable::yourHTML` field reference

---

## [3.1] - 2026-01-13

### Added
- **Tab indentation**: `Char(9)` added to list prefixes for visual alignment
- Ordered lists render as: `[Tab]1. Text`
- Bullet lists render as: `[Tab]• Text`

---

## [3.0] - 2026-01-13

### Added
- **CASE C-SPECIAL**: "Split Chunk" logic to handle Quill's leading newline pattern
- Fixes bug where first list item rendered as `one1.` instead of `1. one`

### Technical
- Detects chunks starting with `\n` but containing more content
- Splits and processes segments independently

---

## [2.0] - 2026-01-13

### Added
- **List support**: Ordered lists (1. 2. 3.) and bullet lists (•)
- **Buffer strategy**: Accumulates styled text before flushing to output
- **CASE A**: Pure line terminator handling
- **CASE B**: Text + return + list attribute handling

### Changed
- Refactored from linear processing to case-based routing

---

## [1.0] - 2026-01-12

### Added
- Initial implementation
- **Bold** support via `TextStyleAdd ( $text ; Bold )`
- **Italic** support via `TextStyleAdd ( $text ; Italic )`
- **Underline** support via `TextStyleAdd ( $text ; Underline )`
- **Strikethrough** support via `TextStyleAdd ( $text ; Strikethrough )`
- **Color support**: Hex (#FF0000) and RGB (rgb(255,0,0)) parsing
- JSON Delta parsing from Quill.js WebViewer
- Basic newline normalization

---

## Development Timeline

| Date | Version | Milestone |
|------|---------|-----------|
| 2026-01-12 | v1.0 | Basic formatting working |
| 2026-01-13 | v2.0 | Lists working |
| 2026-01-13 | v3.0 | List bug fixed |
| 2026-01-13 | v3.1 | Tab indentation |
| 2026-01-13 | v3.2 | HTML persistence |
| 2026-01-13 | v3.3 | Super/subscript |
| 2026-01-13 | v3.4 | Fonts and sizes |
| 2026-01-14 | v3.5 | Giant text bug analysis |
| 2026-01-14 | v3.6 | Attribute fixes |
| 2026-01-14 | v3.7 | 1111px bug fixed |
| 2026-01-14 | v3.8 | List placement bug fixed ✅ |

---

## Contributors

- **Dimitris Kokoutsidis** - Architecture, testing, domain expertise
- **Claude (Anthropic)** - Script development, debugging
- **Gemini (Google)** - Root cause analysis, validation
