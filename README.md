# Text-to-SQL Agent

A minimal, from-scratch AI agent that turns natural language questions into SQL queries, executes them against a live database, and answers in plain English — built with **Google Gemini function calling** and **SQLite**.

> "Which customers are from Algeria?" → `SELECT * FROM customers WHERE country = 'Algeria'` → *"The customer from Algeria is Ali."*

---

## What this is

This notebook walks through building a tool-using LLM agent step by step, starting from a single hardcoded function and ending with a fully autonomous agentic loop that can:

- Understand a natural language question
- Decide which tool to call (and with what arguments)
- Generate and execute real SQL against a database
- Handle errors and reason about the results
- Return a clean, human-readable final answer

No frameworks like LangChain or LlamaIndex — just the raw Gemini API, so you can see exactly how function calling and agent loops work under the hood.

---

## How it works

The notebook builds up the agent in progressive stages:

| Stage | What happens |
|-------|---------------|
| 1. Setup | Connect to the Gemini API and spin up an in-memory SQLite database |
| 2. First tool | Give Gemini a single fixed tool (`get_customers`) to prove function calling works |
| 3. Generalize | Replace it with a flexible `execute_sql` tool so Gemini can write any query |
| 4. Single-turn | Ask one question, let Gemini generate SQL, run it, and feed the result back |
| 5. Full agent | Wrap everything in a `run_agent()` loop that keeps going until Gemini has a final answer — no fixed number of steps |

```
User question
     │
     ▼
 Gemini decides: call execute_sql(query) ──► SQLite runs the query
     │                                              │
     ◄──────────────── result / error ──────────────┘
     │
     ▼
Gemini reasons over the result
     │
     ├─ needs more info? → calls execute_sql again
     └─ done? → returns final natural language answer
```

---

## Getting started

### 1. Requirements

```bash
pip install -U google-genai
```

### 2. Set your API key

The notebook was built on **Kaggle** and reads the key from Kaggle Secrets:

```python
from kaggle_secrets import UserSecretsClient
api_key = UserSecretsClient().get_secret("GEMINI_API_KEY")
```

Running elsewhere (Colab, local, etc.)? Just swap that for your own key, e.g.:

```python
import os
api_key = os.environ["GEMINI_API_KEY"]
```

Get a free Gemini API key at [Google AI Studio](https://aistudio.google.com/).

### 3. Run the notebook

Open `text-to-sql-agent.ipynb` and run all cells top to bottom. It creates a small in-memory `customers` table and lets you ask it questions immediately — no external database setup required.

---

## Example queries

```python
run_agent("Which customers are from Algeria?")
run_agent("How many customers are in the database?")
run_agent("What are the names of customers who are not from Algeria?")
```

Each call prints the SQL Gemini generated, the raw database result, and the final natural-language answer — so you can watch the agent think.

---

## Tech stack

- [Google Gemini API](https://ai.google.dev/) (`google-genai`) — LLM + function calling
- SQLite (`sqlite3`, in-memory) — lightweight database backend
- Pure Python — no agent frameworks

---

## Extending this project

Some natural next steps if you want to take this further:

- Add query validation/sandboxing before executing model-generated SQL
- Swap the toy `customers` table for a real database or a larger schema
- Give the agent multiple tools (e.g. schema lookup, chart generation)
- Wrap `run_agent()` in a simple CLI or Streamlit UI
- Add conversation memory across multiple questions

---

## Note on safety

This is an educational example. `execute_sql` runs whatever SQL the model generates directly against the database. In any real-world/production setting, you'd want to add strict validation, read-only permissions, and query sandboxing before executing LLM-generated SQL.

---

## License

Feel free to use, modify, and build on this project for learning purposes.
