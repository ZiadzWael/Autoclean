# AutoClean – Production‑Ready Data Cleaning Assistant

AutoClean is a fully local, end‑to‑end data cleaning system that combines LLM‑driven reasoning with rule‑based validation.  It can detect, explain and fix data quality issues in tabular datasets via a rich web interface.  The **production‑ready** version goes beyond basic heuristics: it can discover approximate and conditional functional dependencies, correlated missing value patterns, and domain‑specific formats (email, phone, dates, credit cards).  The frontend has been refactored into a multi‑step wizard with interactive rule management, charts summarising the cleaning operations, and export options for CSV and Excel.

## Features

* **Heuristic and domain‑aware rule extraction** – infer not‑null, unique and numeric range constraints from the data profile.  Additional heuristics derive:
  - **Categorical constraints** for low‑cardinality columns
  - **Generic regex patterns** describing allowed characters
  - **Domain‑specific regex patterns** for email addresses, phone numbers, ISO/UK date formats and credit card numbers
  - **Exact and approximate functional dependencies** (``A → B``) where occasional violations are tolerated
  - **Conditional functional dependencies** (``A → B | C``) that hold only within subsets defined by a condition column
  - **Statistical outlier thresholds** (mean ± *z*·σ)
  - **Case standardisation suggestions** to unify uppercase/lowercase strings
  - **Inclusion dependencies** where one column’s values form a subset of another’s
  - **Correlated missing value patterns** indicating that when column ``A`` is missing, column ``B`` should also be missing

* **Interactive rule management** – extracted rules are stored with unique IDs.  The UI allows you to enable/disable or delete individual rules and apply only a selected subset during cleaning.  You can update a rule via a PUT request.

* **Validation and audit** – validate datasets against any stored rule set.  Violations are counted per rule.  After cleaning, the system returns the cleaned dataset plus a detailed log of every cell change, grouped by rule ID.

* **Rich visualisation** – the multi‑step interface displays extracted rules, validation results, and cleaning modifications.  A bar chart summarises how many changes were triggered by each rule type.

* **Export options** – download cleaned data in CSV or Excel format.  Audit logs can be exported via the API.

* **Multi‑step workflow** – the frontend guides you through uploading a dataset, extracting and managing rules, validating the data, and applying selected rules.  Progress indicators help users understand where they are in the process.

* **Extensible architecture** – modular services for profiling, rule extraction, application and feedback.  The rule engine and LLM client can be swapped or extended with domain‑specific logic.  The project lays the foundation for caching results, streaming large datasets and adding authentication or multi‑user support.

## Requirements

- **Python** ≥ 3.11  
- **Node.js** ≥ 18 (for the React frontend)  
- **GPU** with at least 4 GB VRAM for Phi‑3‑mini (or change to a smaller model)  
- Local file system access for storing rules and vector indices

## Installation

