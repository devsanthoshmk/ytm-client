# YouTube Music client: YTM web client and browser music player

A web client and local backend that searches and streams music from YouTube Music.

This application is a YouTube Music client and browser music player. The YTM web client extracts audio streams directly from YouTube Music and plays them in a web browser without video playback.

## Architecture

The project contains three main components: a web frontend, a REST API server, and a media extraction layer.

- Frontend: A Vue 2 single-page application located in `index.html` and `assets/js/script.js`. It uses Bootstrap 5 for layout, FontAwesome for icons, and the browser Audio API for playback. The interface displays song metadata, album covers, playback duration, and current playback time.
- Backend: A Flask web application in `app.py`. It serves static files from `assets/` and exposes two endpoints:
  - `GET /song_results`: Accepts an `input` query parameter with the search term. It calls `YoutubeMusicAPI.py` to query `https://music.youtube.com/search`, parses video IDs with regular expressions, fetches track details from the YouTube oEmbed API, and returns up to 5 results as JSON.
  - `GET /playable_link`: Accepts a `song_url` query parameter with the YouTube watch URL. It calls `ytdlp.py` to extract direct audio stream links.
- Media extractor: `ytdlp.py` uses `yt-dlp` with `download=False` to inspect available streams. It selects audio formats with `webm` or `m4a` extensions that do not contain video tracks, and returns the direct media stream URL.
- Process launcher: `start_gunicorn.py` detects the local network IP address, displays a QR code in the terminal, and launches Gunicorn on port 8000.

## Setup

Install Python 3.10 or newer.

Clone the repository:

```bash
git clone https://github.com/devsanthoshmk/ytm-client.git
cd ytm-client
```

Install the required Python packages:

```bash
pip install flask httpx yt-dlp gunicorn qrcode
```

## Usage

Start the development server with Flask:

```bash
python3 app.py
```

The Flask server listens on port 5005. Open `http://localhost:5005` in a browser.

Start the production server with Gunicorn:

```bash
python3 start_gunicorn.py
```

`start_gunicorn.py` prints an ASCII QR code and the local network URL to the terminal, then starts Gunicorn on `0.0.0.0:8000`. You can scan the QR code with a mobile device on the same local network to access the web player.

To search and play music:

1. Type a song name into the search box and press Enter.
2. The search results show up to 5 matching tracks.
3. Click a track title to fetch the audio stream and start playback.
4. Use the controls to play, pause, seek, jump to the next track, or jump to the previous track.

## Configuration

- Search limit: Edit line 21 in `app.py`. The function call `yt.search(song_name, 5)` specifies how many tracks the search returns.
- Flask port: Edit line 31 in `app.py`. The default configuration runs on port 5005 with `debug=True`.
- Gunicorn port: Edit lines 18 and 25 in `start_gunicorn.py`. The default configuration runs on port 8000.
- Audio formats: Edit lines 17 to 22 in `ytdlp.py`. The filter checks for `webm` and `m4a` audio containers.
- TLS certificates: The repository provides `cert.pem` and `key.pem` for HTTPS bindings.
