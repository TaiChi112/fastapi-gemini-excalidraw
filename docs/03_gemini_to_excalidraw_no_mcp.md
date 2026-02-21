# gemini_to_excalidraw_no_mcp.py - Documentation

## File Location
📁 `gemini_excalidraw/gemini_to_excalidraw_no_mcp.py`

---

## 📌 Overview

**Purpose:** "Orchestrator" ของระบบ - ประสานงานระหว่าง FastAPI, Gemini AI, และ Element Sanitizer

นี่คือตัวกลางที่:
1. รับ user prompt
2. ตรวจหา diagram type
3. เรียก Gemini API
4. Sanitize ผลลัพธ์
5. ส่งกลับไปยัง frontend

---

## 🎯 Core Responsibilities

### 1. Configuration Management
```python
load_dotenv()
GEMINI_API_KEY = os.getenv("GEMINI_API_KEY", "YOUR_GEMINI_API_KEY_HERE")
GEMINI_MODEL = os.getenv("GEMINI_MODEL", "gemini-2.0-flash")

# Initialize the Gemini Client with API key
client = None
if GEMINI_API_KEY and GEMINI_API_KEY != "YOUR_GEMINI_API_KEY_HERE":
    client = genai.Client(api_key=GEMINI_API_KEY)
else:
    print("⚠️  Warning: GEMINI_API_KEY not set. Will use mock data.")
```

**ทำงาน:** โหลด environment variables และ initialize Gemini client

### 2. Async Diagram Generation (Web API Mode)
```python
async def generate_diagram(prompt: str, system_prompt: str) -> dict:
    # Check if API is configured
    # Call Gemini API asynchronously
    # Parse JSON response
    # Sanitize elements
    # Return to frontend
```

**ทำงาน:** Function หลักสำหรับ FastAPI endpoint

### 3. Synchronous Diagram Generation (CLI Mode)
```python
def generate_elements(user_prompt: str, system_prompt: str) -> list:
    # For CLI usage (non-async)
    # Call Gemini API synchronously
    # Returns list of elements
```

**ทำงาน:** Sync version สำหรับ command-line interface

### 4. Mock Data Fallback
```python
def mock_elements():
    # Load from arch.excalidraw file
    # Return as fallback when API fails or not configured
```

**ทำงาน:** ให้ demo ตัวอย่างเมื่อไม่มี API key

### 5. CLI Entry Point
```python
def main():
    # Parse command-line arguments
    # Get user prompt
    # Detect diagram type
    # Generate diagram
    # Save to file
```

**ทำงาน:** Command-line interface สำหรับทดสอบ

---

## 🔍 Key Components

### A. Configuration Section

**Environment Variables:**
```python
GEMINI_API_KEY     # Required: Your Google Gemini API key
GEMINI_MODEL       # Optional: Model name (default: gemini-2.0-flash)
```

**Client Initialization:**
```python
client = genai.Client(api_key=GEMINI_API_KEY)
# This client is used for all async calls
```

### B. Function: generate_diagram() [ASYNC]

**Purpose:** Main web API function

**Input:**
```python
prompt: str           # User's diagram description
system_prompt: str    # System prompt from excalidraw_rules
```

**Output:**
```python
{
    "elements": [
        {element1},
        {element2},
        ...
    ],
    "error": "optional error message"
}
```

**Process Flow:**

```
1. Check if client is initialized
   ├─ NO → Use mock_elements()
   └─ YES → Continue

2. Call Gemini asynchronously
   └─ await client.aio.models.generate_content()

3. Clean JSON response
   ├─ Remove ```json``` markdown
   ├─ Remove trailing ```
   └─ Strip whitespace

4. Parse JSON
   ├─ Extract array of elements
   ├─ Catch JSONDecodeError → use mock
   └─ Catch other errors → use mock

5. Sanitize elements
   ├─ sanitize_elements(elements)
   └─ fix_elements(elements)

6. Return to caller
   └─ {"elements": cleaned_elements}
```

**Error Handling:**
```python
try:
    # Main flow
except json.JSONDecodeError as e:
    # JSON parse error → mock fallback
except Exception as e:
    # Other errors → mock fallback