1. **Clone or extract** this repository.  The tree should look like:

    ```
    AUTOCLEAN_PH1/
    ├── backend/                          # FastAPI application
    │   ├── app/
    │   │   ├── api/                      # REST & WebSocket endpoints
    │   │   │   ├── __init__.py          # Router aggregation
    │   │   │   ├── websocket.py         # Real-time WebSocket updates
    │   │   │   ├── health.py            # Health check endpoint
    │   │   │   ├── ingest.py            # Dataset upload
    │   │   │   ├── extract.py           # Rule extraction
    │   │   │   ├── validate.py          # Data validation
    │   │   │   ├── apply.py             # Rule application
    │   │   │   ├── feedback.py          # User feedback collection
    │   │   │   ├── rules.py             # Rule management
    │   │   │   ├── profile.py           # Data profiling
    │   │   │   ├── actions.py           # Custom actions
    │   │   │   └── ...
    │   │   ├── services/                 # Business logic & LLM services
    │   │   │   ├── llm/                 # LLM clients and planning
    │   │   │   │   ├── client.py        # Base LLM client
    │   │   │   │   ├── phi3_client.py   # Phi3 model implementation
    │   │   │   │   ├── phi3_extractor_planner.py  # Extractor selection
    │   │   │   │   ├── rule_curator.py  # Rule curation
    │   │   │   │   └── prompts.py       # LLM prompts
    │   │   │   ├── rag/                 # Retrieval-Augmented Generation
    │   │   │   │   ├── indexer.py       # Vector index builder
    │   │   │   │   └── retriever.py     # Knowledge retrieval
    │   │   │   ├── profiler.py          # Data profiling logic
    │   │   │   ├── rule_extractor.py    # Rule discovery algorithms
    │   │   │   ├── rule_validator.py    # Rule validation
    │   │   │   ├── rule_store.py        # Rule persistence
    │   │   │   ├── apply_engine.py      # Rule application & cleaning
    │   │   │   ├── rule_conflict_detector.py  # Conflict detection
    │   │   │   ├── rule_converter.py    # Rule format conversion
    │   │   │   ├── rule_scorer.py       # Rule ranking & scoring
    │   │   │   ├── rule_versioning.py   # Version control for rules
    │   │   │   ├── rule_explainer.py    # Rule explanation
    │   │   │   ├── feedback_engine.py   # Feedback processing
    │   │   │   ├── finance_rules.py     # Domain-specific finance rules
    │   │   │   ├── canonical_schema.py  # Schema management
    │   │   │   ├── column_mapper.py     # Column mapping utilities
    │   │   │   ├── metrics.py           # Performance metrics
    │   │   │   ├── storage.py           # File storage operations
    │   │   │   ├── auth.py              # Authentication (future)
    │   │   │   ├── suggest.py           # Suggestion engine
    │   │   │   ├── validator.py         # General validation
    │   │   │   ├── api.js               # Frontend API client (legacy)
    │   │   │   └── ...
    │   │   ├── db/                       # Database models (ORM)
    │   │   │   ├── base.py              # Base model configuration
    │   │   │   └── models.py            # SQLAlchemy models
    │   │   ├── config.py                # Configuration constants
    │   │   ├── main.py                  # FastAPI entry point
    │   │   └── requirements.txt         # Python dependencies
    │   └── scripts/                      # ML/utility scripts
    │       └── phi3_gpu_working.py      # Phi3 GPU testing
    │
    ├── frontend/                         # React + Vite frontend
    │   ├── src/
    │   │   ├── App.jsx                  # Main app component
    │   │   ├── App.css                  # Global styles & design system
    │   │   ├── main.jsx                 # React entry point
    │   │   ├── index.css                # Base styles
    │   │   └── services/
    │   │       └── api.js               # API client library
    │   ├── index.html                   # HTML template
    │   ├── package.json                 # Frontend dependencies
    │   ├── vite.config.js               # Vite configuration
    │   └── ...
    │
    ├── data/                            # Local data storage
    │   ├── storage/                     # Uploaded datasets
    │   ├── rules/                       # Extracted rule definitions (JSON)
    │   └── chroma_db/                   # ChromaDB vector database
    │
    ├── ml/                              # Machine Learning scripts
    │   ├── scripts/
    │   │   └── phi3_gpu_working.py      # Phi3 GPU verification
    │   └── __init__.py
    │
    ├── README.md                        # This file
    ├── __init__.py                      # Package root
    └── ...
    ```

---

## Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   WEB BROWSER                               │
│              (http://localhost:3000)                         │
└──────────────────────────┬──────────────────────────────────┘
                           │
                    HTTP + WebSocket
                           │
        ┌──────────────────┴──────────────────┐
        │                                     │
┌───────▼──────────────────┐     ┌───────────▼──────────────┐
│   FRONTEND (React 18)     │     │  BACKEND (FastAPI)       │
│   ─────────────────       │     │  ─────────────────       │
│ • App.jsx                 │     │ • REST API Routes        │
│ • Multi-step UI wizard    │     │ • WebSocket endpoint     │
│ • Real-time updates       │     │ • Rule extraction        │
│ • Interactive validation  │     │ • Data cleaning          │
│ • Export options          │     │ • Conflict resolution    │
│                           │     │                          │
│ Port: 3000                │     │ Port: 8001               │
└───────┬──────────────────┘     └───────────┬──────────────┘
        │                                    │
        │                        ┌───────────┴────────────┬──────────┐
        │                        │                        │          │
        │              ┌─────────▼─────────┐  ┌──────────▼────┐ ┌─────▼──────┐
        │              │  LLM Services     │  │  Data Storage │ │   Vector   │
        │              │  ─────────────    │  │  ──────────   │ │   Database │
        │              │ • Phi-3 Model     │  │ • CSV files   │ │ ──────────  │
        │              │ • Rule extraction │  │ • JSON rules  │ │ • ChromaDB  │
        │              │ • Rule curation   │  │ • Audit logs  │ │            │
        │              │ • RAG system      │  │               │ │            │
        │              │                   │  │               │ │            │
        └──────────────►                   │  │               │ │            │
                       │ (async requests)  │  │               │ │            │
                       │                   │  │               │ │            │
                       └───────────────────┘  └───────────────┘ └────────────┘
```

### Data Flow

```
User Upload
    ↓
[1] INGEST SERVICE
    • Read CSV/JSON
    • Store in data/storage/
    • Assign dataset_id
    ↓
[2] PROFILE SERVICE
    • Analyze columns
    • Generate statistics
    • Create profile object
    ↓
[3] EXTRACT SERVICE
    • Use LLM Planner to select extractors
    • Run extractors in parallel
    • Extract multiple rule types
    ↓
[4] RULE PROCESSING
    • Rank rules by confidence
    • Curate with LLM
    • Detect conflicts
    • Resolve conflicts
    • Filter low-confidence rules
    ↓
[5] RULE STORAGE
    • Serialize to JSON
    • Save in data/rules/
    • Index in vector DB
    ↓
[6] VALIDATION SERVICE
    • Apply rules to dataset
    • Detect violations
    • Generate violation report
    ↓
[7] APPLY SERVICE
    • Fix violations
    • Log modifications
    • Generate cleaned dataset
    ↓
[8] EXPORT
    • CSV / Excel format
    • Audit trail
    • Summary statistics
```

### Module Dependencies

```
Frontend Dependencies:
├── React 18
├── Vite (build tool)
├── WebSocket (for real-time)
└── CSS (design system)

Backend Dependencies:
├── FastAPI (web framework)
├── Uvicorn (ASGI server)
├── PyTorch (deep learning)
├── Transformers (Phi-3 model)
├── Pandas (data processing)
├── ChromaDB (vector storage)
├── SQLAlchemy (ORM)
└── Python 3.11+
```

### Rule Extraction Pipeline

```
Raw Data
    ↓
1. extract_rules_from_profile()
   └─► Not-null, unique, range rules
    ↓
2. find_categorical_rules()
   └─► Low-cardinality constraints
    ↓
3. find_regex_rules()
   └─► String pattern matching
    ↓
4. find_fd_rules()
   └─► Functional dependencies (A→B)
    ↓
5. find_outlier_rules()
   └─► Statistical outlier detection
    ↓
6. find_standardize_rules()
   └─► Case standardization
    ↓
7. find_inclusion_rules()
   └─► Subset relationships
    ↓
8. extract_finance_rules()
   └─► Domain-specific finance rules
    ↓
Rule Ranking & Scoring
    ↓
LLM Curation
    ↓
Conflict Detection
    ↓
Final Rule Set
```

---

## Installation
    ├── data/                # Local storage for uploaded files, rules and vector DB
    ├── frontend/            # React app powered by Vite
    │   ├── src/
    │   │   ├── App.jsx      # Main UI component
    │   │   └── main.jsx     # React entrypoint
    │   ├── index.html       # HTML template
    │   ├── package.json     # Frontend dependencies
    │   └── vite.config.js   # Vite dev‑server config
    ├── ml/                  # ML scripts (e.g., Phi‑3 GPU test)
    ├── README.md            # This file
    └── ...
    ```

2. **Backend setup** (Python):
   ```bash
   # Create and activate a virtual environment (optional but recommended)
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate

   # Install Python dependencies
   pip install -r backend/app/requirements.txt

   # Start the FastAPI server on port 8001
   uvicorn backend.app.main:app --reload --port 8001
   ```

3. **Frontend setup** (Node.js):
   ```bash
   cd frontend
   npm install        # install React, Vite and related packages
   npm run dev        # launch the development server on port 3000
   ```

   The UI will be available at `http://localhost:3000`.  It proxies API requests to `http://localhost:8001`.

## Usage

1. **Upload** – Choose a CSV or JSON file and click **Upload Dataset**.  The file is stored locally and a unique dataset ID is assigned.
2. **Extract** – Click **Extract Rules** to generate heuristics from your data.  The list of rules (with IDs and types) is displayed.  Proceed to the next step.
3. **Manage** – Review the extracted rules.  Use the checkboxes to enable or disable individual rules; click the **Delete** button to remove a rule entirely.  All enabled rules will be applied by default.
4. **Validate** – Click **Validate Dataset** to see how many rows violate each rule.  Use this information to decide which rules to apply.
5. **Apply** – Click **Apply Selected Rules** to repair the dataset with the enabled rules.  The cleaned data is displayed, along with a per‑rule breakdown of modifications.  Download the cleaned data as CSV or Excel, and view a bar chart summarising the number of modifications per rule type.

## Extending

To support new rule types or customised cleaning behaviours, extend the functions in `backend/app/services/rule_extractor.py` and update the apply engine in `backend/app/services/apply_engine.py`.  Each rule should include an `id` and `type` key.  The frontend will automatically display any new rules and modifications.  For conditional or domain‑specific logic, integrate your own functions or external ontologies.  The rule store can be extended to persist metadata or version rules in a database instead of JSON files.

## Known Limitations

- The heuristics for regex, functional dependencies and inclusion dependencies are approximate and may produce false positives on complex datasets.  They should be reviewed before use in production.
- Repair actions for outliers, regex and inclusion dependencies default to clamping or nullifying values.  More sophisticated fixes could be implemented.
- The UI is built for demonstration purposes.  For production use, you may want to add authentication, persistent storage and a more robust state management solution.

## License

This project is provided “as is” for educational purposes.  Feel free to adapt it to your needs.
