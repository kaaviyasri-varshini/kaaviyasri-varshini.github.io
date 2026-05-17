Title: LangChain Project Setup & First Execution: From Zero to Your First LLM Call
Date: 2026-05-17
Category: AI & Machine Learning
Tags: langchain, python, setup, tutorial, beginner, llm, api-keys, debugging
Slug: langchain-project-setup-first-execution


> *Setting up a LangChain project correctly from the start saves hours of debugging later. Here's how to build a solid foundation — from Python environments to your first working LLM call.*

---

## Why Setup Matters More Than You Think

Most LangChain tutorials skip straight to the exciting part: calling an LLM. But in real development, setup problems cause 80% of beginner frustration:

- "Why can't Python find my packages?"
- "My API key isn't working"
- "It works in the tutorial but breaks in my project"
- "I'm getting SSL errors / connection timeouts"

These aren't LangChain problems — they're environment problems. Get the foundation right once, and everything else becomes easier.

This guide walks through the complete setup process: Python environment, dependency management, secure API key handling, your first LLM call, and debugging common errors.

---

## Python Environment Setup

### Why Virtual Environments Matter

Python projects should **always** use isolated environments. Without isolation:
- Package versions conflict across projects
- System Python gets polluted with project-specific dependencies
- Deployment becomes unpredictable
- Debugging becomes a nightmare

Think of virtual environments as project-specific sandboxes. Each project gets its own Python installation, its own packages, and its own versions.

### Choosing Your Environment Tool

Three main options exist:

| Tool | Best For | Pros | Cons |
|---|---|---|---|
| **venv** | Simple projects | Built into Python, no install needed | Basic features only |
| **conda** | Data science, complex dependencies | Handles non-Python dependencies (C libraries, etc.) | Larger, slower |
| **virtualenv** | Advanced control | More features than venv | Requires separate install |

For LangChain projects, **venv** is sufficient for most cases. Use **conda** if you're also working with data science libraries like NumPy, Pandas, or TensorFlow.

### Creating a Virtual Environment with venv

```bash
# Create project directory
mkdir langchain-project
cd langchain-project

# Create virtual environment
python3 -m venv venv

# Activate it
# On macOS/Linux:
source venv/bin/activate

# On Windows:
venv\Scripts\activate
```

Once activated, your terminal prompt changes to show `(venv)` at the beginning. This confirms you're working inside the isolated environment.

### Creating a Virtual Environment with conda

```bash
# Create environment with Python 3.11
conda create -n langchain-env python=3.11

# Activate it
conda activate langchain-env
```

### Verifying Your Environment

```bash
# Check Python location (should point to your venv)
which python

# Check Python version
python --version

# Verify pip is isolated
which pip
```

If `which python` points to your system Python instead of the virtual environment, the activation didn't work. Retry the activation command.

---

## Dependency Management

### Installing LangChain

LangChain is modular. The core library is small; integrations are separate packages. This keeps installations lean and avoids dependency bloat.

**Core installation:**
```bash
pip install langchain-core
```

**For OpenAI models:**
```bash
pip install langchain-openai
```

**For Google Gemini models:**
```bash
pip install langchain-google-genai
```

**For Anthropic Claude models:**
```bash
pip install langchain-anthropic
```

**Common pattern:**
```bash
# Install core + your chosen provider
pip install langchain-core langchain-openai
```

### Understanding requirements.txt

As your project grows, manually tracking dependencies becomes error-prone. `requirements.txt` solves this by listing all packages in one file.

**Generate requirements.txt from your current environment:**
```bash
pip freeze > requirements.txt
```

**Install from requirements.txt:**
```bash
pip install -r requirements.txt
```

This is essential for:
- Sharing projects with teammates
- Deploying to servers
- Reproducing environments months later

### Pinning Versions vs Flexible Ranges

**Exact pinning (recommended for production):**
```
langchain-core==0.2.5
langchain-openai==0.1.8
```

**Flexible ranges (useful during development):**
```
langchain-core>=0.2.0,<0.3.0
langchain-openai>=0.1.0
```

Exact versions ensure reproducibility. Flexible ranges allow minor updates. For learning projects, flexible is fine. For production, pin everything.

### Common Installation Issues

