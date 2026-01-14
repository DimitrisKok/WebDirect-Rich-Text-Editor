# Quill Delta Format Reference

This document explains the JSON Delta format that Quill.js uses to represent rich text, and how the `saveText` script interprets it.

---

## What is Delta?

Delta is Quill's format for describing rich text content and changes. It's an array of "operations" (ops), where each operation describes what text to insert and what attributes to apply.

**Key Principle:** Delta describes *what the text looks like*, not *how to make it look that way*.

---

## Basic Structure

```json
{
  "ops": [
    { "insert": "text content", "attributes": { "bold": true } },
    { "insert": "more text" },
    { "insert": "\n" }
  ]
}
```

Each operation has:
- **insert** (required): The text to insert
- **attributes** (optional): Formatting to apply

---

## Text Operations

### Plain Text

```json
{ "insert": "Hello World" }
```

No attributes = no special formatting.

### Styled Text

```json
{ "insert": "Bold Text", "attributes": { "bold": true } }
```

### Mixed Styles

Quill creates separate ops for differently-styled text:

```json
{ "insert": "Hello ", "attributes": { "bold": true } },
{ "insert": "World", "attributes": { "italic": true } }
```

---

## Newline Handling

### End of Paragraph

Every paragraph ends with `\n`:

```json
{ "insert": "First paragraph\n" },
{ "insert": "Second paragraph\n" }
```

### List Items

List formatting is applied to the newline, not the text:

```json
{ "insert": "Item one" },
{ "insert": "\n", "attributes": { "list": "ordered" } }
```

This produces: `1. Item one`

### Styled List Items

```json
{ "insert": "Bold item", "attributes": { "bold": true } },
{ "insert": "\n", "attributes": { "list": "bullet" } }
```

This produces: `• Bold item` (bold)

---

## Attribute Reference

### Text Styles

| Attribute | Value | FileMaker Function |
|-----------|-------|-------------------|
| `bold` | `true` | `TextStyleAdd($t ; Bold)` |
| `italic` | `true` | `TextStyleAdd($t ; Italic)` |
| `underline` | `true` | `TextStyleAdd($t ; Underline)` |
| `strike` | `true` | `TextStyleAdd($t ; Strikethrough)` |

### Script (Super/Subscript)

| Attribute | Value | FileMaker Function |
|-----------|-------|-------------------|
| `script` | `"super"` | `TextStyleAdd($t ; Superscript)` |
| `script` | `"sub"` | `TextStyleAdd($t ; Subscript)` |

### Color

| Attribute | Example Value | FileMaker Function |
|-----------|---------------|-------------------|
| `color` | `"#FF0000"` | `TextColor($t ; RGB(255,0,0))` |
| `color` | `"rgb(255,0,0)"` | `TextColor($t ; RGB(255,0,0))` |

### Font

| Attribute | Value | FileMaker Font |
|-----------|-------|----------------|
| `font` | `"arial"` | Arial |
| `font` | `"times"` | Times New Roman |
| `font` | `"courier"` | Courier New |
| `font` | `"georgia"` | Georgia |
| `font` | `"verdana"` | Verdana |
| `font` | `"tahoma"` | Tahoma |
| `font` | `"trebuchet"` | Trebuchet MS |
| `font` | `"helvetica"` | Helvetica |
| `font` | `"serif"` | Times New Roman |
| `font` | `"monospace"` | Courier New |

### Size

| Attribute | Value | FileMaker Points |
|-----------|-------|-----------------|
| `size` | `"small"` | 10 |
| `size` | `"large"` | 18 |
| `size` | `"huge"` | 32 |
| `size` | `"24px"` | 24 |
| `size` | `"48px"` | 48 |

### Lists

| Attribute | Value | Output |
|-----------|-------|--------|
| `list` | `"ordered"` | `1. ` (auto-numbered) |
| `list` | `"bullet"` | `• ` |

---

## Common Patterns

### Pattern 1: Simple Paragraph

**User types:** "Hello World" and presses Enter

```json
{ "insert": "Hello World\n" }
```

### Pattern 2: Multiple Paragraphs

**User types:** "Line 1" Enter "Line 2" Enter

