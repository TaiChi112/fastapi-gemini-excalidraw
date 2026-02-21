# excalidraw_rules.py - Scaling & Feature Expansion Guide

## 📈 Scaling Strategy Overview

**Current State:** 12 diagram types supported
**Vision:** Make it the most comprehensive diagram generation system

---

## 🚀 Phase 1: Near-Term Enhancements (1-3 months)

### 1.1 Add More Diagram Types (Target: 8 more types)

#### 1.1.1 Data Flow Diagram (DFD)
**Why?** Most systems need data flow visualization

**New Diagram Type:**
```python
TYPE_RULES["dfd"] = """\
═══════════════════════════════════
DATA FLOW DIAGRAM (DFD) RULES
═══════════════════════════════════
ELEMENTS:
- Process: circle with number, diameter=50, backgroundColor="#dbe9f9"
- Data Store: two parallel rectangles (top/bottom), width=160, height=80
- External Entity: rectangle with double border, backgroundColor="#f8d7da"
- Data Flow: arrow with label (data name), strokeWidth=2
- Flow Labels: describe what data moves on each arrow

LAYOUT:
- Top: external entities (data sources)
- Middle: processes and data stores
- Bottom: external entities (targets)
- Left-to-right flow
- 200px spacing between elements
"""
```

**Implementation:**
1. Add to DIAGRAM_KEYWORDS
2. Define shape rules
3. Create example layouts for levels 0, 1, 2, 3
4. Test with: "Show a data flow for user registration system"

#### 1.1.2 Deployment Diagram (UML)
**Why?** DevOps/Infrastructure teams need this

**New Diagram Type:**
```python
TYPE_RULES["deployment"] = """\
═══════════════════════════════════
DEPLOYMENT DIAGRAM RULES
═══════════════════════════════════
ARTIFACTS (deployment items):
- Component artifact: box with artifact icon <<artifact>>
- Executable artifact: box with executable icon
- Library artifact: box with library icon

NODES (deployment targets):
- Server: 3D box effect (using multiple rectangles)
- Device: device shape (rectangle with rounded corners on one end)
- Cloud: cloud shape (series of rounded rectangles)

RELATIONSHIPS:
- Deploy: arrow from artifact to node
- Labels: deployment technology, port numbers
- Multiplicity: "1..n" for multiple instances

LAYOUT:
- Top tier: artifact sources
- Bottom tier: deployment nodes
- Vertical connections showing deployment path
"""
```

#### 1.1.3 Activity Diagram (UML)
**Why?** Business process modeling, similar to flowchart but with swimlanes

**Features:**
- Initial/final nodes
- Action nodes (rounded rectangles)
- Decision nodes (diamonds)
- Merge nodes (circle)
- Fork/Join bars (horizontal/vertical lines)
- Arrows with activity labels

#### 1.1.4 Use Case Diagram (UML)
**Why?** Requirements analysis and system scoping

**Features:**
- Actors (stick figures or rectangles)
- Use cases (ovals)
- System boundary (rectangle)
- Relationships: association, generalization, include, extend
- Labels on relationships

#### 1.1.5 Component Diagram (UML)
**Why?** Software architecture at component level

**Features:**
- Components (rectangles with component icon)
- Interfaces (circle with ball/socket notation)
- Dependencies (dashed arrows)
- Provided/required interfaces
- Ports

#### 1.1.6 Package Diagram
**Why?** Large system organization

