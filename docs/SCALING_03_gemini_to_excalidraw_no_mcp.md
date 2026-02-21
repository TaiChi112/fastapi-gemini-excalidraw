# gemini_to_excalidraw_no_mcp.py - Scaling & Feature Expansion Guide

## 📈 Scaling Strategy Overview

**Current State:** Basic async diagram generation with fallback
**Vision:** Enterprise-grade intelligent diagram generation platform

---

## 🚀 Phase 1: Near-Term Enhancements (1-3 months)

### 1.1 Model & API Optimization

#### 1.1.1 Multi-Model Support
Support different Gemini models for different use cases

```python
MODEL_CONFIGS = {
    "gemini-2.0-flash": {
        "speed": "fast",
        "quality": "good",
        "cost": "low",
        "use_for": ["quick_prototypes", "simple_diagrams"]
    },
    "gemini-2.0-pro": {
        "speed": "medium",
        "quality": "excellent",
        "cost": "high",
        "use_for": ["complex_diagrams", "production"]
    },
    "gemini-2.5-flash": {
        "speed": "very_fast",
        "quality": "good",
        "cost": "very_low",
        "use_for": ["batch_processing", "real_time"]
    }
}

async def generate_diagram_auto_model(
    prompt: str,
    system_prompt: str,
    diagram_type: str = "architecture",
    quality_requirement: str = "good"
) -> dict:
    """Automatically select best model for the task"""
    
    # Select model based on complexity
    if diagram_type in ["c4", "class", "erd"]:
        model = "gemini-2.0-pro"  # Complex diagrams need better model
    elif len(prompt) > 500:
        model = "gemini-2.0-pro"  # Long prompts need better parsing
    else:
        model = "gemini-2.0-flash"  # Fast for simple cases
    
    # Override with quality requirement
    if quality_requirement == "maximum":
        model = "gemini-2.0-pro"
    elif quality_requirement == "minimum":
        model = "gemini-2.5-flash"
    
    # Update client with selected model
    original_model = GEMINI_MODEL
    GEMINI_MODEL = model
    
    try:
        result = await generate_diagram(prompt, system_prompt)
    finally:
        GEMINI_MODEL = original_model
    
    return result
```

#### 1.1.2 Prompt Optimization
Enhance prompts for better results

```python
class PromptOptimizer:
    async def optimize_prompt(
        user_prompt: str,
        diagram_type: str
    ) -> str:
        """Use Gemini to rewrite prompt for better generation"""
        
        optimization_request = f"""
        Rewrite this diagram description to be more precise and structured,
        while keeping the user's intent. Make it easier for an AI to understand.
        
        Original: {user_prompt}
        Diagram Type: {diagram_type}
        """
        
        response = await client.aio.models.generate_content(
            model="gemini-2.0-flash",
            contents=optimization_request
        )
        
        return response.text
    
    def add_context_hints(
        user_prompt: str,
        diagram_type: str
    ) -> str:
        """Add context hints to the prompt"""
        
        hints = {
            "sequence": "Include actor names and message descriptions",
            "flowchart": "Use clear start/end labels and decision conditions",
            "architecture": "Specify tier names, component functions, data flows",
            "erd": "Include entity names, attributes, and relationships",
            "c4": "Specify system boundary, containers, and components"
        }
        
        hint = hints.get(diagram_type, "")
        if hint:
            return f"{user_prompt}\n\nHint: {hint}"
        
        return user_prompt
```

#### 1.1.3 Response Validation
Validate and score Gemini responses

```python
class ResponseValidator:
    def score_response(elements: list, diagram_type: str) -> float:
        """Score response quality 0-100"""
        
        score = 100
        
        # Check element count
        element_count = len(elements)
        if element_count < 2:
            score -= 40  # Insufficient elements
        if element_count > 200:
            score -= 20  # Too many elements
        
        # Check diversity
        types = set(el.get("type") for el in elements)
        if len(types) < 2:
            score -= 30  # Not diverse enough
        
        # Check layout
        positions = [(el.get("x", 0), el.get("y", 0)) for el in elements]
        if len(set(positions)) < len(positions):
            score -= 20  # Elements overlap
        
        # Check required fields
        for el in elements:
            if not all(k in el for k in ["x", "y", "width", "height"]):
                score -= 5
        
        return max(0, score)
    
    async def retry_if_low_quality(
        prompt: str,
        system_prompt: str,
        min_score: float = 70
    ) -> dict:
        """Retry if response quality is low"""
        
        for attempt in range(3):
            result = await generate_diagram(prompt, system_prompt)
            quality_score = self.score_response(
                result.get("elements", []),
                system_prompt
            )
            
            if quality_score >= min_score:
                return result
            
            print(f"Quality score {quality_score} < {min_score}, retrying...")
        
        return result  # Return best attempt
```

