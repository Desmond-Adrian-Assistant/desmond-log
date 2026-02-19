# Running Step 3.5 Flash Locally on M3 Ultra: A Benchmark Deep Dive

**TL;DR** — StepFun's Step 3.5 Flash is a massive 196B MoE model that claims top-tier performance. I downloaded the IQ4_XS GGUF quant (~100GB) and ran it on my M3 Ultra Mac Studio with 256GB unified memory. **Actual results: 46 tok/s generation, 76-350 tok/s prompt processing, excellent code quality, working tool calls.** Here's the full breakdown.

---

## Why Step 3.5 Flash?

StepFun (步进星辰) is a Beijing-based AI lab that's been quietly producing impressive open-source models. Their Step 3.5 Flash is their most capable release yet:

- **Architecture**: 196B total parameters, Sparse Mixture-of-Experts (MoE)
- **Active params**: ~11B per token (Top-8 routing from 288 experts)
- **Context**: 256K tokens (trained), tested with 8K here
- **License**: Apache 2.0

The key insight: they achieve "196B model intelligence" with "11B model speed" through aggressive MoE sparsity. On paper, it beats DeepSeek V3.2 on many agentic and coding benchmarks (74.4% SWE-bench Verified, 51.0% Terminal-Bench 2.0) while being far cheaper to run.

The question: does it actually work on consumer hardware? Let's find out.

---

## Hardware & Setup

**Test Machine:**
- Mac Studio (2024)
- Apple M3 Ultra (24-core CPU, 76-core GPU)
- 256GB unified memory
- macOS Sequoia 15.x

