# FastAPI Gemini Excalidraw Generator

**Version:** 1.0.0  
**Last Updated:** February 20, 2026  
**Status:** Active Development

---

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [System Requirements](#-system-requirements)
- [Functional Requirements](#-functional-requirements)
- [Non-Functional Requirements](#-non-functional-requirements)
- [System Architecture](#-system-architecture)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [Project Structure](#-project-structure)
- [Dependencies](#-dependencies)
- [Analysis & Design](#-analysis--design)
- [API Documentation](#-api-documentation)
- [Troubleshooting](#-troubleshooting)
- [Development](#-development)
- [License](#-license)

---

## 🎯 Project Overview

### Executive Summary
The **FastAPI Gemini Excalidraw Generator** is an AI-powered web application that converts natural language descriptions into visual diagrams automatically. The system leverages Google's Gemini AI model to interpret user prompts and generates structured Excalidraw diagram elements, which are then rendered in an interactive canvas.

### Purpose
This application addresses the need for rapid diagram prototyping and visualization by allowing users to create professional diagrams through simple text descriptions, eliminating the manual effort of diagram creation.

### Key Features
- 🤖 AI-powered diagram generation using Google Gemini 2.0 Flash
- 🎨 Interactive Excalidraw canvas for visualization
- 📊 Support for multiple diagram types (Sequence, Flowchart, Architecture, ER, Network, Timeline, Mind Map)
- 🔄 Real-time diagram generation and rendering
- 🌐 Web-based interface accessible via browser
- ⚡ Fast API backend for high performance

### Target Users
- Software developers and architects
- Product managers and business analysts
- Technical documentation writers
- Students and educators
- Anyone needing quick diagram visualization

---

## 💻 System Requirements

### Hardware Requirements
| Component | Minimum | Recommended |
|-----------|---------|-------------|
| Processor | Dual-core 2.0 GHz | Quad-core 2.5 GHz+ |
| RAM | 4 GB | 8 GB+ |
| Storage | 500 MB | 1 GB+ |
| Network | Stable internet connection | Broadband (5+ Mbps) |

### Software Requirements
- **Operating System:** Windows 10/11, macOS 10.15+, Linux (Ubuntu 20.04+)
- **Python:** Version 3.8 or higher
- **Web Browser:** Modern browser (Chrome 90+, Firefox 88+, Edge 90+, Safari 14+)
- **API Access:** Google Gemini API key (required)

---

## ✨ Functional Requirements

### FR-01: User Input Processing
- **Description:** System shall accept natural language text descriptions as input
- **Input:** Text string describing desired diagram (max 2000 characters)
- **Output:** Parsed and validated user prompt
- **Priority:** High

### FR-02: AI Diagram Generation
- **Description:** System shall generate diagram elements using Google Gemini AI
- **Process:**
  1. Detect diagram type from user prompt
  2. Apply appropriate system prompt template
  3. Generate structured JSON elements
  4. Validate and sanitize output
- **Priority:** High

### FR-03: Diagram Type Detection
- **Description:** System shall automatically detect diagram type from user input
- **Supported Types:**
  - Sequence Diagrams
  - Flowcharts
  - Architecture Diagrams
  - Entity-Relationship Diagrams
  - Network Diagrams
  - Timeline Diagrams
  - Mind Maps
- **Priority:** High

### FR-04: Visual Rendering
- **Description:** System shall render generated diagrams in Excalidraw canvas
- **Features:**
  - Interactive canvas
  - Zoom and pan capabilities
  - Export functionality
  - Manual editing support
- **Priority:** High

### FR-05: Error Handling
- **Description:** System shall handle errors gracefully and provide user feedback
- **Cases:**
  - Invalid API key
  - Network failures
  - Malformed AI responses
  - Invalid user input
- **Priority:** Medium

---

## 🔒 Non-Functional Requirements

### NFR-01: Performance
- **Response Time:** Diagram generation shall complete within 10 seconds under normal conditions
- **Concurrent Users:** Support at least 10 concurrent users
- **API Latency:** Maximum 3 seconds for API response
- **Measurement:** 95th percentile response time

### NFR-02: Scalability
- **Vertical Scaling:** Support increased load through resource scaling
- **Horizontal Scaling:** Architecture supports multiple instance deployment
- **Load Handling:** Graceful degradation under high load

### NFR-03: Reliability
- **Availability:** 99% uptime during business hours
- **Error Recovery:** Automatic retry for transient failures
- **Data Validation:** 100% validation of AI-generated elements
- **Fallback Mechanism:** Mock data available for testing without API

### NFR-04: Security
- **API Key Protection:** Environment variable storage for credentials
- **Input Sanitization:** All user inputs validated and sanitized
- **Data Privacy:** No persistent storage of user prompts
- **HTTPS Support:** Production deployment requires HTTPS

### NFR-05: Usability
- **Learning Curve:** Users should create first diagram within 2 minutes
- **Interface:** Clean, intuitive single-page interface
- **Feedback:** Real-time status updates during generation
- **Accessibility:** WCAG 2.1 Level AA compliance (future enhancement)

### NFR-06: Maintainability
- **Code Quality:** Modular architecture with clear separation of concerns
- **Documentation:** Comprehensive inline comments and docstrings
- **Testing:** Unit test coverage (future enhancement)
- **Version Control:** Git-based version management

### NFR-07: Compatibility
- **Browser Support:** All modern browsers (last 2 versions)
- **Python Compatibility:** Python 3.8 through 3.12
- **Platform Independence:** Cross-platform deployment capability

---

## 🏗️ System Architecture

### High-Level Architecture

```
┌─────────────────┐
│   Web Browser   │
│   (Frontend)    │
│  - HTML/CSS/JS  │
│  - React        │
│  - Excalidraw   │
└────────┬────────┘
         │ HTTP/HTTPS
         ▼
┌─────────────────┐
│   FastAPI       │
│   (Backend)     │
│  - REST API     │
│  - Form Handler │
└────────┬────────┘
         │
         ├──────────────────┬────────────────────┐
         ▼                  ▼                    ▼
┌────────────────┐  ┌──────────────┐  ┌─────────────────┐
│ Gemini Client  │  │ Diagram Type │  │ Element         │
│ (google-genai) │  │ Detector     │  │ Sanitizer       │
└────────┬───────┘  └──────┬───────┘  └────────┬────────┘
         │                  │                    │
         ▼                  ▼                    ▼
┌─────────────────────────────────────────────────────┐
│            Google Gemini AI API                     │
│            (gemini-2.0-flash)                       │
└─────────────────────────────────────────────────────┘
```

### Technology Stack

#### Frontend
- **HTML5:** Semantic markup and structure
- **CSS3:** Styling and responsive design
- **JavaScript (ES6+):** Client-side logic
- **React 19.0.0:** UI component framework
- **Excalidraw 0.18.0:** Interactive diagram canvas

#### Backend
- **FastAPI 0.128.0:** Modern Python web framework
- **Python 3.8+:** Core programming language
- **Uvicorn:** ASGI web server
- **Google GenAI 1.64.0:** Gemini AI client library
- **python-dotenv:** Environment configuration

#### AI/ML
- **Google Gemini 2.0 Flash:** Large language model for diagram generation
- **Custom Prompt Engineering:** Specialized system prompts per diagram type

### Component Interaction Flow

```
1. User enters prompt → 2. Frontend sends POST request
   ↓
3. FastAPI receives request → 4. Detect diagram type
   ↓
5. Select appropriate system prompt → 6. Call Gemini API
   ↓
7. Receive JSON elements → 8. Sanitize & validate
   ↓
9. Return to frontend → 10. Excalidraw renders diagram
```

---

## 📥 Installation

### Step 1: Prerequisites Check
```bash
# Verify Python installation
python --version  # Should be 3.8 or higher

# Verify pip
pip --version
```

### Step 2: Clone Repository
```bash
git clone <repository-url>
cd fastapi-gemini-excalidraw
```

### Step 3: Create Virtual Environment
```bash
# On Windows
python -m venv venv
venv\Scripts\activate

# On macOS/Linux
python3 -m venv venv
source venv/bin/activate
```

### Step 4: Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 5: Environment Configuration
1. Create a `.env` file in the project root:
```bash
# Create .env file
touch .env  # macOS/Linux
type nul > .env  # Windows
```

2. Add your configuration to `.env`:
```env
GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_MODEL=gemini-2.0-flash
GEMINI_MODEL3=gemini-2.0-flash
```

### Step 6: Verify Installation
```bash
# Check installed packages
pip list | grep fastapi
pip list | grep google-genai
```

---

## ⚙️ Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `GEMINI_API_KEY` | Yes | - | Your Google Gemini API key |
| `GEMINI_MODEL` | No | gemini-2.0-flash | Gemini model identifier |
| `GEMINI_MODEL3` | No | gemini-2.0-flash | Alternative model configuration |
| `HOST` | No | 0.0.0.0 | Server host address |
| `PORT` | No | 8000 | Server port number |

### Obtaining Gemini API Key
1. Visit [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Sign in with your Google account
3. Click "Create API Key"
4. Copy the generated key to your `.env` file

### Server Configuration
To customize server settings, modify the `uvicorn.run()` call in [main.py](main.py):
```python
uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## 🚀 Usage

### Starting the Application

#### Development Mode (Recommended for testing)
```bash
fastapi dev main.py
```

#### Production Mode
```bash
python main.py
```

The server will start and display:
```
INFO:     Uvicorn running on http://0.0.0.0:8000
INFO:     Application startup complete.
```

### Accessing the Application
1. Open your web browser
2. Navigate to: `http://localhost:8000`
3. You should see the diagram generator interface

### Creating Your First Diagram

**Step-by-step Guide:**

1. **Enter a Description**
   - In the text input field, type a description of your desired diagram
   - Example: "A login flow with 3 steps: user enters credentials, system validates, user is redirected to dashboard"

2. **Generate**
   - Click the "Generate Diagram" button
   - Wait 3-10 seconds for processing

3. **View & Interact**
   - The generated diagram appears in the Excalidraw canvas
   - Use mouse to pan and zoom
   - Click elements to edit manually if needed

4. **Export (Optional)**
   - Use Excalidraw's built-in export features
   - Save as PNG, SVG, or Excalidraw format

### Example Prompts

#### Sequence Diagram
```
Create a sequence diagram for an online payment system with User, Frontend, 
Backend, Payment Gateway, and Database
```

#### Flowchart
```
Draw a flowchart for user registration: start, enter email and password, 
validate input, check if user exists, create account or show error, send 
confirmation email, end
```

#### Architecture Diagram
```
Show a 3-tier web architecture with load balancer, web servers, application 
servers, and database cluster
```

#### ER Diagram
```
Create an ER diagram for a library system with entities: Book, Author, Member, 
Loan. Books have ISBN and title, Authors have name, Members have ID and name
```

---

## 📁 Project Structure

```
fastapi-gemini-excalidraw/
│
├── main.py                      # FastAPI application entry point
├── index.html                   # Frontend single-page application
├── requirements.txt             # Python dependencies
├── pyvenv.cfg                   # Virtual environment configuration
├── .env                         # Environment variables (create manually)
├── README.md                    # This documentation
└── arch.excalidraw             # Sample/fallback diagram file
│
└── gemini_excalidraw/          # Core package
    ├── __init__.py
    ├── gemini_to_excalidraw_no_mcp.py  # Main generation logic
    ├── excalidraw_rules.py             # Diagram type rules & prompts
    ├── sanitize_elements.py            # Element validation & sanitization
    └── __pycache__/                     # Python bytecode cache
```

### Key Files Description

#### `main.py`
FastAPI application server with two main endpoints:
- `GET /` - Serves the HTML frontend
- `POST /generate_excalidraw` - Processes diagram generation requests

#### `index.html`
Single-page application featuring:
- User input form
- Excalidraw React canvas integration
- API communication logic

#### `gemini_excalidraw/gemini_to_excalidraw_no_mcp.py`
Core diagram generation module:
- Gemini API client initialization
- Diagram type detection
- Element generation and mock fallback

#### `gemini_excalidraw/excalidraw_rules.py`
Diagram specification engine:
- Universal Excalidraw element rules
- Diagram type-specific system prompts
- Template definitions for 7+ diagram types

#### `gemini_excalidraw/sanitize_elements.py`
Data validation and cleanup:
- Element schema validation
- Default value injection
- Coordinate and dimension fixes

---

## 📦 Dependencies

### Core Dependencies
- **fastapi** (0.128.0) - Web framework
- **uvicorn** (latest) - ASGI server
- **google-genai** (1.64.0) - Gemini AI client
- **python-dotenv** (latest) - Environment management
- **pydantic** (latest) - Data validation

### Full Dependency List
See [requirements.txt](requirements.txt) for complete list including:
- HTTP clients (httpx, aiohttp)
- Security (cryptography)
- Data processing (fastjsonschema)
- Development tools (gitpython)

### Frontend Dependencies (CDN-loaded)
- React 19.0.0
- React DOM 19.0.0
- Excalidraw 0.18.0

---

## 📊 Analysis & Design

### Use Case Analysis

#### Primary Actors
1. **End User** - Person creating diagrams
2. **Gemini AI** - AI service generating diagram data
3. **System Administrator** - Person managing deployment

#### Use Case 1: Generate Diagram
- **Actor:** End User
- **Precondition:** Application is running, user has browser access
- **Main Flow:**
  1. User accesses application URL
  2. System displays input interface
  3. User enters diagram description
  4. User clicks "Generate Diagram"
  5. System validates input
  6. System detects diagram type
  7. System sends request to Gemini AI
  8. System receives and validates response
  9. System renders diagram on canvas
  10. User views generated diagram
- **Postcondition:** Diagram is displayed in Excalidraw canvas
- **Alternative Flow 8a:** AI response invalid
  - System logs error
  - System returns mock diagram (if available)
  - System shows error message to user

#### Use Case 2: Configure System
- **Actor:** System Administrator
- **Precondition:** Server access, environment files editable
- **Main Flow:**
  1. Admin creates/edits .env file
  2. Admin adds GEMINI_API_KEY
  3. Admin configures optional parameters
  4. Admin starts application
  5. System validates configuration
  6. System starts successfully
- **Postcondition:** Application running with valid configuration

### Domain Model

#### Key Entities

**User Prompt**
- Attributes: text (string), timestamp, length
- Responsibilities: Carry user intent

**Diagram Type**
- Attributes: name, category, system_prompt
- Responsibilities: Define generation rules
- Types: Sequence, Flowchart, Architecture, ER, Network, Timeline, MindMap

**Excalidraw Element**
- Attributes: id, type, x, y, width, height, strokeColor, etc.
- Responsibilities: Represent visual component
- Types: rectangle, ellipse, diamond, arrow, line, text

**AI Response**
- Attributes: elements (array), status, error_message
- Responsibilities: Transport generated diagram data

### Sequence Diagrams

#### Diagram Generation Flow
```
User          Frontend        FastAPI       DiagramDetector    Gemini AI      Sanitizer
 |               |               |                 |               |              |
 |--prompt------>|               |                 |               |              |
 |               |--POST-------->|                 |               |              |
 |               |               |--detect-------->|               |              |
 |               |               |<--type----------|               |              |
 |               |               |--generate--------------------->|              |
 |               |               |<--JSON elements-----------------|              |
 |               |               |--validate---------------------------->|
 |               |               |<--cleaned elements-------------------|
 |               |<--JSON--------|                 |               |              |
 |<--render------|               |                 |               |              |
 |               |               |                 |               |              |
```

### State Diagram: Diagram Generation

```
[Idle] --user enters prompt--> [Input Received]
       |
       v
[Validating Input] --valid--> [Detecting Diagram Type]
       |                              |
       |invalid                       v
       v                      [Selecting System Prompt]
[Show Error]                          |
                                      v
                              [Calling Gemini AI]
                                      |
                      +---------------+----------------+
                      |                                |
                    success                          error
                      |                                |
                      v                                v
              [Sanitizing Elements]            [Loading Mock Data]
                      |                                |
                      v                                |
              [Rendering Diagram] <--------------------+
                      |
                      v
                   [Complete]
```

### Data Flow Diagram (Level 0)

```
                     +------------------------+
                     |                        |
User Prompt -------->|  Diagram Generation    |-------> Excalidraw Elements
                     |  System                |
API Key ------------>|                        |
                     +------------------------+
                              |
                              v
                     Gemini AI Service (External)
```

### Quality Attributes Scenarios

#### Performance Scenario
- **Source:** End User
- **Stimulus:** Submits diagram prompt
- **Environment:** Normal operation, 5 concurrent users
- **Response:** System generates and renders diagram
- **Measure:** 90% of requests complete within 8 seconds

#### Security Scenario
- **Source:** Malicious User
- **Stimulus:** Attempts to access API without key
- **Environment:** Production deployment
- **Response:** System rejects request, logs attempt
- **Measure:** 100% of unauthorized requests blocked

#### Reliability Scenario
- **Source:** Gemini API
- **Stimulus:** Returns malformed JSON
- **Environment:** Normal operation
- **Response:** System catches error, uses fallback
- **Measure:** Zero application crashes

---

## 📚 API Documentation

### REST Endpoints

#### GET /
**Description:** Serves the frontend HTML application

**Request:**
```http
GET / HTTP/1.1
Host: localhost:8000
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: text/html

<!DOCTYPE html>
<html>
...
</html>
```

#### POST /generate_excalidraw
**Description:** Generates diagram from user prompt

**Request:**
```http
POST /generate_excalidraw HTTP/1.1
Host: localhost:8000
Content-Type: application/x-www-form-urlencoded

prompt=Create+a+login+flowchart
```

**Success Response (200):**
```json
{
  "elements": [
    {
      "id": "el1",
      "type": "rectangle",
      "x": 100,
      "y": 100,
      "width": 160,
      "height": 60,
      ...
    },
    ...
  ]
}
```

**Error Response (500):**
```json
{
  "error": "Error description",
  "elements": []
}
```

---

## 🔧 Troubleshooting

### Common Issues & Solutions

#### Issue 1: "GEMINI_API_KEY not found"
**Symptoms:** Application fails to start or returns API errors

**Solution:**
1. Verify `.env` file exists in project root
2. Check `GEMINI_API_KEY` is set correctly
3. Ensure no extra spaces around the key
4. Restart the application

#### Issue 2: "Module not found" errors
**Symptoms:** Import errors when starting application

**Solution:**
```bash
# Reinstall dependencies
pip install -r requirements.txt --force-reinstall

# Verify virtual environment is activated
# Windows: venv\Scripts\activate
# macOS/Linux: source venv/bin/activate
```

#### Issue 3: Diagram not rendering
**Symptoms:** Frontend shows error or blank canvas

**Solution:**
1. Check browser console for JavaScript errors (F12)
2. Verify API response in Network tab
3. Clear browser cache
4. Try a different browser

#### Issue 4: Slow generation (>15 seconds)
**Symptoms:** Long wait times for diagram generation

**Possible Causes & Solutions:**
- **Network latency:** Check internet connection
- **Complex prompts:** Simplify description
- **API rate limits:** Wait and retry
- **Server overload:** Restart application

#### Issue 5: "Address already in use"
**Symptoms:** Port 8000 is already occupied

**Solution:**
```bash
# Find process using port 8000
# Windows:
netstat -ano | findstr :8000
taskkill /PID <process_id> /F

# macOS/Linux:
lsof -i :8000
kill -9 <process_id>

# Or change port in main.py
```

### Debug Mode
Enable detailed logging by modifying `main.py`:
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

### Getting Help
- Check existing issues: [Project Issues](#)
- Documentation: This README
- Community: [Discussions](#)

---

## 👨‍💻 Development

### Development Setup
```bash
# Install development dependencies (if separate)
pip install -r requirements-dev.txt  # If available

# Run in development mode with auto-reload
fastapi dev main.py
```

### Code Style Guidelines
- Follow PEP 8 for Python code
- Use type hints where applicable
- Document functions with docstrings
- Keep functions focused and modular

### Testing (Future Enhancement)
```bash
# Run unit tests
pytest tests/

# Run with coverage
pytest --cov=gemini_excalidraw tests/
```

### Adding New Diagram Types
1. Open `gemini_excalidraw/excalidraw_rules.py`
2. Add detection pattern to `detect_diagram_type()`
3. Create new system prompt following existing patterns
4. Update `get_system_prompt()` function
5. Test with sample prompts

### Contributing
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-diagram-type`)
3. Commit changes (`git commit -am 'Add new diagram type'`)
4. Push to branch (`git push origin feature/new-diagram-type`)
5. Create Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## 📞 Support & Contact

- **Documentation:** This README file
- **Issues:** GitHub Issues
- **Version:** 1.0.0
- **Last Updated:** February 20, 2026

---

## 🔄 Version History

### Version 1.0.0 (February 20, 2026)
- Initial release
- Support for 7 diagram types
- FastAPI backend implementation
- Excalidraw integration
- Gemini AI integration
- Element sanitization system

---

**Note:** This project requires an active Google Gemini API key. Usage may be subject to Google's API quotas and pricing.