**Features:**
- Packages (folder-like rectangles)
- Classes inside packages
- Dependencies between packages
- Visibility indicators (+, -, ~, #)

#### 1.1.7 Composite Structure Diagram
**Why?** Internal structure of complex components

**Features:**
- Parts (components within system)
- Connectors (connections between parts)
- Ports (interaction points)
- Interfaces

#### 1.1.8 Timing Diagram
**Why?** Real-time systems, protocol timing

**Features:**
- Lifelines (vertical lines with time markers)
- State changes (vertical transitions)
- Interactions (arrows showing timing)
- Time axis (horizontal scale)

**Implementation Plan:**
```
Week 1: Design guidelines for each type (this document)
Week 2: Implement 4 types (DFD, Deployment, Activity, Use Case)
Week 3: Implement 4 types (Component, Package, Composite, Timing)
Week 4: Testing & refinement, documentation
```

### 1.2 Enhance Existing Diagram Types

#### 1.2.1 Flowchart + Extensions
**Add:**
- Predefined data types (document, database, input device icons)
- Merge/split junctions
- Parallel processing indicators
- Annotation callouts
- Color-coded swim lane support

**Keywords to add:**
```python
DIAGRAM_KEYWORDS["flowchart"].extend([
    "process flow", "business process",
    "data flow", "workflow diagram",
    "activity diagram", "procedure"
])
```

#### 1.2.2 Sequence Diagram + Extensions
**Add:**
- Combined fragments (alt, opt, loop, par)
- Interaction references
- State invariants
- Gate points
- Destruction markers (X on lifeline)
- Constraint messages

**New feature:**
```
Alternative fragments:
  [if condition1] → sequence A
  [else] → sequence B
```

#### 1.2.3 Architecture + Extensions
**Add:**
- Container registry/artifact storage visualization
- Security zones (firewalls, DMZ)
- Data flow emphasis
- Message bus/event stream visualization
- Kubernetes-specific shapes (pods, services, deployments)

**Kubernetes support:**
```python
# New shapes for K8s components
{
    "pod": {"icon": "📦", "color": "#4a90d9"},
    "service": {"icon": "🔌", "color": "#27ae60"},
    "deployment": {"icon": "📋", "color": "#e67e22"},
    "configmap": {"icon": "⚙️", "color": "#9b59b6"},
    "secret": {"icon": "🔐", "color": "#c0392b"},
}
```

---

## 🎯 Phase 2: Medium-Term Features (3-6 months)

### 2.1 Advanced Diagram Customization

#### 2.1.1 Theming System
Allow users to customize colors per diagram type

```python
THEMES = {
    "default": {
        "primary": "#4a90d9",
        "secondary": "#27ae60",
        "accent": "#e67e22",
        ...
    },
    "dark": {
        "primary": "#2c3e50",
        "secondary": "#16a085",
        ...
    },
    "colorblind": {
        "primary": "#e74c3c",  # More distinct colors
        "secondary": "#3498db",
        ...
    }
}

# Usage in generate_diagram:
system_prompt = get_system_prompt(diagram_type)
system_prompt += f"\n\nColor Palette:\n{THEMES[user_theme]}"
```

#### 2.1.2 Size Presets
Support different diagram complexities

```python
SIZE_PRESETS = {
    "small": {
        "element_width": 100,
        "element_height": 40,
        "spacing": 80,
        "canvas_width": 800,
        "canvas_height": 600
    },
    "medium": {
        "element_width": 160,
        "element_height": 60,
        "spacing": 120,
        "canvas_width": 1200,
        "canvas_height": 900
    },
    "large": {
        "element_width": 200,
        "element_height": 80,
        "spacing": 160,
        "canvas_width": 1600,
        "canvas_height": 1200
    }
}
```

#### 2.1.3 Language Support
Provide rules in multiple languages

```python
SUPPORTED_LANGUAGES = ["en", "ja", "de", "fr", "es", "zh", "ko"]

# Localized keywords
DIAGRAM_KEYWORDS_JA = {
    "sequence": ["シーケンス", "時系列図", "メッセージフロー"],
    "flowchart": ["フローチャート", "流れ図", "ワークフロー"],
    ...
}
```

### 2.2 Smart Diagram Enhancement

#### 2.2.1 Auto-Layout Optimization
Compute optimal element positions to minimize edge crossings

```python
class LayoutOptimizer:
    def optimize_positions(elements: list, diagram_type: str) -> list:
        # Force-directed graph layout for position optimization
        # Prevents overlapping elements
        # Minimizes edge crossings
        # Returns repositioned elements
        pass
```

#### 2.2.2 Consistency Checking
Validate diagrams before rendering

```python
class DiagramValidator:
    def check_consistency(elements: list, diagram_type: str) -> list:
        # Check all connections point to valid elements
        # Verify color consistency
        # Check for orphaned elements
        # Return list of issues
        pass
```

#### 2.2.3 Template Library
Pre-defined diagram templates

```python
TEMPLATES = {
    "microservices": [
        # Elements for typical microservices architecture
        {"id": "api-gateway", "type": "rectangle", ...},
        {"id": "auth-service", "type": "rectangle", ...},
        {"id": "user-service", "type": "rectangle", ...},
        ...
    ],
    "rest-api": [
        # REST API sequence flow
        ...
    ],
    "ci-cd-pipeline": [
        # CI/CD deployment diagram
        ...
    ]
}
```

---

## 💡 Phase 3: Advanced Features (6-12 months)

### 3.1 AI-Powered Enhancements

#### 3.1.1 Natural Language Improvements
- Multi-turn conversations: "Add a load balancer" → "Now add caching"
- Diagram understanding: "Show me the same thing as a flowchart"
- Style transfer: "Make it look like a Netflix architecture diagram"

```python
async def interactive_generate(
    initial_prompt: str,
    conversation_history: list
) -> dict:
    # Combine prompt history into context
    # Generate diagram with full context
    # Support iterative refinement
    pass
```

#### 3.1.2 Diagram-to-Text Generation
Reverse: given a diagram, generate markdown description

```python
def diagram_to_description(elements: list, diagram_type: str) -> str:
    # Analyze diagram structure
    # Generate natural language description
    # Return markdown or plain text
    pass
```

#### 3.1.3 Code Generation from Diagrams
Generate code based on diagrams

```python
def generate_code(elements: list, diagram_type: str, language: str) -> str:
    # Architecture diagram → Terraform/CloudFormation
    # Class diagram → Java/Python class stubs
    # State diagram → State machine code
    # Sequence diagram → API client code
    pass
```

### 3.2 Collaboration Features

#### 3.2.1 Real-Time Collaboration
Using WebSockets

```python
@app.websocket("/ws/diagram/{session_id}")
async def websocket_endpoint(websocket: WebSocket, session_id: str):
    await websocket.accept()
    # Broadcast diagram updates to all connected clients
    # Maintain version history
    # Track concurrent edits
```

#### 3.2.2 Version Control
Track diagram changes over time

```python
class DiagramRepository:
    def save_version(diagram_id: str, elements: list, message: str):
        # Git-style versioning
        # Diff between versions
        # Restore to previous version
        pass
    
    def get_history(diagram_id: str) -> list:
        # Return all versions with timestamps
        pass
    
    def diff(version1: str, version2: str) -> dict:
        # Return added/removed/modified elements
        pass
```

### 3.3 Export & Integration

#### 3.3.1 Export Formats
```python
EXPORT_FORMATS = {
    "excalidraw": "native format",
    "svg": "vector graphics",
    "png": "raster image",
    "pdf": "document",
    "mermaid": "markdown diagram syntax",
    "plantuml": "PlantUML text format",
    "graphviz": "DOT format",
    "json": "custom application format",
}

async def export_diagram(
    elements: list,
    format: str,
    options: dict
) -> bytes:
    # Export to specified format
    # Support format-specific options
    # Return file bytes
    pass
```

#### 3.3.2 Integrations
- Confluence: Export diagram directly to Confluence
- Jira: Embed diagrams in issue descriptions
- Slack: Share diagram snapshots
- GitHub: Auto-embed in README.md
- Notion: Block support

```python
class Integrations:
    async def export_to_confluence(diagram_id: str, page_id: str):
        pass
    
    async def save_to_github(diagram_id: str, repo: str, path: str):
        pass
    
    async def share_on_slack(diagram_id: str, channel: str):
        pass
```

---

## 🔧 Implementation Roadmap

```
Month 1 (Weeks 1-4):
├─ Add 8 new diagram types (DFD, Deployment, etc.)
├─ Enhance existing types (Flowchart, Sequence, Architecture)
└─ Improve keyword detection accuracy

Month 2-3 (Weeks 5-12):
├─ Implement theming system
├─ Add size presets
├─ Build layout optimizer
└─ Create template library

Month 4-6 (Weeks 13-24):
├─ Multi-language support
├─ Diagram validation engine
├─ Diagram-to-text generation
└─ Code generation from diagrams

Month 7-12 (Weeks 25-48):
├─ Real-time collaboration (WebSockets)
├─ Version control system
├─ Multiple export formats
└─ External integrations
```

---

## 📊 Scalability Metrics

### Current Capacity
- **Diagram Types Supported:** 12
- **Shapes Per Diagram:** 50-200 (average)
- **Elements In Registry:** 7 base shapes
- **Color Schemes:** 3 per type (customizable)

### After Phase 1
- **Diagram Types:** 20+
- **Enhanced features per type:** 2-3x
- **Color Schemes:** 5+ per type

### After Phase 2
- **Theme Support:** 5+ themes (dark, light, colorblind, brand colors)
- **Layout Algorithms:** 3+ (hierarchical, force-directed, organic)
- **Template Library:** 20+ pre-designed templates
- **Languages:** 7+ supported

### After Phase 3
- **Collaboration:** Real-time multi-user editing
- **Export Formats:** 8+ formats
- **Code Generation:** Support for 10+ programming languages
- **Integrations:** 10+ platforms

---

## 🎓 Training & Documentation

### New Type Documentation Template
```markdown
## [New Diagram Type] Rules

### Overview
[2-3 sentence description]

### Elements
- [Element 1]: [Description], [Shape], [Color], [Size]
- [Element 2]: [Description], [Shape], [Color], [Size]

### Layout
[Positioning rules, spacing, canvas size]

### Styling
[Colors, fonts, special styling]

### Example
[Sample prompt that should generate this type]
```

### Testing Strategy
1. **Unit Tests:** Each diagram type generates valid JSON
2. **Integration Tests:** Full flow from prompt to diagram
3. **Visual Tests:** Generated diagrams look reasonable
4. **Performance Tests:** Response time < 10 seconds

---

## 🚨 Note on Scalability

### Challenges
1. **Prompt Quality:** More types = harder for Gemini to choose right one
   - **Solution:** Improve DIAGRAM_KEYWORDS matching
   - **Solution:** Add multi-turn disambiguation

2. **Token Usage:** Longer system prompts = higher cost
   - **Solution:** Create compact rule variants
   - **Solution:** Implement caching for system prompts

3. **Maintenance:** More types = more code to maintain
   - **Solution:** Template system for new types
   - **Solution:** Community contribution process

4. **Response Time:** Complex rules may slow down generation
   - **Solution:** Parallel prompt building
   - **Solution:** Model optimization

---

## ✅ Success Criteria

| Metric               | Current | Phase 1 | Phase 2 | Phase 3 |
| -------------------- | ------- | ------- | ------- | ------- |
| Diagram Types        | 12      | 20      | 20      | 20      |
| Export Formats       | 1       | 1       | 8       | 8       |
| Languages            | 1       | 2       | 7       | 7       |
| Avg Response Time    | 5s      | 6s      | 7s      | 8s      |
| User Satisfaction    | TBD     | 4.0/5   | 4.5/5   | 4.8/5   |
| Monthly Active Users | 100     | 500     | 2000    | 5000    |

---

**Document Version:** 1.0
**Last Updated:** February 21, 2026
**Next Review:** Quarterly