```json
{ "insert": "Line 1\nLine 2\n" }
```

### Pattern 3: Formatted Word

**User types:** "Hello **World**"

```json
{ "insert": "Hello " },
{ "insert": "World", "attributes": { "bold": true } },
{ "insert": "\n" }
```

### Pattern 4: Ordered List

**User creates:**
```
1. First
2. Second
```

```json
{ "insert": "First" },
{ "insert": "\n", "attributes": { "list": "ordered" } },
{ "insert": "Second" },
{ "insert": "\n", "attributes": { "list": "ordered" } }
```

### Pattern 5: Mixed Formatting

**User types:** "Hello" (red, bold, Times, 18pt)

```json
{
  "insert": "Hello",
  "attributes": {
    "bold": true,
    "color": "#FF0000",
    "font": "times",
    "size": "18px"
  }
},
{ "insert": "\n" }
```

### Pattern 6: Text Then List (Tricky!)

**User types:**
```
Intro text
1. First item
```

```json
{ "insert": "Intro text\nFirst item" },
{ "insert": "\n", "attributes": { "list": "ordered" } }
```

Note: Quill combines "Intro text\nFirst item" into ONE op! This is why CASE A needs the v3.8 buffer split fix.

### Pattern 7: Leading Newline (Tricky!)

When copying/pasting or in certain edit scenarios:

```json
{ "insert": "\nSome text" }
```

A single op with leading newline. This is why CASE C exists.

---

## How the Script Routes Operations

```
┌─────────────────────────────────────────────────────────────┐
│                    INCOMING OP                               │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
            ┌──────────────────────────────┐
            │  Is chunk exactly "¶"?       │
            └──────────────────────────────┘
                    │              │
                   YES            NO
                    │              │
                    ▼              ▼
              ┌─────────┐   ┌──────────────────────────┐
              │ CASE A  │   │ Ends with ¶ AND has      │
              │ Pure ¶  │   │ list attribute?          │
              └─────────┘   └──────────────────────────┘
                                   │              │
                                  YES            NO
                                   │              │
                                   ▼              ▼
                             ┌─────────┐   ┌──────────────────────────┐
                             │ CASE B  │   │ Starts with ¶ AND        │
                             │ List    │   │ length > 1?              │
                             └─────────┘   └──────────────────────────┘
                                                  │              │
                                                 YES            NO
                                                  │              │
                                                  ▼              ▼
                                            ┌─────────┐   ┌─────────┐
                                            │ CASE C  │   │ CASE D  │
                                            │ Split   │   │ Normal  │
                                            └─────────┘   └─────────┘
```

---

## Edge Cases

### Empty Document

Fresh editor:
```json
{ "insert": "\n" }
```

Just a single newline.

### Only Whitespace

```json
{ "insert": "   \n" }
```

Spaces preserved, ends with newline.

### Multiple Newlines (Blank Lines)

```json
{ "insert": "\n\n\n" }
```

Three consecutive newlines = two blank lines.

### Escaped Characters

Quill handles escaping automatically:
- `"` → `\"`
- `\` → `\\`
- Newlines → `\n` (not `¶` until we convert)

---

## Debugging Tips

### View Raw Delta

In browser console:
```javascript
console.log(JSON.stringify(quill.getContents(), null, 2));
```

### View Specific Op

```javascript
console.log(quill.getContents().ops[0]);
```

### Check Attributes

```javascript
quill.getContents().ops.forEach((op, i) => {
  console.log(`Op ${i}:`, op.insert, op.attributes || '(no attrs)');
});
```

---

## FileMaker Debugging

### Check Raw JSON

```
$$debug
```

### Parse and Inspect

```
JSONGetElement ( $$debug ; "delta.ops[0].insert" )
JSONGetElement ( $$debug ; "delta.ops[0].attributes.bold" )
```

### Count Operations

```
ValueCount ( JSONListKeys ( JSONGetElement ( $$debug ; "delta" ) ; "ops" ) )
```

---

## Reference Links

- [Quill Delta Documentation](https://quilljs.com/docs/delta/)
- [Delta GitHub Repository](https://github.com/quilljs/delta)
- [Quill API: getContents()](https://quilljs.com/docs/api/#getcontents)
