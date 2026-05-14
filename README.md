# 📊 Retail Investor Sentiment Analysis — India vs UK vs USA

> Analysing how retail investors across three major markets discuss and feel about different asset classes using YouTube comments, VADER, and FinBERT.

---

## 🔍 Project Overview

This project analyses **4,500+ YouTube comments** from investing-focused channels across India, UK, and USA to understand:

- What asset classes investors are most interested in
- How sentiment differs across markets and asset types
- How investor confidence shifted from 2025 to 2026
- Whether general-purpose (VADER) vs financial-domain (FinBERT) models produce different results

Data was collected from **11 YouTube videos** — "where to invest" style guides published in 2025 and 2026 — across channels targeting each respective market.

---

## ❓ Research Questions

| # | Question |
|---|---|
| 1 | What asset classes do investors discuss most across India, UK and USA? |
| 2 | What is the sentiment surrounding each asset class per country? |
| 3 | How has investor sentiment shifted from 2025 to 2026? |
| 4 | Which country shows the most aggressive vs conservative investor profile? |
| 5 | Where does VADER disagree with FinBERT — and what does that tell us? |

---

## 🗂️ Data Sources

| Source | Details |
|---|---|
| **YouTube Data API v3** | Comments from 11 investing videos — 4 India, 4 UK, 3 USA |
| **Year Coverage** | 2025 and 2026 videos for year-on-year comparison |
| **Total Raw Comments** | ~5,100 collected |
| **After Cleaning** | 4,512 comments retained |
| **After Sentence Tokenisation** | 9,392 sentence-level data points |

### YouTube Videos Used

**🇮🇳 India (2025 & 2026)**
- Ankur Warikoo — Investment Strategy 2025 & 2026
- Various Indian finance creators covering stocks, mutual funds, SIPs, crypto

**🇬🇧 UK (2025 & 2026)**
- Damien Talks Money — If I Started Investing in 2026
- UK ETF and ISA focused investing channels

**🇺🇸 USA (2025 & 2026)**
- Nischa — If I Started Investing in 2026
- Humphrey Yang — Full Beginner's Guide to Investing
- The Iced Coffee Hour — Where to Invest Your Money in 2026

---

## ⚙️ Pipeline Architecture

    Raw YouTube Comments
            ↓
    1. Data Collection        YouTube Data API v3 + Pagination
            ↓
    2. Data Cleaning          Word count filter (> 5 words)
                              Duplicate removal
                              Spam and promotional comment detection
            ↓
    3. Sentence Tokenisation  NLTK sent_tokenize
            ↓
    4. Rule-Based Pipeline    Keyword detection via Regex
                              Multi-category sentence handling
                              General_Investing suppression
            ↓
    5. Investing Type Mapping Aggressive / Moderate / Conservative
            ↓
    6. Sentiment Analysis     VADER (general purpose)
                              FinBERT (financial domain specific)
            ↓
    7. Visualisation          Country comparison
                              Year-on-year sentiment velocity
                              VADER vs FinBERT divergence

---

## 🏷️ Asset Categories

| Category | Keywords (Sample) |
|---|---|
| **ETF** | etf, index fund, vanguard, ishares, s&p 500, vwrp |
| **Crypto** | crypto, bitcoin, btc, ethereum, blockchain, solana |
| **Stocks** | stocks, shares, equity, dividend, nasdaq, nifty50 |
| **Mutual Fund** | mutual fund, sip, nav, elss, flexi cap |
| **Property** | property, real estate, reit, mortgage, buy to let |
| **ISA** | isa, stocks and shares isa, lisa, lifetime isa |
| **Pension** | pension, sipp, 401k, roth ira, nps |
| **Gold** | gold, silver, sovereign gold, sgb |
| **FD & Bonds** | fd, fixed deposits, bonds |
| **General Investing** | invest, portfolio, diversify, compound, returns |

---

## 📏 Rule Pipeline Logic

Sentences containing multiple asset category keywords are handled as follows:

    Single keyword found
        → assign to that category

    Multiple keywords found
        → Rule 1: Walk back from 2nd keyword to find a separation symbol
                  Symbols: .  ,  ;  /  -  —  |  !  ?
                  If found → split sentence, assign each segment to its category
        → Rule 2: No symbol found → assign to first keyword's category only

    Post-processing
        → If specific asset categories exist alongside General_Investing
          then remove General_Investing from that sentence's assignment

---

## 🤖 Sentiment Models

### VADER
- General purpose lexicon-based sentiment analyser
- Fast, no model training required
- **Limitation:** Not trained on financial vocabulary — tends to overestimate positivity in financial text

### FinBERT (ProsusAI/finbert)
- BERT model fine-tuned specifically on financial news and text
- Understands domain-specific terms like "bearish", "correction", "volatile"
- **Compound score:** Probability(Positive) - Probability(Negative)
- Consistently scored lower than VADER — validating the financial domain gap

Both models were run and compared to show where general vs domain-specific NLP disagrees.

---

## 📊 Key Findings

