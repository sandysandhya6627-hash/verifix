# VeriFix — document-checking prototype

VeriFix accepts PDF and DOCX files, checks basic file structure, calculates a SHA-256 fingerprint, and displays a transparent report. Passing these checks does **not** prove a document is genuine or that its contents are true.

## Project structure

- `frontend/` — Next.js frontend
- `backend/` — FastAPI API
- `render.yaml` — deployment blueprint for the API on Render

## Run locally (Windows PowerShell)

### 1. Backend

```powershell
cd backend
py -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
uvicorn main:app --reload
```

API health check: http://127.0.0.1:8000/health
API docs: http://127.0.0.1:8000/docs

### 2. Frontend (open a second terminal)

```powershell
cd frontend
npm install
Copy-Item .env.example .env.local
npm run dev
```

Open http://localhost:3000. The frontend reads `NEXT_PUBLIC_API_URL` from `.env.local`. Restart the dev server after changing environment variables.

## Deploy the real app

GitHub stores the code; it does not run the Python API by itself. GitHub Pages is not suitable for this full-stack application.

### Backend on Render

1. Create a new Web Service from this GitHub repository.
2. Use the repository root; the included `render.yaml` configures `backend` as the service root. If configuring manually, set root directory `backend`, build command `pip install -r requirements.txt`, start command `uvicorn main:app --host 0.0.0.0 --port $PORT`, and health check `/health`.
3. Set `FRONTEND_ORIGIN` to the exact Vercel frontend origin after you deploy the frontend, for example `https://your-app.vercel.app` (no path and no trailing slash).
4. Confirm `https://YOUR-API/health` returns `{"status":"ok","service":"VeriFix API"}`.

### Frontend on Vercel

1. Import this GitHub repository into Vercel.
2. Set **Root Directory** to `frontend`.
3. Add environment variable `NEXT_PUBLIC_API_URL` with the backend base URL, e.g. `https://your-api.onrender.com` (no trailing slash).
4. Deploy. If you change the environment variable, redeploy.
5. Update the backend `FRONTEND_ORIGIN` to the Vercel URL and redeploy/restart the backend.

## Limitations and safety

- Accepts PDF and DOCX only; max upload size is 10 MB.
- Checks basic file signature/container structure and computes SHA-256 only.
- Files are processed in memory and are not saved permanently by this prototype.
- A matching fingerprint can help compare identical files; it does not identify an issuer or establish authenticity.
- Before public use, add rate limiting, stronger validation, malware scanning as appropriate, monitoring, and a clear privacy policy. Do not upload sensitive documents to a public demo.
