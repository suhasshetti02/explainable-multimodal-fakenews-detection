# TruthLab — Multimodal Fake News Forensics

TruthLab is a multimodal news-claim analysis prototype. It combines a trained text-and-image classifier with external search snippets and LLM-assisted interpretation. The repository also contains a separate video-forensics flow that extracts audio, obtains a transcript, and asks a model to assess the video's claims.

> **Project status:** Experimental decision-support tool. Model output, search snippets, generated explanations, and video analysis are not definitive fact-checks. Human verification of primary sources is required.

## Capabilities visible in the source

- Text/image analysis through a PyTorch model combining MuRIL text features and a torchvision Vision Transformer image encoder using a cross-attention fusion layer.
- Binary fake/real output and a claim-topic classification head.
- Text-token and image explanation utilities, including SHAP-related code and image gradient / attention visualisation helpers.
- External search through SerpAPI for evidence snippets.
- Gemini-based explanatory / fallback verification logic.
- Video workflow that extracts audio using MoviePy, transcribes in 30-second chunks through Sarvam AI (configured for Kannada), and evaluates the transcript/video through a Gemini model.
- React/Vite interface for investigation, dashboard, and video-forensics workflows.

## Technology stack

| Layer | Technologies |
|---|---|
| Frontend | React 18, Vite, React Router, Tailwind CSS |
| API | Python, FastAPI |
| Deep learning | PyTorch, torchvision, Hugging Face Transformers |
| Text model | Google MuRIL |
| Image model | torchvision ViT-B/16 |
| Explanations | SHAP and custom image/token attribution helpers |
| External services | SerpAPI, Google Gemini, Sarvam AI |
| Media processing | MoviePy, FFmpeg-related tooling, pydub / Whisper dependencies |

## Architecture

```text
React / Vite client
  ├── POST /analyze ──► FastAPI
  │                       ├── MuRIL text encoder
  │                       ├── ViT image encoder
  │                       ├── cross-attention / classifier heads
  │                       └── SerpAPI evidence + Gemini assistance
  ├── POST /ai_summarise ─► Gemini explanation
  └── POST /video_verify ─► video/audio extraction
                              └── Sarvam transcription + Gemini review
```

## Repository layout

- `fake_news_detection/backend/` — FastAPI application, model, prediction, fusion, and video helpers.
- `fake_news_detection/frontend/vite-project/` — React/Vite client.
- `fake_news_detection/README.md` — nested project notes.
- Root `README.md` — project overview.

## Run locally

Requirements: Python 3.10+, Node.js 18+, FFmpeg, model weights, and credentials for services you enable.

Backend:

```bash
cd fake_news_detection/backend
python -m venv .venv
# Activate the environment
pip install -r requirements.txt
```

Create a backend environment file with the credential names read by the code (for example `gemini_api_key2`, `SERP_API_KEY`, and `SARVAM_API_KEY`). The backend loads model weights from `model_multimodal/best_model.pth`; ensure the artifact and compatible model files exist before starting the service.

Start FastAPI with the repository's configured entry module, for example:

```bash
uvicorn server:app --host 0.0.0.0 --port 8000
```

Frontend:

```bash
cd fake_news_detection/frontend/vite-project
npm install
npm run dev
```

## API routes implemented

- `POST /analyze` — multipart form fields `title` and `image`.
- `POST /ai_summarise` — accepts an evidence object and returns a generated summary.
- `POST /video_verify` — accepts a video file and returns analysis output.
- `GET /` — basic server health response.

## Limitations and security notes

- The backend CORS configuration currently allows all origins; restrict it before deployment.
- Uploaded filenames are used to construct local paths and files are stored under local upload directories. Add generated filenames, strict file validation, size limits, cleanup, and isolated storage before production.
- Model accuracy depends on the exact trained weights and evaluation data; do not quote accuracy without a reproducible evaluation report.
- SerpAPI results are snippets / search results, not proof that a claim is true or false. Gemini output may be wrong or outdated.
- The video transcription code is configured for Kannada and uses an external Sarvam endpoint; it should not be described as language-agnostic transcription.
- The current explanation helpers and classifier outputs require careful interpretation; a heatmap is not proof that an image is manipulated.
- Keep all provider credentials and model artifacts out of source control. Rotate any credential that has ever been committed.

## License

See repository licensing and model / dataset terms.
