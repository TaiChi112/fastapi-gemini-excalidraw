# sanitize_elements.py - Scaling & Feature Expansion Guide

## 📈 Scaling Strategy Overview

**Current State:** Basic validation and field injection
**Vision:** Intelligent element management and optimization system

---

## 🚀 Phase 1: Near-Term Enhancements (1-3 months)

### 1.1 Advanced Validation Rules

#### 1.1.1 Semantic Validation
Check if elements make sense together

```python
class SemanticValidator:
    def validate_connections(elements: list) -> list:
        """Check that arrows point to valid elements"""
        element_ids = {el["id"] for el in elements}
        issues = []
        
        for el in elements:
            if el["type"] == "arrow":
                if el.get("startBinding"):
                    target_id = el["startBinding"].get("elementId")
                    if target_id and target_id not in element_ids:
                        issues.append({
                            "element": el["id"],
                            "issue": f"Arrow points to non-existent element {target_id}",
                            "severity": "error"
                        })
        return issues
    
    def validate_hierarchy(elements: list) -> list:
        """Check parent-child relationships"""
        issues = []
        for el in elements:
            if el.get("containerId"):
                container = next((e for e in elements 
                                if e["id"] == el["containerId"]), None)
                if not container:
                    issues.append({
                        "element": el["id"],
                        "issue": f"Container {el['containerId']} not found",
                        "severity": "warning"
                    })
        return issues
```

#### 1.1.2 Constraint Checking
Ensure elements follow diagram-specific rules

```python
class ConstraintChecker:
    CONSTRAINTS = {
        "sequence": {
            "max_actors": 10,
            "max_messages": 50,
            "min_lifeline_height": 100
        },
        "flowchart": {
            "max_nodes": 50,
            "max_decisions": 10,
            "require_start_end": True
        },
        "architecture": {
            "max_tiers": 5,
            "max_components_per_tier": 10
        }
    }
    
    def check_constraints(
        elements: list,
        diagram_type: str
    ) -> (list, dict):
        """Return (elements, violations_dict)"""
        constraints = self.CONSTRAINTS.get(diagram_type, {})
        violations = {}
        
        for constraint, limit in constraints.items():
            count = self.count_constraint(elements, constraint)
            if count > limit:
                violations[constraint] = {
                    "current": count,
                    "limit": limit,
                    "status": "exceeded"
                }
        
        return elements, violations
```

#### 1.1.3 Color Harmony Validation
Ensure colors are visually harmonious

```python
class ColorValidator:
    def check_color_harmony(elements: list) -> dict:
        """Analyze color distribution"""
        colors = [el.get("backgroundColor") for el in elements 
                 if el.get("backgroundColor")]
        
        # Convert hex to RGB
        rgb_colors = [hex_to_rgb(c) for c in colors]
        
        # Check contrast ratios
        issues = []
        for i, color1 in enumerate(rgb_colors):
            for color2 in rgb_colors[i+1:]:
                contrast = calculate_contrast(color1, color2)
                if contrast < 4.5:  # WCAG AA standard
                    issues.append({
                        "issue": "Low contrast between colors",
                        "contrast_ratio": contrast,
                        "severity": "warning"
                    })
        
        return {"issues": issues, "colors_used": len(colors)}
    
    def auto_adjust_colors(elements: list, threshold=4.5) -> list:
        """Automatically adjust colors for accessibility"""
        # Modify colors to meet WCAG standards
        for el in elements:
            bg_color = el.get("backgroundColor")
            stroke_color = el.get("strokeColor")
            
            if bg_color and stroke_color:
                contrast = calculate_contrast(bg_color, stroke_color)
                if contrast < threshold:
                    el["strokeColor"] = adjust_for_contrast(
                        stroke_color, bg_color, threshold
                    )
        
        return elements
```

### 1.2 Enhanced Dimension Management

#### 1.2.1 Smart Sizing
Automatically calculate appropriate sizes based on content

