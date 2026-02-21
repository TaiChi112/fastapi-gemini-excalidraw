# sanitize_elements.py - Documentation

## File Location
📁 `gemini_excalidraw/sanitize_elements.py`

---

## 📌 Overview

**Purpose:** "Cleaner" ของระบบ - แก้ไข validate และทำให้ elements ที่ได้จาก Gemini AI เป็นไปตามมาตรฐาน Excalidraw

นี่คือ element post-processor ที่ทำงานหลังจาก Gemini generate JSON มาแล้ว

---

## 🎯 Core Responsibilities

### 1. Default Field Injection
เติม fields ที่ขาดหาย ให้กับทุก element

```python
BASE_DEFAULTS = {
    "angle": 0,
    "seed": lambda: random.randint(1, 999999),
    "version": 1,
    "versionNonce": lambda: random.randint(1, 999999),
    "isDeleted": False,
    "groupIds": [],
    "boundElements": [],
    "updated": lambda: int(time.time() * 1000),
    "link": None,
    "locked": False,
    "opacity": 100,
}
```

**ทำงาน:** ถ้า element ขาด field ใดๆ ก็จะเพิ่มค่า default เข้าไป

### 2. Type-Specific Defaults
ให้ค่า default เฉพาะสำหรับแต่ละ shape type

```python
TYPE_DEFAULTS = {
    "rectangle": {
        "strokeColor": "#1e1e1e",
        "backgroundColor": "transparent",
        "fillStyle": "solid",
        "strokeWidth": 2,
        "strokeStyle": "solid",
        "roughness": 1,
        "roundness": None,
    },
    "arrow": {
        "strokeColor": "#1e1e1e",
        "points": [[0, 0], [100, 0]],
        "endArrowhead": "arrow",
        ...
    },
    "text": {
        "fontSize": 16,
        "fontFamily": 1,
        "textAlign": "left",
        "verticalAlign": "top",
        "originalText": "",
        "lineHeight": 1.25,
        "autoResize": True,
    },
    ...
}
```

### 3. Problem Fixing

#### Fix 1: Negative Width/Height
Gemini บางครั้งสร้าง arrow/line ด้วย width/height ติดลบ

```python
if w < 0 or h < 0:
    el["x"] = el.get("x", 0) + w      # ย้าย x ให้ชดเชย
    el["y"] = el.get("y", 0) + h      # ย้าย y ให้ชดเชย
    el["width"] = abs(w)              # ทำให้เป็นบวก
    el["height"] = abs(h)             # ทำให้เป็นบวก
    el["points"] = [[0, 0], [abs(w), abs(h)]]
```

**ผลลัพธ์:** Elements ไม่เคยมี negative dimensions

#### Fix 2: Arrow Lifeline Detection
ตรวจหา arrows ที่ควรจะเป็น "line" (vertical lifelines)

```python
if el_type == "arrow" and el.get("height", 0) > el.get("width", 0):
    # Tall arrow ที่ height > width → ควรเป็น line
    el["type"] = "line"
    el["endArrowhead"] = None
    el["startArrowhead"] = None
    h = el.get("height", 300)
    el["points"] = [[0, 0], [0, h]]
```

**ผลลัพธ์:** Sequence diagram lifelines สร้างถูกต้อง

#### Fix 3: Points Array Sync
ตรวจสอบว่า points array ตรงกับ width/height

```python
if el_type in ("arrow", "line"):
    w = el.get("width", 0)
    h = el.get("height", 0)
    current_pts = el.get("points", [])
    if not current_pts or current_pts[-1] != [w, h]:
        el["points"] = [[0, 0], [w, h]]
```

**ผลลัพธ์:** Arrows และ lines เสมอมี points ที่ถูกต้อง

#### Fix 4: Text Original Text Sync
ตรวจสอบว่า "originalText" เท่ากับ "text"

```python
if el_type == "text":
    el["originalText"] = el.get("text", "")
```

**ผลลัพธ์:** Excalidraw ไม่ complain เรื่อง text mismatch

#### Fix 5: Remove Invalid Labels
ลบ "label" field จาก shapes (ต้องเป็น text element แยก)

```python
if el_type == "rectangle" and "label" in el:
    del el["label"]
```

**ผลลัพธ์:** Shapes ไม่มี inline labels ที่ invalid

---

## 🔍 Key Components

### A. BASE_DEFAULTS

**11 Universal Fields:**
| Field           | Data Type | Default Value | Purpose              |
| --------------- | --------- | ------------- | -------------------- |
| `angle`         | int       | 0             | Rotation angle       |
| `seed`          | int       | random        | Styling seed         |
| `version`       | int       | 1             | Schema version       |
| `versionNonce`  | int       | random        | Change tracker       |
| `isDeleted`     | bool      | False         | Deletion status      |
| `groupIds`      | array     | []            | Group membership     |
| `boundElements` | array     | []            | Connected elements   |
| `updated`       | int       | timestamp     | Last update time     |
| `link`          | null/str  | null          | URL link             |
| `locked`        | bool      | False         | Lock state           |
| `opacity`       | int       | 100           | Transparency (0-100) |

