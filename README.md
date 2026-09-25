# NutriGuard 🥗🤖
### Evidence-Grounded, Uncertainty-Aware Nutrition Intelligence for Indian Diets

> An AI nutrition agent that doesn't just give advice — it knows when to trust itself, when to ask a follow-up question, and when to defer to a dietitian or doctor.

---

## 📌 Problem Statement

India's nutrition-AI tools log meals against Western food databases and issue static, one-size-fits-all advice, with no mechanism for knowing when a recommendation is safe to give autonomously versus when it should be deferred to a dietitian or doctor. Existing India-specific resources like IFCT provide nutrient grounding but lack any uncertainty-aware autonomy control, and no system separates "what happened today" from "is this becoming a persistent pattern" when deciding how to act. This project builds a nutrition intelligence agent that computes nutrient gaps deterministically against IFCT/ICMR-NIN data, reasons across daily and weekly timescales, and dynamically calibrates its own autonomy — answering directly, asking for clarification, or escalating to a human — based on confidence and risk rather than fixed thresholds.

---

## 🎯 Objectives

1. Build a deterministic nutrient computation engine grounded in IFCT 2017 and ICMR-NIN RDA data — eliminating LLM-hallucinated nutrient facts.
2. Design a natural-language meal parser that converts free-text/code-switched logs (e.g., *"2 roti, dal, sabzi"*) into structured food-quantity data with calibrated confidence scores.
3. Implement dual-timescale reasoning — daily gap analysis and weekly/longitudinal pattern detection — as two separate evidence streams.
4. Develop an evidence-grounded recommendation module (RAG over IFCT + ICMR-NIN + Dietary Guidelines) with a verification layer to check outputs before they reach the user.
5. Create an uncertainty-aware autonomy policy that dynamically routes each case to **AI Recommend**, **Clarify**, or **Human Review** based on confidence and risk — not fixed rules.
6. Evaluate the system on a real pilot dataset to measure parsing accuracy, hallucination reduction, and appropriateness of escalation decisions.

---

## 📖 Background

India faces a dual nutrition burden — widespread micronutrient deficiencies alongside rapidly rising obesity and diabetes (**100+ million diabetics and 136+ million prediabetics**, per ICMR 2023; NFHS-6 shows 30%+ of women aged 15–49 are overweight/obese). Existing nutrition apps (MyFitnessPal, HealthifyMe) rely on Western food databases, don't fit Indian meals, and give static, generic advice. LLM-based tools add flexibility but risk hallucinating nutrition facts — a real danger in healthcare. Recent research (RAG-based and GraphRAG nutrition systems) improved grounding and explainability, but none model **uncertainty-aware autonomy** — knowing when it's safe to act versus when to defer to a human expert. This project addresses that exact gap.

---

## 🧠 Key Concepts

1. **RAG (Retrieval-Augmented Generation)** – grounding AI answers in real nutrition data (IFCT, ICMR-NIN), not the model's memory.
2. **Deterministic Nutrient Calculation** – nutrient totals computed by lookup/math, not AI guessing.
3. **Food Parsing** – turning free-text meal logs into structured food + quantity data.
4. **Daily vs. Weekly Reasoning** – separating today's gap from long-term patterns.
5. **Graded Autonomy** – AI decides to answer, ask a question, or escalate — based on confidence/risk.
6. **Human-in-the-Loop** – risky/uncertain cases go to a dietitian or doctor.

---

## ❓ Why This Project Matters

1. Addresses India's growing diabetes/obesity crisis.
2. Existing tools use Western data, not Indian diets.
3. Prevents AI hallucination risk in health advice.
4. Knows when to defer to a doctor/dietitian.
5. Supports, doesn't replace, human experts.
6. Understands natural, everyday meal logging.

---

## 🤖 Role of AI & Technology

