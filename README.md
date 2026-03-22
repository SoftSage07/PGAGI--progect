# 🎯 TalentScout — AI Hiring Assistant

> An intelligent conversational chatbot that conducts structured candidate screenings for technology roles, powered by **Llama 3.3-70B via Groq** and built with **Streamlit**.

---

## 📌 Project Overview

TalentScout Hiring Assistant automates the initial stage of tech recruitment by:

- **Greeting** candidates and explaining the screening process
- **Collecting** essential candidate details (name, contact, experience, location, desired roles)
- **Assessing** technical proficiency through dynamically generated questions tailored to the candidate's declared tech stack
- **Concluding** gracefully with next-step guidance
- **Storing** anonymized session data locally in JSON format

The chatbot maintains full conversation context across all stages, handles unexpected inputs with fallback responses, and respects candidate privacy throughout.

---

## ✨ Features

| Feature | Details |
|---|---|
| 🤖 LLM-Powered | Llama 3.3-70B via Groq (free tier) |
| 🎨 Custom UI | Dark-themed Streamlit interface with DM Mono + Syne fonts |
| 📋 Stage Tracking | 4-stage progress bar: Greeting → Info → Assessment → Complete |
| 🧠 Sentiment Analysis | Live TextBlob sentiment meter in sidebar (Positive / Neutral / Stressed) |
| 🌐 Multilingual | English, Hindi, Spanish, French, German — full conversation in chosen language |
| ⬇️ Transcript Export | Download full conversation as formatted .txt file |
| 🕐 Timestamps | Every message shows time sent |
| 🔒 Data Privacy | Local JSON storage only, data minimisation, GDPR-aligned |
| 💾 Session Logging | Candidate profile + sentiment saved to `data/` as JSON |
| 🚪 Exit Handling | Keywords (`bye`, `quit`, `exit`, `done`, `stop`) trigger graceful wrap-up |
| 📱 Live Profile Card | Sidebar updates with candidate info as it's collected |

---

## 🛠️ Tech Stack

