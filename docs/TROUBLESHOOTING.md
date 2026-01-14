# Troubleshooting Guide

Common issues and their solutions for the FileMaker WebDirect Rich Text Editor.

---

## Quick Diagnosis

Before diving into specific issues, check these globals in the Data Viewer:

| Variable | What to Look For |
|----------|------------------|
| `$$debug` | Does it contain valid JSON? |
| `$$debugFinalText` | Is the content correct? |
| `$$debugCSS` | What styles are being applied? |

---

## Common Issues

### 1. Script Not Found Error

**Symptom:** WebViewer shows "Script not found" or nothing happens when typing.

**Cause:** The script name doesn't match the JavaScript call.

**Solution:**
1. Verify your script is named exactly `saveText` (case-sensitive)
2. Check that the script is not in a folder that restricts access
3. In WebDirect, verify "Allow JavaScript to perform FileMaker scripts" is enabled

---

### 2. WebViewer Shows Blank or Error

**Symptom:** The editor doesn't load, or shows an error page.

**Causes & Solutions:**

**A. No Internet Connection**
- Quill.js loads from CDN: `cdn.jsdelivr.net`
- Check network connectivity
- Consider bundling Quill locally for offline use

**B. HTML Encoding Error**
- Check the Web Address calculation for proper escaping
- All double quotes must be escaped as `\"`
- Special characters may need URL encoding

**C. CSP Restriction (WebDirect)**
- Some server configurations block external scripts
- Check FileMaker Server's web configuration

---

### 3. Formatting Not Saving

**Symptom:** You type and format, but the native field stays empty.

**Cause:** Script error or field access issue.

**Solution:**
1. Check `$$debug` - is JSON being received?
2. Check field permissions - can the script write to the fields?
3. Check for field validation rules blocking the save
4. Ensure `Commit Records/Requests` isn't being blocked

---

### 4. Formatting Not Persisting After Refresh

**Symptom:** Content saves, but formatting is lost on page refresh.

**Cause:** HTML field not being used to reload the WebViewer.

**Solution:**
1. Verify `yourHTML` field is being populated
2. Check your Web Address calculation includes the conditional load:
   ```
   If ( IsEmpty ( yourTable::yourHTML ) ;
     "data:text/html," & defaultHTML ;
     "data:text/html," & yourTable::yourHTML
   )
   ```
3. In WebDirect, verify the layout refreshes on record load

---

### 5. Giant Text / 1111px Bug

**Symptom:** Plain text renders at enormous size.

**Cause:** Using an older script version with styled text concatenation bug.

**Solution:**
1. Update to script version 3.7 or later
2. The fix: Use inline styled CR instead of `$styledCR` variable
3. Verify Step 4.0 (Emergency Flush) re-applies base styling

---

### 6. List Markers on Wrong Lines

**Symptom:** "1." appears before text that isn't part of the list.

**Cause:** Buffer contains multiple lines when list marker is applied.

**Solution:**
1. Update to script version 3.8 or later
2. The fix adds this check in CASE A:
   ```
   If [ PatternCount ( $Buffer ; ¶ ) > 0 ]
       // Split buffer before applying marker
   End If
   ```

---

### 7. Colors Not Working

**Symptom:** Color picker works in editor but native field shows black text.

**Cause:** Variable name mismatch in script (v3.5 bug).

**Solution:**
1. Update to script version 3.6 or later
2. Verify color parsing uses `$chunk` (CASE C/D) or `$textPart` (CASE B)
3. Check `$$debugCSS` to see what color is being applied

---

### 8. Fonts Not Working

**Symptom:** Font dropdown changes editor view but native field stays default.

**Cause:** Incomplete font mapping or variable name error.

**Solution:**
1. Verify full 10-font mapping exists in script
2. Check font names match exactly (e.g., "Times New Roman" not "Times")
3. Ensure font is available on the server for WebDirect

---

### 9. Sizes Not Working