**Issue: `pip: command not found`**
- Solution: Ensure virtual environment is activated

**Issue: `Permission denied` errors**
- Solution: Never use `sudo pip`. Activate your virtual environment first.

**Issue: SSL certificate errors**
- Solution: Update pip: `pip install --upgrade pip`

**Issue: Conflicting dependencies**
- Solution: Create a fresh virtual environment and reinstall

---

## Secure API Key Handling

### Why Hardcoding Keys Is Dangerous

Never do this:

```python
# ❌ NEVER DO THIS
api_key = "sk-proj-abc123..."
```

Hardcoded keys:
- Get committed to version control (and exposed publicly if you push to GitHub)
- Are visible in logs and error messages
- Can't be changed without modifying code
- Violate security best practices

### Using Environment Variables

Environment variables keep secrets out of code. They're loaded at runtime, not stored in files.

**Step 1: Create a `.env` file**

In your project root:

```bash
# .env
OPENAI_API_KEY=sk-proj-your-actual-key-here
GOOGLE_API_KEY=your-google-key-here
```

**Step 2: Add `.env` to `.gitignore`**

```bash
# .gitignore
.env
venv/
__pycache__/
*.pyc
```

This prevents accidentally committing secrets.

**Step 3: Install python-dotenv**

```bash
pip install python-dotenv
```

**Step 4: Load environment variables in your code**

```python
import os
from dotenv import load_dotenv

# Load .env file
load_dotenv()

# Access keys
api_key = os.getenv("OPENAI_API_KEY")
```

### Alternative: System Environment Variables

For production systems, set environment variables at the system level instead of using `.env` files:

**macOS/Linux:**
```bash
export OPENAI_API_KEY="sk-proj-..."
```

**Windows (PowerShell):**
```powershell
$env:OPENAI_API_KEY="sk-proj-..."
```

**Permanent (add to shell profile):**
```bash
# Add to ~/.bashrc or ~/.zshrc
export OPENAI_API_KEY="sk-proj-..."
```

### Obtaining API Keys