```

### C. Function: generate_elements() [SYNC]

**Use Case:** CLI/testing only

**Input:**
```python
user_prompt: str      # Description
system_prompt: str    # Rules from excalidraw_rules
```

**Output:**
```python
list  # Raw elements from Gemini (before sanitization)
```

**Difference from async:**
- Uses `client.models.generate_content()` (sync)
- Used for CLI tool, not web API
- Doesn't apply sanitization (caller does)

**Example:**
```python
elements = generate_elements(
    "Create a flowchart for login",
    system_prompt
)
elements = sanitize_elements(elements)  # Caller sanitizes
```

### D. Function: mock_elements()

**Purpose:** Fallback data when API unavailable

**Implementation:**
```python
def mock_elements():
    with open("arch.excalidraw", "r") as f:
        data = json.load(f)
        elements = data["elements"]
    return elements
```

**Returns:** List of pre-created diagram elements from `arch.excalidraw`

**Use Cases:**
- API key not configured
- Gemini API call fails
- JSON parsing error
- Network error

---

## 📊 Data Flow

### Async Flow (Web API)

```
FastAPI Request
    ↓
/generate_excalidraw endpoint (main.py)
    ↓
generate_diagram(prompt, system_prompt)
    ↓
├─ [IF no API key]
│  └─ mock_elements()
│
└─ [IF API key exists]
   ├─ client.aio.models.generate_content()
   │  └─ Gemini API returns JSON
   ├─ Clean JSON (remove markdown)
   ├─ json.loads() parse
   ├─ sanitize_elements()
   ├─ fix_elements()
   └─ Return {"elements": [...]}
    ↓
FastAPI Response
    ↓
Frontend (index.html)
    ↓
Excalidraw Canvas renders shapes
```

### Sync Flow (CLI)

```
Command Line: python main.py
    ↓
main() function
    ↓
detect_diagram_type(prompt)
    ↓
get_system_prompt(diagram_type)
    ↓
generate_elements(prompt, system_prompt)
    ↓
client.models.generate_content() [SYNC]
    ↓
json.loads() parse
    ↓
sanitize_elements()
    ↓
fix_elements()
    ↓
Write to arch.excalidraw file
```

---

## 🔗 Public API

### Async Functions

#### 1. `async generate_diagram(prompt: str, system_prompt: str) -> dict`

**Input:**
```python
prompt = "Create a simple login flow"
system_prompt = """You are an Excalidraw expert...
                  [rules for diagram type]"""
```

**Process:**
```
Check API → Call Gemini → Clean JSON → Parse → Sanitize → Return
```

**Output:**
```python
{
    "elements": [
        {"id": "el1", "type": "rectangle", "x": 100, ...},
        {"id": "ar1", "type": "arrow", "x": 260, ...},
        ...
    ]
}
```

**Error Response:**
```python
{
    "elements": [mock_element1, mock_element2, ...],
    "error": "JSON parse error: ..."
}
```

**Example Usage:**
```python
from gemini_excalidraw import gemini_to_excalidraw_no_mcp
from gemini_excalidraw.excalidraw_rules import detect_diagram_type, get_system_prompt

async def handler(prompt):
    diagram_type = detect_diagram_type(prompt)
    system_prompt = get_system_prompt(diagram_type)
    result = await gemini_to_excalidraw_no_mcp.generate_diagram(
        prompt, 
        system_prompt
    )
    return result["elements"]
```

### Sync Functions

#### 2. `generate_elements(user_prompt: str, system_prompt: str) -> list`

**Input:** Same as async
**Output:** List of elements (not wrapped in dict)
**Error:** Raises exception (doesn't catch)

**Use Case:** CLI tool, manual element processing

### Utility Functions

#### 3. `mock_elements() -> list`

**Output:** Pre-canned elements from arch.excalidraw
**Error:** Raises FileNotFoundError if file missing

#### 4. `main()`

**Purpose:** CLI entry point

**Command-line Arguments:**
```bash
python -m gemini_excalidraw.gemini_to_excalidraw_no_mcp \
    --prompt "Create a flowchart" \
    --output mydiagram.excalidraw
```

**Arguments:**
- `--prompt, -p` - Diagram description
- `--session, -s` - Session name (unused)
- `--output, -o` - Output file (unused)

---

## 🛠️ Integration Points

### 1. Input: From main.py
```python
# main.py does this:
from gemini_excalidraw import gemini_to_excalidraw_no_mcp

gemini_to_excalidraw_no_mcp.GEMINI_API_KEY = os.getenv("GEMINI_API_KEY")
gemini_to_excalidraw_no_mcp.GEMINI_MODEL = os.getenv("GEMINI_MODEL")
gemini_to_excalidraw_no_mcp.client = genai.Client(...)

# Then calls:
await gemini_to_excalidraw_no_mcp.generate_diagram(prompt, system_prompt)
```

### 2. Input: From excalidraw_rules.py
```python
from .excalidraw_rules import detect_diagram_type, get_system_prompt

