# sitesense
> Voice-first construction progress tracking and delay prediction for Hindi, Gujarati and Hinglish site teams.

![Status](https://img.shields.io/badge/status-in%20development-orange)
![Python](https://img.shields.io/badge/python-3.11+-blue)
![License](https://img.shields.io/badge/license-MIT-green)

**sitesense** (Hindi/Gujarati for "measurement") is an AI agent that turns daily WhatsApp/Telegram voice notes and photos from construction site supervisors into structured progress logs, compares them against the BOQ and schedule, and flags likely delays before they become expensive.

> ⚠️ **Project status:** actively under development. Features below are marked as planned or done in the roadmap. All data in this repository is **synthetic**; no client or proprietary data is used.

---

## The problem

On most small and mid-size construction sites, progress reporting is informal. The supervisor knows what happened today, but the office learns about it late, incompletely, or in a format that cannot be analysed. Delays are discovered when fixing them is already costly.

Existing AI progress-monitoring tools typically depend on 360° photo capture, BIM models and enterprise budgets. Naap targets the teams those tools miss: **low cost, messaging-app native, vernacular voice, BOQ-based, no BIM required.**

## How it works

1. **Capture:** the supervisor sends a voice note (e.g. *"aaj slab ka shuttering 60 percent hua, cement kam pad gaya"*) and 2-3 photos.
2. **Transcribe:** voice activity detection plus Indic speech recognition handles Hindi, Gujarati and code-mixed Hinglish.
3. **Extract:** an LLM converts the transcript into a validated, structured log (activity, quantity, labour, material issues, blockers).
4. **Match:** the log is mapped to the correct BOQ line item with a confidence score.
5. **Verify:** a multimodal model checks whether the photos support the claim; mismatches are flagged.
6. **Compare:** planned vs actual quantity and schedule performance are computed per activity.
7. **Predict:** the system estimates which activities are likely to slip and the probable cause.
8. **Review:** a project manager approves, flags or comments in a dashboard; every decision is stored in an audit trail.

## Architecture

```
 Supervisor (voice + photos)
          |
          v
   Telegram / WhatsApp bot
          |
          v
 +---------------------+      +-------------------+
 |  VAD + ASR          | ---> |  LLM extraction   |
 |  (Hindi/Gujarati)   |      |  (JSON schema)    |
 +---------------------+      +---------+---------+
                                         |
                    +--------------------+-------------------+
                    v                                        v
          +------------------+                    +---------------------+
          |  BOQ matcher     |                    | Photo verification  |
          |  (embed + rerank)|                    | (multimodal model)  |
          +--------+---------+                    +----------+----------+
                   |                                         |
                   +--------------------+--------------------+
                                        v
                          +--------------------------+
                          | Progress engine          |
                          | planned vs actual, SPI   |
                          +------------+-------------+
                                       v
                          +--------------------------+
                          | Delay prediction + alerts|
                          +------------+-------------+
                                       v
                          +--------------------------+
                          | Review dashboard         |
                          | approve / flag + audit   |
                          +--------------------------+
```

## Tech stack (planned)

| Layer | Choice |
|---|---|
| Input | Telegram bot (WhatsApp Cloud API later) |
| Speech | Indic ASR models (e.g. IndicConformer, Sarvam) with Whisper as baseline; Silero VAD |
| Extraction | LLM with JSON schema + Pydantic validation |
| BOQ matching | Embeddings, fuzzy matching, LLM reranking |
| Vision | Multimodal LLM for claim-vs-photo consistency |
| Backend | FastAPI, PostgreSQL, Redis |
| Memory / routing | Hindsight (site memory), CascadeFlow (cost-aware model routing) |
| Forecasting | Rule-based schedule performance index first, then gradient boosting |
| Dashboard | Streamlit |
| Evaluation | pytest, labelled test set, experiment tracking |

## Roadmap

- [ ] **Week 1:** repo setup, bot receives voice and photos, VAD + ASR, transcripts stored
- [ ] **Week 2:** structured extraction, BOQ matching, planned vs actual, audit log
- [ ] **Week 3:** photo verification, review dashboard, memory, model routing
- [ ] **Week 4:** delay alerts, evaluation report, demo video, documentation

## Evaluation plan

Results will be published here once measured. Planned metrics:

- **Transcription:** word error rate on a labelled set of Hindi/Gujarati/Hinglish site voice notes
- **Extraction:** field-level precision and recall
- **BOQ matching:** top-1 and top-3 accuracy
- **Photo verification:** agreement with human labels
- **Delay prediction:** how many days in advance slippage is flagged, compared with a naive baseline
- **Business metric:** time to produce the daily progress report

## Data and privacy

All BOQs, schedules, transcripts and logs in this repository are **synthetic**, generated by scripts in `data/synthetic/`. No real client or personal data is stored here.

## Known limitations

- Speech models are benchmarked mostly on clean audio; accuracy on noisy site recordings must be measured, not assumed.
- Gujarati coverage depends on the chosen ASR model and will be validated before any claims are made.
- Delay prediction needs historical project outcomes; the first version uses transparent rule-based metrics, with learned models added as data allows.

## Repository structure

```
naap/
├── src/naap/
│   ├── bot/            # messaging handlers
│   ├── asr/            # VAD + transcription
│   ├── extraction/     # LLM structured extraction
│   ├── boq/            # BOQ loader + matcher
│   ├── vision/         # photo-claim verification
│   ├── progress/       # planned vs actual, SPI
│   ├── forecast/       # delay prediction
│   ├── review/         # approve/flag workflow, audit log
│   └── api/
├── dashboard/          # review UI
├── data/synthetic/     # synthetic data generators
├── eval/               # labelled cases + evaluation scripts
├── tests/
└── docs/
```

## License

MIT
