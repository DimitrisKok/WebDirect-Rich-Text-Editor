# FileMaker WebDirect Rich Text Editor

**Two problems. One solution. Both "impossible" — until now.**

[![FileMaker 19+](https://img.shields.io/badge/FileMaker-19%2B-blue)](https://www.claris.com/filemaker/)
[![WebDirect Compatible](https://img.shields.io/badge/WebDirect-Compatible-green)](https://www.claris.com/filemaker/)
[![Native Output](https://img.shields.io/badge/Output-Native%20Text-brightgreen)](https://www.claris.com/filemaker/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 🎯 Two Problems We Solved
### Problem #1: Rich Text in WebDirect (The "Not Supported" Problem)

According to [Claris official documentation](https://help.claris.com/en/webdirect-guide/content/step-two-understand-the-capabilities-of-filemaker-webdirect.html):

> *"FileMaker WebDirect offers limited text styles... Web users cannot enter data with rich text formatting. Editing a field in FileMaker WebDirect removes any existing rich text formatting."*

**Translation:** If you're using WebDirect, you're stuck with plain text. Period. That's what Claris says.

**We solved it.** ✅

---

### Problem #2: The Bitmap Problem (Even Claris's Add-on Has This)

Claris offers a [Rich Text Editor Add-on](https://marketplace.claris.com/detail/1551.html) that uses Quill.js (version from 2020). It works in FileMaker Pro and Go. **But it has a fatal flaw:**

The content stays **inside the WebViewer**. When you print or export to PDF:

| Problem | Impact |
|---------|--------|
| 🖼️ **Bitmap rendering** | WebViewer prints as a low-resolution screenshot |
| 🔍 **Not searchable** | Text inside WebViewer isn't indexed by FileMaker |
| 📄 **Broken layouts** | Your professional vector reports get a blurry image blob |
| 🖨️ **Print quality** | Pixelated, unprofessional output |

**Every WebViewer-based rich text solution has this problem.**

**We solved it.** ✅

---

## 💡 Our Solution: The Best of Both Worlds

```
┌─────────────────────────────────────────────────────────────────┐
│  EDITING (WebDirect + Pro + Go)                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │         Quill.js 2.0.3 in WebViewer                         ││
│  │         Modern WYSIWYG editing experience                   ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ JSON Delta Translation
┌─────────────────────────────────────────────────────────────────┐
│  STORAGE & OUTPUT                                               │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │         Native FileMaker Text Field                         ││ 
│  │         Vector printing • Searchable • Layout-ready         ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

The `saveText` script translates Quill's formatting into FileMaker's native text functions:
- `TextStyleAdd()` for bold, italic, underline, strikethrough, super/subscript
- `TextColor()` for text colors
- `TextFont()` for font families
- `TextSize()` for font sizes

**Result:** Rich text that works in WebDirect AND prints beautifully.

---

## 📊 Comparison: Problems & Solutions

| Problem | Claris Says | Claris Add-on | **This Solution** |
|---------|-------------|---------------|-------------------|
| **Rich Text in WebDirect** | "Not supported" | ❌ Doesn't work | ✅ **SOLVED** |
| **Vector Printing** | N/A | ❌ Bitmap only | ✅ **SOLVED** |
| **Searchable Text** | N/A | ❌ No | ✅ **Yes** |
| **Native Field Output** | N/A | ❌ WebViewer only | ✅ **Yes** |

### Feature Comparison

| Feature | Claris Rich Text Add-on | This Solution |
|---------|------------------------|---------------|
| **Quill Version** | 1.x (2020) | 2.0.3 (2024) |
| **WebDirect Support** | ❌ No | ✅ Yes |
| **Pro/Go Support** | ✅ Yes | ✅ Yes |
| **Output Type** | WebViewer (bitmap) | Native text field |
| **Print Quality** | Screenshot/pixelated | Vector/crisp |
| **PDF Export** | Blurry image | Sharp text |
| **FileMaker Search** | ❌ No | ✅ Yes |
| **Layout Integration** | Iframe blob | Native field |
| **Source Code** | Open | Open (MIT) |
| **Customizable** | Yes | Fully |

---

## ✨ Features

### Text Formatting
- **Bold**, *Italic*, <u>Underline</u>, ~~Strikethrough~~
- Superscript and Subscript
- Text colors (hex and RGB)
- 10 font families (Arial, Times, Courier, Georgia, Verdana, Tahoma, Trebuchet, Helvetica, Serif, Monospace)
- Named sizes (Small, Normal, Large, Huge) and numeric px sizes

### Structure
- Ordered lists with auto-numbering (1. 2. 3.)
- Bullet lists (•)
- Tab indentation for proper list alignment

### Technical
- Native FileMaker text output (prints as vector, not bitmap)
- HTML persistence for WebViewer reload
- Debounced saving (500ms) to prevent script overload
- Cross-version JSON compatibility (FM19+)

---

## 📋 Requirements

- FileMaker Pro 19 or later
- FileMaker Server 19 or later (for WebDirect)
- Modern web browser (Chrome, Firefox, Safari, Edge)

---

## 🚀 Quick Start

### 1. Create Required Fields

In your target table, create:

| Field Name | Type | Purpose |
|------------|------|---------|
| `yourNativeField` | Text | Stores the native FileMaker rich text |
| `yourHTML` | Text | Stores HTML for WebViewer persistence |

### 2. Add the Custom Function

Create a custom function called `SafeJSONParse`:

```
// SafeJSONParse ( json )
// Cross-version JSON parse wrapper for FM19+

Let ( 
  [ 
    // Version detection logic - variation with < 22
    versionCheck = GetAsNumber ( 
      Substitute ( 
        Get ( ApplicationVersion ) ; 
        "." ; 
        Filter ( 1/2 ; ".," ) 
      ) 
    ) < 22
  ] ; 
    Case ( 
      versionCheck ; json ;
      JSONParse ( json ) 
    ) 
)
```

### 3. Import the Script

Copy the `saveText` script from `scripts/saveText_v3.8.txt` into your solution.

**Important:** Update these references in the script:
- `yourTable::yourNativeField` → Your actual table and field names
- `yourTable::yourHTML` → Your actual HTML storage field

### 4. Add the WebViewer

1. Create a WebViewer object on your layout
2. Set the Web Address to: `"data:text/html," & yourTable::yourHTML`
3. For initial content, use the HTML from `webviewer/QuillEditor_v4.html`

### 5. Configure the WebViewer

Set the WebViewer calculation to load either:
- Stored HTML (if `yourHTML` is not empty)
- Default editor HTML (if starting fresh)

```
If ( IsEmpty ( yourTable::yourHTML ) ; 
  // Your default HTML here
  "data:text/html,<html>..." ;
  "data:text/html," & yourTable::yourHTML
)
```

---

## 📁 File Structure

```
fm-webdirect-richtext/
├── README.md                    # This file
├── CHANGELOG.md                 # Version history
├── LICENSE                      # MIT License
├── INSTALLATION.md              # Detailed installation guide
├── scripts/
│   └── saveText_v3.8.txt        # FileMaker script (copy/paste)
├── webviewer/
│   └── QuillEditor_v4.html      # WebViewer HTML/JS
├── custom-functions/
│   └── SafeJSONParse.txt        # Required custom function
└── docs/
    ├── ARCHITECTURE.md          # Technical deep-dive
    ├── TROUBLESHOOTING.md       # Common issues
    └── DELTA_FORMAT.md          # Quill Delta JSON reference
```
## 📚 Documentation Index

Documentation stored under [`/docs`](./docs/)  

---

### 🔹 Core
- [ARCHITECTURE](./docs/ARCHITECTURE.md)
- [DELTA FORMAT](./docs/DELTA_FORMAT.md)
- [CHANGELOG](./docs/CHANGELOG.md)
- [TROUBLESHOOTING](./docs/TROUBLESHOOTING.md)

---
---

## 🔧 How It Works

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      WebViewer                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                  Quill.js Editor                    │    │
│  │  ┌─────────────────────────────────────────────┐    │    │
│  │  │  User types and formats text                │    │    │
│  │  └─────────────────────────────────────────────┘    │    │
│  │                       │                             │    │
│  │                       ▼                             │    │
│  │              JSON Delta + HTML                      │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                  │
│                          ▼                                  │
│         FileMaker.PerformScript("saveText", payload)        │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   saveText Script                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Parse JSON Delta → Apply TextStyleAdd/TextColor/   │    │
│  │  TextFont/TextSize → Build Native FileMaker Text    │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                  │
│                          ▼                                  │
│              ┌──────────────────────┐                       │
│              │  yourNativeField     │  ← Native Rich Text   │
│              │  yourHTML            │  ← WebViewer Source   │
│              └──────────────────────┘                       │
└─────────────────────────────────────────────────────────────┘
```

### The Delta Format

Quill.js outputs a "Delta" format—an array of operations describing text and formatting:

```json
{
  "delta": {
    "ops": [
      { "insert": "Hello ", "attributes": { "bold": true } },
      { "insert": "World", "attributes": { "color": "#FF0000" } },
      { "insert": "\n" }
    ]
  },
  "html": "<p><strong>Hello </strong><span style=\"color: #FF0000;\">World</span></p>"
}
```

The `saveText` script iterates through each operation, applying FileMaker's native text functions:
- `TextStyleAdd()` for bold, italic, underline, etc.
- `TextColor()` for text colors
- `TextFont()` for font families
- `TextSize()` for font sizes

---

## 🐛 Known Limitations

| Limitation | Reason | Workaround |
|------------|--------|------------|
| No inline images | FileMaker text fields don't support embedded images | Use container fields alongside |
| No text alignment | FileMaker's text functions don't support paragraph alignment | Use layout alignment |
| No indentation | Beyond tab characters for lists | Manual tab insertion |
| No highlight/background | FileMaker has no background color function | Use TextColor for emphasis |

**Note:** These are FileMaker platform limitations, not limitations of our approach. The Claris add-on has the same constraints *plus* the bitmap problem.

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

This solution was developed through collaborative AI-human partnership, solving a problem that Claris's own engineering team approached differently (keeping content in WebViewer). The native text translation approach required:

- Deep understanding of FileMaker's styled text internals
- Reverse-engineering Quill's Delta format edge cases  
- Debugging FileMaker's text concatenation quirks (the 1111px bug)
- Testing across WebDirect, Pro, and various browsers

**Contributors:**
- **Dimitris Kokoutsidis** - Project architect, testing, domain expertise, and the insight that native output was the right goal
- **Claude (Anthropic)** - Script architecture, debugging, and documentation
- **Gemini (Google)** - Root cause analysis (v3.5 styled CR bug) and validation

Special thanks to the FileMaker community for decades of creative problem-solving, and to the Quill.js team for their excellent editor.

---

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/your-repo/fm-webdirect-richtext/issues)
- **Discussions**: [GitHub Discussions](https://github.com/your-repo/fm-webdirect-richtext/discussions)
- **Author**: [Dimitris Kokoutsidis](https://axelar.eu)

---

*"Web users cannot enter data with rich text formatting"* — Not anymore.

*"WebViewers print as bitmaps"* — Not ours.

**Two problems. One solution. Open source.** ✨
