# excalidraw_rules.py - Documentation

## File Location
📁 `gemini_excalidraw/excalidraw_rules.py`

---

## 📌 Overview

**Purpose:** ศูนย์กลางสำหรับรูปแบบการสร้าง diagram ต่างๆ นี่คือ "brain" ของระบบ

ไฟล์นี้มีความรับผิดชอบสำคัญ 2 ประการ:
1. **UNIVERSAL RULES** - กฎการสร้าง Excalidraw JSON ที่ต้องใช้ในทุก diagram type
2. **TYPE RULES** - กฎเฉพาะของแต่ละประเภท diagram (12 ประเภท)

---

## 🎯 Core Responsibilities

### 1. Universal Rules Definition
```python
UNIVERSAL_RULES = """
  - วิธีสร้าง JSON elements ที่ถูกต้อง
  - ฟิลด์ที่ต้องมี: angle, seed, version, versionNonce, isDeleted, etc.
  - Schema ของ shape ต่างๆ: rectangle, ellipse, diamond, arrow, line, text
  - Rules สำหรับ colors, positions, dimensions
"""
```

**เนื้อหาหลัก:**
- ✅ Excalidraw elements ต้องมีฟิลด์บังคับ 11 ตัว
- ✅ เนื้อที่ width/height ต้องเป็นเลขบวกเสมอ
- ✅ ห้ามใช้ "label" ภายใน shape ต้องสร้าง text element แยก
- ✅ Arrows และ Lines ต้องมี "points" array

### 2. 12 Diagram Type Rules

เก็บไว้ใน dictionary `TYPE_RULES` โดยมี 12 ประเภท:

| #   | Type             | ชื่อย่อ  | ใช้สำหรับ                                         |
| --- | ---------------- | ----- | ---------------------------------------------- |
| 1   | **sequence**     | SEQ   | API flows, actor interactions, message passing |
| 2   | **flowchart**    | FC    | Algorithms, processes, decision flows          |
| 3   | **architecture** | ARCH  | System design, infrastructure, microservices   |
| 4   | **erd**          | ERD   | Database schemas, entity relationships         |
| 5   | **class**        | CLASS | OOP designs, UML class diagrams                |
| 6   | **state**        | STATE | State machines, FSM, event transitions         |
| 7   | **mindmap**      | MM    | Brainstorming, concept mapping, hierarchies    |
| 8   | **swimlane**     | SWL   | Cross-functional flows, responsibility mapping |
| 9   | **network**      | NET   | Network topology, infrastructure, devices      |
| 10  | **timeline**     | TL    | Roadmaps, milestones, historical events        |
| 11  | **gitflow**      | GIT   | Git branching, version control, release cycle  |
| 12  | **c4**           | C4    | C4 model, system context, architecture levels  |

---

## 🔍 Key Components

### A. UNIVERSAL_RULES Content

**Shape Element Examples:**
```json
// RECTANGLE - Basic box
{
  "id": "el1", "type": "rectangle",
  "x": 100, "y": 100, "width": 160, "height": 60,
  "angle": 0, "seed": 123456, "version": 1, 
  "versionNonce": 654321,
  "isDeleted": false, "groupIds": [], "boundElements": [],
  "updated": 1700000000000, "link": null, "locked": false,
  "strokeColor": "#333333", "backgroundColor": "#dbe9f9",
  "fillStyle": "solid", "strokeWidth": 2, "strokeStyle": "solid",
  "roughness": 1, "opacity": 100, "roundness": {"type": 3}
}

// ARROW - Connection with arrowhead
{
  "id": "ar1", "type": "arrow",
  "x": 100, "y": 200, "width": 160, "height": 0,
  "points": [[0, 0], [160, 0]],
  "startArrowhead": null, "endArrowhead": "arrow",
  ...
}

// TEXT - Label element
{
  "id": "t1", "type": "text",
  "x": 100, "y": 100, "width": 160, "height": 25,
  "text": "My Label", "originalText": "My Label",
  "fontSize": 16, "fontFamily": 1,
  ...
}
```

**Required Fields (11 ตัว):**
1. `angle` - Rotation angle (0 unless rotated)
2. `seed` - Random integer for styling
3. `version` - Version number (always 1)
4. `versionNonce` - Change tracker
5. `isDeleted` - Deletion flag (always false)
6. `groupIds` - Group membership array
7. `boundElements` - Refs to connected elements
8. `updated` - Timestamp (milliseconds)
9. `link` - URL link (null if none)
10. `locked` - Lock state (false)
11. `opacity` - Transparency (0-100)