### 1.2 Request/Response Handling

#### 1.2.1 Request Caching
Cache responses for identical or similar prompts

```python
from functools import lru_cache
import hashlib

class RequestCache:
    def __init__(self, max_size: int = 1000):
        self.cache = {}
        self.max_size = max_size
    
    def get_cache_key(self, prompt: str, system_prompt: str) -> str:
        """Generate cache key from prompts"""
        combined = f"{prompt}|{system_prompt}"
        return hashlib.md5(combined.encode()).hexdigest()
    
    async def get_or_generate(
        self,
        prompt: str,
        system_prompt: str
    ) -> dict:
        """Get from cache or generate"""
        cache_key = self.get_cache_key(prompt, system_prompt)
        
        if cache_key in self.cache:
            print(f"Cache hit for {prompt[:30]}...")
            return self.cache[cache_key]
        
        result = await generate_diagram(prompt, system_prompt)
        
        # Store in cache
        if len(self.cache) >= self.max_size:
            # Remove oldest entry
            oldest_key = next(iter(self.cache))
            del self.cache[oldest_key]
        
        self.cache[cache_key] = result
        return result
    
    def clear_cache(self):
        """Clear entire cache"""
        self.cache.clear()

# Global cache instance
diagram_cache = RequestCache(max_size=1000)
```

#### 1.2.2 Streaming Responses
Stream diagram generation for large diagrams

```python
async def generate_diagram_streaming(
    prompt: str,
    system_prompt: str
) -> AsyncGenerator[str, None]:
    """Stream diagram generation progress"""
    
    yield json.dumps({"status": "starting", "progress": 0})
    
    # Call API with streaming
    async with client.aio.models.generate_content_stream(
        model=GEMINI_MODEL,
        config={"system_instruction": system_prompt},
        contents=prompt
    ) as stream:
        collected_text = ""
        
        async for chunk in stream:
            collected_text += chunk.text
            yield json.dumps({
                "status": "generating",
                "progress": min(100, int(len(collected_text) / 10))
            })
        
        # Parse final result
        try:
            clean_json = collected_text.replace("```json", "").replace("```", "")
            elements = json.loads(clean_json)
            elements = sanitize_elements(elements)
            elements = fix_elements(elements)
            
            yield json.dumps({
                "status": "complete",
                "elements": elements,
                "progress": 100
            })
        except Exception as e:
            yield json.dumps({
                "status": "error",
                "error": str(e),
                "progress": 100
            })
```

#### 1.2.3 Batch Processing
Process multiple diagrams efficiently

```python
class BatchProcessor:
    async def generate_batch(
        self,
        prompts: list,
        system_prompts: list = None
    ) -> list:
        """Generate multiple diagrams concurrently"""
        
        if system_prompts is None:
            # Generate system prompts for each
            system_prompts = [
                get_system_prompt(detect_diagram_type(p))
                for p in prompts
            ]
        
        # Run all concurrently using asyncio
        import asyncio
        tasks = [
            generate_diagram(prompt, sys_prompt)
            for prompt, sys_prompt in zip(prompts, system_prompts)
        ]
        
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        # Handle any errors
        processed_results = []
        for i, result in enumerate(results):
            if isinstance(result, Exception):
                processed_results.append({
                    "error": str(result),
                    "prompt": prompts[i]
                })
            else:
                processed_results.append(result)
        
        return processed_results
    
    async def generate_with_backoff(
        self,
        prompt: str,
        system_prompt: str,
        max_retries: int = 3,
        initial_backoff: float = 1.0
    ) -> dict:
        """Retry with exponential backoff on rate limit"""
        
        backoff = initial_backoff
        last_error = None
        
        for attempt in range(max_retries):
            try:
                return await generate_diagram(prompt, system_prompt)
            except Exception as e:
                last_error = e
                
                if "429" in str(e) or "rate_limit" in str(e):
                    print(f"Rate limited, backing off {backoff}s...")
                    await asyncio.sleep(backoff)
                    backoff *= 2  # Exponential backoff
                else:
                    raise
        
        raise last_error
```

