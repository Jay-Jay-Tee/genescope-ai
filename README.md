# GeneScope 🧬💙

A web-based AI preventive health platform that predicts an individual's genetically inherited disease risks using family history, personal health records, and lifestyle data — then acts as a dynamic health coach by recommending prevention plans, screening tests, and urgent consultation when needed.

Built for the **ChangeMaker: AI for Good** hackathon.

---

## The Problem

Many people carry a genetic predisposition to serious conditions — Type 2 Diabetes, Coronary Artery Disease, Hereditary Cancer, Thalassemia, Sickle Cell Disease — without ever knowing until symptoms appear. Preventive healthcare, especially in India, remains reactive. Genetic testing is expensive and inaccessible. Adolescents and young adults rarely seek screening.

This platform bridges that gap: using family history and health records as a proxy for genetic risk, making early risk awareness accessible without expensive lab tests.

---

## What It Does

1. **Collects user profile** — age, weight, height, known conditions, lifestyle habits
2. **Collects family medical history** — up to 2 generations (parents, grandparents), including their conditions and vitals
3. **Optionally parses uploaded medical documents** via AI-powered OCR (lab reports, prescriptions)
4. **Runs multi-disease risk scoring** using rule-based AI models with full explainability
5. **Returns risk classes I–IV** (Low → Very High) for each disease, with probability scores
6. **Generates personalized output** per disease:
   - Plain-language disease explanation
   - Personalized prevention plan (diet, exercise, lifestyle)
   - Recommended screening tests with frequency and prep instructions
   - Urgency guidance — from "routine annual checkup" to "consult a doctor this week"

---

## Diseases Covered

| Disease | Category |
|---|---|
| Type 2 Diabetes | Metabolic |
| Coronary Artery Disease | Cardiovascular |
| Hypertension | Cardiovascular |
| Familial Hypercholesterolemia | Cardiovascular |
| Hereditary Breast & Ovarian Cancer (BRCA) | Oncological |
| Thalassemia | Hematological |
| Sickle Cell Disease | Hematological |
| Asthma | Respiratory |
| Hypothyroidism | Endocrine |
| PCOS | Endocrine |

---

## Risk Classification

| Class | Label | Action |
|---|---|---|
| I | Low Risk | Routine annual checkup |
| II | Moderate Risk | Screen within 3–6 months |
| III | High Risk | Consult doctor within 1 month |
| IV | Very High Risk | Urgent — consult within 1–2 weeks |

Each class drives personalized prevention advice, test selection, frequency adjustments, and urgency messaging throughout the platform.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  FRONTEND (React + Vite)             │
│  LandingPage → RegistrationForm → FamilyHistoryForm  │
│  → LifestyleForm → UploadReport → ResultsDashboard   │
│                                                      │
│  React · Vite · TailwindCSS · Lucide Icons · Axios   │
└─────────────────┬───────────────────────────────────┘
                  │  REST API (Axios / JSON)
                  ▼
┌─────────────────────────────────────────────────────┐
│                   BACKEND (FastAPI)                  │
│                                                      │
│  POST /api/predict    → prediction_service           │
│  POST /api/ocr        → ocr_service                  │
│  GET  /api/disease-info/{id}                         │
│                                                      │
│  FastAPI · Uvicorn · Pydantic · Python-dotenv        │
└─────────────────┬───────────────────────────────────┘
                  │  Python imports
                  ▼
