# Multi-Agent RAG Enhanced Text-to-SQL System

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.9+-blue.svg" alt="Python">
  <img src="https://img.shields.io/badge/LangGraph-Multi_Agent-green.svg" alt="LangGraph">
  <img src="https://img.shields.io/badge/FAISS-Vector_Store-orange.svg" alt="FAISS">
  <img src="https://img.shields.io/badge/Ollama-LLM-purple.svg" alt="Ollama">
  <img src="https://img.shields.io/badge/SQLglot-SQL_Processing-red.svg" alt="SQLglot">
</p>

## 🎯 Project Overview

A **Multi-Agent System** that converts natural language questions into SQL queries and executes them on databases. The system implements and compares **three different approaches**:

| Pipeline | Description | Use Case |
|----------|-------------|----------|
| **Single Agent** | Single LLM call for SQL generation | Baseline comparison |
| **Multi-Agent (No RAG)** | 5 specialized agents with direct schema access | Agent orchestration |
| **Multi-Agent (RAG)** | 5 agents + semantic schema retrieval | Full system with RAG |

### Key Features

- 🤖 **5 Specialized Agents**: Planner, Schema, SQL, Executor, Evaluator
- 🔍 **3 RAG Methods**: Dense (FAISS), Sparse (BM25), Hybrid
- 📊 **Comprehensive Evaluation**: Execution accuracy, component matching, charts
- 🌐 **Interactive Web UI**: Streamlit-based interface
- 📈 **Benchmark Testing**: Spider dataset with 5 databases

---

## 📊 System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    USER QUESTION                             │
│         "How many countries have a republic government?"     │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                   PLANNER AGENT                              │
│  • Analyzes the question                                     │
│  • Determines query type (SELECT, COUNT, AGGREGATE, etc.)   │
│  • Predicts required tables                                  │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                   SCHEMA AGENT                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │    Dense     │  │    Sparse    │  │    Hybrid    │       │
│  │   (FAISS)    │  │    (BM25)    │  │   (Dense+    │       │
│  │              │  │              │  │    Sparse)   │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│  Retrieves relevant schema information                       │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    SQL AGENT                                 │
│  • Receives schema + plan + question                         │
│  • Generates executable SQL with few-shot prompting          │
│  • SQLglot post-processing for validation                    │
│  • Retry mechanism on errors                                 │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                  EXECUTOR AGENT                              │
│  • Executes SQL on SQLite database                           │
│  • Security checks (no DROP, DELETE, etc.)                   │
│  • Formats and returns results                               │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                  EVALUATOR AGENT                             │
│  • Evaluates SQL correctness                                 │
│  • Calculates execution, syntax, semantic scores             │
│  • Provides feedback for improvement                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 RAG Schema Retrieval Methods

### 1. Dense Retrieval (FAISS + Sentence Embeddings)
- Uses semantic similarity between query and schema
- High natural language understanding capability
- Does not require exact keyword matching
- Model: `all-MiniLM-L6-v2`

### 2. Sparse Retrieval (BM25)
- Keyword-based retrieval algorithm
- Fast and efficient
- Effective for technical terms and exact matches

### 3. Hybrid Retrieval (Recommended)
- Combines Dense + Sparse with weighted fusion
- Default weights: Dense 60%, Sparse 40%
- Best balance between semantic and keyword matching

---

## 🛠️ Installation

### Prerequisites

```bash
# Python 3.9+ required
python --version

# Install Ollama (Local LLM)
# Windows: Download from https://ollama.ai/download
# Mac/Linux: curl -fsSL https://ollama.ai/install.sh | sh

# Download the LLM model
ollama pull llama3.2:3b
```

### Project Setup

```bash
# 1. Clone the repository
git clone https://github.com/your-username/TextToSQL_Agentic_AI.git
cd TextToSQL_Agentic_AI

# 2. Create virtual environment
python -m venv venv

# 3. Activate virtual environment
# Windows (PowerShell):
.\venv\Scripts\Activate
# Windows (CMD):
venv\Scripts\activate.bat
# Linux/Mac:
source venv/bin/activate

# 4. Install dependencies
pip install -r requirements.txt
```

### Spider Dataset Setup

The Spider dataset is already included in `data/spider_data/`. If you need to download it manually:

```bash
# Download from: https://yale-lily.github.io/spider
# Extract to: data/spider_data/spider_data/
# Structure:
#   data/spider_data/spider_data/database/
#   data/spider_data/spider_data/dev.json
#   data/spider_data/spider_data/train_spider.json
```

---

## 🚀 Quick Start

### Option 1: Web Interface (Recommended)

```bash
# Start the Streamlit web application
streamlit run app.py
```

This opens a browser at `http://localhost:8501` with:
- Database selection (World, Concert Singer, Student Transcripts, Dog Kennels, Car)
- Pipeline selection (Single, Multi, Multi+RAG, Compare All)
- Sample questions to try
- Visual results with charts

### Option 2: Command Line