```python
class SmartSizer:
    def auto_size_text_box(text: str, font_size: int = 16) -> dict:
        """Calculate optimal width/height for text element"""
        
        # Approximate: each character is about 0.6x font height wide
        char_width = font_size * 0.6
        text_width = len(text) * char_width
        
        # Add padding: 10px on each side
        width = int(text_width + 20)
        height = int(font_size * 1.25 + 10)  # 1.25 = line height
        
        return {"width": width, "height": height}
    
    def auto_size_container(children: list) -> dict:
        """Calculate optimal size for container holding children"""
        if not children:
            return {"width": 100, "height": 100}
        
        max_x = max(el.get("x", 0) + el.get("width", 100)
                   for el in children)
        max_y = max(el.get("y", 0) + el.get("height", 100)
                   for el in children)
        
        # Add 20px padding
        return {
            "width": int(max_x + 20),
            "height": int(max_y + 20)
        }
    
    def proportion_elements(elements: list, target_size: tuple) -> list:
        """Scale all elements to fit within target canvas size"""
        current_width = max((el.get("x", 0) + el.get("width", 0))
                          for el in elements)
        current_height = max((el.get("y", 0) + el.get("height", 0))
                           for el in elements)
        
        scale_x = target_size[0] / current_width if current_width > 0 else 1
        scale_y = target_size[1] / current_height if current_height > 0 else 1
        scale = min(scale_x, scale_y, 1.0)  # Don't enlarge
        
        for el in elements:
            el["x"] = int(el.get("x", 0) * scale)
            el["y"] = int(el.get("y", 0) * scale)
            el["width"] = int(el.get("width", 0) * scale)
            el["height"] = int(el.get("height", 0) * scale)
        
        return elements
```

#### 1.2.2 Collision Detection
Detect and prevent overlapping elements

```python
class CollisionDetector:
    def find_overlaps(elements: list) -> list:
        """Find all overlapping elements"""
        overlaps = []
        
        for i, el1 in enumerate(elements):
            for el2 in elements[i+1:]:
                if self.rectangles_overlap(el1, el2):
                    overlaps.append({
                        "element1": el1["id"],
                        "element2": el2["id"],
                        "overlap_area": self.calculate_overlap_area(el1, el2)
                    })
        
        return overlaps
    
    def rectangles_overlap(self, el1: dict, el2: dict) -> bool:
        """Check if two rectangles overlap"""
        x1_start, y1_start = el1.get("x", 0), el1.get("y", 0)
        x1_end = x1_start + el1.get("width", 0)
        y1_end = y1_start + el1.get("height", 0)
        
        x2_start, y2_start = el2.get("x", 0), el2.get("y", 0)
        x2_end = x2_start + el2.get("width", 0)
        y2_end = y2_start + el2.get("height", 0)
        
        return (x1_start < x2_end and x1_end > x2_start and
                y1_start < y2_end and y1_end > y2_start)
    
    def auto_fix_overlaps(elements: list, spacing: int = 20) -> list:
        """Automatically shift overlapping elements"""
        # Use simple greedy approach: shift second element right
        for i, el1 in enumerate(elements):
            for el2 in elements[i+1:]:
                if self.rectangles_overlap(el1, el2):
                    el2["x"] = el1.get("x", 0) + el1.get("width", 0) + spacing
        
        return elements
```

### 1.3 Performance Optimization

#### 1.3.1 Lazy Validation
Only validate what's necessary

```python
class LazyValidator:
    def validate_essential_only(elements: list) -> (list, list):
        """Fast validation for essential fields only"""
        critical_errors = []
        
        for el in elements:
            # Only check absolutely required fields
            if "id" not in el:
                critical_errors.append(f"Element missing id")
            if "type" not in el:
                critical_errors.append(f"Element {el.get('id')} missing type")
            if "x" not in el or "y" not in el:
                critical_errors.append(f"Element {el.get('id')} missing x/y")
            if "width" not in el or "height" not in el:
                critical_errors.append(f"Element {el.get('id')} missing width/height")
        
        return elements, critical_errors
    
    def validate_full(self, elements: list) -> (list, dict):
        """Complete validation with all checks"""
        _, critical = self.validate_essential_only(elements)
        
        # Only proceed if critical errors are resolved
        if critical:
            return elements, {"critical": critical}
        
        # Now do full validation
        connector_issues = self.validate_connections(elements)
        hierarchy_issues = self.validate_hierarchy(elements)
        color_issues = self.check_color_harmony(elements)
        
        return elements, {
            "connectors": connector_issues,
            "hierarchy": hierarchy_issues,
            "colors": color_issues
        }
```