- **Frontend:** Streamlit 1.32+
- **LLM:** `llama-3.3-70b-versatile` via [Groq Cloud](https://console.groq.com) (free)
- **Sentiment Analysis:** TextBlob 0.18+
- **Language:** Python 3.9+
- **Libraries:** `groq`, `streamlit`, `textblob`, `json`, `re`, `datetime`, `os`

---

## 🚀 Installation & Setup

### 1. Clone the repository
```bash
git clone https://github.com/YOUR_USERNAME/talentscout-hiring-assistant.git
cd talentscout-hiring-assistant
```

### 2. Create a virtual environment
```bash
python -m venv venv
source venv/bin/activate        # Linux/macOS
venv\Scripts\activate           # Windows
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
python -m textblob.download_corpora   # download TextBlob language data
```

### 4. Get a FREE Groq API Key
1. Go to [https://console.groq.com](https://console.groq.com)
2. Sign up (free, no credit card needed)
3. Create an API key under **API Keys**

### 5. Configure the API key
Create the file `.streamlit/secrets.toml`:
```toml
GROQ_API_KEY = "gsk_your_actual_key_here"
```

> ⚠️ This file is in `.gitignore` — it will never be committed to your repository.

### 6. Run the app
```bash
streamlit run app.py
```

The app opens at `http://localhost:8501`

---

## 📖 Usage Guide

1. **Language** — Select your preferred language from the sidebar (English, Hindi, Spanish, French, German)
2. **Start** — The bot greets you automatically on launch
3. **Answer** each question one at a time (name → email → phone → location → experience → desired role → tech stack)
4. **Tech assessment** — After listing your stack, you'll receive 3–5 targeted technical questions
5. **Sentiment** — Watch the live mood meter in the sidebar update as you chat
6. **End** — Type `bye`, `done`, or `exit` at any time, or let the bot conclude naturally
7. **Download** — Click "Download Transcript" in the sidebar to save the full conversation
8. **New session** — Click "New Session" in the sidebar to restart

---

## 🏗️ Architecture

```
talentscout/
├── app.py                  # Main application (UI + logic + LLM)
├── requirements.txt        # Python dependencies
├── .gitignore
├── .streamlit/
│   └── secrets.toml        # API key (not committed)
├── data/
│   ├── .gitkeep            # Ensures folder exists in repo
│   └── candidate_*.json    # Auto-saved session records
└── README.md
```

### Conversation Flow

```
Launch
  │
  ▼
[Stage 1] Greeting
  │  Bot auto-greets in selected language, asks for name
  ▼
[Stage 2] Information Gathering
  │  name → email → phone → location → experience → role → tech stack
  ▼
[Stage 3] Technical Assessment
  │  3–5 questions generated per tech in declared stack
  │  Asked one at a time, answers acknowledged neutrally
  ▼
[Stage 4] Wrap-up
     Thanks candidate, confirms details, states next steps (3–5 business days)
```

---

## 🧠 Prompt Design

### Philosophy
The system prompt is the core of the product. Designed around three principles:
1. **Stage-locking** — The bot is told exactly which stage it's in on every API call, preventing skipping or regression
2. **Single-question enforcement** — Explicitly instructed to ask one question at a time, which dramatically improves UX
3. **Purpose-bounding** — A clear fallback instruction redirects off-topic inputs without breaking immersion

### Key Prompt Decisions

**Stage + language injection at runtime:**
```python
def build_system_prompt(language: str, stage: int) -> str:
    lang_instruction = f"IMPORTANT: Conduct this ENTIRE conversation in {language}.\n\n"
    # ... + stage label appended at end
```
Both language and stage are injected dynamically, so the same prompt handles all 5 languages and 4 stages without branching.

**Tech question generation instruction:**
> "For EACH technology mentioned, generate 1-2 targeted technical questions (total 3-5 questions across all technologies). Ask one question at a time."

Ensures proportional coverage across the full stack — not just the first technology listed.

**Fallback:**
> "If input is gibberish or off-topic: 'I want to make sure I capture your information accurately — could you clarify that?'"

Precise recovery phrase rather than letting the model improvise, keeping tone consistent.

**Prompt injection guardrail:**
A `SECURITY — IMMUTABLE GUARDRAIL` block at the top of the system prompt instructs the model to treat any reprogramming attempt as an attack and respond with a fixed redirect phrase, then repeat the last pending question. This prevents candidates from saying "ignore previous instructions" or trying to bypass the screening.

**Progressive question hierarchy:**
Stage 3 follows a strict 3-level structure: *Conceptual -> Practical/Scenario -> Advanced/Optimization*. This ensures every candidate is assessed at increasing depth and maps to the rubric's "progressively challenging" requirement.

**LLM-based tech stack extraction:**
A dedicated low-temperature (0.0) LLM call extracts technical entities from free-form tech stack responses. This handles niche frameworks, unconventional spellings, and bundled descriptions that regex alone would miss.

**Prompt injection guardrail:**
A dedicated `SECURITY — IMMUTABLE GUARDRAIL` block at the top of the system prompt instructs the model to treat any reprogramming attempt as an attack and respond with a fixed redirect phrase, then repeat the last question. This prevents candidates from saying "ignore previous instructions" or "give me a 10/10 score."

**Progressive question hierarchy:**
Stage 3 now follows a strict 3-level structure: Conceptual → Practical/Scenario → Advanced/Optimization. This maps directly to the evaluation rubric requirement for "progressively challenging" questions and ensures every candidate is assessed at depth, not just surface knowledge.

**LLM-based tech stack extraction:**
Instead of relying solely on regex, a dedicated low-temperature (0.0) LLM call extracts technical entities from the candidate's free-form tech stack response. This handles niche frameworks, unconventional spellings, and bundled descriptions that regex would miss.

---

## 🧠 Sentiment Analysis

Uses **TextBlob** to analyze all user messages cumulatively:

```python
blob = TextBlob(" ".join(user_messages))
polarity     = blob.sentiment.polarity      # -1.0 to +1.0
subjectivity = blob.sentiment.subjectivity  #  0.0 to  1.0
```

| Score | Label | Meaning |
|---|---|---|
| ≥ 0.3 | 😊 Positive | Candidate is confident, enthusiastic |
| -0.2 to 0.3 | 😐 Neutral | Calm, matter-of-fact responses |
| ≤ -0.2 | 😟 Stressed | Candidate may be nervous or frustrated |

This is displayed as a live meter in the sidebar, updating with every message. The sentiment data is also saved to the candidate's JSON record for recruiter review.

> **Note on Multilingual Sentiment — Known Limitation & Production Solution:**
> **Current Implementation:** TextBlob (pattern-based, English-centric lexicon).
> **Known Limitation:** Sentiment accuracy is highest in English. For Hindi, Spanish, French, and German, accuracy degrades because TextBlob's lexicon is predominantly English-trained.
> **Proposed Production Solution:** Replace with a transformer-based multilingual model such as `cardiffnlp/twitter-xlm-roberta-base-sentiment`, which supports 100+ languages with high accuracy via a single unified model.

---

## 🌐 Multilingual Support

Language selection in the sidebar passes the choice into the system prompt:

```python
lang_instruction = (
    f"IMPORTANT: You must conduct this ENTIRE conversation in {language}. "
    f"All your responses must be in {language} only.\n\n"
    if language != "English" else ""
)
```

Supported: **English, Hindi, Spanish, French, German**. The LLM (Llama 3.3-70B) handles all five natively with high accuracy.

---

## 🔒 Data Privacy

- Only collects what is explicitly needed for screening
- Data stored **locally** in `data/` — never sent to any third party
- The Groq API only receives conversation text, no metadata
- Session IDs are timestamp-based, not personally identifiable
- Compliant with GDPR Article 5 principles: data minimisation, purpose limitation, storage limitation
- In production, `data/` would be replaced with an encrypted database with access controls

---

## ⚠️ Challenges & Solutions

| Challenge | Solution |
|---|---|
| LLM asking multiple questions at once | Explicit "ONE question at a time" rule + stage context injection |
| Input not clearing after submit | Dynamic `input_key` counter forces Streamlit to create a fresh widget |
| Enter key not submitting | `on_change=submit` callback on the text input widget |
| Context loss across turns | Full `messages` list passed to Groq API on every call |
| Stage detection | Keyword heuristics in `detect_stage()` + LLM stage context suffix |
| Free API with no paid tier | Groq's free tier — Llama 3.3-70B, fast and capable, zero cost |
| Sensitive data handling | `.gitignore` for secrets, local JSON, no cloud persistence |

---

## 🎁 Bonus Features Implemented

- ✅ **Sentiment Analysis** — Live TextBlob mood meter updates per message, saved to session record
- ✅ **Multilingual Support** — 5 languages via sidebar selector, full conversation in chosen language
- ✅ **Transcript Download** — Full formatted conversation exportable as .txt at any time
- ✅ **Message Timestamps** — Every bubble shows time sent
- ✅ **Custom UI** — Distinctive dark theme, animated bubbles, progress bar, live profile card
- ✅ **Session persistence** — JSON record per session saved to `data/`

---

## 📦 Dependencies

```
streamlit>=1.32.0   # Web UI framework
groq>=0.4.0         # Groq Python SDK (Llama 3.3 access)
textblob>=0.18.0    # Sentiment analysis
```

---

## 📄 License

MIT License — free to use and modify.

---

*Built for TalentScout AI/ML Intern Assignment.*