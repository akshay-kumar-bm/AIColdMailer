# AI Cold Email Automation

A Streamlit tool that scrapes target websites, uses a Groq-hosted LLM through LangChain to write a personalised cold email for each target, and sends it via Gmail SMTP while tracking status in a CSV.

## Features
- Navigation-aware web scraper (`requests` + BeautifulSoup) that gathers page content from a target site.
- `LangChainEmailGenerator`: prompt chain returning a structured `EmailStructure` (subject, body) via Pydantic parsing; custom prompt and purpose supplied from the UI.
- LLM factory pattern (`src/llm/factory.py`, `options.py`) with two Groq models: `llama3-groq-70b-8192-tool-use-preview` and `llama-3.2-11b-text-preview`.
- `GmailSender` over SMTP; results stored in `data/generated_emails.csv` with status (Pending / Sent / ...), editable from the UI.
- Logging to `logging/cold_email_automation.log`; LangSmith tracing settings in config.
- `cold_email.ipynb`: the original notebook prototype.

## Flow
```
URL + recipient -> scraper -> LLM chain (Groq) -> subject/body -> CSV (Pending) -> review in UI -> Gmail SMTP -> status update
```

## Structure
```
main.py                     # wires Config -> ColdEmailAutomation -> StreamlitUI
src/config.py
src/llm/{factory,options}.py
src/models/schemas.py
src/services/{scraper,email_generator,email_sender,automation}.py
src/ui/streamlit_app.py
data/generated_emails.csv   logging/*.log   cold_email.ipynb
```

## Setup
```
pip install -r requirements.txt
```
`.env` variables: `GROQ_API_KEY`, `LANGCHAIN_API_KEY`, `LANGCHAIN_PROJECT`, `SENDER_EMAIL`, `SENDER_PASSWORD` (Gmail app password). Note `src/llm/factory.py` reads the Groq key from `st.secrets['GROQ_API_KEY']` (for Streamlit Cloud) - provide `.streamlit/secrets.toml` or switch to the commented `os.getenv` line when running locally.

Run:
```
streamlit run main.py
```
(`main.py` builds the UI; the original README suggests `src/ui/streamlit_app.py`, which only defines the class.)

## Limitations
- `requirements.txt` lacks `streamlit`-adjacent pins and `langchain_core.pydantic_v1` needs a compatible LangChain version; Groq preview model ids are likely retired.
- Some generated rows in the CSV record generation errors; no rate limiting or unsubscribe handling is implemented in code.
- Cold emailing carries legal/spam-compliance considerations.