**OpenAI:**
1. Go to [platform.openai.com](https://platform.openai.com)
2. Sign up / log in
3. Navigate to API Keys section
4. Create new secret key
5. Copy immediately (you won't see it again)

**Google Gemini:**
1. Go to [ai.google.dev](https://ai.google.dev)
2. Get API key from Google AI Studio
3. Enable Gemini API in your project

**Anthropic Claude:**
1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Sign up / log in
3. Generate API key from settings

---

## First LLM Call

### Minimal Working Example

Here's the simplest possible LangChain program:

```python
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage

# Load environment variables
load_dotenv()

# Initialize model
model = ChatOpenAI(model="gpt-4o-mini")

# Create message
message = HumanMessage(content="What is LangChain?")

# Invoke model
response = model.invoke([message])

print(response.content)
```

**What's happening:**
1. Environment variables load from `.env`
2. `ChatOpenAI` initializes with your API key (automatically read from `OPENAI_API_KEY`)
3. `HumanMessage` wraps your prompt
4. `invoke()` sends the message and waits for a response
5. `response.content` contains the model's text output

### Using Prompt Templates

Hardcoding prompts is like hardcoding API keys — it works once but doesn't scale. Prompt templates make prompts reusable:

```python
from langchain_core.prompts import ChatPromptTemplate

# Define template
template = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful AI assistant."),
    ("human", "Explain {topic} in simple terms.")
])

# Format with input
prompt = template.invoke({"topic": "neural networks"})

# Send to model
response = model.invoke(prompt)
print(response.content)
```

### Building Your First Chain

Chains connect components into pipelines:

```python
from langchain_core.output_parsers import StrOutputParser

# Build chain
chain = template | model | StrOutputParser()

# Execute
result = chain.invoke({"topic": "transformers"})
print(result)
```

**What changed:**
- `template | model | StrOutputParser()` creates a pipeline
- `invoke()` now accepts a dictionary (matching template variables)
- Output is a clean string, not a message object

### Streaming Responses

For chat interfaces, streaming provides better UX:

```python
for chunk in chain.stream({"topic": "attention mechanisms"}):
    print(chunk, end="", flush=True)
```

Output appears incrementally, like ChatGPT's interface.

---

## Debugging Common Beginner Errors

### Error: `AuthenticationError: Invalid API key`

**Cause:** API key is wrong, missing, or not loaded

**Solutions:**
1. Verify `.env` file exists and contains correct key
2. Check `load_dotenv()` is called before model initialization
3. Print `os.getenv("OPENAI_API_KEY")` to verify it's loaded
4. Ensure no extra spaces or quotes in `.env` file

```python
# Debug: Print loaded key (first 10 chars only)
key = os.getenv("OPENAI_API_KEY")
print(f"Key loaded: {key[:10]}..." if key else "Key not found!")
```

### Error: `ModuleNotFoundError: No module named 'langchain_openai'`

**Cause:** Package not installed, or wrong environment active

**Solutions:**
1. Verify virtual environment is activated (check terminal prompt)
2. Install missing package: `pip install langchain-openai`
3. Check installed packages: `pip list | grep langchain`

### Error: `RateLimitError: You exceeded your current quota`

**Cause:** API usage limits reached (free tier exhausted or billing issue)

**Solutions:**
1. Check your API usage dashboard
2. Add payment method if on free tier
3. Wait if you hit rate limits (they reset)
4. Use a different model (e.g., `gpt-4o-mini` instead of `gpt-4`)

### Error: `Timeout` or `Connection refused`

**Cause:** Network issues, firewall, or API service down

**Solutions:**
1. Check internet connection
2. Verify API service status (status.openai.com)
3. Try with increased timeout:

```python
model = ChatOpenAI(
    model="gpt-4o-mini",
    timeout=60  # seconds
)
```

### Error: `ValidationError: 1 validation error for ChatOpenAI`

**Cause:** Invalid parameter passed to model initialization

**Solutions:**
1. Check model name spelling: `gpt-4o-mini` not `gpt4-mini`
2. Verify parameter names match documentation
3. Check for typos in configuration

### Error: Output is a message object, not a string

**Cause:** Missing output parser

**Solution:** Add `StrOutputParser()` to your chain:

```python
from langchain_core.output_parsers import StrOutputParser

chain = template | model | StrOutputParser()
```

### Debugging Strategy

When something breaks:

1. **Isolate the problem** — Test each component separately
2. **Check the basics** — Environment active? Packages installed? API key loaded?
3. **Read the full error** — Don't just look at the last line; the traceback shows where it broke
4. **Print intermediate values** — Add `print()` statements to verify data flow
5. **Start minimal** — Strip your code down to the simplest working example, then add complexity back

---

## Project Structure Best Practices

As your project grows, organization matters:

```
langchain-project/
├── .env                 # API keys (never commit)
├── .gitignore          # Ignore .env, venv, etc.
├── requirements.txt    # Dependencies
├── venv/               # Virtual environment (never commit)
├── src/
│   ├── __init__.py
│   ├── chains.py       # Chain definitions
│   ├── prompts.py      # Prompt templates
│   └── models.py       # Model configurations
├── tests/
│   └── test_chains.py
└── main.py             # Entry point
```

**Key principles:**
- Keep prompts separate from logic
- One file per major component
- Tests live in their own directory
- Configuration (API keys, model names) stays in `.env` or config files

---

## Verification Checklist

Before moving forward, confirm:

- ✅ Virtual environment created and activated
- ✅ LangChain packages installed (`pip list | grep langchain`)
- ✅ `.env` file created with API key
- ✅ `.env` added to `.gitignore`
- ✅ `python-dotenv` installed
- ✅ First LLM call executes successfully
- ✅ Output is readable (string, not message object)
- ✅ No hardcoded API keys in code

---

## Summary

Setting up a LangChain project correctly means:

1. **Isolated environments** — Use `venv` or `conda` to avoid dependency conflicts
2. **Modular installation** — Install only what you need (`langchain-core` + provider packages)
3. **Secure secrets** — Use `.env` files and `python-dotenv`, never hardcode keys
4. **Structured projects** — Separate prompts, chains, and configuration from the start
5. **Debugging mindset** — Isolate problems, check basics first, read full errors

With this foundation in place, you're ready to build real LangChain applications. The next steps involve prompt engineering, memory, retrieval, and agents — but all of that builds on the setup you've just completed.

---