| Component | AI/Tech Role |
|---|---|
| **LLM** | Parses natural meal text into structured food data |
| **RAG** | Grounds recommendations in real IFCT/ICMR-NIN evidence |
| **Deterministic Engine** | Handles nutrient math — not AI-generated, no hallucination |
| **Confidence/Risk Scoring** | AI judges how safe/certain a decision is |
| **Verification Layer** | Checks AI output against real data before showing it |
| **Autonomy Router** | AI decides: answer, clarify, or escalate to human |

---

## 🏗️ Proposed Solution & System Architecture

**NutriGuard** combines deterministic Indian nutrient data with uncertainty-aware autonomy across six stages:

```
                              USER
                               │
                               ▼
                    Meal Logging Interface
                    (free text / voice, e.g. "2 roti, dal, sabzi")
                               │
                               ▼
                    Food Parser Agent (LLM)
              → structured {food, quantity, unit} + confidence
                               │
                               ▼
                Food Normalization + IFCT Mapping
          (fuzzy match + synonym table; unresolved → flagged,
                 not silently blended with foreign DBs)
                               │
                               ▼
                 Deterministic Nutrient Engine
        (only component that does arithmetic — full auditability)
                               │
                 ┌─────────────┴─────────────┐
                 ▼                           ▼
          DAILY STATE                LONGITUDINAL STORE
     (current + remaining +                  │
      logging completeness L)                ▼
                 │                   WEEKLY PATTERN
                 │                   (persistence P)
                 └─────────────┬─────────────┘
                                ▼
              Evidence Retrieval + Recommendation Agent
        (RAG: IFCT + ICMR-NIN + Dietary Guidelines — primary)
                                │
                                ▼
                   Minimal-Intervention Optimizer
      Utility = NutrientGain − Cost − Preference − Uncertainty
                                │
                                ▼
                       Verification Layer
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
   Evidence validity   Nutrient validity   Constraint validity
  (citation supports  (matches nutrient   (respects user diet
        claim?)             engine?)          restrictions?)
        └───────────────────┼───────────────────┘
                             ▼
              Confidence + Risk Decomposition
        Cₘ (measurement) · Cₑ (evidence) · R (risk)
              · P (persistence) · L (log completeness)
                             │
                             ▼
                     AUTONOMY POLICY
   knowledge state (Cₘ, Cₑ, L) insufficient → CLARIFY
   risk state (R, confirmed by P) concerning → HUMAN REVIEW
                             │
                             ▼
                     Autonomy Router
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
     CLARIFY          AI RECOMMEND         HUMAN REVIEW
                                           ┌────┴────┐
                                           ▼         ▼
                                     Dietitian    Doctor

═══════════════════════════════════════════════════════════
        AUDIT / EVIDENCE GRAPH (cross-cutting)
log → parsed food → IFCT record → nutrient calc → gap →
retrieved evidence → recommendation → verification → decision
═══════════════════════════════════════════════════════════
```

### Core Contributions
1. **Uncertainty-aware graded autonomy** — a policy that chooses between autonomous recommendation, clarification, and human review using separately tracked confidence and risk signals.
2. **Dual-timescale nutritional reasoning** — daily current/remaining-state estimation plus longitudinal persistence reasoning as two distinct evidence streams.
3. **Verified, evidence-grounded, minimal-intervention recommendation** — deterministic nutrient computation + RAG retrieval + two-part verification + a minimal-intervention utility function.

---

## ⚠️ Challenges

1. **Unstructured source data** – IFCT/ICMR-NIN exist only as PDFs, requiring manual digitization and verification.
2. **Messy input parsing** – accurately handling code-switched, regional meal descriptions (Hindi/English mix).
3. **Confidence calibration** – LLM confidence scores don't reflect true accuracy without validation.
4. **No standard benchmark** – no existing dataset/metric to evaluate "correct" triage decisions.
5. **Balancing autonomy vs. safety** – avoiding both overconfident wrong advice and excessive escalation.

---

## 🛠️ Tools Required

- **Python** (pandas, NumPy)
- **Jupyter / Kaggle Notebook**
- **LLM API** (GPT / Gemini / LLaMA)
- **LangChain + ChromaDB** (RAG pipeline)
- **Sentence Transformers** (semantic matching)
- **FastAPI / Flask** (backend)
- **Streamlit** (frontend UI)

