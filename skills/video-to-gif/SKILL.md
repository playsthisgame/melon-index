---
name: video-to-gif
description: >
  Converts video files into high-quality GIFs using ffmpeg's two-pass palette
  generation technique. Use this skill whenever the user wants to turn a video
  (mp4, mov, mkv, webm, avi, etc.) into a GIF — whether they say "make a gif
  from this video", "convert my clip to gif", "extract this scene as a gif",
  "create an animated gif from my screen recording", or anything similar. Also
  triggers when the user wants to clip/trim a video and save it as an animated
  image. Always prefer this skill over ad-hoc ffmpeg commands whenever video-to-gif
  conversion is requested.
compatibility:
  tools:
    - Bash
  dependencies:
    - ffmpeg
---

# video-to-gif

Generates high-quality GIFs from video files using ffmpeg. The key to a
great GIF is the **two-pass palette technique**: first extract an optimal
256-color palette from the video, then use that palette to encode the GIF.
This avoids the muddy, banded look you get from naive conversion.

## Step 1: Gather parameters

Before running anything, collect these from the user (or infer from context):

| Parameter | Default | Notes |
|-----------|---------|-------|
| `input`   | (required) | Path to the source video |
| `output`  | `<input-basename>.gif` | Where to save the GIF |
| `start`   | beginning of video | e.g. `0:05` or `5` (seconds) |
| `duration` | full video | How many seconds to capture after `start` |
| `fps`     | `24` | Frames per second — 24 gives smooth motion; drop to 15 only to reduce file size |
| `width`   | `640` | Output width in pixels; height scales automatically |

If the user provides a start + end time instead of duration, compute
`duration = end - start` before building the command.

If the video is short (< 10 s) or the user says "the whole thing", skip
`-ss` and `-t`.

Confirm the parameters with the user if anything is ambiguous before running.

## Step 2: Check ffmpeg is available

```bash
which ffmpeg || echo "ffmpeg not found"
```

If ffmpeg is missing, tell the user to install it:
- macOS: `brew install ffmpeg`
- Ubuntu/Debian: `sudo apt install ffmpeg`
- Windows: download from https://ffmpeg.org/download.html

## Step 3: Generate the palette (pass 1)

Save the palette to a temp file:

```bash
PALETTE=$(mktemp /tmp/palette_XXXXXX.png)

ffmpeg -y \
  [-ss <start>] \
  -i "<input>" \
  [-t <duration>] \
  -vf "fps=<fps>,scale=<width>:-1:flags=lanczos,palettegen=stats_mode=diff:reserve_transparent=0" \
  "$PALETTE"
```

**Why this matters:** `palettegen=stats_mode=diff` builds the palette from
the *differences* between frames (motion areas), so the 256 colors are
spent where they matter most rather than on static background pixels.

**Important:** place `-ss` *before* `-i` for fast seeking; place `-t` *after*
`-i` to keep the duration precise.

## Step 4: Encode the GIF (pass 2)

```bash
ffmpeg -y \
  [-ss <start>] \
  -i "<input>" \
  [-t <duration>] \
  -i "$PALETTE" \
  -lavfi "fps=<fps>,scale=<width>:-1:flags=lanczos [x]; [x][1:v] paletteuse=dither=sierra2_4a:diff_mode=rectangle" \
  "<output>"

rm "$PALETTE"
```

**Why these settings:**
- `flags=lanczos` — high-quality downscaling filter
- `reserve_transparent=0` — uses all 256 palette slots for visible colors
  instead of reserving one for transparency, maximizing color fidelity
- `dither=sierra2_4a` — error-diffusion dithering that produces smoother
  gradients and fewer visible patterns than Bayer dithering; the best
  general-purpose dither mode for photographic and screen-recording content
- `diff_mode=rectangle` — only re-dithers pixels that changed between frames,
  significantly shrinking file size for content with static regions

## Step 5: Report results

After the GIF is created, show the user:

```bash
ls -lh "<output>"
ffprobe -v quiet -show_entries format=duration -of default=noprint_wrappers=1 "<output>" 2>/dev/null || true
```

Report the file size and confirm the output path. If the file is very large
(> 10 MB), proactively suggest ways to reduce it:
- Lower `fps` (e.g. 10 instead of 15)
- Reduce `width` (e.g. 320 instead of 480)
- Trim to a shorter duration

## Full example

User: "Make a gif of my screen recording from 0:12 to 0:18, make it 600px wide"

```bash
# Parameters: start=12, duration=6, width=600, fps=24
PALETTE=$(mktemp /tmp/palette_XXXXXX.png)

ffmpeg -y -ss 12 -i "screen_recording.mov" -t 6 \
  -vf "fps=24,scale=600:-1:flags=lanczos,palettegen=stats_mode=diff:reserve_transparent=0" \
  "$PALETTE"

ffmpeg -y -ss 12 -i "screen_recording.mov" -t 6 \
  -i "$PALETTE" \
  -lavfi "fps=24,scale=600:-1:flags=lanczos [x]; [x][1:v] paletteuse=dither=sierra2_4a:diff_mode=rectangle" \
  "screen_recording.gif"

rm "$PALETTE"
ls -lh screen_recording.gif
```

## Troubleshooting

**"Invalid option" or filter errors:** Older ffmpeg versions may not support
`diff_mode=rectangle` or `sierra2_4a`. Fall back to: `paletteuse=dither=bayer:bayer_scale=5`

**GIF looks choppy:** Verify the source video's frame rate isn't lower than the
target fps — don't go higher than the source. Use `ffprobe <input>` to check.

**GIF file is huge:** Lower fps or width; trim to a shorter clip.

**Colors look washed out:** Try `stats_mode=full` in palettegen (uses all
frames equally) — better for videos where the background changes a lot.

**Seeking is off:** If `-ss` before `-i` skips too aggressively, move it
to after `-i`: `ffmpeg -i input.mp4 -ss 12 ...` (slower but frame-accurate).
