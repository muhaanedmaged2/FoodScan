# ScanFood API 🍽️

Estimate calories from a food photo using **YOLO segmentation + MiDaS depth**.

## Project structure

```
scan_food/
├── inference.py       ← core pipeline (YOLO + MiDaS + calorie math)
├── main.py            ← FastAPI app
├── requirements.txt
├── Dockerfile
└── weights/
    └── best.pt        ← your trained YOLO weights (NOT committed to git)
```

## Run locally (without Docker)

```bash
pip install -r requirements.txt
YOLO_WEIGHTS=weights/best.pt uvicorn main:app --reload
```

Open http://localhost:8000/docs for the interactive Swagger UI.

## Run with Docker

```bash
# Build
docker build -t scanfood .

# Run  (mount your weights file)
docker run -p 8000:8000 \
  -v $(pwd)/weights/best.pt:/app/weights/best.pt \
  scanfood
```

## API

### POST /predict

Send a multipart form with one field `file` containing the image.

```bash
curl -X POST http://localhost:8000/predict \
  -F "file=@photo.jpg" | python -m json.tool
```

Response:
```json
{
  "items": [
    {
      "name": "rice",
      "confidence": 0.96,
      "area_cm2": 132.22,
      "depth_cm": 3.5,
      "volume_cm3": 462.77,
      "weight_g": 370.2,
      "calories": 481.3
    }
  ],
  "total_calories": 481.3,
  "annotated_image": "<base64 JPEG string>",
  "error": null
}
```

### GET /health

Returns `{"status": "ok"}` — use this for server health checks.

## Notes

- Place a **Egyptian 1-pound coin** (⌀ 2.3 cm) next to the food in every photo.
- The coin is used as a size reference to convert pixels → cm.