#### 1.3.2 Batch Processing
Handle large element sets efficiently

```python
class BatchProcessor:
    BATCH_SIZE = 100
    
    def process_large_set(elements: list, processor_func) -> list:
        """Process elements in batches"""
        result = []
        
        for i in range(0, len(elements), self.BATCH_SIZE):
            batch = elements[i:i + self.BATCH_SIZE]
            processed_batch = [processor_func(el) for el in batch]
            result.extend(processed_batch)
        
        return result
    
    def parallel_sanitize(elements: list, num_workers: int = 4) -> list:
        """Use multiprocessing for large sets"""
        from multiprocessing import Pool
        
        if len(elements) < 50:
            # Use single-threaded for small sets
            return [sanitize_element(el) for el in elements]
        
        with Pool(num_workers) as pool:
            return pool.map(sanitize_element, elements)
```

---

## 🎯 Phase 2: Medium-Term Features (3-6 months)

### 2.1 Intelligent Sanitization

#### 2.1.1 Context-Aware Fixes
Fix elements based on surrounding context

```python
class ContextAwareFixer:
    def fix_with_context(elements: list, diagram_type: str) -> list:
        """Apply diagram-type-specific fixes"""
        
        if diagram_type == "sequence":
            elements = self.fix_sequence_diagram(elements)
        elif diagram_type == "flowchart":
            elements = self.fix_flowchart(elements)
        elif diagram_type == "architecture":
            elements = self.fix_architecture(elements)
        
        return elements
    
    def fix_sequence_diagram(self, elements: list) -> list:
        """
        - Ensure lifelines are vertical (height > width)
        - Ensure messages are horizontal (width > height)
        - Fix message y-coordinates to not overlap
        """
        actors = [el for el in elements if el["type"] == "rectangle"]
        lifelines = [el for el in elements if el["type"] == "line"]
        messages = [el for el in elements if el["type"] == "arrow"]
        
        # Ensure lifelines extend full height
        for lifeline in lifelines:
            lifeline["height"] = max(el.get("y", 0) + el.get("height", 0)
                                    for el in actors + messages)
        
        # Sort messages by y-coordinate
        messages.sort(key=lambda x: x.get("y", 0))
        
        # Adjust y to avoid overlaps
        min_y = min(a.get("y", 0) + a.get("height", 0) for a in actors) + 60
        for i, msg in enumerate(messages):
            msg["y"] = min_y + (i * 70)
        
        return elements
    
    def fix_flowchart(self, elements: list) -> list:
        """
        - Center elements horizontally
        - Ensure proper vertical spacing
        - Check decision diamond orientation
        """
        # Implementation similar to above
        pass
    
    def fix_architecture(self, elements: list) -> list:
        """
        - Ensure tier rectangles contain components
        - Check tier ordering
        - Verify arrow directions
        """
        # Implementation similar to above
        pass
```

#### 2.1.2 Learning-Based Correction
Learn from past corrections

```python
class CorrectionLearner:
    def __init__(self):
        self.correction_history = {}  # timestamp -> (problem, solution)
        self.pattern_database = {}     # pattern -> solution
    
    def record_correction(self, problem_element: dict, solution: dict):
        """Record successful corrections"""
        pattern = self.extract_pattern(problem_element)
        self.pattern_database[pattern] = solution
    
    def apply_learned_corrections(self, elements: list) -> list:
        """Apply previously successful corrections"""
        for el in elements:
            pattern = self.extract_pattern(el)
            if pattern in self.pattern_database:
                solution = self.pattern_database[pattern]
                el.update(solution)
        
        return elements
    
    def extract_pattern(self, element: dict) -> str:
        """Identify problem pattern"""
        patterns = []
        
        if element.get("width", 0) < 0:
            patterns.append("negative_width")
        if element.get("height", 0) < 0:
            patterns.append("negative_height")
        if not element.get("backgroundColor"):
            patterns.append("missing_background")
        if "originalText" not in element and element["type"] == "text":
            patterns.append("missing_original_text")
        
        return "|".join(patterns) if patterns else "none"
```

