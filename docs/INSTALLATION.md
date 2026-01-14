# Installation Guide

This guide walks you through setting up the FileMaker WebDirect Rich Text Editor in your solution.

---

## Prerequisites

- FileMaker Pro 19 or later (for development)
- FileMaker Server 19 or later (for WebDirect hosting)
- Basic familiarity with FileMaker scripting and calculations

---

## Step 1: Create the Database Structure

### 1.1 Create or Select a Table

Choose an existing table or create a new one to store the rich text content.

### 1.2 Add Required Fields

| Field Name | Type | Options | Purpose |
|------------|------|---------|---------|
| `yourNativeField` | Text | — | Stores the native FileMaker rich text |
| `yourHTML` | Text | — | Stores HTML for WebViewer persistence |

> **Note:** You can rename these fields, but you'll need to update the script references accordingly.

---

## Step 2: Add the Custom Function

### 2.1 Open Manage Custom Functions

1. Go to **File → Manage → Custom Functions**
2. Click **New**

### 2.2 Create SafeJSONParse

**Function Name:** `SafeJSONParse`

**Parameters:** `json`

**Calculation:**
```
Let ( 
  [ 
    // Use your proven version detection logic - variation with < 22
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

**Description:**
> Cross-version JSON parse wrapper for FileMaker 19+. Handles edge cases where JSONGetElement returns error markers.

3. Click **OK** to save

---

## Step 3: Import the Script

### 3.1 Create the Script

1. Go to **Scripts → Manage Scripts**
2. Click **New Script**
3. Name it exactly: `saveText`

### 3.2 Copy the Script Content

1. Open [`[saveText_v3.8.txt)`](https://github.com/DimitrisKok/WebDirect-Rich-Text-Editor/blob/main/scripts/saveText_v3.8.txt) from this repository
2. Copy the entire content
3. In FileMaker:
   - Click in the script workspace
   - Press **Cmd+A** (Mac) or **Ctrl+A** (Windows) to select all
   - Paste the copied content

### 3.3 Update Field References

Find and replace these placeholders with your actual table and field names:

| Find | Replace With |
|------|--------------|
| `yourTable::yourNativeField` | Your actual field reference |
| `yourTable::yourHTML` | Your actual HTML field reference |

### 3.4 Save the Script

Press **Cmd+S** or click **Save**

---

## Step 4: Set Up the Layout

### 4.1 Create or Modify a Layout

1. Open the layout where you want the rich text editor
2. Enter Layout Mode

### 4.2 Add the WebViewer

1. Insert → Web Viewer
2. Draw the WebViewer at your desired size
3. In the Web Viewer Setup dialog:
   - **Name:** `richTextEditor` (or your preference)
   - **Web Address:** (see calculation below)
   - Check: **Allow interaction with web viewer content**
   - Check: **Allow JavaScript to perform FileMaker scripts**

### 4.3 Set the Web Address Calculation

For the Web Address, use this calculation pattern:

```
If ( IsEmpty ( yourTable::yourHTML ) ;
  /* Default: Load empty editor */
  "data:text/html,<!DOCTYPE html><html>..." ;
  /* Existing: Load saved HTML */
  "data:text/html," & yourTable::yourHTML
)
```

**Full Default HTML:**

Copy the entire content of [`QuillEditor_v4.html`](https://github.com/DimitrisKok/WebDirect-Rich-Text-Editor/blob/main/webviewer/QuillEditor_v4.html) and:
1. Replace all double quotes with escaped quotes (`\"`)
2. Remove line breaks or join with `& ¶ &`
3. Wrap in quotes

Or use this simplified approach:

```
Let ([
  defaultHTML = 
    "<!DOCTYPE html><html><head>..." // (paste minified HTML here)
] ;
  If ( IsEmpty ( yourTable::yourHTML ) ;
    "data:text/html," & defaultHTML ;
    "data:text/html," & yourTable::yourHTML
  )
)
```

### 4.4 Add the Native Text Field

1. Add a field for `yourNativeField` to display the result
2. Position it next to or below the WebViewer
3. (Optional) Set the field to not allow entry in Browse mode if you want it read-only

---

## Step 5: Test Your Setup

### 5.1 In FileMaker Pro

1. Switch to Browse mode
2. Click in the WebViewer editor
3. Type some text
4. Apply formatting (bold, color, etc.)
5. Verify the native field updates

### 5.2 In WebDirect

1. Host your file on FileMaker Server
2. Access via WebDirect
3. Repeat the same test
4. Verify formatting persists after page refresh

---

## Step 6: Optional Enhancements

### 6.1 Add a Refresh Button

Create a button that reloads the WebViewer with saved HTML:

**Script:**
```
Set Web Viewer [ Object Name: "richTextEditor" ; 
  URL: "data:text/html," & yourTable::yourHTML ]
```

### 6.2 Add a Print-Friendly Layout

The native text field prints as vector graphics. Create a print layout that:
1. Shows only `yourNativeField`
2. Hides the WebViewer
3. Uses appropriate margins and fonts

### 6.3 Set Default Formatting

To set a default font/size for new records, use an Auto-Enter calculation on `yourNativeField`:

```
TextSize ( TextFont ( "" ; "Arial" ) ; 12 )
```

---

## Troubleshooting

### Issue: Script not found error

**Cause:** The script name doesn't match the JavaScript call.

**Fix:** Ensure your script is named exactly `saveText` (case-sensitive).

### Issue: WebViewer shows error

**Cause:** HTML encoding issue or missing Quill library.

**Fix:** 
1. Check your internet connection (Quill loads from CDN)
2. Verify HTML escaping in the Web Address calculation

### Issue: Formatting doesn't persist

**Cause:** HTML field not saving properly.

**Fix:**
1. Verify `yourHTML` field exists and is accessible
2. Check for field validation rules that might block saving
3. Review the script for errors (check `$$debug` variable)

### Issue: Giant text / 1111px bug

**Cause:** Using an older script version.

**Fix:** Ensure you're using v3.8 or later of the `saveText` script.

---

## File Checklist

Before going live, verify you have:

- [ ] `SafeJSONParse` custom function created
- [ ] `saveText` script imported and field references updated
- [ ] `yourNativeField` text field created
- [ ] `yourHTML` text field created
- [ ] WebViewer configured with correct URL calculation
- [ ] WebViewer allows JavaScript to perform FileMaker scripts
- [ ] Tested in both FileMaker Pro and WebDirect

---

## Need Help?

- Check the [Troubleshooting Guide](./TROUBLESHOOTING.md)
- Review the [Architecture Documentation](./ARCHITECTURE.md)

