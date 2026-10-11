# Mochi the Explorer 🩷

<p align="center">
    <img width="600" height="300" alt="mochi-banner" src="https://github.com/user-attachments/assets/f1786128-673d-44dd-95f1-e02d9f0aca06" />
</p>


[Mochi the Explorer](https://mochitheexplorer.onrender.com/) is a cute wee pixel mascot who helps you rediscover the city or town you already live in.

The inspiration came from two weeks spent trying to organise something new and fun with my friends in the city we've lived in for the past 10 years: 
none of us could come up with anything that felt even remotely exciting, and so off we went to our usual pub, defeated once again.

Sounds familiar? Cue Mochi to the rescue! All you have to do is tell Mochi where you live, what mood you're in, 
how much time you have and what time of day you're planning to head out and she'll plan the perfect outing for you! 
Mochi the Explorer will suggest between one and six stops (depending on how much time you have) 
and then encourage you to 'Go touch grass!' by showing you the location(s) she suggested on OpenStreeMap.

```python
DURATIONS = {
    "30min":         {"radius_m": 1500,  "stops": 1},
    "1-2hrs":        {"radius_m": 4000,  "stops": 1},
    "half_day":      {"radius_m": 10000, "stops": 2},
    "full_day":      {"radius_m": 20000, "stops": 4},
    "multiple_days": {"radius_m": 40000, "stops": 6},
}
```

You only need to spend a couple of minutes on screen before being ready to put your shoes on and leave for an adventure! 
The goal is to suggest spots that fit your mood, but that you are less likely to have heard of or been to. 
In fact, Mochi deliberately avoids big chains and the most obvious picks, 
favouring indie shops, cafes and unusual experiences instead (try the `unusual` tag if you don't believe me 😉). 
The results change between requests, so even locals who feel they've seen it all will find something they haven't tried yet.

Let Mochi find your next favourite spot in town! ✨

<p align="center">
    <img width="800" height="520" alt="mochi-screens" src="https://github.com/user-attachments/assets/4031cb4c-47a1-4d07-8e3d-6a03bb8a548d" />
</p>


## Demo

Mochi the Explorer is available here: https://mochitheexplorer.onrender.com/

You can also watch a wee demo here: https://www.youtube.com/watch?v=_fleqSGD_4k

_A wee heads up: the backend runs on Render’s free tier and sleeps when idle, 
so the first request can take about a minute to wake up. 
But don't despair! Mochi will keep you entertained while you wait:_
<p align="center"> 

<p align="center">
    <img alt="mochi-gif" width="350" height="175" src="https://github.com/user-attachments/assets/5e51de7f-dbc1-4b01-bb6c-590806bfd8d6" />
</p>

## How I built it

The open pieces making up Mochi the Explorer are: 

- [Gemma 4](https://ai.google.dev/gemma/docs/core/model_card_4), who picks and describes Mochi’s recommendations.
The deployed demo uses [gemma-4-26b-a4b-it](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/maas/google/gemma-4-26b-a4b-it) on [Google AI Studio’s free tier](https://ai.google.dev/gemini-api/docs/pricing). While developing the project locally, I ran [gemma4:e2b](https://ollama.com/library/gemma4:e2b) through [Ollama](https://ai.google.dev/gemma/docs/integrations/ollama?utm_source=deepmind.google&utm_medium=referral&utm_campaign=gdm&utm_content) with the same code.

- The geographic data is from [OpenStreetMap](https://www.openstreetmap.org/). In particular, [Nominatim](https://nominatim.org/) 
turns a place name into coordinates, while [Overpass](https://wiki.openstreetmap.org/wiki/Overpass_API) finds real places (parks, galleries, cafes, bookshops...) around them.

- For the frontend, I used React + Vite, [NES.css](https://nostalgic-css.github.io/NES.css/) for the pixel graphics, 
and [Leaflet](https://leafletjs.com/) for map visualisation. 

- I used [Gumdrop Studio](https://docs.gumdropstudio.app/) to design the sprite, Mochi.

- Mochi is deployed on Render's free tier.

<p align="center">
    <img src="./frontend/frontend/public/mochi-happy.png"
</p>

## How it works

I designed a fixed pipeline rather than a free-roaming agent. 
This is primarily because (small) AI models are not always reliable at deciding which tools to call, so my Python code runs the steps in a fixed order:
(1) find the geocode, (2) find relevant places, (3) ask Gemma to **choose** the most suitable spot(s), **explain** why it's a good choice, 
and give fun and cheerful tips to the user.


### Making a small model trustworthy

- **Gemma never writes place names.** Each candidate gets an ID (`P1`, `P2`, ...). Gemma answers with IDs, 
and the code looks up the real name and coordinates, so it cannot invent a place.
- **Picks are checked in code.** Duplicates and invented IDs are dropped, each stop in a multi-stop trip comes from a different OpenStreetMap tag, 
and missing stops are topped up from the candidate list.
- **Mochi's closing message is a template**, filled with the real first stop's name. 
The model kept making false claims in free-written messages (wrong counts, wrong place types).
- **If Gemma fails twice,** the app falls back to plain top places instead of showing an error.

### Making results less obvious

OpenStreetMap doesn't record popularity, so places are ranked in tiers: independent places first, well-documented landmarks next, chains last. 
Within a tier the choice is random. Each request queries different tags, and the search radius widens only when too little is found.

## Tech stack

| Part | Tool |
|---|---|
| Model | **Gemma 4** (open weights): `gemma4:e2b` locally via **Ollama**, `gemma-4-26b-a4b-it` hosted on Google AI Studio |
| Map data | **OpenStreetMap**: Nominatim (geocoding) and Overpass (places) |
| Backend | Python, FastAPI, httpx |
| Frontend | React, Vite, NES.css, Leaflet / react-leaflet |
| Hosting | Render (free web service + static site) |

## Project structure

```
backend/
  main.py          FastAPI app and the /api/recommend endpoint
  geo.py           Nominatim + Overpass, ranking and randomisation
  llm.py           Prompt, calls to Gemma, validation, message templates
  moods.py         Mood → OpenStreetMap tags, duration → radius and stop count
  requirements.txt
frontend/
  src/             App.jsx, Mascot.jsx, ResultMap.jsx, useTypewriter.js
  public/          Mochi's sprites
```


<p align="center">
    <img src="./frontend/frontend/public/mochi-idle.png"
</p>

## Run it locally

You need Python 3.10+, Node 20+, and either [Ollama](https://ollama.com) or a Google AI Studio API key.

### 1. Backend

```bash
cd backend
python -m venv venv
# Windows (PowerShell):
venv\Scripts\Activate.ps1
# Mac/Linux:
source venv/bin/activate

pip install -r requirements.txt
```

Create `backend/.env` using **one** of these model setups.

**Option A: local Gemma with Ollama** (nothing leaves your machine):

```
LLM_API_STYLE=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=gemma4:e2b
LLM_API_KEY=ollama
LLM_TIMEOUT=120
```

First run `ollama pull gemma4:e2b` and make sure Ollama is running.

**Option B: Google AI Studio's hosted Gemma:**

```
LLM_API_STYLE=gemini
LLM_MODEL=gemma-4-26b-a4b-it
LLM_API_KEY=your_google_ai_studio_key
LLM_TIMEOUT=60
```

Then start the server:

```bash
uvicorn main:app --reload
```

Check `http://localhost:8000/api/health` returns `{"status":"ok"}`. Interactive API docs are at `http://localhost:8000/docs`.

Before running, open `backend/geo.py` and put your own email in the `User-Agent` header. 
[Nominatim's usage policy](https://operations.osmfoundation.org/policies/nominatim/) requires an identifying User-Agent and a maximum of one request per second.

### 2. Frontend

```bash
cd frontend
npm install
npm run dev
```

Open `http://localhost:5173`. The frontend expects the backend at `http://localhost:8000`.

## Configuration

Backend environment variables:

| Variable | Purpose |
|---|---|
| `LLM_API_STYLE` | `gemini` (Google native API), `ollama` (Ollama native API), or leave unset for any OpenAI-compatible server |
| `LLM_BASE_URL` | Model server address (not used with `gemini`) |
| `LLM_MODEL` | Model name |
| `LLM_API_KEY` | API key (any value for Ollama) |
| `LLM_TIMEOUT` | Seconds to wait for the model |
| `OVERPASS_PRIVATE_URL` | Optional private Overpass endpoint, tried before the public mirrors |
| `ALLOWED_ORIGINS` | Comma-separated frontend addresses allowed by CORS (default `http://localhost:5173`) |

Frontend environment variable (baked in at build time):

| Variable | Purpose |
|---|---|
| `VITE_API_URL` | Backend address (with no trailing slash!) |

    
## API

`POST /api/recommend`

```json
{
  "location": "Edinburgh",
  "mood": "cosy",
  "duration": "30min",
  "time_of_day": "afternoon"
}
```

- `mood`: `cosy`, `outdoors`, `adventurous`, `creative`, `social`, `food`, `history`, `thrifting`, `unusual`, `surprise`
- `duration`: `30min`, `1-2hrs`, `half_day`, `full_day`, `multiple_days`
- `time_of_day`: `morning`, `afternoon`, `evening`, `night`, `anytime`

The response contains a `message`, a list of `stops` (name, category, coordinates, distance, why, tip), the `mood_used`, and the map `center`.

## Deploying on Render

Two free services, both from this repo:

| Service | Type | Root directory | Build | Start / publish |
|---|---|---|---|---|
| Backend | Web Service (Python) | `backend` | `pip install -r requirements.txt` | `uvicorn main:app --host 0.0.0.0 --port $PORT` |
| Frontend | Static Site | `frontend` | `npm install && npm run build` | publish directory `dist` |

1. Deploy the backend with its environment variables and check `/api/health`.
2. Deploy the frontend with `VITE_API_URL` set to the backend's URL **before** the first build.
3. Set `ALLOWED_ORIGINS` on the backend to the frontend's URL (with `https://`, no trailing slash).

## Limitations

- **Public Overpass servers can be slow or rate-limited.** The app retries, tries mirrors and caches results in memory, and it supports a private endpoint.
- **Small towns can have thin data.** If too few places are found, the search widens its radius, and Mochi says so if it finds nothing.
- **Caches live in memory** and reset when the server restarts or sleeps. There is no database, no login and no stored user data.
- **The hosted demo sends the city, mood and time of day to Google** for the Gemma call. For full privacy, run it locally with Ollama.
- **Recommendations are only as good as OpenStreetMap's tags.** Mis-tagged places occasionally slip through.

<p align="center">
    <img src="./frontend/frontend/public/mochi-talking.png"
</p>
    
## Credits

- Map data © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors.
- Model: [Gemma](https://ai.google.dev/gemma) by Google.
- Pixel UI: [NES.css](https://nostalgic-css.github.io/NES.css/) and the [Press Start 2P](fonthttps://fonts.google.com/specimen/Press+Start+2P).
- Pixel sprite designed with [Gumdrop Studio](gumdrop Studio | Gumdrop Studio).


## License

The project is released under the [MIT License](LICENSE).

## Author

Mochi the Explorer was built by Paola Ruffo (cobaltspiral) for the [Hacktoberfest Open-Source AI Challenge - Week 1: Touch Grass](https://dev.to/challenges/hacktoberfest-week1-2026-10-05)
[**Hackathon Write-up**](https://dev.to/cobaltspiral/want-to-touch-grass-but-run-out-of-options-let-mochi-the-explorer-guide-you-to-your-next-favourite-243g)