### B. TYPE_DEFAULTS by Shape

**Rectangle:**
```python
{
    "strokeColor": "#1e1e1e",
    "backgroundColor": "transparent",
    "fillStyle": "solid",
    "strokeWidth": 2,
    "strokeStyle": "solid",
    "roughness": 1,
    "roundness": None,
}
```

**Arrow:**
```python
{
    "strokeColor": "#1e1e1e",
    "points": [[0, 0], [100, 0]],
    "lastCommittedPoint": None,
    "startBinding": None,
    "endBinding": None,
    "startArrowhead": None,
    "endArrowhead": "arrow",
    "elbowed": False,
}
```

**Line:**
```python
{
    "strokeColor": "#1e1e1e",
    "points": [[0, 0], [0, 300]],
    "startArrowhead": None,
    "endArrowhead": None,
    # ไม่มี arrowhead เลย (ต่างจาก arrow)
}
```

**Text:**
```python
{
    "strokeColor": "#1e1e1e",
    "fontSize": 16,
    "fontFamily": 1,
    "textAlign": "left",
    "verticalAlign": "top",
    "containerId": None,
    "originalText": "",
    "lineHeight": 1.25,
    "autoResize": True,
}
```

### C. Fix Functions

#### `sanitize_element(el: dict) -> dict`

**Input:** Single element as dict
**Process:**
1. เติม BASE_DEFAULTS
2. เติม TYPE_DEFAULTS ตามประเภท
3. แก้ negative dimensions
4. ตรวจสอบ points arrays
5. ทำให้ originalText ตรงกับ text
6. ลบ invalid labels

**Output:** Fixed & complete element dict

#### `sanitize_elements(elements: list) -> list`

**Input:** List of raw elements from Gemini
**Process:** Apply `sanitize_element()` กับแต่ละตัว
**Output:** List of sanitized elements

#### `fix_elements(elements: list) -> list`

**Input:** Already-sanitized elements
**Process:** Apply fix logic สำหรับ:
- Arrow → Line conversion
- Points sync
- Text sync

**Output:** Fixed elements

---

## 📊 Data Flow

```
Gemini API Response (raw JSON)
         ↓
    [Element 1: might have incomplete fields,
     Element 2: might have negative width,
     Element 3: arrow instead of line,
     ...]
         ↓
  sanitize_elements()
         ↓
    [Element 1: all fields filled,
     Element 2: width/height fixed,
     Element 3: type corrected,
     ...]
         ↓
   fix_elements()
         ↓
    [Element 1: points synced,
     Element 2: text synced,
     Element 3: complete,
     ...]
         ↓
Frontend Rendering
```

---

## 🔗 Public API

### Functions

#### 1. `sanitize_element(el: dict) -> dict`

**Input:**
```python
{
    "id": "el1",
    "type": "rectangle",
    "x": 100,
    "y": 100,
    "width": 160,
    "height": 60
    # Missing: angle, seed, colors, etc.
}
```

**Process:**
- Check element type
- Add missing fields from BASE_DEFAULTS
- Add type-specific defaults
- Fix negative dimensions
- Handle points arrays

**Output:**
```python
{
    "id": "el1",
    "type": "rectangle",
    "x": 100,
    "y": 100,
    "width": 160,
    "height": 60,
    "angle": 0,
    "seed": 123456,
    "version": 1,
    "versionNonce": 654321,
    "isDeleted": False,
    "groupIds": [],
    "boundElements": [],
    "updated": 1708457800000,
    "link": None,
    "locked": False,
    "opacity": 100,
    "strokeColor": "#1e1e1e",
    "backgroundColor": "transparent",
    "fillStyle": "solid",
    "strokeWidth": 2,
    "strokeStyle": "solid",
    "roughness": 1,
    "roundness": None
}
```

#### 2. `sanitize_elements(elements: list) -> list`

**Input:** List of 10 incomplete elements
**Output:** List of 10 complete elements
**Time:** O(n) - linear

#### 3. `fix_elements(elements: list) -> list`

**Input:** Sanitized list
**Output:** Fixed list
**Handles:**
- Lifeline detection
- Points synchronization
- Text field synchronization

---

## 🛠️ Problem Fixing Examples