### 2.2 Visual Quality Improvements

#### 2.2.1 Style Consistency Checker
Ensure styles match diagram type conventions

```python
class StyleConsistencyChecker:
    STYLE_GUIDELINES = {
        "sequence": {
            "actor_background": "#dbe9f9",
            "lifeline_style": "dashed",
            "message_width_min": 50
        },
        "flowchart": {
            "start_end_background": "#d4edda",
            "process_background": "#dbe9f9",
            "decision_background": "#fff3cd",
            "arrow_width_min": 2
        },
        "architecture": {
            "tier_opacity": 0.2,
            "component_width_min": 100,
            "spacing_min": 60
        }
    }
    
    def check_style_consistency(
        elements: list,
        diagram_type: str
    ) -> dict:
        """Check if styles match conventions"""
        guidelines = self.STYLE_GUIDELINES.get(diagram_type)
        if not guidelines:
            return {"status": "unknown_type"}
        
        issues = {}
        for el in elements:
            el_issues = self.check_element_style(el, guidelines)
            if el_issues:
                issues[el["id"]] = el_issues
        
        return issues
    
    def auto_fix_styles(self, elements: list, diagram_type: str) -> list:
        """Automatically correct style issues"""
        guidelines = self.STYLE_GUIDELINES.get(diagram_type)
        if not guidelines:
            return elements
        
        for el in elements:
            el_type = el.get("type")
            el_subtype = self.infer_subtype(el, diagram_type)
            
            if el_subtype in ["start_end", "process", "decision"]:
                guideline_key = f"{el_subtype}_background"
                if guideline_key in guidelines:
                    el["backgroundColor"] = guidelines[guideline_key]
        
        return elements
```

#### 2.1.2 Accessibility Improvements
WCAG 2.1 AA compliance

```python
class AccessibilityChecker:
    def check_contrast(elements: list) -> list:
        """Check color contrast ratios"""
        issues = []
        
        for el in elements:
            bg = el.get("backgroundColor")
            stroke = el.get("strokeColor")
            text_color = el.get("strokeColor")  # text usually uses stroke color
            
            if bg and stroke:
                ratio = calculate_contrast_ratio(bg, stroke)
                if ratio < 4.5:  # WCAG AA, large text
                    issues.append({
                        "element": el["id"],
                        "issue": "Low contrast",
                        "ratio": ratio,
                        "required": 4.5
                    })
        
        return issues
    
    def add_labels_to_icons(elements: list) -> list:
        """Ensure all icon-only elements have text labels"""
        for el in elements:
            # Elements with restrictive dimensions (icons)
            if el.get("width", 100) < 50 and el.get("height", 100) < 50:
                # Check if has associated text
                has_label = any(
                    other.get("x", 0) == el.get("x", 0) and
                    other.get("y", 0) > el.get("y", 0)
                    for other in elements if other["type"] == "text"
                )
                
                if not has_label:
                    # Flag for manual review
                    el["_accessibility_warning"] = "Add text label"
        
        return elements
```

---

## 💡 Phase 3: Advanced Features (6-12 months)

### 3.1 Intelligent Auto-Repair

#### 3.1.1 Anomaly Detection
Detect and fix unusual element properties

```python
class AnomalyDetector:
    def detect_anomalies(elements: list) -> list:
        """Find statistically unusual elements"""
        
        # Calculate statistics
        widths = [el.get("width", 0) for el in elements]
        heights = [el.get("height", 0) for el in elements]
        
        width_mean = sum(widths) / len(widths)
        height_mean = sum(heights) / len(heights)
        
        # Find outliers (> 2 standard deviations)
        width_std = self.std_dev(widths)
        height_std = self.std_dev(heights)
        
        anomalies = []
        for el in elements:
            width = el.get("width", 0)
            height = el.get("height", 0)
            
            if abs(width - width_mean) > 2 * width_std:
                anomalies.append({
                    "element": el["id"],
                    "issue": "Unusual width",
                    "value": width,
                    "expected_range": (width_mean - 2*width_std, width_mean + 2*width_std)
                })
        
        return anomalies
    
    def auto_fix_anomalies(self, elements: list) -> list:
        """Fix anomalous elements"""
        anomalies = self.detect_anomalies(elements)
        
        for anomaly in anomalies:
            el = next((e for e in elements if e["id"] == anomaly["element"]), None)
            if el:
                el["width"] = int(anomaly["expected_range"][0] + anomaly["expected_range"][1]) // 2
        
        return elements
```