### 1.3 Error Handling & Resilience

#### 1.3.1 Comprehensive Error Recovery
```python
class ErrorRecovery:
    async def auto_recover(
        prompt: str,
        system_prompt: str,
        original_error: Exception
    ) -> dict:
        """Automatically attempt recovery strategies"""
        
        strategies = [
            self.simplify_and_retry,
            self.use_fallback_model,
            self.load_mock_data,
            self.return_error_response
        ]
        
        for strategy in strategies:
            try:
                result = await strategy(prompt, system_prompt)
                if result:
                    return result
            except Exception as e:
                print(f"Recovery strategy failed: {e}")
                continue
        
        # Last resort
        return {
            "elements": [],
            "error": str(original_error),
            "status": "failed"
        }
    
    async def simplify_and_retry(
        self,
        prompt: str,
        system_prompt: str
    ) -> dict:
        """Simplify prompt and retry"""
        simplified = prompt.split('.')[0]  # Use only first sentence
        return await generate_diagram(simplified, system_prompt)
    
    async def use_fallback_model(
        self,
        prompt: str,
        system_prompt: str
    ) -> dict:
        """Try with different model"""
        original = GEMINI_MODEL
        try:
            GEMINI_MODEL = "gemini-2.5-flash"
            return await generate_diagram(prompt, system_prompt)
        finally:
            GEMINI_MODEL = original
```

#### 1.3.2 Request Timeout Management
```python
async def generate_diagram_with_timeout(
    prompt: str,
    system_prompt: str,
    timeout_seconds: int = 30
) -> dict:
    """Generate with timeout protection"""
    
    try:
        result = await asyncio.wait_for(
            generate_diagram(prompt, system_prompt),
            timeout=timeout_seconds
        )
        return result
    
    except asyncio.TimeoutError:
        print(f"Request timed out after {timeout_seconds}s")
        return {
            "elements": mock_elements(),
            "error": f"Generation timeout after {timeout_seconds}s",
            "status": "timeout"
        }
```

---

## 🎯 Phase 2: Medium-Term Features (3-6 months)

### 2.1 Advanced Orchestration

#### 2.1.1 Workflow Engine
Multi-step diagram generation workflows

```python
class WorkflowEngine:
    async def execute_workflow(
        self,
        prompt: str,
        workflow_name: str
    ) -> dict:
        """Execute multi-step diagram workflow"""
        
        workflows = {
            "architecture_full": [
                ("detect_type", {}),
                ("generate_initial", {}),
                ("validate_quality", {"min_score": 70}),
                ("enhance_styling", {}),
                ("add_annotations", {}),
                ("export_formats", {"formats": ["svg", "png"]})
            ],
            "design_review": [
                ("generate_initial", {}),
                ("check_accessibility", {}),
                ("get_suggestions", {}),
                ("auto_fix_issues", {}),
                ("generate_report", {})
            ]
        }
        
        workflow_steps = workflows.get(workflow_name, [])
        
        context = {"prompt": prompt, "elements": None}
        
        for step_name, step_config in workflow_steps:
            print(f"Executing step: {step_name}")
            context = await self.execute_step(step_name, context, step_config)
        
        return context
    
    async def execute_step(self, step_name: str, context: dict, config: dict) -> dict:
        """Execute individual workflow step"""
        
        if step_name == "detect_type":
            context["diagram_type"] = detect_diagram_type(context["prompt"])
        
        elif step_name == "generate_initial":
            diagram_type = context.get("diagram_type", "architecture")
            system_prompt = get_system_prompt(diagram_type)
            result = await generate_diagram(context["prompt"], system_prompt)
            context["elements"] = result["elements"]
        
        # ... more steps
        
        return context
```

#### 2.1.2 Multi-Format Output
```python
class MultiFormatExporter:
    async def export_all_formats(
        self,
        elements: list,
        diagram_type: str
    ) -> dict:
        """Export to multiple formats simultaneously"""
        
        formats = {
            "excalidraw": elements,  # Native format
            "svg": await self.to_svg(elements),
            "png": await self.to_png(elements),
            "pdf": await self.to_pdf(elements),
            "json": json.dumps(elements),
            "mermaid": self.to_mermaid(elements, diagram_type),
            "plantuml": self.to_plantuml(elements, diagram_type)
        }
        
        return formats
```

