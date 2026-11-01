# Insurance Premium Prediction

A machine-learning application that predicts an insurance premium category from a user's demographic, lifestyle, location, and income details. The project includes a FastAPI prediction API, a Streamlit frontend, and a Docker image for running the API.

## Docker image

Docker Hub repository: https://hub.docker.com/repository/docker/varunvadhwani/insurance-premium-api/general

```bash
docker pull varunvadhwani/insurance-premium-api:latest
docker run --rm -p 8000:8000 varunvadhwani/insurance-premium-api:latest
```

The API will be available at `http://localhost:8000`.

To build the image from this project instead:

```bash
docker build -t insurance-premium-predictor .
docker run --rm -p 8000:8000 insurance-premium-predictor
```

The Dockerfile uses Python 3.11, installs dependencies from `requirements.txt`, and starts Uvicorn on port 8000.

## Features

- FastAPI endpoints for health checks and premium-category predictions.
- Pydantic validation of user input.
- Automatic derivation of BMI, age group, lifestyle risk, and city tier.
- Prediction confidence and per-category probabilities from the bundled model.
- Streamlit form-based frontend.

## Project structure

```text
.
├── app.py                       # FastAPI application
├── frontend.py                  # Streamlit user interface
├── Dockerfile                   # Container image for the API
├── requirements.txt             # API dependencies
├── config/
│   └── city_tier.py             # Tier 1 and tier 2 city lists
├── model/
│   ├── model.pkl                # Trained prediction model
│   └── predict.py               # Model loading and prediction logic
└── schema/
    ├── prediction_response.py   # Prediction response schema
    └── user_input.py            # Input validation and derived fields
```

## Run locally

Use Python 3.11. From the project root, create and activate a virtual environment, then install the API dependencies:

```bash
python -m venv .venv
```

Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

macOS/Linux:

```bash
source .venv/bin/activate
```

Install dependencies and start the API:

```bash
python -m pip install -r requirements.txt
uvicorn app:app --host 0.0.0.0 --port 8000
```

Open `http://localhost:8000/docs` for the interactive API documentation.

### Run the Streamlit frontend

The current `requirements.txt` contains the API dependencies. Install the frontend packages separately:

```bash
python -m pip install streamlit requests
streamlit run frontend.py
```

By default, the frontend sends requests to `http://127.0.0.1:8000/predict`. Set the `API_URL` environment variable if the API is running elsewhere. For example, in PowerShell:

```powershell
$env:API_URL = "http://localhost:8000/predict"
streamlit run frontend.py
```

## API

### `GET /`

Returns a service welcome message.

### `GET /health`

Returns the service status, model version, and whether the model loaded.

### `POST /predict`

Example request:

```json
{
  "age": 30,
  "weight": 65,
  "height": 1.7,
  "income_lpa": 10,
  "smoker": false,
  "city": "Mumbai",
  "occupation": "private_job"
}
```

`occupation` must be one of `retired`, `freelancer`, `student`, `government_job`, `business_owner`, `unemployed`, or `private_job`. Age must be between 1 and 119; weight, height, and income must be positive; height must be below 2.5 metres.

The input schema derives the following model features:

- `bmi`: weight divided by height squared.
- `age_group`: `young` (under 25), `adult` (25–44), `middle_aged` (45–59), or `senior` (60+).
- `lifestyle_risk`: based on smoking status and BMI.
- `city_tier`: tier 1 for listed major cities, tier 2 for listed secondary cities, and tier 3 for other cities. City names are normalized before matching.

The prediction contains a category, confidence score, and probability for each class. **Current implementation note:** the route currently places these fields inside a top-level `response` object, while the declared response schema and Streamlit frontend expect them at the top level. Align the route and consumers before relying on the documented schema or using the frontend end to end.

## Model

The trained model is loaded from `model/model.pkl` by `model/predict.py` when the API starts. The model version returned by the health endpoint is currently `1.0.0`.