diagram_type = detect_diagram_type(prompt)
system_prompt = get_system_prompt(diagram_type)
# Pass system_prompt to generate_diagram()
```

### 3. Input: From sanitize_elements.py
```python
from .sanitize_elements import sanitize_elements, fix_elements

elements = sanitize_elements(raw_elements)
elements = fix_elements(elements)
# Done!
```

### 4. Output: To FastAPI/Frontend
```python
return {
    "elements": elements,
    "error": optional_error_message
}
```

---

## 📈 Current Capabilities

### ✅ API Integration
- ✅ Async/await support (FastAPI compatible)
- ✅ Gemini 2.0 Flash model support
- ✅ Configurable model name via environment
- ✅ API key validation

### ✅ Error Handling
- ✅ Graceful fallback to mock data
- ✅ JSON parsing error recovery
- ✅ Network error handling
- ✅ Missing API key detection

### ✅ Response Formats
- ✅ Clean Excalidraw JSON array
- ✅ Complete elements (all 11 required fields)
- ✅ Error message included when applicable

### ✅ CLI Support
- ✅ Command-line diagram generation
- ✅ File-based output
- ✅ Interactive user input

---

## ⚙️ Configuration & Customization

### Change Gemini Model
Edit .env file:
```env
GEMINI_MODEL=gemini-2.5-flash
```

Or modify code:
```python
GEMINI_MODEL = os.getenv("GEMINI_MODEL", "custom-model-name")
```

### Customize Mock Fallback
Edit mock_elements():
```python
def mock_elements():
    # Load from different file
    with open("mydiagrams/default.excalidraw", "r") as f:
        ...
```

### Add Retry Logic
Wrap generate_diagram():
```python
async def generate_diagram_with_retry(prompt, system_prompt, max_retries=3):
    for attempt in range(max_retries):
        try:
            return await generate_diagram(prompt, system_prompt)
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            await asyncio.sleep(1)  # Backoff
```

### Add Caching
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def get_system_prompt_cached(diagram_type: str) -> str:
    return get_system_prompt(diagram_type)
```

---

## 🐛 Common Issues & Solutions

| Issue                    | Cause                          | Solution                           |
| ------------------------ | ------------------------------ | ---------------------------------- |
| "GEMINI_API_KEY not set" | No .env or wrong variable      | Create .env with valid key         |
| JSON parse error         | Gemini returned malformed JSON | Review Gemini rules, adjust prompt |
| Mock data shown          | API call failed silently       | Check console logs, API status     |
| Slow response (>10s)     | Network lag or API overload    | Retry, check internet connection   |
| Invalid elements         | Sanitize step skipped          | ensure fix_elements() called       |

---

## 📋 Code Statistics

- **Total Lines:** 150+
- **Functions:** 5 (generate_diagram, generate_elements, mock_elements, main, dump_result)
- **Imports:** 10 (asyncio, argparse, json, os, re, sys, time, dotenv, genai, local modules)
- **Error Handlers:** 3 (JSONDecodeError, general Exception, missing client)
- **Async/Await:** Full async/await support for web API

---

## 🔄 Async/Await Explanation

**Why async?**
- FastAPI handles multiple concurrent requests
- Waiting for Gemini API blocks without async
- Async allows other requests to proceed while waiting

**How it works:**
```python
async def generate_diagram(...):
    response = await client.aio.models.generate_content(...)
    # ^^ Awaits API call, yields control to other tasks
```

**Calling from FastAPI:**
```python
@app.post("/generate_excalidraw")
async def endpoint(prompt: str = Form(...)):
    result = await generate_diagram(prompt, system_prompt)
    return result
```

---

## 📝 Notes

- Client is initialized once at startup
- Mock data is in `arch.excalidraw` in project root
- All Gemini calls use specified GEMINI_MODEL
- Response always includes "elements" key (never null)
- Timestamps in response use milliseconds
- Sanitization is automatic in async flow
- CLI tool doesn't auto-sanitize (responsibility on caller)

---

## 🚀 Future Enhancements

Potential improvements:
1. Add request caching by prompt hash
2. Implement retry with exponential backoff
3. Add metrics/logging for API calls
4. Support multiple Gemini models simultaneously
5. Add prompt optimization/engineering
6. Implement batch processing
7. Add webhook support for async notifications

---

**Last Updated:** February 21, 2026
**Version:** 1.0.0
**Maintainer:** FastAPI Gemini Excalidraw Team