### 2.2 Analytics & Monitoring

#### 2.2.1 Usage Analytics
```python
class UsageAnalytics:
    def __init__(self):
        self.stats = {
            "total_requests": 0,
            "successful": 0,
            "failed": 0,
            "total_response_time": 0.0,
            "diagram_type_counts": {},
            "error_counts": {}
        }
    
    async def track_request(
        self,
        prompt: str,
        diagram_type: str,
        response_time: float,
        success: bool,
        error: str = None
    ):
        """Track request for analytics"""
        
        self.stats["total_requests"] += 1
        self.stats["total_response_time"] += response_time
        
        if success:
            self.stats["successful"] += 1
        else:
            self.stats["failed"] += 1
            self.stats["error_counts"][error] = \
                self.stats["error_counts"].get(error, 0) + 1
        
        self.stats["diagram_type_counts"][diagram_type] = \
            self.stats["diagram_type_counts"].get(diagram_type, 0) + 1
    
    def get_statistics(self) -> dict:
        """Get usage statistics"""
        avg_time = (self.stats["total_response_time"] / 
                   self.stats["total_requests"] 
                   if self.stats["total_requests"] > 0 else 0)
        
        return {
            **self.stats,
            "average_response_time": avg_time,
            "success_rate": (self.stats["successful"] / 
                           self.stats["total_requests"] * 100
                           if self.stats["total_requests"] > 0 else 0)
        }

# Global analytics instance
analytics = UsageAnalytics()
```

#### 2.2.2 Performance Monitoring
```python
class PerformanceMonitor:
    def __init__(self):
        self.request_times = []
        self.memory_usage = []
    
    async def monitor_generation(
        self,
        prompt: str,
        system_prompt: str
    ) -> dict:
        """Monitor performance during generation"""
        
        import time
        import psutil
        
        start_time = time.time()
        start_memory = psutil.Process().memory_info().rss / 1024 / 1024
        
        result = await generate_diagram(prompt, system_prompt)
        
        end_time = time.time()
        end_memory = psutil.Process().memory_info().rss / 1024 / 1024
        
        self.request_times.append(end_time - start_time)
        self.memory_usage.append(end_memory - start_memory)
        
        return {
            **result,
            "metrics": {
                "response_time_s": end_time - start_time,
                "memory_delta_mb": end_memory - start_memory,
                "avg_response_time_s": sum(self.request_times) / len(self.request_times)
            }
        }
```

### 2.3 Integration Framework

#### 2.3.1 Webhook Support
```python
class WebhookManager:
    def __init__(self):
        self.webhooks = {}  # event -> [list of webhook URLs]
    
    async def register_webhook(self, event: str, webhook_url: str):
        """Register webhook for event"""
        if event not in self.webhooks:
            self.webhooks[event] = []
        self.webhooks[event].append(webhook_url)
    
    async def trigger_webhooks(self, event: str, data: dict):
        """Trigger all webhooks for event"""
        import aiohttp
        
        if event not in self.webhooks:
            return
        
        async with aiohttp.ClientSession() as session:
            for webhook_url in self.webhooks[event]:
                try:
                    async with session.post(webhook_url, json=data) as resp:
                        if resp.status != 200:
                            print(f"Webhook {webhook_url} returned {resp.status}")
                except Exception as e:
                    print(f"Webhook error: {e}")
```

#### 2.3.2 Plugin System
```python
class PluginManager:
    def __init__(self):
        self.plugins = {}
    
    def register_plugin(self, name: str, plugin_class):
        """Register a plugin"""
        self.plugins[name] = plugin_class()
    
    async def pre_generate_hooks(self, prompt: str, diagram_type: str):
        """Run before diagram generation"""
        for plugin in self.plugins.values():
            if hasattr(plugin, 'pre_generate'):
                prompt = await plugin.pre_generate(prompt, diagram_type)
        return prompt
    
    async def post_generate_hooks(self, elements: list, diagram_type: str):
        """Run after diagram generation"""
        for plugin in self.plugins.values():
            if hasattr(plugin, 'post_generate'):
                elements = await plugin.post_generate(elements, diagram_type)
        return elements
```

---

## 💡 Phase 3: Enterprise Features (6-12 months)

### 3.1 Advanced AI Integration

