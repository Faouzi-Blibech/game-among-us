# ai_service/ — Python AI service rules

Read root `CLAUDE.md` first. API contract: `docs/AI.md` > `ai_service` API contract. Do not change it without a note in `PROGRESS.md` > Decisions.

## Purpose
Runs on the host laptop (or VPS) next to the Godot server. Serves `/health`, `/classify`, `/director`, `/telemetry`. Must work fully **offline** at the venue.

## Stack
- Python 3.11+, FastAPI + uvicorn, PyTorch (or ONNX Runtime for inference), Pillow.
- Use a virtual env: `python -m venv .venv` then install from `requirements.txt` (pin versions).
- Model weights are not committed to git if over ~50 MB. Store in a shared drive, path in `PROGRESS.md`; download script in `scripts/`.

## Layout (to create)
```
ai_service/
  app/main.py          # FastAPI app, routes only
  app/classifier.py    # load model once at startup, predict(image) -> label, conf, top3
  app/director.py      # rule curve
  app/telemetry.py     # append JSON lines to data/telemetry.jsonl
  training/            # notebooks/scripts for the classifier; not imported by app/
  data/                # datasets and own photos (gitignored)
  tests/
  requirements.txt
```

## Run
`uvicorn app.main:app --host 0.0.0.0 --port 8000` (0.0.0.0 so phones on the hotspot can reach it).

## Rules
- Load the model once at startup; `/classify` must answer in under 1 s on CPU.
- Never crash on bad input: invalid image -> 400 with a message; model error -> `label: "other"`.
- Confidence under 0.5 -> `label: "other"`.
- Log each classify call to telemetry (label, confidence, ms) for the pitch numbers.

## Tests
- `pytest -q` for: `/classify` with a sample image returns a valid label; bad upload returns 400; director output stays within bounds for extreme inputs.
- Model quality: report accuracy on own held-out photos in `PROGRESS.md`, not just the public split.