### B. Type-Specific Rules

**Example: FLOWCHART Rules**
```
SHAPES BY NODE TYPE:
- Start/End:      ellipse, backgroundColor="#d4edda"
- Process:        rectangle roundness=3, backgroundColor="#dbe9f9"
- Decision:       diamond, backgroundColor="#fff3cd"
- Input/Output:   rectangle skewed, backgroundColor="#e8d5f5"
- Database:       ellipse squashed, backgroundColor="#fde8d8"

LAYOUT:
- Direction: top-to-bottom
- Center: x=400
- Vertical spacing: 120px
- Branching: left x≈200, right x≈600

ARROWS:
- Connect center-bottom to center-top
- Labels: YES/NO on decision branches
```

**Example: SEQUENCE DIAGRAM Rules**
```
ACTORS (top row):
- Rectangles spaced 200px apart
- Lifelines: vertical dashed lines below actors
- Messages: horizontal arrows between lifelines
- Spacing: 70px per message

CONNECTIONS:
- Forward call: solid arrow
- Return: dashed arrow
- Self-call: small loop path
```

### C. Diagram Type Detection

```python
DIAGRAM_KEYWORDS = {
    "sequence": ["sequence", "sequence diagram", "actor", "lifeline", ...],
    "flowchart": ["flowchart", "flow chart", "decision", "if else", ...],
    "architecture": ["architecture", "tier", "layer", "microservice", ...],
    ...
}

def detect_diagram_type(user_prompt: str) -> str:
    # ค้นหา keywords ใน prompt
    # คิด score สำหรับแต่ละ type
    # return type ที่ได้ score สูงสุด
    # fallback: "architecture"
```

---

## 🎨 Color Schemes

### Flowchart Colors
- **Start/End:** `#d4edda` (light green)
- **Process:** `#dbe9f9` (light blue)
- **Decision:** `#fff3cd` (light yellow)
- **Input/Output:** `#e8d5f5` (light purple)

### Architecture Colors (by tier)
- **Frontend:** `#e8f4f8` (light cyan)
- **Backend/API:** `#e8f9e8` (light green)
- **Data/Storage:** `#fff8e8` (light orange)
- **External:** `#f8e8f8` (light magenta)

### Sequence Diagram Colors
- **Actor boxes:** `#dbe9f9`
- **Return arrows:** `#888888`, dashed
- **Lifelines:** `#999999`, dashed

### Entity Relationship Colors
- **Entity header:** `#4a90d9` (darker blue)
- **Entity body:** `#ffffff` (white)
- **Relationship:** `#333333` (dark gray)

---

## 📊 Data Flow

```
user_prompt (string)
         ↓
   detect_diagram_type()
         ↓
   diagram_type (string)
         ↓
   get_system_prompt(diagram_type)
         ↓
   system_prompt (string)
         ↓
 [ใช้ไป Gemini API together with user_prompt]
```

---

## 🔗 Public API

### Functions

#### 1. `detect_diagram_type(user_prompt: str) -> str`
**Input:** User's natural language description
**Output:** Detected diagram type (e.g., "flowchart")
**Logic:**
- Tokenize prompt
- Count keyword matches per type
- Return type with highest score
- Default: "architecture"

**Example:**
```python
detect_diagram_type("Create a sequence flow for login")
# → "sequence"

detect_diagram_type("Draw a 3-layer architecture")
# → "architecture"
```

#### 2. `get_system_prompt(diagram_type: str) -> str`
**Input:** Diagram type identifier
**Output:** Complete system prompt for Gemini AI
**Content:** UNIVERSAL_RULES + TYPE_RULES[diagram_type]
**Fallback:** Uses "architecture" rules if type unknown

**Example:**
```python
prompt = get_system_prompt("flowchart")
# → Returns multi-line string with flowchart generation rules
```

#### 3. `get_all_types() -> list`
**Output:** List of all supported diagram types
**Returns:** 12 type strings

---

## 🔄 Integration Points

### Incoming Data
- **From:** `gemini_to_excalidraw_no_mcp.py::generate_diagram()`
- **Input:** `user_prompt`, system prompt needed

