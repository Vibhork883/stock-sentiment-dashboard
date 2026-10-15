# 📈 Stock Intelligence & Social Sentiment Dashboard

An end-to-end Python project that scrapes Reddit finance communities, scores sentiment for each stock, forecasts next-day prices with machine learning, and shows everything in an interactive Streamlit dashboard. The dashboard also checks every forecast against the real closing price.

## 🔗 Live demo

**https://stock-sentiment-dashboard-n7mzf2omhybecp2hqcdvgs.streamlit.app/**

This is the dashboard exactly as I ran and deployed it. Just click and explore, there is nothing to install. The free hosting puts the app to sleep when idle, so if it shows a "wake up" screen, wait 30 to 60 seconds.

## What do you want to do?

| I want to... | What to do |
|---|---|
| **Just look at the project** | Open the live demo above |
| **Run the dashboard on my own computer** | Follow [Option A](#option-a-run-the-dashboard-locally). The data I generated is already included in this repo |
| **Collect fresh Reddit data and retrain the models** | Follow [Option B](#option-b-re-run-the-full-pipeline-fresh-data). You need your own free Reddit API credentials |

## How it works

1. **Scrape:** collects posts and comments from 20 finance subreddits with PRAW (`1_scraper.py`)
2. **Sentiment:** detects stock mentions with spaCy and scores sentiment with VADER (`2_sentiment.py`)
3. **Forecast:** downloads 1 year of prices from Yahoo Finance, builds technical indicators (RSI, MACD, EMA, SMA, Bollinger Bands), trains XGBoost, Random Forest, Ridge, SVR and Gradient Boosting models, and keeps the best one per stock (`3_ml_predictor.py`)
4. **Summaries (optional, in progress):** a CrewAI agent writes a narrative summary of the Reddit discussion per stock (`4_crewai_summary.py`)
5. **Dashboard:** candlestick charts, sentiment gauges, model metrics and summaries in Streamlit (`app.py`)

Steps 1 to 4 save their results into `data/` and `models/`. The dashboard (step 5) only reads those saved files.

## Prediction vs actual

The pipeline holds back the most recent trading day. The models are trained only on prices *before* that day and forecast its closing price. The dashboard then shows the forecast next to the real close, the percentage error, and a scorecard across all stocks. This is a single-day check, not a claim of long-run accuracy.

The dashboard is a self-contained snapshot: prices, forecasts and sentiment are all saved by the pipeline, so everything on screen describes the same date. Reddit sentiment is shown alongside the forecasts as context; the price models use technical indicators only.

## Option A: Run the dashboard locally

Requires Python 3.11 or newer and Git.

```bash
git clone https://github.com/Vibhork883/stock-sentiment-dashboard.git
cd stock-sentiment-dashboard

python -m venv .venv
# Windows:      .venv\Scripts\activate
# macOS/Linux:  source .venv/bin/activate

pip install -r requirements.txt
streamlit run app.py
```

The repo already contains the data and trained models behind the live demo, so you will see the same dashboard on your own machine. No Reddit account or API keys are needed.

## Option B: Re-run the full pipeline (fresh data)

Use this if you want newer Reddit data and freshly trained models.

**1. Install everything the pipeline needs**

```bash
pip install -r requirements-pipeline.txt
python -m spacy download en_core_web_sm
```

**2. Get free Reddit API credentials** at https://www.reddit.com/prefs/apps (create an app of type *script*), then create a file named `.env` in the project folder:

```
REDDIT_CLIENT_ID=your_client_id
REDDIT_CLIENT_SECRET=your_client_secret
REDDIT_USER_AGENT=StockDashboard/1.0 by your_username
```

**3. Run the steps in order.** Run them *after the US stock market has closed*, so the latest trading day is a complete daily bar:

```bash
python 1_scraper.py
python 2_sentiment.py
python 3_ml_predictor.py
streamlit run app.py
```

Step 4 (`4_crewai_summary.py`) is optional. It needs CrewAI and a local Ollama model, and it is currently being reworked (see limitations below).

All settings (subreddits, ticker aliases, models, paths) live in `config.py`. If you fork this repo and push the refreshed data, Streamlit Cloud redeploys automatically.

## Why are there two requirements files?

| File | What it contains | Who needs it |
|---|---|---|
| `requirements.txt` | Only what the dashboard needs: Streamlit, Plotly, pandas, yfinance, python-dotenv | Anyone who just wants to view the dashboard. Streamlit Cloud also reads this file to deploy the live demo |
| `requirements-pipeline.txt` | Everything: the dashboard libraries **plus** the scraping, NLP and machine-learning libraries (PRAW, spaCy, XGBoost, scikit-learn and others) | Anyone who wants to re-run the full pipeline |

The split keeps the live deployment fast and reliable, because the deployed app never scrapes or trains anything. It only displays saved results, so it has no need for the heavy libraries. **You only install one of the two files**, not both: `requirements.txt` to view, `requirements-pipeline.txt` to regenerate the data.

## Project structure

```
app.py                    Streamlit dashboard
1_scraper.py              Reddit scraper
2_sentiment.py            Stock detection + sentiment scoring
3_ml_predictor.py         Feature engineering, model training, next-day forecast
4_crewai_summary.py       CrewAI narrative summaries (optional)
config.py                 All settings
data/                     Saved Reddit data, sentiment results, summaries
models/                   Trained models + model_metadata.json (forecasts, saved prices)
requirements.txt          Dashboard only
requirements-pipeline.txt Full pipeline
```

## Tech stack

Python, Streamlit, Plotly, pandas, scikit-learn, XGBoost, spaCy, VADER, PRAW, yfinance, CrewAI

## Known limitations

- The CrewAI narrative summaries were generated with a small local model on limited hardware and did not complete successfully, so that panel currently shows placeholder results. Switching to a stronger model is planned.
- The forecast check covers one trading day, so treat the scorecard as a demonstration, not evidence of long-run accuracy.
- Price forecasts are a learning project and not financial advice.
