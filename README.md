# 📈 Stock Intelligence & Social Sentiment Dashboard

An end-to-end Python pipeline that scrapes Reddit finance communities, measures sentiment per stock, predicts next-day prices with machine learning, and presents everything in an interactive Streamlit dashboard.

**Live demo:** _add your Streamlit link here after deploying_

## What it does

1. **Scrape** posts and comments from 20 finance subreddits using PRAW (`1_scraper.py`)
2. **Detect stocks and score sentiment** using spaCy entity recognition and VADER (`2_sentiment.py`)
3. **Predict prices:** downloads 1 year of price history with yfinance, builds technical indicators (RSI, MACD, EMA, SMA, Bollinger Bands), trains XGBoost, Random Forest, Ridge, SVR and Gradient Boosting models, and keeps the best one per ticker (`3_ml_predictor.py`)
4. **Summarise** the Reddit narrative per ticker with a CrewAI agent (`4_crewai_summary.py`)
5. **Display** candlestick charts, sentiment gauges, model metrics and summaries in Streamlit (`app.py`)

## Tech stack

Python, Streamlit, Plotly, pandas, scikit-learn, XGBoost, spaCy, VADER, PRAW, yfinance, CrewAI, Ollama

## Run the dashboard locally

```bash
pip install -r requirements.txt
streamlit run app.py
```

The dashboard reads pre-generated results from `data/` and `models/`, so no API keys are needed to view it. Price charts are fetched live from Yahoo Finance.

## Re-run the full pipeline

```bash
pip install -r requirements-pipeline.txt
python -m spacy download en_core_web_sm
```

Create a `.env` file with your Reddit credentials from https://www.reddit.com/prefs/apps:

```
REDDIT_CLIENT_ID=your_client_id
REDDIT_CLIENT_SECRET=your_client_secret
REDDIT_USER_AGENT=StockDashboard/1.0 by your_username
```

Then run the scripts in order:

```bash
python 1_scraper.py
python 2_sentiment.py
python 3_ml_predictor.py
python 4_crewai_summary.py   # needs Ollama running locally: ollama pull llama3
```

All settings (subreddits, tickers, model list, paths) are in `config.py`.

## Known limitations

- The CrewAI summaries were generated with a small local Ollama model on limited hardware and did not complete successfully, so that panel currently shows placeholder results. Switching to a stronger model is planned.
- Price predictions are a learning project and not financial advice.