┌─────────────────────────────────────────────────────┐
│                     AI MODULE                        │
│                                                      │
│  ai/risk/                                            │
│    risk_model.py       ← predict_risks() entry point │
│    scoring_rules.py    ← per-disease scoring rules   │
│    risk_classes.py     ← I–IV classification logic   │
│    explainability.py   ← human-readable reasons      │
│                                                      │
│  ai/coaching/                                        │
│    consult_logic.py       ← backend↔AI bridge        │
│    prevention_engine.py   ← personalized plans       │
│    test_recommender.py    ← screening test selection │
│                                                      │
│  ai/ocr/                                             │
│    ocr_pipeline.py     ← medical document parsing    │
│                                                      │
│  ai/data/                                            │
│    diseases_config.json   ← disease metadata         │
│    guidelines.json        ← prevention guidelines    │
│    tests_map.json         ← screening tests database │
│    sample_inputs.json     ← test payloads            │
└─────────────────────────────────────────────────────┘
```

---

## Project Structure

```
genescope/
├── ai/
│   ├── coaching/
│   │   ├── consult_logic.py          # Backend↔AI bridge, predict_risk()
│   │   ├── prevention_engine.py      # Personalized prevention plans
│   │   └── test_recommender.py       # Screening test selection & prioritization
│   ├── data/
│   │   ├── diseases_config.json      # Disease metadata and descriptions
│   │   ├── guidelines.json           # Prevention guidelines for all diseases
│   │   ├── sample_inputs.json        # Sample request payloads for testing
│   │   └── tests_map.json            # Screening test database per disease
│   ├── ocr/
│   │   └── ocr_pipeline.py           # Medical document OCR pipeline
│   ├── risk/
│   │   ├── explainability.py         # Human-readable risk reasons
│   │   ├── risk_classes.py           # Risk class I–IV assignment
│   │   ├── risk_model.py             # Main predict_risks() function
│   │   └── scoring_rules.py          # Per-disease scoring rules
│   └── requirements.txt
├── backend/
│   ├── app/
│   │   ├── main.py                   # FastAPI app entry point
│   │   ├── config.py                 # Settings and environment config
│   │   ├── routers/
│   │   │   ├── diseases.py           # GET /api/disease-info/
│   │   │   ├── ocr.py                # POST /api/ocr
│   │   │   └── predict.py            # POST /api/predict
│   │   ├── schemas/
│   │   │   ├── prediction.py         # RiskRequest, RiskResponse schemas
│   │   │   ├── diseases.py           # Disease info schemas
│   │   │   ├── family.py             # Family member schema
│   │   │   ├── lab_values.py         # Lab results schema
│   │   │   ├── lifestyle.py          # Lifestyle inputs schema
│   │   │   └── profile.py            # Patient profile schema
│   │   └── services/
│   │       ├── prediction_service.py # Calls AI risk module
│   │       ├── ocr_service.py        # Calls OCR pipeline
│   │       └── disease_service.py    # Disease info lookup
│   ├── .env.example
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   │   ├── client.js             # Axios instance with interceptors
│   │   │   ├── predict.js            # POST /api/predict
│   │   │   ├── ocr.js                # POST /api/ocr
│   │   │   └── diseases.js           # GET /api/disease-info/
│   │   ├── components/
│   │   │   ├── FileUpload.jsx        # Drag-and-drop medical document upload
│   │   │   ├── Navbar.jsx
│   │   │   ├── Footer.jsx
│   │   │   ├── RiskCard.jsx          # Disease risk result card
│   │   │   ├── RiskLegend.jsx        # Risk class color legend
│   │   │   ├── ProgressStepper.jsx   # Multi-step form progress indicator
│   │   │   ├── ErrorBanner.jsx
│   │   │   └── LoadingSpinner.jsx
│   │   ├── pages/
│   │   │   ├── LandingPage.jsx
│   │   │   ├── RegistrationForm.jsx  # Patient profile input
│   │   │   ├── FamilyHistoryForm.jsx # Family medical history (2 generations)
│   │   │   ├── LifestyleForm.jsx     # Lifestyle habits input
│   │   │   ├── UploadReport.jsx      # Medical document upload (OCR)
│   │   │   ├── Assessment.jsx
│   │   │   ├── ResultsDashboard.jsx  # Final results with all disease risks
│   │   │   └── DiseaseDetail.jsx     # Per-disease detail with prevention plan
│   │   ├── hooks/
│   │   │   └── useLocalStorage.js
│   │   └── utils/
│   │       ├── formatters.js
│   │       └── validators.js
│   ├── .env.example
│   └── package.json
├── docs/
│   └── api_contract.json             # Full OpenAPI-style contract
├── render.yaml                       # Render.com deployment config
└── requirements.txt
```

---

## Quickstart

### Prerequisites

- Python 3.9+
- Node.js 18+

### 1. Clone

```bash
git clone https://github.com/Jay-Jay-Tee/genescope.git
cd genescope
```

### 2. Backend

```bash
cd backend
cp .env.example .env
pip install -r requirements.txt
pip install -r ../ai/requirements.txt
uvicorn app.main:app --reload --port 8000
```

The API will be running at `http://localhost:8000`.

### 3. Frontend

```bash
cd frontend
cp .env.example .env
# Edit .env and set: VITE_API_URL=http://localhost:8000/api
npm install
npm run dev
```