```bash
# Single query with RAG pipeline
python main.py --mode rag --question "How many countries are there?" --db "data/spider_data/spider_data/database/world_1/world_1.sqlite"

# Compare all pipelines on one question
python main.py --compare --question "What countries became independent after 1950?" --db "data/spider_data/spider_data/database/world_1/world_1.sqlite"

# Interactive mode
python main.py --interactive
```

### Option 3: Full Evaluation (All 5 Databases)

```bash
# Quick test (5 questions per database = 25 total, ~20-30 min)
python full_evaluation.py --limit 5

# Full evaluation (~195 questions, ~3-4 hours)
python full_evaluation.py

# Single database only
python full_evaluation.py --db world_1
```

---

## 📁 Project Structure

```
TextToSQL_Agentic_AI/
│
├── agents/                          # Multi-Agent Components
│   ├── __init__.py
│   ├── base_agent.py               # Base agent class
│   ├── planner_agent.py            # Question analysis & planning
│   ├── schema_agent.py             # Schema extraction/retrieval
│   ├── sql_agent.py                # SQL generation with few-shot
│   ├── executor_agent.py           # SQL execution
│   └── evaluator_agent.py          # Result evaluation
│
├── pipelines/                       # Pipeline Implementations
│   ├── __init__.py
│   ├── single_agent_pipeline.py    # Single LLM call (baseline)
│   ├── multi_agent_no_rag_pipeline.py  # Multi-agent without RAG
│   └── multi_agent_rag_pipeline.py     # Multi-agent with RAG
│
├── rag/                             # RAG Components
│   ├── __init__.py
│   ├── faiss_store.py              # FAISS vector store
│   ├── schema_embeddings.py        # Embedding generation + SQL patterns
│   └── retrieval_methods.py        # Dense/Sparse/Hybrid retrievers
│
├── evaluation/                      # Evaluation Framework
│   ├── __init__.py
│   ├── schema_retrieval_comparison.py
│   └── pipeline_evaluation.py
│
├── utils/                           # Utilities
│   ├── __init__.py
│   ├── database.py                 # SQLite management
│   ├── llm_client.py               # Ollama client
│   └── sql_utils.py                # SQL evaluation utilities
│
├── config/
│   └── config.py                   # Configuration settings
│
├── data/
│   └── spider_data/                # Spider benchmark dataset
│
├── results/                         # Evaluation Results
│   ├── world_1/                    # Per-database results
│   ├── concert_singer/
│   ├── student_transcripts_tracking/
│   ├── dog_kennels/
│   ├── car_1/
│   └── combined/                   # Cross-database comparison (all 5)
│
├── app.py                          # Streamlit Web UI
├── main.py                         # CLI entry point
├── full_evaluation.py              # Full benchmark evaluation
├── requirements.txt                # Python dependencies
└── README.md                       # This file
```

---

## 📊 Evaluation Databases

The system is evaluated on **5 databases** from the Spider benchmark:

| Database | Description | Tables | Questions |
|----------|-------------|--------|-----------|
| **world_1** | Countries, cities, languages | 3 | ~120 |
| **concert_singer** | Concerts, singers, stadiums | 4 | ~45 |
| **student_transcripts_tracking** | Students, courses, grades | 11 | ~78 |
| **dog_kennels** | Dogs, breeds, kennels, treatments | 8 | ~46 |
| **car_1** | Car makers, models, countries | 6 | ~64 |

---

## 📈 Evaluation Metrics

### Execution Metrics
- **Execution Accuracy**: Does the generated SQL produce correct results?
- **Exact Set Match**: Do the results exactly match the gold standard?
- **Execution Valid Rate**: Is the SQL syntactically valid and executable?

### Component Matching
Component matching evaluates SQL structure by comparing individual SQL components between generated and gold standard queries:

- **SELECT Match**: Correct aggregate functions used (COUNT, SUM, AVG, MAX, MIN, etc.)?
- **FROM Match**: Correct tables used in the query?
- **WHERE Match**: Correct columns referenced in WHERE conditions?
- **GROUP BY Match**: Correct columns used for grouping?
- **ORDER BY Match**: Correct columns used for sorting?

The overall component score is the average of all 5 component matches, providing a detailed breakdown of SQL structure accuracy.

### Performance
- **Average Time**: Time per question (seconds)
- **Total Time**: Complete evaluation time

---

## 📊 Results Output Structure

After running `full_evaluation.py`, results are saved as:

```
results/
├── world_1/
│   ├── detailed_results.json       # Every question with all 3 pipelines
│   ├── summary_metrics.json        # Aggregated metrics (JSON)
│   ├── summary_metrics.csv         # Aggregated metrics (CSV)
│   ├── evaluation_results.csv      # All results in CSV
│   ├── evaluation_report.html      # Beautiful HTML report
│   ├── evaluation_report.md        # Markdown report
│   └── charts/
│       ├── accuracy_comparison.png
│       ├── time_comparison.png
│       ├── component_breakdown.png
│       └── accuracy_by_difficulty.png
│
├── concert_singer/                  # Same structure
├── student_transcripts_tracking/    # Same structure
├── dog_kennels/                     # Same structure
├── car_1/                           # Same structure
│
└── combined/                        # Cross-database comparison (all 5 databases)
    ├── combined_summary.json
    ├── all_databases_comparison.csv
    ├── all_databases_comparison.html
    └── charts/
        ├── combined_accuracy.png
        ├── per_database_comparison.png
        ├── combined_time.png
        └── accuracy_heatmap.png
```

