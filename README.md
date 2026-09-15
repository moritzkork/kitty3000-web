# Kitty3000 – web front end (standalone HTML/JS)

The real design (Ceven's branding), not Streamlit. One `index.html` + assets, calls the
Cloud Run API directly with `fetch`. Splash → one click → the tool. Same features as the
Streamlit app: Samples (with demo-mode masking), Upload, Record, waveform, snippet range,
mascot model buttons, Analyse, result with the "not a cat?" soft warning.

## Three things to do

### 1. Add the four samples
Copy them from the Streamlit repo (same files, same names):

    cp ~/code/Kitty3000/samples/cat_complaining.mp3 \
       ~/code/Kitty3000/samples/cat_meow.mp3 \
       ~/code/Kitty3000/samples/human_imitating_a_cat.mp3 \
       ~/code/Kitty3000/samples/two_cats_fighting.mp3 \
       samples/

### 2. Enable CORS on the API (REQUIRED)
A browser page on another origin can't call the API until it sends CORS headers.
In the ML repo, `src/kitty3000_ml/api.py`, right after `app = FastAPI(...)`:

    from fastapi.middleware.cors import CORSMiddleware
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"],        # a public read-only inference API; fine for a demo
        allow_methods=["*"],
        allow_headers=["*"],
    )

Then redeploy the API (this is a backend change — do it before Wednesday's freeze):

    cd ~/code/Kitty3000-ML && bash deploy.sh

Without this, the page loads but every /models and /predict call fails in the browser.

### 3. Host it
Easiest: a new GitHub repo + GitHub Pages.

    cd ~/Downloads/kitty3000-web
    git init && git add -A && git commit -m "Kitty3000 web front end"
    gh repo create kitty3000-web --public --source=. --push
    # then on github.com: Settings -> Pages -> Source: main / root

Your URL: `https://moritzkork.github.io/kitty3000-web/`

Or run locally for a quick look (mic + fetch work on localhost, not on file://):

    python3 -m http.server 8000   # then open http://localhost:8000

## Config
The API URL is the one constant at the top of the `<script>` in index.html:
`https://kitty3000-api-1097344477099.europe-west1.run.app`

## Not included yet
- The Analysis page (spectrogram / compare-all / explainer) is still the Streamlit one.
  Keep Streamlit for that, or port it here later — the demo's main tool is this page.
- Record encodes the mic to WAV in the browser so the API can read it (libsndfile can't do webm).
