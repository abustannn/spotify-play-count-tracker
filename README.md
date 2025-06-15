import json
from spotipy import Spotify
from spotipy.oauth2 import SpotifyOAuth

CLIENT_ID = "769eb334464f4783bba7608dae7c3101"
CLIENT_SECRET = "0ab7958715f04cb39909c73594219359"
REDIRECT_URI = "http://localhost:8888/callback"
SCOPE = "user-read-recently-played"

sp = Spotify(auth_manager=SpotifyOAuth(
    client_id=CLIENT_ID,
    client_secret=CLIENT_SECRET,
    redirect_uri=REDIRECT_URI,
    scope=SCOPE
))

def load_counts(filename="play_counts.json"):
    try:
        with open(filename, "r") as f:
            return json.load(f)
    except FileNotFoundError:
        return {}

def save_counts(counts, filename="play_counts.json"):
    with open(filename, "w") as f:
        json.dump(counts, f, indent=2)

def update_counts():
    counts = load_counts()
    results = sp.current_user_recently_played(limit=50)

    for item in results['items']:
        track = item['track']
        name = track['name']
        artist = track['artists'][0]['name']
        key = f"{artist} - {name}"
        counts[key] = counts.get(key, 0) + 1

    save_counts(counts)
    print("✅ Play counts updated!")

if __name__ == "__main__":
    update_counts()