---

## 🔧 Configuration

Edit `config/config.py` to customize:

```python
# LLM Settings
OLLAMA_MODEL = "llama3.2:3b"  # or codellama, mistral, etc.

# RAG Settings
RAG_TOP_K = 5                      # Number of schema chunks to retrieve
EMBEDDING_MODEL = "all-MiniLM-L6-v2"

# Hybrid Retrieval Weights
DENSE_WEIGHT = 0.6
SPARSE_WEIGHT = 0.4

# SQL Generation
SQL_MAX_RETRIES = 3
SQL_TEMPERATURE = 0.1
```

---

## 🚀 Usage Examples

### 1. Web Interface (Streamlit)

```bash
streamlit run app.py
```

Features:
- Select from 5 featured databases
- Choose pipeline (Single, Multi, Multi+RAG, or Compare All)
- Click sample questions or type your own
- View generated SQL and results
- Batch evaluation mode

### 2. Command Line Interface

```bash
# Basic query
python main.py --mode rag --question "How many countries are there?" \
  --db "data/spider_data/spider_data/database/world_1/world_1.sqlite"

# Compare all pipelines
python main.py --compare --question "List countries in Europe" \
  --db "data/spider_data/spider_data/database/world_1/world_1.sqlite"

# Different retrieval methods
python main.py --mode rag --retrieval dense --question "..."
python main.py --mode rag --retrieval sparse --question "..."
python main.py --mode rag --retrieval hybrid --question "..."
```

### 3. Full Benchmark Evaluation

```bash
# All 5 databases (recommended for final evaluation)
python full_evaluation.py

# Quick test
python full_evaluation.py --limit 5

# Include hard questions
python full_evaluation.py --include-hard

# Specific database
python full_evaluation.py --db world_1
python full_evaluation.py --db concert_singer
python full_evaluation.py --db student_transcripts_tracking
python full_evaluation.py --db dog_kennels
python full_evaluation.py --db car_1
```

### 4. Interactive Mode

```bash
python main.py --interactive

# Then in interactive mode:
> set db data/spider_data/spider_data/database/world_1/world_1.sqlite
> set mode rag
> How many countries have population over 100 million?
> What languages are spoken in Germany?
> exit
```

---

## 📋 Command Reference

| Command | Description |
|---------|-------------|
| `streamlit run app.py` | Start web interface |
| `python main.py --interactive` | Interactive CLI mode |
| `python main.py --compare --question "..." --db ...` | Compare all pipelines |
| `python full_evaluation.py` | Full benchmark (5 DBs) |
| `python full_evaluation.py --limit 5` | Quick test (25 questions) |
| `python full_evaluation.py --db world_1` | Single database evaluation |

---

## 🏆 Expected Results

Based on the Spider benchmark evaluation across 5 databases (284 questions):

| Pipeline | Execution Accuracy | Avg Time/Question | Notes |
|----------|-------------------|-------------------|-------|
| Single Agent | ~48.6% | ~3.3s | Baseline, often adds unnecessary JOINs |
| Multi-Agent | ~53.2% | ~10.0s | Better SQL structure, improved accuracy |
| Multi-Agent RAG | ~60.6% | ~18.1s | Best overall with schema retrieval |

*Results may vary based on LLM model and question complexity.*

---

## 🛡️ Technologies Used

- **LangGraph**: Multi-agent orchestration and state management
- **FAISS**: Fast vector similarity search for dense retrieval
- **BM25**: Sparse keyword-based retrieval
- **Ollama**: Local LLM inference (llama3.2:3b)
- **Sentence Transformers**: Text embeddings (all-MiniLM-L6-v2)
- **SQLglot**: SQL parsing, validation, and optimization
- **Streamlit**: Interactive web interface
- **Spider Dataset**: Text-to-SQL benchmark

---

## 📄 License

MIT License - See [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- [Spider Dataset](https://yale-lily.github.io/spider) - Text-to-SQL benchmark
- [Ollama](https://ollama.ai/) - Local LLM inference
- [LangGraph](https://github.com/langchain-ai/langgraph) - Agent orchestration
- [FAISS](https://github.com/facebookresearch/faiss) - Vector search
- [SQLglot](https://github.com/tobymao/sqlglot) - SQL parsing

---

## 📞 Support

For issues or questions:
1. Check existing issues on GitHub
2. Create a new issue with detailed description
3. Include error logs and system information

---

<p align="center">
  <strong>Built with ❤️ for Natural Language to SQL Research</strong>
</p>