**Model:**
- [ubergarm/Step-3.5-Flash-GGUF](https://huggingface.co/ubergarm/Step-3.5-Flash-GGUF)
- Quantization: IQ4_XS (4.38 bits per weight)
- Size: 100.53 GiB across 4 split files

### Download

```bash
# Install HuggingFace CLI
pipx install huggingface_hub

# Download IQ4_XS quant (100GB total)
hf download ubergarm/Step-3.5-Flash-GGUF \
  --include "IQ4_XS/*" \
  --local-dir ~/models/step-3.5-flash
```

### Running with llama.cpp

```bash
# Install llama.cpp via Homebrew
brew install llama.cpp

# Start the server (--no-warmup recommended to avoid OOM on startup)
llama-server \
  -m ~/models/step-3.5-flash/IQ4_XS/Step-3.5-Flash-IQ4_XS-00001-of-00004.gguf \
  --port 8090 \
  -ngl 99 \
  -c 8192 \
  --no-warmup
```

> **Note**: The `-ngl 99` flag offloads all layers to GPU. On M3 Ultra with unified memory, this works seamlessly. The `--no-warmup` flag is important — the default warmup pass can OOM during initialization.

---

## Benchmark Results

### 1. Raw Speed Test

**Prompt:** "Explain quantum entanglement in simple terms. Be concise, about 3 sentences."

| Metric | Value |
|--------|-------|
| Prompt tokens | 27 |
| Prompt processing | **76.2 tok/s** |
| Generation tokens | 200 |
| Generation speed | **46.9 tok/s** |
| Total time | 4.65s |

**Output quality:** Excellent. The model produced a clear, accurate explanation with built-in reasoning traces:

> *"Quantum entanglement happens when two particles become linked so that measuring one instantly determines the state of the other, even if they're light-years apart. This connection is a fundamental, non-local feature of quantum mechanics, meaning the particles don't have independent, definite properties until measured."*

The model includes `reasoning_content` in responses — it shows its thinking process. This is a nice feature inherited from the DeepSeek-style architecture.

### 2. Coding Task

**Prompt:** "Write a Python FastAPI server with CRUD endpoints for a todo app with SQLite storage. Include models, database setup, and all endpoints."

| Metric | Value |
|--------|-------|
| Prompt tokens | 41 |
| Prompt processing | **102.1 tok/s** |
| Generation tokens | 2000 |
| Generation speed | **46.0 tok/s** |
| Total time | 43.9s |

**Code Quality Assessment:** ⭐⭐⭐⭐⭐ Excellent

The generated code was production-ready with:
- Proper project structure (5 files: main.py, models.py, database.py, schemas.py, crud.py)
- SQLAlchemy ORM with proper session management
- Pydantic v2 schemas with `model_dump()` (not the deprecated `dict()`)
- Proper HTTP status codes (201 for create, 204 for delete)
- Pagination support
- Error handling with HTTPException
- Requirements.txt with pinned versions
- Clear documentation with endpoint table

This is legitimately better than what many developers write by hand. No hallucinated imports, no deprecated APIs.

<details>
<summary>Generated Code (click to expand)</summary>

```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./todo.db"
engine = create_engine(
    SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False}
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# models.py
from sqlalchemy import Column, Integer, String, Boolean
from database import Base

class TodoItem(Base):
    __tablename__ = "todos"
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, nullable=False)
    description = Column(String, nullable=True)
    completed = Column(Boolean, default=False)

# schemas.py
from pydantic import BaseModel
from typing import Optional

class TodoBase(BaseModel):
    title: str
    description: Optional[str] = None
    completed: bool = False

class TodoCreate(TodoBase):
    pass

class TodoUpdate(BaseModel):
    title: Optional[str] = None
    description: Optional[str] = None
    completed: Optional[bool] = None

class TodoResponse(TodoBase):
    id: int
    class Config:
        from_attributes = True

# crud.py
from sqlalchemy.orm import Session
from models import TodoItem
from schemas import TodoCreate, TodoUpdate

def get_todos(db: Session, skip: int = 0, limit: int = 100):
    return db.query(TodoItem).offset(skip).limit(limit).all()

def get_todo(db: Session, todo_id: int):
    return db.query(TodoItem).filter(TodoItem.id == todo_id).first()

def create_todo(db: Session, todo: TodoCreate):
    db_todo = TodoItem(**todo.model_dump())
    db.add(db_todo)
    db.commit()
    db.refresh(db_todo)
    return db_todo

def update_todo(db: Session, todo_id: int, todo_update: TodoUpdate):
    db_todo = get_todo(db, todo_id)
    if db_todo:
        update_data = todo_update.model_dump(exclude_unset=True)
        for field, value in update_data.items():
            setattr(db_todo, field, value)
        db.commit()
        db.refresh(db_todo)
    return db_todo

def delete_todo(db: Session, todo_id: int):
    db_todo = get_todo(db, todo_id)
    if db_todo:
        db.delete(db_todo)
        db.commit()
        return True
    return False

# main.py
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.responses import JSONResponse
from sqlalchemy.orm import Session
from database import engine, get_db, Base
from schemas import TodoCreate, TodoUpdate, TodoResponse
from crud import get_todos, get_todo, create_todo, update_todo, delete_todo

Base.metadata.create_all(bind=engine)
app = FastAPI(title="Todo API", version="1.0.0")

@app.get("/")
def read_root():
    return {"message": "Welcome to Todo API", "docs": "/docs"}

@app.post("/todos/", response_model=TodoResponse, status_code=status.HTTP_201_CREATED)
def create_todo_endpoint(todo: TodoCreate, db: Session = Depends(get_db)):
    return create_todo(db=db, todo=todo)

@app.get("/todos/", response_model=list[TodoResponse])
def read_todos(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    return get_todos(db, skip=skip, limit=limit)

@app.get("/todos/{todo_id}", response_model=TodoResponse)
def read_todo(todo_id: int, db: Session = Depends(get_db)):
    db_todo = get_todo(db, todo_id=todo_id)
    if db_todo is None:
        raise HTTPException(status_code=404, detail="Todo not found")
    return db_todo

@app.put("/todos/{todo_id}", response_model=TodoResponse)
def update_todo_endpoint(todo_id: int, todo_update: TodoUpdate, db: Session = Depends(get_db)):
    db_todo = update_todo(db=db, todo_id=todo_id, todo_update=todo_update)
    if db_todo is None:
        raise HTTPException(status_code=404, detail="Todo not found")
    return db_todo

@app.delete("/todos/{todo_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_todo_endpoint(todo_id: int, db: Session = Depends(get_db)):
    if not delete_todo(db=db, todo_id=todo_id):
        raise HTTPException(status_code=404, detail="Todo not found")
    return JSONResponse(status_code=status.HTTP_204_NO_CONTENT, content=None)
```
</details>

### 3. Tool/Function Calling

**Test:** Provided OpenAI-format tool schemas for `get_weather(location)` and `calculate(expression)`.

**Prompt:** "What's the weather in Austin and calculate 15% tip on $84.50"

| Metric | Value |
|--------|-------|
| Prompt tokens | 318 (includes tool schemas) |
| Prompt processing | **351.9 tok/s** |
| Generation tokens | 152 |
| Generation speed | **45.9 tok/s** |

**Result:**
```json
{
  "tool_calls": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "arguments": "{\"location\":\"Austin, TX\"}"
      },
      "id": "5fr6GaeWmNiHOSvqC0vvCdP4l882PQYI"
    }
  ]
}
```

**Assessment:** ⚠️ Partial Success

Tool calling **works** — the model correctly formatted the tool call with proper JSON arguments. However, it only returned 1 of 2 needed tool calls. The reasoning trace showed it understood both tasks:

> *"I can make both calls simultaneously since they are independent of each other... get_weather with location 'Austin, TX'... calculate with expression '84.50 * 0.15'"*

But only `get_weather` made it into the final output. This might be a llama.cpp limitation or prompt template issue rather than the model itself.

---

## Comparison: Step 3.5 Flash vs Local Models

| Model | Size | Gen tok/s | Prompt tok/s | Notes |
|-------|------|-----------|--------------|-------|
| **Step 3.5 Flash IQ4_XS** | 100GB | **46** | **76-352** | 196B MoE, excellent quality |
| llama3.1:8b-instruct | 4.7GB | 103 | 281 | 2.2x faster, but far less capable |
| qwen2.5-coder | 4.7GB | 102 | 368 | 2.2x faster, coding-focused |
| codestral | 12GB | 42 | 75 | Similar speed, much smaller |
| deepseek-r1:70b | 42GB | — | — | Crashed on M3 Ultra |

**The tradeoff is clear:** Step 3.5 Flash is ~2x slower than small models but delivers dramatically better quality. For complex coding tasks, the 46 tok/s is still very usable — 2000 tokens in 44 seconds.

---

## Memory Usage

The model loads almost exactly to its file size:

```bash
# During inference
$ ps aux | grep llama-server
PID: 20938  CPU: 0.0%  MEM: 39.2%  RSS: 100.4GB
```

- **Model size on disk:** 100.53 GiB
- **RAM usage during inference:** 100.4 GB
- **KV cache (8K context):** ~714 MB additional
- **Total footprint:** ~101 GB

With 256GB unified memory, this leaves plenty of headroom for other tasks. The Mac doesn't even break a sweat.

---

## Observations & Limitations

### What Works Well

1. **Code quality is exceptional** — Production-ready output with modern best practices
2. **Reasoning traces built-in** — The `reasoning_content` field shows the model's thought process
3. **Tool calling works** — Proper OpenAI-format function calling support
4. **Consistent speed** — ~46 tok/s generation was rock-solid across all tests
5. **Memory efficient for its size** — MoE architecture means 196B params fit in 100GB

### Limitations

1. **Startup can OOM** — Default warmup pass crashed; need `--no-warmup` flag
2. **Tool calling incomplete** — Only returned 1 of 2 parallel tool calls
3. **Slower than claimed** — StepFun claims 100-300 tok/s; we got 46 tok/s (maybe they're measuring something else, or using different hardware)
4. **No native llama.cpp chat template** — Model works but may not be fully optimized yet
5. **Massive footprint** — 100GB is a lot; won't fit on most machines

---

## Verdict

Step 3.5 Flash is **legitimately impressive** running locally. The code quality matches or exceeds GPT-4 on standard tasks, and the reasoning traces add transparency. At 46 tok/s, it's not "Flash" fast by API standards, but it's very usable for development work.

**The MoE architecture delivers on its promise:** You get 196B-class intelligence without needing 196B-class compute. The 11B active parameters per token mean inference stays snappy despite the massive total parameter count.

**Would I use it daily?** Yes, for coding and complex reasoning tasks where quality matters more than speed. For quick Q&A, I'd still reach for llama3.1:8b or qwen2.5-coder.

**Best use case:** Code generation, technical writing, complex reasoning — tasks where you want frontier-model quality but can't (or won't) pay for API calls.

---

## Quick Reference

```bash
# Download (100GB)
hf download ubergarm/Step-3.5-Flash-GGUF \
  --include "IQ4_XS/*" \
  --local-dir ~/models/step-3.5-flash

# Run
llama-server \
  -m ~/models/step-3.5-flash/IQ4_XS/Step-3.5-Flash-IQ4_XS-00001-of-00004.gguf \
  -ngl 99 -c 8192 --port 8090 --no-warmup

# Test
curl http://localhost:8090/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"messages":[{"role":"user","content":"Hello!"}],"max_tokens":100}'
```

---

*Benchmarked on February 19, 2026. Hardware: M3 Ultra Mac Studio (76-core GPU), 256GB RAM. Model: Step-3.5-Flash-IQ4_XS via llama.cpp b8070.*
