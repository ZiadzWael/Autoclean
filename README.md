# AutoClean v1

> Professional Data Cleaning & Validation System with AI-Powered Rule Extraction

AutoClean PH1 is an intelligent data quality platform that automatically discovers, 
validates, and applies data cleaning rules using advanced LLM capabilities. Built with 
Phi-3 AI model, it provides real-time rule extraction, conflict detection, and automated 
data cleaning with a modern, intuitive interface.

## Key Features

- **AI-Powered Rule Discovery**: Automatically extract data quality rules using Phi-3 LLM
- **Real-Time Processing**: WebSocket-enabled live updates during data processing
- **Comprehensive Validation**: Multi-stage data validation with detailed violation reports
- **Intelligent Curation**: LLM-based rule ranking and conflict resolution
- **Rule Versioning**: Track rule changes and maintain rule version history
- **Domain-Specific Rules**: Built-in finance and business rule extractors
- **Production Ready**: Scalable FastAPI backend with React frontend

## Tech Stack

- **Backend**: FastAPI, Python 3.11, PyTorch, Phi-3-mini LLM
- **Frontend**: React 18, Vite, WebSocket
- **Storage**: ChromaDB (vector database), Local file storage
- **GPU Support**: CUDA-enabled PyTorch for faster inference

#How to run
(once you open the folder open terminal)
1.python -m venv venv
2.venv\Scripts\activate
3.pip install -r backend/app/requirements.txt
4.python -m uvicorn backend.app.main:app --host 127.0.0.1 --port 8001 --reload

(click on frontend folder and open another new terminal)
1.rm -r node_modules
2.rm package-lock.json
3.npm install
4.npm run dev (if for server)

(do not close any terminal of both)
