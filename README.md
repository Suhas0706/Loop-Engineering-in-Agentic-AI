# Loop Engineering in Agentic AI: Code Demo

A hands-on demo comparing a **one-shot (non-loop) agent** with a **loop agent** that generates, evaluates, and improves its own output until it meets the requirements.


---

## What Is Loop Engineering?

A typical AI agent answers once and stops, even when the answer is wrong. A loop agent follows an iterative workflow:

```
Generate → Evaluate → Improve → Repeat (until it passes or hits the retry limit)
```

| | Non-Loop Agent | Loop Agent |
|---|---|---|
| Attempts | 1 | Up to `MAX_ITERATIONS` |
| Checks its own output | No | Yes |
| Uses feedback | No | Yes, errors go back into the next prompt |
| Reliability | Lower | Higher |

---

## What's in the Notebook

**Example 1: Non-loop agent**
Sends a single request to the LLM and prints the reply, with no validation.

**Example 2: Loop agent**
Runs the same task, but:
1. Generates a reply with the LLM
2. Evaluates it against rule-based checks
3. Feeds the list of problems and the previous answer back into the prompt
4. Retries until the reply passes or the iteration cap is reached

### The Task

Write a customer-support reply for a customer who received a damaged laptop (`ORD-2026-101`). The reply must:

- Apologize to the customer
- Mention the order ID
- Offer a replacement
- Stay under 60 words

### Sample Output

```
--- Iteration 1 ---
Sorry for the damaged laptop (ORD-2026-101). We'll replace it promptly.
Evaluation: FAILED
- Clearly offer a replacement.

--- Iteration 2 ---
Sorry for the damaged laptop (ORD-2026-101). We will provide a replacement laptop.
Evaluation: PASSED
```

---

## Tech Stack

- Python 3
- [Groq API](https://console.groq.com/)
- Model: `openai/gpt-oss-120b`
- Google Colab

---

## Getting Started

### 1. Get a Groq API key

Create a free key at [console.groq.com](https://console.groq.com/keys).

### 2. Run in Google Colab

1. Open the notebook in Colab.
2. Click the 🔑 **Secrets** icon in the left sidebar.
3. Add a secret named `GROQ_API_KEY` and paste your key. Enable notebook access.
4. Replace the hardcoded key cell with:

```python
from google.colab import userdata
api_key = userdata.get("GROQ_API_KEY")
```

5. Run all cells.

### 3. Run locally

```bash
pip install groq
export GROQ_API_KEY="your_key_here"   # Windows: set GROQ_API_KEY=your_key_here
```

Then in the notebook or script:

```python
import os
api_key = os.environ["GROQ_API_KEY"]
```

> ⚠️ **Never commit API keys to GitHub or share them in notebooks.** If a key has been exposed, revoke it in the Groq console and create a new one.

---

## How the Loop Works

```python
for iteration in range(1, MAX_ITERATIONS + 1):
    answer = call_llm(prompt)           # 1. Generate
    problems = evaluate_answer(answer)  # 2. Evaluate

    if not problems:                    # 3. Stop on success
        break

    feedback = "\n".join(problems)      # 4. Feedback becomes next input
    previous_answer = answer
```

### Key Design Ideas

- **Clear evaluation criteria:** Testable checks (apology words, order ID, keyword, word count) give the agent actionable feedback.
- **Feedback loop:** The previous answer and the list of problems are included in the next prompt.
- **Stopping conditions:** `MAX_ITERATIONS` prevents infinite loops and runaway API costs.

---

## Configuration

| Variable | Default | Description |
|---|---|---|
| `MODEL_NAME` | `openai/gpt-oss-120b` | Groq model used for generation |
| `MAX_ITERATIONS` | `3` | Maximum retry attempts |
| `ORDER_ID` | `ORD-2026-101` | Order ID the reply must include |

---

## Ideas for Extending

- Use an **LLM-as-judge** evaluator instead of rule-based checks
- Track the best answer across iterations, not only the last one
- Add exponential backoff and error handling for API calls
- Apply the pattern to code generation, report writing, or data validation
- Log each iteration's score for analysis

---

## Real-World Applications

- Customer-support reply generation
- Code generation with automated tests as the evaluator
- Structured data extraction with schema validation
- Report and document drafting with style checks

--

## License

MIT, or choose the license that fits your project.
