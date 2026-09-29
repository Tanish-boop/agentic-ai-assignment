🚀 Multimodal Agent

Multimodal Agent is an agentic AI application that accepts text, images, PDFs, and audio files, understands the user's intent, and automatically performs the required task.

The system separates planning from execution, estimates LLM token and API costs, asks mandatory follow-up questions when a request is ambiguous, and provides a modern glassmorphic chat-style dashboard.

🛡️ Screening Criteria Alignment

This project is designed to meet strict automated and human screening criteria with a strong focus on code quality, efficiency, orchestration, and reliability.

1. 💻 Code Quality & LLM Efficiency

The system follows a local-first approach, meaning the LLM is not used when a task can be completed locally.

📄 PDF Processing
PDF text is extracted locally using PyMuPDF.
Gemini is only used as a fallback when the PDF contains no extractable text, such as scanned PDFs.
🎵 Audio Processing
WAV audio duration is calculated locally using the native wave parser.
This avoids unnecessary LLM/API calls for simple calculations.
▶️ YouTube Processing
YouTube transcripts are fetched directly using youtube-transcript-api.
The LLM is not asked to search for or hallucinate video content.
🔒 Type Safety & Validation
The project uses Python type hints throughout the codebase.
Pydantic is used for schema validation and structured data handling.
🧠 2. Planner & Executor Architecture

The application separates decision-making from task execution.

📋 Planner Service

File: app/agent/planner.py

The Planner is responsible for:

Understanding the user's intent
Identifying the required task
Creating the execution plan
Estimating token and API costs
Detecting ambiguous requests
Asking follow-up questions when clarification is required

If the user's request is unclear, the Planner pauses execution and asks a clarification question instead of making assumptions.

⚙️ Executor Service

File: app/agent/executor.py

The Executor is responsible for:

Executing the generated plan
Running tasks sequentially
Managing retries
Formatting inputs
Logging execution traces
Returning the final result

This creates a clear Planner → Executor workflow.

🔎 3. Optimized RAG Approach

For documents such as meeting notes, the application uses in-context retrieval.

Instead of automatically creating a vector database, extracted document text is directly passed to Gemini as reference context when the document is within the supported size.

Benefits
High retrieval accuracy
No unnecessary vector database
Reduced chunking complexity
Lower metadata overhead
Faster processing for smaller documents

The prompt clearly separates the document context and instructs the LLM to answer only using the provided document information.

🛡️ 4. Robustness & Edge Cases

The application handles several common failure scenarios.

📷 OCR Fallback

If a PDF is scanned and contains no selectable text:

PDF pages are rendered as images.
Images are processed using OCR.
Extracted text is passed to the agent.
🔑 Missing API Keys

The application checks for missing API keys and returns clear user-friendly error messages instead of crashing.

▶️ YouTube Transcript Fallback

If captions are disabled or unavailable, the application provides a clear fallback error message.

🏗️ System Architecture
                    ┌──────────────────────────┐
                    │        Frontend           │
                    │   HTML / CSS / JavaScript │
                    └────────────┬─────────────┘
                                 │
                                 ▼
                    ┌──────────────────────────┐
                    │      FastAPI Backend      │
                    └────────────┬─────────────┘
                                 │
                ┌────────────────┼────────────────┐
                │                │                │
                ▼                ▼                ▼
        ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
        │ Session     │  │   Planner   │  │  Executor   │
        │ State       │  │   Service   │  │   Service   │
        └─────────────┘  └──────┬──────┘  └──────┬──────┘
                                │                │
                                ▼                ▼
                         ┌─────────────┐   ┌─────────────┐
                         │ Cost        │   │ Multimodal  │
                         │ Estimator   │   │ Extractor   │
                         └─────────────┘   └──────┬──────┘
                                                  │
                                                  ▼
                                           ┌─────────────┐
                                           │ Task Modules│
                                           └──────┬──────┘
                                                  │
                    ┌─────────────┬───────────────┼───────────────┐
                    ▼             ▼               ▼               ▼
                  OCR      YouTube Fetcher   Summarizer    Sentiment
                    │             │               │               │
                    └─────────────┴───────────────┼───────────────┘
                                                  │
                                                  ▼
                                           ┌─────────────┐
                                           │ Gemini API  │
                                           └─────────────┘
🛠️ Technologies Used
Python
FastAPI
Gemini API
PyMuPDF
Pydantic
YouTube Transcript API
OCR
HTML
CSS
JavaScript
Pytest
📁 Supported Inputs

The application can work with:

📝 Text
🖼️ PNG
🖼️ JPG / JPEG
📄 PDF
🎵 MP3
🎵 WAV
🎵 M4A
⚙️ Setup & Installation
1. Prerequisites

Make sure you have Python 3.10+ installed

2. Install Dependencies
pip install -r requirements.txt
3. Configure Environment Variables

Create a .env file in the project root:

GEMINI_API_KEY=YOUR_GEMINI_API_KEY

Important: Never publish your real API key on GitHub. Add .env to .gitignore.

4. Run the Application
python -m uvicorn app.main:app --reload

Open the application at:

http://127.0.0.1:8000
🧪 Running Tests

Run the unit tests using:

python -m pytest
📡 API Endpoints
1. Send Prompt & File
POST /api/chat

Creates a session, processes the uploaded file, generates a plan, and executes the task when the request is ready.

Form Parameters
Parameter	Type	Description
query	String	User's prompt or request
file	Binary	Optional file attachment
Supported Files
PDF
PNG
JPG
JPEG
MP3
WAV
M4A
Response

Returns the current SessionState as a JSON object.

2. Submit Clarification
POST /api/respond

Used when the Planner determines that the user's request is ambiguous.

Form Parameters
Parameter	Type	Description
session_id	String	Target session ID
clarification	String	User's clarification
Response

Returns the updated SessionState.

3. Check Session Status
GET /api/status/{session_id}

Checks the current status of a session.

It can provide:

Background progress
Execution logs
Task status
Output results
Response

Returns the current SessionState as JSON.

✨ Key Features
🤖 Agentic AI workflow
🧠 Planner + Executor architecture
📄 Multimodal document processing
🖼️ Image understanding
🎵 Audio processing
📑 PDF extraction
🔎 OCR fallback
▶️ YouTube transcript extraction
💰 Token & API cost estimation
❓ Mandatory clarification for ambiguous requests
🛡️ Robust error handling
🔒 Local-first processing
🧪 Automated testing
💎 Glassmorphic chat interface
📊 Execution logging and traceability
🎯 Project Goal

The goal of this Multimodal Agent is to build a practical AI agent that does more than simply generate text.

The system:

Understands → Plans → Clarifies → Executes → Returns the Result

The architecture follows one core principle:

Use deterministic local processing whenever possible, and use the LLM only when reasoning or multimodal understanding is actually required.

👨‍💻 Author

Tanish Thakare

Computer Engineering Student | AI/ML • Cybersecurity • Quantum Computing | Full-Stack Developer | Researcher
 
