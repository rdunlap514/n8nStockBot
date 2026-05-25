
# StockBot AI Workflows (n8n)

A compact, portfolio-ready set of n8n workflows that demonstrate an AI-powered short-term stock sentiment assistant. These workflows fetch market data (Twelve Data), pull recent news (NewsAPI), and use an LLM to produce structured short-term sentiment summaries. Designed to showcase integration, data cleaning, LLM orchestration, and practical automation in n8n.

Author: Ryan Dunlap

---

## Contents
- workflows/
  - StockBot.json
  - analyze_stock.json
- .env.example
- README.md
- .gitignore
- LICENSE (MIT)
- tests/
  - example_input.json (suggested)
- assets/
  - demo.gif (optional)

---

## Important disclaimer — Not financial advice
This project is for demonstration and educational purposes only. It is NOT financial, investment, tax, or legal advice. The AI outputs may be incomplete, incorrect, or out of date. Do not make real investment decisions based solely on these workflows or their outputs. Always do your own research and consult a licensed professional.

If you expose these tools to other users, display a visible reminder that the assistant is not a licensed financial advisor.

---

## What this repo demonstrates
- Integrating multiple APIs (Twelve Data, NewsAPI) in n8n.
- Orchestrating an LLM-based analysis pipeline (LangChain / OpenAI nodes).
- Data cleaning/normalization of time series and article content.
- A chat agent that can call a tool workflow and keep short-term memory.
- Basic production practices: sanitized exports, .env usage, README guidance.

---

## Prerequisites
- n8n (self-hosted or n8n cloud). Recommended: n8n >= 0.230.0.
- API keys / accounts:
  - OpenAI (or another LLM provider supported by LangChain nodes)
  - Twelve Data
  - NewsAPI
- Familiarity with importing workflows into n8n and adding credentials.

---

## Quick setup & import

1. Clone repository
   ```
   git clone <your-repo-url>
   cd stock-ai-workflows
   ```

2. Create `.env` (optional)
   - Copy `.env.example` to `.env` and fill real keys if you want to use environment variable expressions:
     ```
     cp .env.example .env
     # edit .env and add:
     # OPENAI_API_KEY=...
     # TWELVEDATA_API_KEY=...
     # NEWSAPI_KEY=...
     ```

3. Recommended: create credentials in n8n (safer)
   - Open n8n → Credentials → Create:
     - OpenAI / OpenAI API
     - HTTP / Generic credentials (or custom credentials for Twelve Data / NewsAPI)
   - Using n8n Credentials is preferred for production.

4. Import workflows
   - n8n → Workflows → Import → Choose file → select `workflows/StockBot.json` and `workflows/analyze_stock.json`.
   - After import, open each workflow and assign the appropriate credentials to nodes (LLM and HTTP Request nodes).

5. Re-link the tool workflow (StockBot → analyze_stock)
   - Open the StockBot workflow → open the tool node that calls `analyze_stock` → use the workflow dropdown to select your imported `analyze_stock` workflow. This avoids stale workflow IDs.

---

## Quick test

- analyze_stock (manual)
  - Open `analyze_stock` in editor → click Execute Workflow trigger and provide this example input:
    ```json
    {
      "symbol": "AAPL",
      "name": "Apple"
    }
    ```
  - Expected steps: Candles 1h/2h/8h (Twelve Data) → Quote → News search (NewsAPI) → article cleanup → LLM sentiment analysis.

- StockBot (chat)
  - Test the chat trigger (editor or configured chat webhook):
    ```
    "Analyze AAPL for short-term sentiment and summarize the latest news."
    ```
  - The agent should call `analyze_stock` and reply with a natural language summary derived from the tool output.

Example expected LLM output structure:
```json
{
  "shortTermSentiment": {
    "category": "Positive",
    "score": 0.68,
    "rationale": "Multiple articles report better-than-expected earnings and increased guidance; market reaction appears favorable..."
  }
}
```

---

## Recommended improvements (for production-readiness / portfolio polish)
- Credentials & secrets
  - Ensure no API keys in repo. Use `.env` + `.gitignore` or n8n Credentials.
- Error handling & validation
  - Add IF nodes to verify HTTP responses, and Code nodes to validate LLM JSON.
  - Implement retry/backoff for transient API failures.
- Limit LLM input size
  - Trim article content and cap the number of articles to control cost and latency.
- Modularize workflows
  - Split into smaller flows: `get_market_data`, `get_news`, `analyze_sentiment`.
- Tests & demos
  - Add `tests/example_input.json` and `tests/expected_output.json`.
  - Add an assets/demo.gif or demo.png showing a run.
- Observability
  - Log runs and failures (Google Sheets/Airtable/DB) and add notification on errors (Slack/email).

---

## Security & cost considerations
- Do NOT commit real API keys. Add `.env` to `.gitignore`.
- Truncate or summarize article text before sending to LLMs to reduce cost.
- Watch API rate limits for Twelve Data and NewsAPI — add backoff/retry logic.
- Mask or remove proprietary/personal data before sending to third-party LLMs.

---

## Troubleshooting
- Agent doesn’t call tool:
  - Re-open the tool node in StockBot and re-select the `analyze_stock` workflow.
- HTTP requests failing:
  - Verify API keys and rate limits. Inspect raw responses in n8n execution logs.
- LLM returns malformed JSON:
  - Tighten the prompt; add a post-LLM Code node that attempts to parse/validate and retries with a stricter prompt if needed.

---

## License & attribution
- License: MIT (see LICENSE file).  

---

## Want me to do one of these for you?
- Add post-LLM validation code nodes into `analyze_stock` and provide the updated workflow JSON for import.
- Sanitize webhookId / instanceId metadata to make imports even cleaner.
- Create `tests/example_input.json` and a demo GIF placeholder.

Tell me which and I’ll add it to the repo.