The app will be running at `http://localhost:5173`.

---

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/predict` | Submit patient + family + lifestyle data, receive disease risk results |
| `POST` | `/api/ocr` | Upload medical document (PDF/image), extract structured health data |
| `GET` | `/api/disease-info/` | List all supported diseases |
| `GET` | `/api/disease-info/{id}` | Detailed info for a specific disease |

Full API contract: [`docs/api_contract.json`](docs/api_contract.json)

### Sample prediction request

```json
{
  "patient": {
    "age": 22,
    "gender": "M",
    "weight": 56.7,
    "height": 171,
    "known_issues": ["hypertension"]
  },
  "lifestyle": {
    "diet": "high_sugar",
    "exercise": "sedentary",
    "smoking": false,
    "stress_level": "high",
    "sleep_hours": 5
  },
  "family": [
    {
      "role": "mother",
      "generation": 1,
      "known_issues": ["diabetes", "hypertension", "heart-disease"]
    },
    {
      "role": "father",
      "generation": 1,
      "known_issues": ["diabetes", "cancer"]
    }
  ]
}
```

### Sample prediction response (per disease)

```json
{
  "disease_name": "Type 2 Diabetes",
  "disease_id": "type2_diabetes",
  "probability": 0.74,
  "risk_class": "III",
  "reasons": [
    "Mother has Type 2 Diabetes — first-degree family history significantly increases risk",
    "Father has Type 2 Diabetes — bilateral parental history compounds risk",
    "High-sugar diet identified as a modifiable risk factor",
    "Sedentary lifestyle increases insulin resistance risk"
  ],
  "prevention": { "diet": [...], "exercise": [...], "lifestyle": [...] },
  "recommended_tests": ["HbA1c", "Fasting Blood Glucose", "..."],
  "consult": "Consult doctor within 1 month"
}
```

---

## AI Module Design

The risk engine is fully rule-based and explainable — no black-box ML. This was a deliberate design decision: in a health context, every factor that contributes to a risk score is surfaced to the user as a plain-language reason.

**Scoring pipeline:**

1. `risk_model.py` — `predict_risks(user_data)` iterates over all covered diseases
2. `scoring_rules.py` — per-disease rules weight family history, lifestyle, vitals, and lab values into a 0–1 probability
3. `risk_classes.py` — probability mapped to class I / II / III / IV
4. `explainability.py` — generates human-readable reason strings per contributing factor
5. `prevention_engine.py` — personalized plan based on disease + class + user profile
6. `test_recommender.py` — selects and prioritizes screening tests; adjusts frequency by risk class

**Test recommendations by risk class:**

| Class | Max tests shown |
|---|---|
| I | 2 |
| II | 3 |
| III | 5 |
| IV | 6 |

Test frequency is automatically tightened as risk class increases (e.g. "Annually" → "Every 6 months" for Class III, "Every 3–6 months" for Class IV).

---

## Deployment

Deployed on **Render** using `render.yaml`.

- **Backend**: Python web service running `uvicorn app.main:app`
- **Frontend**: Static site — Vite builds to `dist/`, served as a static deploy

Environment variables on Render:

| Variable | Where | Description |
|---|---|---|
| `VITE_API_URL` | Frontend build env | Backend API base URL |
| Backend secrets | Backend service env | See `backend/.env.example` |

---

## SDG Alignment

**SDG 3: Good Health and Well-Being**

GeneScope directly addresses the gap between knowing a disease runs in your family and understanding what to do about it. By making risk awareness accessible, actionable, and free, it supports early detection and preventive intervention — reducing long-term healthcare burden and avoidable complications.

---

## Disclaimer

⚠️ GeneScope is a **preventive decision-support tool**, not a licensed medical diagnostic system. All high-risk outputs explicitly recommend professional medical consultation. Results should not replace clinical diagnosis, testing, or treatment by a qualified healthcare provider.

---

## Team

Built at **NIT Calicut, 2026**:

- Joshua Jacob Thomas (Lead)
- HR Soorya Dev
- Dinesh Paliwal
- Mayank Singh
- Aditya Singh
- Nirmal Kumar Prajapath
- Satrajit Mondal
- Sai Pranav Parasuraman
- Sagnik Dutta
- Siddharth Madhavan

---

## License

MIT License — see [LICENSE](LICENSE)