#### 3.1.1 Multi-Turn Conversations
```python
class ConversationManager:
    def __init__(self):
        self.sessions = {}  # session_id -> conversation history
    
    async def continue_conversation(
        self,
        session_id: str,
        user_message: str
    ) -> dict:
        """Continue multi-turn conversation"""
        
        if session_id not in self.sessions:
            self.sessions[session_id] = {
                "history": [],
                "current_diagram": None,
                "diagram_type": None
            }
        
        session = self.sessions[session_id]
        session["history"].append({"role": "user", "content": user_message})
        
        # Build context from history
        context = f"Conversation history:\n"
        for msg in session["history"][-5:]:  # Last 5 messages
            context += f"{msg['role']}: {msg['content']}\n"
        
        # Generate response
        response = await self.generate_with_context(context, user_message)
        
        session["history"].append({"role": "assistant", "content": response})
        
        return {
            "response": response,
            "session_id": session_id,
            "history_length": len(session["history"])
        }
    
    async def generate_with_context(self, context: str, message: str) -> dict:
        """Generate diagram with conversation context"""
        
        # If user said "change it to a flowchart", update diagram type
        if "flowchart" in message.lower():
            diagram_type = "flowchart"
        elif "class diagram" in message.lower():
            diagram_type = "class"
        # ... more pattern matching
        else:
            diagram_type = "architecture"  # default
        
        system_prompt = get_system_prompt(diagram_type)
        full_prompt = f"{context}\n\nNew request: {message}"
        
        return await generate_diagram(full_prompt, system_prompt)
```

#### 3.1.2 Diagram Understanding
```python
class DiagramUnderstanding:
    async def analyze_diagram(self, elements: list) -> dict:
        """Use Gemini to understand and describe diagram"""
        
        # Convert elements to description
        description = self.elements_to_text(elements)
        
        prompt = f"""
        Analyze this Excalidraw diagram and provide:
        1. Summary of what the diagram shows
        2. Key components identified
        3. Data flows or relationships
        4. Potential issues or improvements
        
        Diagram description: {description}
        """
        
        response = await client.aio.models.generate_content(
            model="gemini-2.0-flash",
            contents=prompt
        )
        
        return {
            "analysis": response.text,
            "elements_analyzed": len(elements)
        }
```

### 3.2 Personalization & Learning

#### 3.2.1 User Preferences
```python
class UserPreferences:
    def __init__(self):
        self.preferences = {}  # user_id -> preferences
    
    def set_preference(self, user_id: str, key: str, value):
        """Set user preference"""
        if user_id not in self.preferences:
            self.preferences[user_id] = {}
        self.preferences[user_id][key] = value
    
    def get_preferences(self, user_id: str) -> dict:
        """Get user preferences"""
        return self.preferences.get(user_id, {})
    
    async def generate_with_preferences(
        self,
        user_id: str,
        prompt: str,
        diagram_type: str
    ) -> dict:
        """Generate using user preferences"""
        
        prefs = self.get_preferences(user_id)
        theme = prefs.get("theme", "default")
        quality = prefs.get("quality", "good")
        
        # Apply preferences
        system_prompt = get_system_prompt(diagram_type)
        system_prompt += f"\nUser theme preference: {theme}"
        system_prompt += f"\nQuality level: {quality}"
        
        return await generate_diagram(prompt, system_prompt)
```

---

## 📊 Implementation Timeline

```
Month 1:
├─ Multi-model support
├─ Prompt optimization
├─ Request caching
└─ Response validation

Month 2-3:
├─ Batch processing
├─ Error recovery strategies
├─ Timeout management
└─ Performance improvements

Month 4-6:
├─ Workflow engine
├─ Multi-format export
├─ Analytics & monitoring
└─ Webhook support

Month 7-12:
├─ Plugin system
├─ Multi-turn conversations
├─ User preferences
└─ Advanced ML integration
```

---

## 🎯 Success Metrics

| Metric               | Current | Target (12 mo) |
| -------------------- | ------- | -------------- |
| Avg Response Time    | 5s      | 3s             |
| Success Rate         | 85%     | 95%            |
| Cache Hit Rate       | 0%      | 30%            |
| Concurrent Users     | 10      | 1000+          |
| Diagrams/Day         | 100     | 100K+          |
| Enterprise Customers | 0       | 50+            |
| API Uptime           | 99%     | 99.9%          |

---

**Document Version:** 1.0
**Last Updated:** February 21, 2026
**Next Review:** Quarterly
