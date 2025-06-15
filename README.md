import json
import time
from collections import defaultdict
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

def show_top_artists_and_tracks(filename="play_counts.json", top_n=10):
    try:
        with open(filename, "r") as f:
            play_data = json.load(f)
    except FileNotFoundError:
        print("⚠️ play_counts.json not found. Run the script to generate it.")
        return

    artist_counts = defaultdict(int)
    track_counts = []

    for full_title, count in play_data.items():
        if " - " in full_title:
            artist, song = full_title.split(" - ", 1)
            artist_counts[artist] += count
            track_counts.append((full_title, count))

    sorted_artists = sorted(artist_counts.items(), key=lambda x: x[1], reverse=True)
    print(f"\nTop {top_n} Artists by Play Count:\n")
    for i, (artist, count) in enumerate(sorted_artists[:top_n], 1):
        print(f"{i}. {artist} — {count} plays")

    sorted_tracks = sorted(track_counts, key=lambda x: x[1], reverse=True)
    print(f"\nTop {top_n} Songs by Play Count:\n")
    for i, (track, count) in enumerate(sorted_tracks[:top_n], 1):
        print(f"{i}. {track} — {count} plays")

if __name__ == "__main__":
    update_counts()
    show_top_artists_and_tracks(top_n=10)
