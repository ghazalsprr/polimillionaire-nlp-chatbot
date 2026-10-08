# 🏆 Who Wants to Be a PoliMillionaire? — LLM Quiz Chatbot

An LLM-powered chatbot that plays an online *Who Wants to Be a Millionaire*–style quiz game, in both **text** and **speech** mode, using only **local open-weight models** (no paid APIs).

> NLP Group Assignment 2025/26 — **Politecnico di Milano** · Group **AGAH**

---

## Overview

The game server asks up to 15 multiple-choice questions of increasing difficulty. A wrong answer ends the game, and each answer must be submitted within **30 seconds**. Our chatbot:

1. receives the question and four options from the game API (as text, or as audio in speech mode),
2. transcribes the audio with **Whisper** when in speech mode,
3. reasons over the question with a local LLM — optionally augmented with retrieval or tools,
4. parses a single valid answer and submits it before the timer runs out.

We evaluated the system across **six competitions** — Math, Ancient History & Politics, News, Science, Philosophy & Psychology, and Entertainment — comparing models, prompt strategies, retrieval, tools and ensembles.

```
            ┌──────────── speech mode only ────────────┐
Game API ──►│ audio ──► Whisper ASR ──► transcript      │──► Prompt builder ──► Local LLM ──► Answer parser ──► Game API
            └───────────────────────────────────────────┘          ▲
                                                                   │
                                          RAG (Wikipedia / web / static corpus) · Calculator tools (SymPy)
```

## What we explored

| Area | Techniques |
|---|---|
| **Models** | Qwen2.5 (1.5B / 3B / 7B / Math-7B), Qwen3 (4B / 8B), Phi-3 / Phi-3.5-mini, Llama 3 / 3.1 8B, Mistral-7B, Gemma, SmolLM2, DeepSeek-R1-Distill, FLAN-T5 |
| **Prompting** | Zero-shot, few-shot, anti-bias few-shot, chain-of-thought, structured "think → compute → answer", domain-expert prompts, elimination prompts |
| **Retrieval (RAG)** | Wikipedia, live web search with date windows (News), static corpora (SciQ, psychology), FAISS + Sentence-Transformers, TF-IDF memory of past questions |
| **Tools** | Python / SymPy calculator, LaTeX normaliser, two-turn tool agent and question router (Math) |
| **Robustness** | Self-consistency voting, model ensembles, two-stage elimination, confidence calibration, adaptive prompting by difficulty level |
| **Speech** | Whisper `tiny.en` → `medium.en` via faster-whisper, domain vocabulary hints, WER-based ASR diagnostics |
| **Efficiency** | 4-bit NF4 quantisation (BitsAndBytes), one-model-at-a-time loading to fit Colab memory |

## Key results

Accuracy is measured over the full game (correct answers ÷ 15), so a model that loses early is not over-rewarded.

### Text mode

| Competition | Best system | Result |
|---|---|---|
| Philosophy & Psychology | Phi-3.5-mini + short domain prompt | **14.2** avg questions · **92%** game accuracy · ~0.44 s/answer |
| Science | Phi-3.5-mini + short domain prompt | **13.8** avg questions · **86.7%** game accuracy · ~0.42 s/answer |
| Ancient History & Politics | Qwen2.5-3B (zero-shot) | **9.0** avg level; self-consistency reached level 14 |
| News | Phi-3.5-mini + date-window web RAG | **7.4** avg questions · best run 10 |
| Entertainment | Phi-3.5-mini + online RAG | **6.1** avg questions · top prize of 1,024,000 |
| Math | Qwen2.5 / Qwen3 + calculator router + RAG | compared across 5 models and 8 strategies |

### Speech mode

| Competition | Best system | Result |
|---|---|---|
| Philosophy & Psychology | Phi-3.5-mini + few-shot + Whisper base.en | **10.8** avg questions · perfect 15/15 game |
| Science | Phi-3.5-mini + elimination + Whisper base.en | **9.0** avg level · perfect 15/15 game |
| Ancient History & Politics | Phi-3-mini + few-shot + RAG | **6.7** avg level · perfect 15/15 game |
| News | Phi-3.5-mini + date-window RAG + Whisper small.en | **6.4** avg questions · best run 12 |

All speech pipelines stayed within the 30-second limit (ASR ≈ 7–8 s + LLM ≈ 1–2 s per question).

### Main takeaways

- **Bigger isn't automatically better.** Compact models like Phi-3.5-mini and Qwen2.5-3B regularly beat 7–8B models.
- **Short, controlled prompts win in a timed game.** Chain-of-thought and "thinking" models were slower and usually less accurate.
- **Retrieval helps most where knowledge is time-specific** (News, Entertainment), and much less where the model already knows the domain.
- **Speech adds a real penalty** — ASR errors on names, dates and numbers cost several levels — but domain vocabulary hints and RAG recover part of it.

## Repository structure

```
notebooks/
├── text/
│   ├── math.ipynb
│   ├── history_politics.ipynb
│   ├── news.ipynb
│   ├── science.ipynb
│   ├── philosophy_psychology.ipynb
│   └── entertainment.ipynb
└── speech/
    ├── history_politics.ipynb
    ├── news.ipynb
    ├── science.ipynb
    └── philosophy_psychology.ipynb
```

Each notebook is self-contained: setup, model loading, prompt strategies, game loop, experiments, and a written analysis of the results.

## Running the notebooks

The notebooks were developed on **Google Colab** with a GPU runtime.

1. Open a notebook in Colab and select a GPU runtime.
2. Place the course-provided `millionaire_client` package in your Google Drive (the notebooks copy it into the runtime).
3. Set your game credentials as Colab Secrets or environment variables — **never hard-code them**:
   - `MILLIONAIRE_USERNAME`
   - `MILLIONAIRE_PASSWORD`
   - `HF_TOKEN` for gated Hugging Face models (e.g. Llama)
4. Run the cells top to bottom.

To install the main dependencies locally:

```bash
pip install -r requirements.txt
```

> The game server and the `millionaire_client` package were provided by the course and are not included in this repository.

## Team

| Name | Contact |
|---|---|
| Hedieh Raeisi | hedieh.raeisi@mail.polimi.it |
| Ghazal Sepehrirad | ghazal.sepehrirad@mail.polimi.it |
| Amin Sakhaei | amin.sakhaei@mail.polimi.it |
| Amirhossein Sharifisadr | amirhossein.sharifisadr@mail.polimi.it |

---

*Developed for the Natural Language Processing course at Politecnico di Milano, 2025/26.*