### Outgoing Data
- **To:** `gemini_to_excalidraw_no_mcp.py::generate_diagram()`
- **Output:** Complete system prompt → sent to Gemini API

---

## 📈 Current Capabilities

### ✅ Supported Diagram Types
1. Sequence Diagrams - Actor lifelines, messages
2. Flowcharts - Decision flows, loops
3. Architecture - Multi-tier systems
4. ER Diagrams - Database schemas
5. Class Diagrams - OOP structures
6. State Machines - FSM transitions
7. Mind Maps - Hierarchical concepts
8. Swimlanes - Cross-functional flows
9. Network Diagrams - Infrastructure topology
10. Timelines - Roadmaps, milestones
11. Git Flow - Branch strategies
12. C4 Model - Architecture context

### ✅ Shape Support
- Rectangles (with roundness control)
- Ellipses (circles, ovals)
- Diamonds (decisions)
- Arrows (connections with arrowheads)
- Lines (structural, decorative)
- Text (labels, annotations)

### ✅ Layout Features
- Customizable colors per type
- Spacing rules
- Position calculations (x, y coordinates)
- Canvas size recommendations

---

## 🚀 Usage Examples

### Example 1: Detect Flowchart
```python
from gemini_excalidraw.excalidraw_rules import detect_diagram_type, get_system_prompt

user_input = "I want to create a flowchart for my login process with two paths"
diagram_type = detect_diagram_type(user_input)
# → "flowchart"

system_prompt = get_system_prompt(diagram_type)
# → Flowchart-specific rules passed to Gemini
```

### Example 2: Generate Architecture Rules
```python
system_prompt = get_system_prompt("architecture")
# Contains rules for:
# - Tier/zone rectangles
# - Component boxes
# - Synchronous/async arrows
# - Color coding by tier
```

---

## 📋 Code Statistics

- **Total Lines:** ~815
- **Lines of Code:** ~500 (rules + logic)
- **Lines of Documentation:** ~300 (inline strings)
- **Diagram Types:** 12
- **Shape Types:** 7 (rectangle, ellipse, diamond, arrow, line, text, image)
- **Keyword Patterns:** 100+
- **Schema Definitions:** 7 (one per shape type)

---

## ⚙️ Configuration & Customization

### Add New Diagram Type
Steps to add a new diagram type:

1. **Add to DIAGRAM_KEYWORDS**
   ```python
   DIAGRAM_KEYWORDS["new_type"] = ["keyword1", "keyword2", ...]
   ```

2. **Create TYPE_RULES entry**
   ```python
   TYPE_RULES["new_type"] = """\
   ═══════════════════════════════════
   NEW TYPE RULES
   ═══════════════════════════════════
   ... detailed rules ...
   """
   ```

3. **Test detection**
   ```python
   assert detect_diagram_type("mention keyword1") == "new_type"
   ```

### Modify Colors
Edit the color hex codes in any TYPE_RULES section:
```python
backgroundColor="#dbe9f9"  # Change to desired hex
```

### Adjust Spacing
Modify pixel values in layout rules:
```
- Vertical spacing: 120px → change to desired value
- Horizontal spacing: 200px → change to desired value
```

---

## 🐛 Common Issues & Solutions

| Issue                       | Cause                | Solution                              |
| --------------------------- | -------------------- | ------------------------------------- |
| Wrong diagram type detected | Keywords not matched | Add more keywords to DIAGRAM_KEYWORDS |
| Elements misaligned         | Wrong spacing values | Review TYPE_RULES layout section      |
| Colors not matching         | Hex code typo        | Verify hex code format (#RRGGBB)      |
| Missing shape support       | Unsupported type     | Add to schema examples                |

---

## 🔍 Dependencies

- **Language:** Pure Python (no imports needed)
- **Used By:** `gemini_to_excalidraw_no_mcp.py`
- **Uses:** None (self-contained)

---

## 📝 Notes

- All shapes require ALL 11 universal fields
- Color values must be valid hex codes
- Coordinates are in pixels (Excalidraw units)
- Points arrays define shape corners/paths
- Text elements need "originalText" to match "text"
- Roundness types: 1=circle, 2=adaptive, 3=adaptive

---

**Last Updated:** February 21, 2026
**Version:** 1.0.0
**Maintainer:** FastAPI Gemini Excalidraw Team