---

## 📊 Datasets

| # | Dataset | Role | Link |
|---|---|---|---|
| 1 | **Indian Food Composition Tables (IFCT 2017)** | Primary nutrient engine data source (~528 foods, 151 nutrients) | [ifct2017.github.io](https://ifct2017.github.io) / [GitHub repo](https://github.com/ifct2017/ifct2017.github.io) |
| 2 | **ICMR-NIN RDA 2020 / Dietary Guidelines for Indians 2024** | Age/sex-specific nutrient targets + RAG corpus | [nin.res.in/dietaryguidelines](https://www.nin.res.in/dietaryguidelines/index.html) |
| 3 | **NFHS-6 (2023–24)** | Problem-statement evidence (obesity/diabetes/anemia prevalence) | [IIPS/NFHS Fact Sheets](http://rchiips.org/nfhs/NFHS-6Reports.shtml) |
| 4 | **FKG.in — Indian Food Knowledge Graph** | Secondary ingredient/recipe resolver (~9,600 recipes) | [arxiv.org/abs/2409.00830](https://arxiv.org/abs/2409.00830) |
| 5 | **Indian Food 101 (Kaggle)** | Dish metadata — diet type, region, flavor profile | [kaggle.com/datasets/nehaprabhavalkar/indian-food-101](https://www.kaggle.com/datasets/nehaprabhavalkar/indian-food-101) |
| 6 | **Nutrition Details for Most Common Foods (Kaggle)** | Secondary/fallback nutrient reference (non-Indian, sanity-check only) | [kaggle.com/datasets/niharika41298/nutrition-details-for-most-common-foods](https://www.kaggle.com/datasets/niharika41298/nutrition-details-for-most-common-foods) |

> ⚠️ **Note:** IFCT and ICMR-NIN RDA are not natively available as clean Kaggle datasets — they are digitized from official PDFs and uploaded as custom Kaggle datasets for notebook use. Secondary sources (USDA FoodData Central, UK McCance & Widdowson, Ministry of AYUSH guidance) are used **only in ablation experiments**, never blended into the primary IFCT-only pipeline, to avoid cross-database inconsistency.

---

## 📈 Performance Metrics

1. **Parser Accuracy** – precision/recall on food & quantity extraction.
2. **Calibration Error (ECE / Brier Score)** – how well AI confidence matches real accuracy.
3. **Nutrient Computation Accuracy** – correctness of IFCT-based calculations (target: 100%, deterministic).
4. **Hallucination Rate** – wrong/unverified facts, before vs. after verification layer.
5. **Escalation Precision/Recall** – correctness of Answer/Clarify/Escalate decisions.
6. **Response Latency** – time from meal log input to final recommendation.

---

## 🖥️ Demonstration (GUI)

1. **Meal Logging Screen** – user types a natural meal log (e.g., *"2 roti, dal, sabzi"*).
2. **Parsed Output** – structured food + quantity + confidence score per item.
3. **Nutrient Dashboard** – today's intake vs. RDA targets (charts/progress bars).
4. **Weekly Trend View** – nutrient gaps over the past 7 days.
5. **Recommendation Card** – AI suggestion with short evidence-based reasoning.
6. **Decision Tag** – 🟢 **AI Recommend** / 🟡 **Clarify** / 🔴 **Human Review**.

Built using **Streamlit** — lightweight, fast to prototype, ideal for demoing live meal-logging → real-time recommendation flow.

---

## 📚 Reference Papers

### Base / Foundational Papers
1. Gavai, A.K. & van Hillegersberg, J. (2025). *AI-driven personalized nutrition: RAG-based digital health solution for obesity and type 2 diabetes.* PLOS Digital Health. [DOI: 10.1371/journal.pdig.0000758](https://doi.org/10.1371/journal.pdig.0000758)
2. Dindukurthi, V., Jain, D., Tripathi, A., Obbineni, J.M., & Kandasamy, I. (2026). *An explainable graph retrieval augmented generation framework for personalized nutrition recommendation.* Frontiers in Artificial Intelligence. [DOI: 10.3389/frai.2026.1808444](https://doi.org/10.3389/frai.2026.1808444)

### Graded Autonomy / Uncertainty-Aware AI
3. Sarkar & Singh-Wolkenhauer (2026). *Artificial intelligence in nutritional oncology: From isolated screening tools to agentic intervention systems.* Oncotarget (editorial).
4. AT-CXR (2025). Chest X-ray triage agent using conformal-prediction-based autonomy control.
5. Conformal-prediction autonomy control in process automation (2026).

### RAG + Nutrition-Specific Grounding
6. Edge, D. et al. (2024). *From local to global: A graph RAG approach to query-focused summarization.* [arXiv:2404.16130](https://arxiv.org/abs/2404.16130)
7. Zhou, H. et al. (2026). *NutriRAG: Unleashing the power of large language models for food identification and classification through retrieval methods.* JAMIA. [DOI: 10.1093/jamia/ocag003](https://doi.org/10.1093/jamia/ocag003)
8. Parameswaran, V. et al. (2025). *Evaluating LLMs and RAG enhancement for delivering guideline-adherent nutrition information for cardiovascular disease prevention.* JMIR. [DOI: 10.2196/78625](https://doi.org/10.2196/78625)
9. Miao, J. et al. (2024). *Integrating retrieval-augmented generation with LLMs in nephrology.* Medicina. [DOI: 10.3390/medicina60030445](https://doi.org/10.3390/medicina60030445)

### India-Specific Grounding Data
10. Gupta, S.K. et al. (2024). *Building FKG.in: A knowledge graph for Indian food.* JOWO/FOIS. [arXiv:2409.00830](https://arxiv.org/abs/2409.00830)
11. Vijayakumar, A. et al. (2024). *Development of an Indian Food Composition Database.* Current Developments in Nutrition. [DOI: 10.1016/j.cdnut.2024.103790](https://doi.org/10.1016/j.cdnut.2024.103790)

### Survey / Gap-Justification
12. Systematic Review (2026). *Generative AI in Precision Nutrition: A Review.* Nutrients (21 eligible studies) — explicitly names uncertainty representation as an unmet challenge.

---

## 📁 Repository Structure (suggested)

```
nutriguard/
├── data/
│   ├── raw/                  # Original IFCT/ICMR-NIN PDFs
│   └── processed/            # Digitized, cleaned CSVs
├── src/
│   ├── parser/                # LLM meal-parsing agent
│   ├── nutrient_engine/       # Deterministic IFCT-based calculator
│   ├── rag/                   # Retrieval + recommendation pipeline
│   ├── verification/          # Evidence + nutrient + constraint validity checks
│   └── autonomy/              # Confidence/risk scoring + routing logic
├── app/
│   └── streamlit_app.py       # GUI demo
├── notebooks/
│   └── kaggle_pipeline.ipynb  # End-to-end Kaggle notebook
├── evaluation/
│   └── metrics.py             # Parser accuracy, ECE, hallucination rate, etc.
├── docs/
│   └── architecture.png
├── requirements.txt
└── README.md
```

---

## 🚀 Getting Started

```bash
# Clone the repository
git clone https://github.com/<your-username>/nutriguard.git
cd nutriguard

# Install dependencies
pip install -r requirements.txt

# Run the Streamlit demo
streamlit run app/streamlit_app.py
```

---

## 📄 License

This project is for academic/research purposes. Please check individual dataset licenses (IFCT/ICMR-NIN — NIN copyright terms apply; FKG.in — refer to source repo license) before redistribution.

---

## 🙏 Acknowledgements

Built on the grounding provided by NIN/ICMR's IFCT 2017 and RDA guidelines, and inspired by prior work in RAG-based (Gavai & van Hillegersberg, 2025) and GraphRAG-based (Dindukurthi et al., 2026) nutrition recommendation systems.
