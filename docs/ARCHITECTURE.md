# Architecture Documentation

This document provides a technical deep-dive into how the FileMaker WebDirect Rich Text Editor works.

---

## Overview

This solution solves **two problems** that have plagued FileMaker developers:

### Problem #1: Rich Text in WebDirect
Claris documentation explicitly states WebDirect users "cannot enter data with rich text formatting." We solved this by using a WebViewer with Quill.js for editing, then translating to native text.

### Problem #2: The Bitmap Problem
Even solutions that use WebViewers for rich text (including Claris's own add-on) suffer from a fatal flaw: the content prints as a bitmap screenshot. We solved this by storing the output in a native text field.

### The Bridge

The solution bridges two worlds:
1. **Quill.js** - A modern JavaScript rich text editor running in a WebViewer (editing)
2. **FileMaker's native text functions** - TextStyleAdd, TextColor, TextFont, TextSize (storage/output)

The magic happens in the `saveText` script, which translates Quill's JSON "Delta" format into FileMaker-native rich text.

### Why This Approach?

| Approach | WebDirect Editing | Print/PDF Result | Searchable |
|----------|-------------------|------------------|------------|
| Native field | ❌ Not supported | Vector text | ✅ |
| Claris Add-on | ❓ Limited | Screenshot (bitmap) | ❌ |
| **Our Solution** | ✅ Full support | Vector text | ✅ |

By using WebViewer for editing AND translating to native text for storage, we get the best of both worlds.

---

## Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER ACTION                             │
│                    Types/formats in editor                      │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       QUILL.JS (WebViewer)                      │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ text-change event → getContents() → JSON Delta            │  │
│  │                   → root.innerHTML → HTML                 │  │
│  │                   → getText() → Plain text                │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  JSON Payload:                                                  │
│  {                                                              │
│    "delta": { "ops": [...] },                                   │
│    "html": "<p>...</p>",                                        │
│    "plain": "..."                                               │
│  }                                                              │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ FileMaker.PerformScript
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      saveText SCRIPT                            │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Parse JSON → Iterate ops → Apply FileMaker text styles   │  │
│  │                                                           │  │
│  │  For each op:                                             │  │
│  │    1. Normalize newlines (LF → ¶)                         │  │
│  │    2. Force base style (Arial 12)                         │  │
│  │    3. Route to appropriate CASE                           │  │
│  │    4. Apply attributes (bold, color, font, size)          │  │
│  │    5. Handle list markers                                 │  │
│  │    6. Add to buffer or flush to output                    │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      OUTPUT FIELDS                              │
│  ┌─────────────────────┐  ┌─────────────────────┐               │
│  │  yourNativeField    │  │  yourHTML           │               │
│  │  (Native Rich Text) │  │  (WebViewer Source) │               │
│  └─────────────────────┘  └─────────────────────┘               │
└─────────────────────────────────────────────────────────────────┘
```

---

## The Delta Format

Quill uses a format called "Delta" to represent rich text as JSON. It's an array of operations, where each operation describes:
- **insert**: The text content
- **attributes**: The formatting applied

### Example Delta

**Input:** "Hello World" with "Hello" bold and "World" red

```json
{
  "ops": [
    { "insert": "Hello ", "attributes": { "bold": true } },
    { "insert": "World", "attributes": { "color": "#FF0000" } },
    { "insert": "\n" }
  ]
}
```

### Key Patterns

| Quill sends... | Meaning | Script handles via... |
|----------------|---------|----------------------|
| `{"insert":"\n"}` | End of line | CASE A |
| `{"insert":"text\n","attributes":{"list":"ordered"}}` | List item | CASE B |
| `{"insert":"\ntext"}` | Leading newline | CASE C |
| `{"insert":"text"}` | Plain text | CASE D |

---

## Script Architecture

### Processing Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     MAIN LOOP (for each op)                     │
├─────────────────────────────────────────────────────────────────┤
│  Step 3.1: Get chunk and attributes from JSON                   │
│  Step 3.2: Normalize newlines (LF → ¶)                          │
│            Apply base style (Arial 12)                          │
│  Step 3.3: Calculate flags (endsWithReturn, startsWithReturn)   │
├─────────────────────────────────────────────────────────────────┤
│                      CASE ROUTING                               │
├───────────────┬───────────────┬───────────────┬─────────────────┤
│    CASE A     │    CASE B     │    CASE C     │    CASE D       │
│  Pure ¶       │  Text + ¶     │  Leading ¶    │  Standard       │
│               │  + List attr  │  + Text       │  Text           │
├───────────────┼───────────────┼───────────────┼─────────────────┤
│ • Add list    │ • Strip ¶     │ • Flush prev  │ • Apply styles  │
│   marker      │ • Apply styles│ • Strip lead  │ • Add to buffer │
│ • Flush       │ • Add marker  │ • Apply styles│ • Flush if ¶    │
│   buffer      │ • Flush       │ • Add to buf  │                 │
└───────────────┴───────────────┴───────────────┴─────────────────┘
```

### Case Details

#### CASE A: Pure Line Terminator
**Condition:** `$chunk = ¶`

Handles end-of-line markers, including list terminators:
- Ordered list → Prepend "1. " with tab
- Bullet list → Prepend "• " with tab
- Plain paragraph → Just flush

**v3.8 Fix:** Splits buffer if it contains embedded returns, ensuring list markers only apply to the last line.

#### CASE B: Text + Return + List Attribute
**Condition:** `$endsWithReturn AND $hasListAttr`

Handles inline list items where Quill sends text with formatting AND a list attribute:
```json
{"insert":"Item one\n","attributes":{"list":"ordered","bold":true}}
```

#### CASE C: Leading Return (Split Chunk)
**Condition:** `$startsWithReturn AND Length($chunk) > 1`

Handles Quill's pattern of combining newline with following text:
```json
{"insert":"\nNext line text"}
```

This was the source of the infamous "1. one" bug where list numbering appeared in wrong places.

#### CASE D: Standard Text
**Condition:** Everything else

Handles normal text with optional formatting:
```json
{"insert":"Regular text","attributes":{"italic":true}}
```

---

## Styling Architecture

### The Base Style Strategy

Every chunk receives Arial 12 as a base style **before** any formatting is applied:

```
$chunk = TextFont ( $chunk ; "Arial" )
$chunk = TextSize ( $chunk ; 12 )
```

This prevents FileMaker's styled text engine from inheriting corrupt or unexpected formatting from the field's state.

### Style Application Order

```
1. Base style (Arial 12)
2. Bold/Italic/Underline/Strike
3. Superscript/Subscript
4. Color (hex or RGB)
5. Font family (overrides base)
6. Font size (overrides base)
```

Explicit attributes always override the base style.

---

## Buffer Strategy

The script uses a buffer to accumulate styled text before flushing to the output:

```
┌─────────────────────────────────────────────────────────────────┐
│                      BUFFER LIFECYCLE                           │
├─────────────────────────────────────────────────────────────────┤
│  1. Text arrives → Apply styles → Append to $Buffer             │
│  2. When ¶ arrives → Flush $Buffer to $FinalText                │
│  3. If list → Prepend marker BEFORE flushing                    │
│  4. Clear $Buffer for next line                                 │
└─────────────────────────────────────────────────────────────────┘
```

This allows list markers to be prepended to complete lines rather than individual text chunks.

---

## Critical Bug Fixes

### The 1111px Bug (v3.5 → v3.7)

**Problem:** Plain text rendered with 1111px font size instead of 12px.

**Root Cause:** FileMaker's styled text concatenation corrupts style metadata when mixing pre-styled text (like `$styledCR`) with post-styled chunks.

**Fix:** Remove `$styledCR` variable entirely. Apply styling inline at each flush point:

```
$FinalText = $FinalText & $Buffer & TextSize ( TextFont ( ¶ ; "Arial" ) ; 12 )
```

### The List Marker Bug (v3.7 → v3.8)

**Problem:** Text preceding a list would incorrectly receive the list marker.

**Root Cause:** When Quill sends multi-line content before a list, the entire buffer (containing multiple lines) received the list marker.

**Fix:** Before applying list markers, check if buffer contains embedded returns. If so, flush all but the last line first:

```
If [ PatternCount ( $Buffer ; ¶ ) > 0 ]
    // Split and flush preceding lines
    // Keep only last line in buffer for marker
End If
```

---

## FileMaker Text Functions Used

| Function | Purpose | Example |
|----------|---------|---------|
| `TextStyleAdd()` | Apply text styles | `TextStyleAdd($text ; Bold)` |
| `TextColor()` | Apply text color | `TextColor($text ; RGB(255,0,0))` |
| `TextFont()` | Apply font family | `TextFont($text ; "Georgia")` |
| `TextSize()` | Apply font size | `TextSize($text ; 24)` |
| `GetAsCSS()` | Debug: View applied styles | `GetAsCSS($text)` → `<span style="...">` |

---

## Font Mapping

| Quill Value | FileMaker Font |
|-------------|----------------|
| `arial` | Arial |
| `times` | Times New Roman |
| `courier` | Courier New |
| `georgia` | Georgia |
| `verdana` | Verdana |
| `tahoma` | Tahoma |
| `trebuchet` | Trebuchet MS |
| `helvetica` | Helvetica |
| `serif` | Times New Roman |
| `monospace` | Courier New |

---

## Size Mapping

| Quill Value | FileMaker Points |
|-------------|------------------|
| `small` | 10 |
| (default) | 12 |
| `large` | 18 |
| `huge` | 32 |
| `NNpx` | NN (extracted number) |

---

## Debugging

### Global Variables

The script sets these globals for debugging:

| Variable | Contents |
|----------|----------|
| `$$debug` | Raw JSON payload |
| `$$debugFinalText` | Output analysis |
| `$$debugCSS` | GetAsCSS of final output |

### Using GetAsCSS

To inspect styled text, use:

```
GetAsCSS ( yourTable::yourNativeField )
```

This returns HTML-style CSS showing exactly what styles FileMaker has applied:

```html
<span style="font-family: 'Arial';font-size: 12px;font-weight: bold;">Hello</span>
<span style="font-family: 'Arial';font-size: 12px;color: #FF0000;">World</span>
```

---

## Performance Considerations

1. **Debouncing:** The WebViewer uses a 500ms debounce to prevent script overload during typing
2. **Loop Efficiency:** The script processes ops in a single pass with O(n) complexity
3. **Buffer Strategy:** Reduces string concatenation operations by batching
4. **Base Style:** Prevents style recalculation by establishing predictable defaults

---

## Limitations

| Limitation | Reason |
|------------|--------|
| No images | FileMaker text fields don't support embedded images |
| No alignment | No FileMaker function for paragraph alignment |
| No indentation | Beyond tabs for lists, no indent control |
| WebDirect only | Solution designed for browser-based editing |
| CDN dependency | Quill.js loads from CDN (requires internet) |

---

## Future Enhancements

Potential improvements for future versions:

1. **Tables** - Complex but theoretically possible with careful delta parsing
2. **Offline mode** - Bundle Quill.js in container field for no-internet scenarios