**Symptom:** Size changes in editor but native field stays 12pt.

**Cause:** Missing numeric/px size parsing.

**Solution:**
1. Verify size handling includes both named sizes AND px parsing:
   ```
   Else If [ PatternCount ( $sizeVal ; "px" ) > 0 ]
       Set Variable [ $sizeNum ; Value: Filter ( $sizeVal ; "0123456789" ) ]
       ...
   ```

---

### 10. WebViewer Works in Pro, Fails in WebDirect

**Symptom:** Everything works in FileMaker Pro but not WebDirect.

**Causes & Solutions:**

**A. JavaScript Permissions**
- Check: WebViewer object → "Allow JavaScript to perform FileMaker scripts"

**B. Server Security Settings**
- FileMaker Server admin console → Web Publishing → Security

**C. Browser Console Errors**
- Open browser Developer Tools (F12)
- Check Console tab for JavaScript errors

**D. Cross-Origin Issues**
- If loading external resources, check for CORS errors

---

## Debugging Steps

### Step 1: Check JSON Payload

Add to your script or check in Data Viewer:
```
$$debug = Get(ScriptParameter)
```

Valid payload looks like:
```json
{"delta":{"ops":[...]},"html":"<p>...</p>","plain":"..."}
```

### Step 2: Check Intermediate Values

Add debug variables throughout the script:
```
$$debug1 = "After normalize: " & GetAsCSS($chunk)
$$debug2 = "After styles: " & GetAsCSS($chunk)
$$debug3 = "Buffer: " & GetAsCSS($Buffer)
$$debug4 = "FinalText: " & GetAsCSS($FinalText)
```

### Step 3: Check Output CSS

After the script runs:
```
GetAsCSS ( yourTable::yourNativeField )
```

Look for unexpected values like `font-size: 1111px` or missing attributes.

### Step 4: Test Minimal Case

Type just "Hello" (no formatting) and check:
1. JSON payload contains `{"insert":"Hello\n"}`
2. Native field shows "Hello" in Arial 12
3. No extra formatting or corruption

---

## Browser-Specific Issues

### Chrome

Usually works without issues. If problems:
- Check for extension conflicts (disable extensions)
- Clear cache and reload

### Safari

- May have stricter CSP enforcement
- Check for "blocked content" warnings
- Enable "Allow JavaScript from Apple Events" if using Pro

### Firefox

- Check for enhanced tracking protection blocking CDN
- Whitelist `cdn.jsdelivr.net` if needed

### Edge

- Similar to Chrome (Chromium-based)
- Check IE Mode isn't accidentally enabled

---

## Server Configuration Issues

### FileMaker Server Web Publishing

1. Admin Console → Web Publishing
2. Verify WebDirect is enabled
3. Check concurrent session limits
4. Review security settings

### Firewall / Proxy

If CDN is blocked:
- Whitelist `cdn.jsdelivr.net`
- Or bundle Quill.js locally

### SSL/TLS

- WebDirect should use HTTPS
- Mixed content (HTTP in HTTPS) will be blocked
- Quill CDN supports HTTPS

---

## Getting More Help

If none of the above solves your issue:

1. **Check the GitHub Issues** - Someone may have encountered the same problem
2. **Open a New Issue** - Include:
   - FileMaker Pro/Server version
   - Browser and version
   - Content of `$$debug` variable
   - Content of `$$debugCSS` variable
   - Steps to reproduce
3. **FileMaker Community** - Post on [community.claris.co](https://community.claris.com/en/s/) with link to this repo

---

## Version Compatibility

| FileMaker Version | Compatible? | Notes |
|-------------------|-------------|-------|
| 19.x | ✅ Yes | Full support |
| 20.x | ✅ Yes | Full support |
| 21.x | ✅ Yes | Full support |
| 18.x | ⚠️ Partial | May need JSON function adjustments |
| < 18 | ❌ No | Missing required JSON functions |