### Finding 1 — VADER Overestimates Positivity in Financial Text
FinBERT corrected downward from VADER on approximately 80% of asset-country combinations. The biggest divergence was UK Mutual Fund (-0.80 delta) and USA ISA (-0.57 delta). Studies using general-purpose sentiment models on financial data are likely telling an overly optimistic story.

### Finding 2 — US Investor Confidence Dropped Sharply Year on Year
FinBERT scores show US sentiment fell from +0.17 in 2025 to +0.05 in 2026. UK was already negative at -0.04 in 2025 and barely recovered to -0.01 in 2026. India remained the most stable at +0.09 to +0.05.

### Finding 3 — Crypto Sentiment Is Globally Homogenous
VADER crypto sentiment was nearly identical across all three markets — India: 0.14, UK: 0.14, USA: 0.14. Every other asset class diverged significantly by country. Digital asset sentiment appears to be the most globally uniform investment emotion in online discourse.

### Finding 4 — USA Skews Aggressive, UK Skews Conservative
60% of US investor comments fell into aggressive risk categories (Stocks + Crypto). UK had the highest conservative profile driven by ISA and pension discussion. India sat between the two — reflecting each country's underlying investment culture.

### Finding 5 — India Pension Sentiment Collapsed in 2026
India pension sentiment velocity was -0.47 from 2025 to 2026 — the sharpest single drop in the dataset — suggesting growing concern about retirement planning adequacy among Indian retail investors.

---

## ⚠️ Limitations

| Limitation | Detail |
|---|---|
| **Sample size** | Some asset-country combinations had fewer than 10 sentences — treat as indicative only |
| **Platform bias** | YouTube comments represent engaged viewers of investing content, not the general investing population |
| **Language** | Hinglish comments in Indian data were not filtered — neither VADER nor FinBERT handles these reliably |
| **Demographics** | User age and verified location are not available via YouTube API |
| **Category overlap** | The rule pipeline assigns ambiguous sentences to the first keyword's category |
| **VADER financial gap** | All VADER findings should be read alongside FinBERT results |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| Python 3.12 | Core language |
| Google Colab | Development environment |
| YouTube Data API v3 | Data collection |
| Pandas | Data manipulation |
| NLTK | Sentence tokenisation and VADER sentiment |
| Regex (re) | Keyword detection and rule pipeline |
| Transformers (HuggingFace) | FinBERT model loading and inference |
| PyTorch | FinBERT inference backend |
| Matplotlib + Seaborn | Visualisation |
| Google Drive | Data persistence across Colab sessions |

---

## 🚀 How To Run

**Step 1 — Install dependencies**

    pip install requests pandas nltk transformers torch matplotlib seaborn

**Step 2 — Get a YouTube Data API v3 key**

Go to [Google Cloud Console](https://console.cloud.google.com), create a project, enable YouTube Data API v3, and generate an API key.

**Step 3 — Add your key to the notebook**

Replace the placeholder in the notebook:

    KEY = "YOUR_API_KEY_HERE"

**Step 4 — Mount Google Drive in Colab**

Run the Drive mount cell at the top of the notebook to save your data between sessions.

**Step 5 — Run cells in order**

    1. Data Collection
    2. Data Cleaning
    3. Sentence Tokenisation
    4. Rule Pipeline
    5. Investing Type Mapping
    6. VADER Sentiment Analysis
    7. FinBERT Sentiment Analysis   ← takes 5-10 mins on CPU
    8. Visualisations

> ⚠️ FinBERT inference is slow on CPU. Google Colab GPU runtime is strongly recommended.

---

## 📈 Visualisations

| Visual | Description |
|---|---|
| Asset Sentiment Heatmap | Mean VADER score per country-asset combination |
| Investor Risk Profile | Aggressive / Moderate / Conservative % by country |
| Sentiment Boxplot | Score distribution by risk profile and country |
| Mean Sentiment Bar Chart | Average score per risk profile per country |
| VADER vs FinBERT | Side-by-side model comparison per asset class |
| FinBERT Risk Profile | FinBERT scores by country and risk profile |
| Divergence Chart | Where FinBERT most strongly corrects VADER |
| YoY Change | Sentiment trajectory 2025 vs 2026 per country |
| Sentiment Shift | Asset-level 2025 vs 2026 comparison |
| Sentiment Velocity | Rate of change heatmap by asset and country |

---

## 🧑‍💻 Author

**Harshit**
MSc Advanced Computer Science (AI) — University of Leeds

Self-directed data science project built to develop practical NLP, API integration and data pipeline skills.

---

## 📄 Licence

This project is for educational and portfolio purposes. YouTube comment data collected via official API in compliance with YouTube Terms of Service. No personally identifiable information is stored or published.

---

## 🙏 Acknowledgements

- [ProsusAI/FinBERT](https://huggingface.co/ProsusAI/finbert) — Financial sentiment model
- [NLTK VADER](https://www.nltk.org/) — General sentiment analysis
- YouTube Data API v3 — Data collection
- All finance content creators whose public comment sections formed the dataset
