# FileMaker WebDirect Rich Text Bridge

**Extending WebDirect with native rich text editing for professional reporting.**

[![FileMaker 19+](https://img.shields.io/badge/FileMaker-19%2B-blue)](https://www.claris.com/filemaker/)
[![WebDirect Compatible](https://img.shields.io/badge/WebDirect-Compatible-green)](https://www.claris.com/filemaker/)
[![Native Output](https://img.shields.io/badge/Output-Native%20Text-brightgreen)](https://www.claris.com/filemaker/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 🎯 What This Module Does

FileMaker WebDirect is an incredible deployment tool that has revolutionized how we deliver apps. One common request from clients is **rich text editing** that also **prints perfectly** on reports—something that has traditionally been challenging in a web environment.

Rather than seeing this as a limitation, we saw it as an opportunity to leverage the power of the FileMaker Script Engine.

This open-source module enables robust rich text editing in WebDirect while maintaining the **data integrity** and **print fidelity** that FileMaker is known for.

---

## 💡 The Approach: A Translation Engine

This isn't a workaround—it's a **bridge**. It uses a modern web interface ([Quill.js 2.0](https://quilljs.com/)) for the user experience, but crucially, it respects the FileMaker database by converting that input into **native FileMaker styled text**.

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

---

## ✨ Why This Matters for Your Solution

### Data Purity
By storing data as **native text** (not HTML blobs), your database remains clean, searchable, and fast. FileMaker's indexing and Find operations work perfectly on the formatted content.

### Report Quality
Because the data is native, your printed PDFs leverage FileMaker's powerful **vector rendering engine**. Your invoices, letters, and reports look razor-sharp at any resolution.

### Seamless Integration
It works alongside your existing layouts. The native text field respects your Theme styling and integrates naturally with your design.

### No External Dependencies
Built entirely with standard FileMaker tools—Scripting and WebViewers. No plugins required.

---

## 📋 Features

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
- Native FileMaker text output
- HTML persistence for WebViewer state
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

| Field Name | Type | Purpose |
|------------|------|---------|
| `RichText_Native` | Text | Stores the native FileMaker rich text |
| `RichText_HTML` | Text | Stores HTML for WebViewer persistence |

### 2. Add the Custom Function

Create `SafeJSONParse` (see `custom-functions/SafeJSONParse.txt`)

### 3. Import the Script

Copy `scripts/saveText_v3.8.txt` into your solution and update field references.

### 4. Add the WebViewer

Configure with the HTML from `webviewer/QuillEditor_v4.html`

**Detailed instructions:** See [INSTALLATION.md](docs/INSTALLATION.md)

---

## 📁 Repository Structure

```
fm-webdirect-richtext/
├── README.md                    # This file
├── CHANGELOG.md                 # Version history
├── LICENSE                      # MIT License
├── INSTALLATION.md              # Detailed setup guide
├── scripts/
│   └── saveText_v3.8.txt        # FileMaker script
├── webviewer/
│   └── QuillEditor_v4.html      # WebViewer HTML/JS
├── custom-functions/
│   └── SafeJSONParse.txt        # Required custom function
└── docs/
    ├── ARCHITECTURE.md          # Technical deep-dive
    ├── TROUBLESHOOTING.md       # Common issues
    └── DELTA_FORMAT.md          # Quill Delta JSON reference
```

---

## 🔧 How It Works

The `saveText` script iterates through Quill's JSON Delta format and applies FileMaker's native text functions:

| Quill Attribute | FileMaker Function |
|-----------------|-------------------|
| `bold: true` | `TextStyleAdd($text ; Bold)` |
| `color: "#FF0000"` | `TextColor($text ; RGB(255,0,0))` |
| `font: "georgia"` | `TextFont($text ; "Georgia")` |
| `size: "24px"` | `TextSize($text ; 24)` |
| `list: "ordered"` | Prefix with `1. ` (auto-numbered) |

For the full technical breakdown, see [ARCHITECTURE.md](docs/ARCHITECTURE.md)

---

## 🎯 A Victory for the Platform

This project demonstrates the incredible flexibility of Claris FileMaker. By combining a **WebViewer for UI** and **native Scripting for logic**, we can solve complex challenges without external plugins or dependencies.

The platform gave us all the tools we needed:
- `FileMaker.PerformScript()` for WebViewer-to-script communication
- `TextStyleAdd()`, `TextColor()`, `TextFont()`, `TextSize()` for native formatting
- JSON functions for parsing complex data structures

We're sharing this as an **open-source resource** to help the community deliver even better solutions to their clients.

---

## 📝 Known Considerations

| Consideration | Context |
|---------------|---------|
| No inline images | FileMaker text fields are optimized for text; use container fields for images |
| No text alignment | Paragraph alignment handled at layout level |
| No background colors | FileMaker's text functions focus on foreground styling |

These reflect platform design choices, not limitations of this approach.

---

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Submit a Pull Request

---

## 📜 License

MIT License — Use freely in your commercial and personal projects.
- see the [LICENSE](LICENSE) file for details.
---

## 🙏 Acknowledgments

This solution was developed through collaborative effort:

- **Dimitris Kokoutsidis** ([Axelar](https://axelar.eu)) — Architecture and domain expertise
- **Claude Opues 4.5 (Anthropic)** — Script development and documentation
- **Gemini 3 Pro (Google)** — Analysis and validation

Thanks to the FileMaker community for decades of knowledge sharing, and to Claris for building a platform flexible enough to make solutions like this possible.

---

## 📞 Resources

- **Documentation**: [docs/](docs/)
- **Author**: [axelar.eu](https://axelar.eu)

---

*Helping FileMaker developers say "Yes" to rich text in WebDirect.*