### Example 1: Negative Width Arrow
```python
# Input from Gemini (BAD)
{
    "id": "ar1",
    "type": "arrow",
    "x": 300,
    "y": 200,
    "width": -160,  # ❌ Negative!
    "height": 0
}

# After fix_elements() (GOOD)
{
    "id": "ar1",
    "type": "arrow",
    "x": 140,        # x moved left by 160
    "y": 200,
    "width": 160,    # ✅ Now positive
    "height": 0,
    "points": [[0, 0], [160, 0]]
}
```

### Example 2: Arrow Should Be Line
```python
# Input from Gemini (might work, but wrong type)
{
    "id": "ar2",
    "type": "arrow",
    "x": 100,
    "y": 100,
    "width": 0,      # Very thin
    "height": 300,   # Very tall → should be line!
    "endArrowhead": "arrow"
}

# After fix_elements() (CORRECT)
{
    "id": "ar2",
    "type": "line",     # ✅ Changed to line
    "x": 100,
    "y": 100,
    "width": 0,
    "height": 300,
    "points": [[0, 0], [0, 300]],
    "endArrowhead": None,      # ✅ No arrowhead
    "startArrowhead": None
}
```

### Example 3: Missing Fields
```python
# Input from Gemini (INCOMPLETE)
{
    "id": "el1",
    "type": "rectangle",
    "x": 100,
    "y": 100,
    "width": 160,
    "height": 60,
    "text": "Button"
}

# After sanitize_element() (COMPLETE)
{
    "id": "el1",
    "type": "rectangle",
    "x": 100,
    "y": 100,
    "width": 160,
    "height": 60,
    "angle": 0,
    "seed": 456789,
    "version": 1,
    "versionNonce": 234567,
    "isDeleted": False,
    "groupIds": [],
    "boundElements": [],
    "updated": 1708457900000,
    "link": None,
    "locked": False,
    "opacity": 100,
    "strokeColor": "#1e1e1e",
    "backgroundColor": "transparent",
    "fillStyle": "solid",
    "strokeWidth": 2,
    "strokeStyle": "solid",
    "roughness": 1,
    "roundness": None,
    "text": "Button"
}
```

---

## 📈 Current Capabilities

### ✅ Fixes Applied
1. ✅ Missing field injection
2. ✅ Negative dimension correction
3. ✅ Arrow ↔ Line type detection
4. ✅ Points array synchronization
5. ✅ Text field duplication
6. ✅ Invalid label removal

### ✅ Supported Shape Types
- rectangle
- ellipse
- diamond
- arrow
- line
- text
- image

---

## ⚙️ Configuration & Customization

### Add New Type-Specific Defaults
```python
TYPE_DEFAULTS["new_shape"] = {
    "strokeColor": "#1e1e1e",
    "backgroundColor": "transparent",
    "fillStyle": "solid",
    "strokeWidth": 2,
    ...
}
```

### Change Default Colors
Edit color hex codes:
```python
TYPE_DEFAULTS["rectangle"]["strokeColor"] = "#ff0000"  # Changed to red
```

### Add New Fix Logic
Add to `fix_elements()` function:
```python
if el_type == "new_shape":
    # Custom fix logic
    el["custom_field"] = value
```

---

## 🐛 Common Issues & Solutions

| Issue                     | Cause               | Solution                       |
| ------------------------- | ------------------- | ------------------------------ |
| Elements very small       | dimensions=0        | Check width/height calculation |
| Text not showing          | originalText ≠ text | Already fixed by sanitize      |
| Arrows pointing wrong way | negative width      | Already fixed by fix_elements  |
| Colors missing            | no backgroundColor  | Added by TYPE_DEFAULTS         |
| Points don't match size   | points not synced   | Already fixed by fix_elements  |

---

## 🔍 Dependencies

- **Built-in:** `time`, `random`
- **Used By:** `gemini_to_excalidraw_no_mcp.py`
- **Uses:** None (self-contained)

---

## 📊 Code Statistics

- **Total Lines:** 147
- **Functions:** 3 (sanitize_element, sanitize_elements, fix_elements)
- **Default Values:** 11 (BASE_DEFAULTS) + 5 (TYPE_DEFAULTS per type)
- **Fix Logic Blocks:** 5 major fixes
- **Shape Types Supported:** 7

---

## 🎯 Integration

### Input Source
- **From:** `gemini_to_excalidraw_no_mcp.py::generate_diagram()`
- **Contents:** JSON array from Gemini

### Output Destination
- **To:** `main.py` and frontend
- **Format:** Clean Excalidraw elements array

---

## 📝 Notes

- Always sanitize BEFORE fix_elements
- Functions are stateless (no side effects)
- All operations are non-destructive (don't lose data, only add/correct)
- Timestamps use milliseconds (standard for JavaScript)
- Random seeds ensure consistent styling per element

---

**Last Updated:** February 21, 2026
**Version:** 1.0.0
**Maintainer:** FastAPI Gemini Excalidraw Team
