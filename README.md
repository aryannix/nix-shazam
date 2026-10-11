# nix-shazam

Find out what song is playing from an audio or video file. Also search songs and get full track details, powered by Shazam.

```js
const { recognize } = require("nix-shazam");

const result = await recognize("./clip.mp3");
console.log(result.track.title, "-", result.track.artist);
```

## Install

```bash
npm install nix-shazam
```

Requires Node.js 18 or newer.

## Quick start

CommonJS:

```js
const { recognize } = require("nix-shazam");
```

ES Modules:

```js
import { recognize } from "nix-shazam";
```

Then:

```js
const result = await recognize("./clip.mp3");

if (result.success) {
  console.log("Title :", result.track.title);
  console.log("Artist:", result.track.artist);
  console.log("Cover :", result.track.cover);
} else {
  console.log(result.message);
}
```

## recognize(input, options?)

Identifies a song from a short piece of audio or video.

### What you can pass as `input`

```js
await recognize("./song.mp3");                 // file path
await recognize("https://example.com/a.mp4");  // direct link
await recognize(fs.readFileSync("./song.mp3")); // Buffer
await recognize(uint8Array);                   // Uint8Array
await recognize(arrayBuffer);                  // ArrayBuffer
```

Audio files and video files both work.

### What you get back

When a song is found:

```json
{
  "success": true,
  "track": {
    "id": "546891609",
    "title": "Blinding Lights",
    "artist": "The Weeknd",
    "album": "After Hours",
    "label": "Republic Records",
    "release_year": "2020",
    "genres": "Pop",
    "cover": "https://is1-ssl.mzstatic.com/image/.../400x400cc.jpg",
    "share_url": "https://www.shazam.com/track/546891609/blinding-lights",
    "apple_music_url": "https://music.apple.com/us/album/...",
    "spotify_url": "https://open.spotify.com/search/Blinding%20Lights%20The%20Weeknd",
    "has_lyrics": true,
    "lyrics": "..."
  },
  "matches": [
    { "id": "546891609", "offset": 0.012, "timeskew": 0.0001, "frequencyskew": 0.0002 }
  ],
  "raw": { "...": "full response from Shazam" }
}
```

When no song is found:

```json
{
  "success": false,
  "message": "No matching song found in Shazam database",
  "matches": []
}
```

The values above are examples. Your result will have the real song data.

### The `track` fields

| Field | Type | What it is |
| --- | --- | --- |
| `id` | string | Shazam track id (use it with `getTrack`) |
| `title` | string | Song name |
| `artist` | string | Artist name |
| `album` | string or null | Album name |
| `label` | string or null | Record label |
| `release_year` | string or null | Release year |
| `genres` | string or null | Main genre |
| `cover` | string or null | Cover image link |
| `share_url` | string or null | Shazam page for the song |
| `apple_music_url` | string or null | Apple Music link |
| `spotify_url` | string or null | Spotify link |
| `has_lyrics` | boolean | `true` if Shazam has lyrics for the song |
| `lyrics` | string or null | Lyrics text, if available |

Some fields can be `null` when Shazam does not have that information, so check before using them.

### Options

All options are optional.

| Option | Default | What it does |
| --- | --- | --- |
| `offsets` | `[0, 12, 30]` | Seconds in the file where listening starts. The next position is tried when the song is not found |
| `offset` | none | Listen from one position only, for example `offset: 20` |
| `seconds` | `12` | How many seconds of audio to listen to each time |
| `country` | `"US"` | Shazam country code |
| `language` | `"en-US"` | Shazam language |
| `timezone` | `"UTC"` | Timezone sent to Shazam |

```js
const result = await recognize("./long-video.mp4", {
  offset: 40,
  seconds: 10,
  country: "BD",
  language: "en-US"
});
```

Tip: if the song starts late in your file, pass `offset` or add more values in `offsets`.

## search(query, options?)

Search songs by name.

```js
const { search } = require("nix-shazam");

const songs = await search("blinding lights", { limit: 3 });
console.log(songs[0].title, "-", songs[0].artist);
```

Returns an array (empty array if nothing is found):

```json
[
  {
    "shazam_id": "546891609",
    "apple_id": 1499378108,
    "title": "Blinding Lights",
    "artist": "The Weeknd",
    "album": "After Hours",
    "release_date": "2020-03-20",
    "duration": 200040,
    "duration_formatted": "3:20",
    "genre": "R&B/Soul",
    "preview_url": "https://audio-ssl.itunes.apple.com/.../preview.m4a",
    "cover": "https://is1-ssl.mzstatic.com/image/.../800x800bb.jpg",
    "apple_music_url": "https://music.apple.com/us/album/..."
  }
]
```

| Option | Default | What it does |
| --- | --- | --- |
| `limit` | `5` | Maximum number of results |
| `country` | `"US"` | Store country |
| `language` | `"en-US"` | Language |

`duration` is in milliseconds. `duration_formatted` is the same as `m:ss`.

## getTrack(trackId, options?)

Get full details of a song using its Shazam id (from `recognize` or `search`).

```js
const { getTrack } = require("nix-shazam");

const track = await getTrack("546891609");
console.log(track.title, "-", track.artist);
```

Returns the same fields as `track` above, plus `success` and `raw`:

```json
{
  "success": true,
  "id": "546891609",
  "title": "Blinding Lights",
  "artist": "The Weeknd",
  "album": "After Hours",
  "label": "Republic Records",
  "release_year": "2020",
  "genres": "Pop",
  "cover": "https://is1-ssl.mzstatic.com/image/.../400x400cc.jpg",
  "share_url": "https://www.shazam.com/track/546891609/blinding-lights",
  "apple_music_url": "https://music.apple.com/us/album/...",
  "spotify_url": "https://open.spotify.com/search/Blinding%20Lights%20The%20Weeknd",
  "has_lyrics": true,
  "lyrics": "...",
  "raw": { "...": "full response from Shazam" }
}
```

## Handling errors

A song that is not found is **not** an error. You get `success: false`.

Real problems (bad input, network error, Shazam not reachable) throw an error, so wrap your call in `try / catch`:

```js
try {
  const result = await recognize("./clip.mp3");

  if (!result.success) {
    console.log("Song not found");
    return;
  }

  console.log(result.track.title);
} catch (err) {
  console.error("Failed:", err.message);
}
```

## Example: use with a bot

Download the audio or video to a Buffer, then pass it in:

```js
const { recognize } = require("nix-shazam");

async function whatSong(buffer) {
  const result = await recognize(buffer);
  if (!result.success) return "Could not identify the song.";

  const t = result.track;
  return `${t.title} - ${t.artist}\n${t.album || ""}\n${t.share_url || ""}`;
}
```

## License

MIT