#### 3.1.2 Predictive Correction
Predict what user intended

```python
class PredictiveCorrector:
    def predict_intent(element: dict, context: list) -> str:
        """Predict what the element should be"""
        
        # Based on position, size, connections
        # ML model trained on correct diagrams
        
        neighbors = self.find_neighbors(element, context)
        
        # If all neighbors are rectangles, this should probably be rectangle too
        if all(n["type"] == "rectangle" for n in neighbors):
            return "rectangle"
        
        # If surrounded by arrows pointing to it, should be container
        connections = [c for c in context if c.get("id") == element["id"]]
        if len(connections) > 3:
            return "container"
        
        return element["type"]  # Keep as-is
```

### 3.2 Integration with LLM

#### 3.2.1 Element Description Generation
Generate human-readable descriptions

```python
class DescriptionGenerator:
    async def generate_description(element: dict) -> str:
        """Use Gemini to generate element description"""
        
        prompt = f"""
        Describe this Excalidraw element in one sentence:
        Type: {element['type']}
        Color: {element.get('backgroundColor')}
        Size: {element.get('width')}x{element.get('height')}
        Position: ({element.get('x')}, {element.get('y')})
        """
        
        response = await gemini_client.models.generate_content(
            model="gemini-2.0-flash",
            contents=prompt
        )
        
        return response.text
```

#### 3.2.2 Smart Suggestion System
Suggest improvements to elements

```python
class SmartSuggester:
    async def suggest_improvements(
        elements: list,
        diagram_type: str
    ) -> list:
        """Suggest element improvements"""
        
        suggestions = []
        
        for el in elements:
            issues = []
            
            # Check various aspects
            if el.get("width", 0) < 50:
                issues.append("Element too small, consider enlarging")
            
            if not el.get("backgroundColor"):
                issues.append("Add background color for better visibility")
            
            if el["type"] == "text" and not el.get("fontSize"):
                issues.append("Set explicit font size")
            
            # Use Gemini for more complex suggestions
            if issues:
                llm_suggestions = await self.get_llm_suggestions(
                    el, diagram_type, issues
                )
                suggestions.extend(llm_suggestions)
        
        return suggestions
```

---

## 🔧 Implementation Roadmap

```
Month 1 (Weeks 1-4):
├─ Add semantic validation
├─ Implement constraint checking
├─ Build color harmony validator
└─ Test with various diagram types

Month 2-3 (Weeks 5-12):
├─ Smart auto-sizing
├─ Collision detection & fixing
├─ Context-aware fixes
└─ Style consistency checker

Month 4-6 (Weeks 13-24):
├─ Accessibility improvements
├─ Learning-based corrections
├─ Batch processing optimization
└─ Performance profiling

Month 7-12 (Weeks 25-48):
├─ Anomaly detection
├─ Predictive correction
├─ LLM integration for descriptions
└─ Smart suggestion system
```

---

## 📊 Scalability Metrics

### Current Capacity
- **Elements per diagram:** 200 (typical)
- **Processing time:** < 100ms
- **Memory per element:** ~2KB
- **Validation checks:** 5 major categories

### After Phase 1
- **Processing time:** < 150ms (slower validation)
- **Validation checks:** 12+ categories
- **Error detection:** 90%+ accuracy

### After Phase 2
- **Processing time:** < 200ms (but more intelligent)
- **Collision detection:** Automatic fixing
- **Style consistency:** 95%+ compliance
- **Accessibility:** WCAG 2.1 AA compliant

### After Phase 3
- **Processing time:** < 300ms (learning & ML)
- **Self-healing capability:** Auto-fix 85%+ issues
- **Predictive accuracy:** 80%+ for intent
- **User satisfaction:** 4.7/5 for quality

---

**Document Version:** 1.0
**Last Updated:** February 21, 2026
**Next Review:** Quarterly
