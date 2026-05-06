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
| `fps`     | source fps | Match the source — only lower to reduce file size |
| `width`   | source width | Original dimensions — only lower to reduce file size |

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

## Step 3: Detect source properties

Run ffprobe to get the source fps and dimensions so the GIF matches the
original video as closely as possible:

```bash
ffprobe -v quiet -select_streams v:0 \
  -show_entries stream=r_frame_rate,width,height \
  -of default=noprint_wrappers=1 "<input>"
```

- `r_frame_rate` returns a fraction like `30000/1001` (≈29.97 fps) — evaluate
  it (e.g. `python3 -c "print(30000/1001)"`) to get the decimal fps to pass
  to the `fps=` filter.
- Use the reported `width` as the `scale` value unless the user requested
  a different size.

If the user already specified fps or width, skip this step for that parameter.

## Step 4: Generate the palette (pass 1)

Save the palette to a temp file:

```bash
PALETTE=$(mktemp /tmp/palette_XXXXXX.png)

ffmpeg -y \
  [-ss <start>] \
  -i "<input>" \
  [-t <duration>] \
  -vf "fps=<fps>,scale=<width>:-1:flags=lanczos,palettegen=stats_mode=full:max_colors=256:reserve_transparent=0" \
  "$PALETTE"
```

**Why this matters:** `palettegen=stats_mode=full` samples *all* frames
equally to build the 256-color palette, so every part of the video —
including static areas — is well represented. This gives the best overall
color fidelity across the entire clip.

**Important:** place `-ss` *before* `-i` for fast seeking; place `-t` *after*
`-i` to keep the duration precise.

## Step 5: Encode the GIF (pass 2)

```bash
ffmpeg -y \
  [-ss <start>] \
  -i "<input>" \
  [-t <duration>] \
  -i "$PALETTE" \
  -lavfi "fps=<fps>,scale=<width>:-1:flags=lanczos [x]; [x][1:v] paletteuse=dither=sierra2_4a" \
  "<output>"

rm "$PALETTE"
```

**Why these settings:**
- `flags=lanczos` — highest-quality resampling filter, critical when scaling
- `stats_mode=full` — palette covers all frames equally, not just motion areas
- `max_colors=256` — use all 256 slots; the GIF format's hard ceiling
- `reserve_transparent=0` — uses all 256 palette slots for visible colors
  instead of reserving one for transparency, maximizing color fidelity
- `dither=sierra2_4a` — error-diffusion dithering that produces smoother
  gradients and fewer visible patterns than Bayer dithering; the best
  general-purpose dither mode for photographic and screen-recording content
- No `diff_mode=rectangle` — every frame is fully re-dithered independently,
  preserving maximum quality (omitting it trades some file size for better
  per-frame accuracy)

## Step 6: Report results

After the GIF is created, show the user:

```bash
ls -lh "<output>"
ffprobe -v quiet -show_entries format=duration -of default=noprint_wrappers=1 "<output>" 2>/dev/null || true
```

Report the file size and confirm the output path. If the file is very large
(> 50 MB), proactively suggest ways to reduce it while keeping the best
quality possible:
- Reduce `width` (e.g. half the source width)
- Lower `fps` (e.g. half the source fps — motion still looks smooth at 15 fps)
- Trim to a shorter duration

## Full example

User: "Make a gif of my screen recording from 0:12 to 0:18, make it 600px wide"

```bash
# Detect source fps first
ffprobe -v quiet -select_streams v:0 \
  -show_entries stream=r_frame_rate \
  -of default=noprint_wrappers=1 "screen_recording.mov"
# e.g. output: r_frame_rate=60/1  →  use fps=60

# Parameters: start=12, duration=6, width=600, fps=60
PALETTE=$(mktemp /tmp/palette_XXXXXX.png)

ffmpeg -y -ss 12 -i "screen_recording.mov" -t 6 \
  -vf "fps=60,scale=600:-1:flags=lanczos,palettegen=stats_mode=full:max_colors=256:reserve_transparent=0" \
  "$PALETTE"

ffmpeg -y -ss 12 -i "screen_recording.mov" -t 6 \
  -i "$PALETTE" \
  -lavfi "fps=60,scale=600:-1:flags=lanczos [x]; [x][1:v] paletteuse=dither=sierra2_4a" \
  "screen_recording.gif"

rm "$PALETTE"
ls -lh screen_recording.gif
```

## Troubleshooting

**"Invalid option" or filter errors:** Older ffmpeg versions may not support
`sierra2_4a`. Fall back to: `paletteuse=dither=bayer:bayer_scale=5`

**GIF looks choppy:** Verify the source video's frame rate isn't lower than
the target fps — don't set fps higher than the source. Use `ffprobe <input>`
to check.

**GIF file is very large:** Reduce width or fps; trim to a shorter clip.
Adding `diff_mode=rectangle` to `paletteuse` can shrink file size at a small
quality cost: `paletteuse=dither=sierra2_4a:diff_mode=rectangle`

**Colors look banded in motion areas:** Switch `stats_mode=full` to
`stats_mode=diff` in palettegen — this focuses the palette on pixels that
change between frames, which can improve motion fidelity at the cost of
static-area color accuracy.

**Seeking is off:** If `-ss` before `-i` skips too aggressively, move it
to after `-i`: `ffmpeg -i input.mp4 -ss 12 ...` (slower but frame-accurate).
