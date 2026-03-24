---
name: content-producer
description: >
  Create cinematic MP4 videos using Python (Pillow) for frame-by-frame rendering piped to ffmpeg.
  Supports multiple content types: tech demos, explainers, promos, tutorials, and showcases — each
  with its own narrative structure and slide composition. Fully generic and brandable — supports
  custom color palettes, fonts, and icon sets, or works with no branding at all. Use this skill
  when the user wants to create a video, demo video, product video, explainer video, animated
  presentation, promo, tutorial, or motion graphics piece. Also trigger when the user mentions
  "demo video", "product video", "explainer video", "animated video", "MP4 presentation",
  "promo video", "event video", "tutorial video", or asks to present content as a video.
---

# Content Producer

Generate cinematic, animated MP4 videos using Python frame-by-frame rendering with Pillow and
ffmpeg encoding. Choose a content type template to match your topic, then generate.

## Before You Begin

1. **Read the shared references:**
   - `references/design-system.md` — color palettes, typography, spacing rules
   - `references/story-arcs.md` — narrative structures and slide design principles
   - `references/icons.md` — open-source icon sets and setup instructions
   - `references/media-assets.md` — how to embed user-provided images and video clips
   - `references/music.md` — dynamic music generation system
   - `references/branding-guide.md` — how to set up branded video production

2. **Check for a brand config**: If a `brand/brand.md` file exists, read it and apply
   the brand colors, fonts, logo, icons, and design rules to the generated video.

4. **Check for user-provided assets**: If the user provides images, screenshots, or screen
   recordings (in an `assets/` directory or as file paths), examine them before generating:
   - **Images**: Read them directly (Claude is multimodal)
   - **Video clips**: Extract sample frames via ffmpeg, read those to understand the content
   - Write narration that describes or references the visual content
   - Auto-map each asset to the most appropriate slide
   - See `references/media-assets.md` for the full embedding system

5. **Ask the user** (if not already clear from their request):
   - **What content type?** (or infer from their request)
   - **What style/brand?** Custom colors and fonts, or use the generic dark theme (skip if `brand/brand.md` exists)

6. **Read the content type template** based on their choice:

| Content Type | Template | Best For |
|-------------|----------|----------|
| **Tech Demo** | `templates/demo.md` | Product demos, feature walkthroughs, tool introductions |
| **Explainer** | `templates/explainer.md` | Concepts, architectures, "what is X?" topics |
| **Promo** | `templates/promo.md` | Events, product launches, announcements, teasers |
| **Tutorial** | `templates/tutorial.md` | How-to guides, step-by-step instructions, workshops |
| **Showcase** | `templates/showcase.md` | Portfolios, highlight reels, retrospectives, recaps |

Each template defines the recommended story arc, slide plan, slide-specific design guidance,
and music style — all producing MP4 video output.

## What This Skill Produces

A Python script (e.g., `make_video.py`) that generates an MP4 video file:
- 1920x1080 resolution at 30fps
- Rendered frame-by-frame using Pillow (PIL), piped as raw RGB bytes to ffmpeg
- Encoded with `libopenh264` codec, `yuv420p` pixel format
- Animated slides with eased transitions, kinetic typography, and icon reveals
- Crossfade transitions between slides
- Optional synthetic background music with 7 mood presets
- Optional TTS narration via Kokoro (auto-adjusting slide durations)
- Optional embedding of user-provided images and screen recordings
- Configurable color palette, fonts, and accent colors

## Prerequisites

- **Python 3** with `Pillow` installed
  - Fedora: `sudo dnf install python3-pillow`
  - Ubuntu: `sudo apt install python3-pil`
- **ffmpeg** installed on the system
  - Fedora: `sudo dnf install ffmpeg-free`
  - Ubuntu: `sudo apt install ffmpeg`
- **numpy** (optional, for background music)
  - Fedora: `sudo dnf install python3-numpy`
  - Ubuntu: `sudo apt install python3-numpy`
- **Fonts**: At least one font stack (see `references/design-system.md`)
  - Fedora: `sudo dnf install inter-fonts jetbrains-mono-fonts`
  - Ubuntu: `sudo apt install fonts-inter fonts-jetbrains-mono`
- **Icons** (optional): Clone one of the recommended icon sets (see `references/icons.md`)

## Rendering Pipeline

```
Python (Pillow)                    ffmpeg
+--------------+    raw RGB     +----------+
| Render frame | --stdin pipe-->| Encode   |--> output.mp4
|  as PIL Image|    bytes       | H.264    |
+--------------+                +----------+
```

Each slide is a Python function: `slide_name(frame, total_frames) -> PIL.Image`

- `frame`: current frame number within this slide (0 to total_frames-1)
- `total_frames`: total frames for this slide (duration_seconds * FPS)
- Returns a 1920x1080 RGB `Image` for that frame.

### ffmpeg Command

```python
cmd = [
    "ffmpeg", "-y", "-loglevel", "warning",
    "-f", "rawvideo",
    "-vcodec", "rawvideo",
    "-s", "1920x1080",
    "-pix_fmt", "rgb24",
    "-r", "30",
    "-i", "-",
    "-c:v", "libopenh264",
    "-pix_fmt", "yuv420p",
    OUTPUT,
]
proc = subprocess.Popen(cmd, stdin=subprocess.PIPE)
# For each frame:
proc.stdin.write(img.tobytes())
```

**Note**: Use `libopenh264` — `libx264` may not be available on all systems.

## Design System

See `references/design-system.md` for the complete palette. Key highlights:

### Color Palette — Default Dark Theme

```python
# Backgrounds
BLACK    = (0, 0, 0)         # bg-primary
GRAY_90  = (24, 24, 27)      # bg-secondary, card backgrounds
GRAY_80  = (39, 39, 42)      # bg-surface
GRAY_70  = (63, 63, 70)      # borders, muted elements

# Text
GRAY_40  = (161, 161, 170)   # muted text, labels
GRAY_30  = (212, 212, 216)   # secondary text
WHITE    = (255, 255, 255)   # primary text

# Accents — configurable per project
ACCENT       = (59, 130, 246)   # Primary accent (blue by default)
ACCENT_LIGHT = (96, 165, 250)   # Lighter accent
ACCENT_DARK  = (29, 78, 216)    # Darker accent

# Semantic colors
TEAL     = (20, 184, 166)    # Solution, positive
GREEN    = (34, 197, 94)     # Success, approved
ORANGE   = (249, 115, 22)    # Warnings, caution
RED      = (239, 68, 68)     # Error, danger
PURPLE   = (139, 92, 246)    # Expressive accent
```

Six accent presets: Blue (default), Cyan, Violet, Amber, Emerald, Rose.

**Color rules:**
- ACCENT on every slide (accent bar, glow, progress bar, or text highlight)
- TEAL for "solution" / "how it works" content
- GREEN for success/approved states
- ORANGE for warnings and caution
- RED for errors and critical states
- Background: BLACK or GRAY_90
- Text hierarchy: WHITE > GRAY_30 > GRAY_40

### Typography

```python
def font(name, size):
    paths = {
        "bold":    "/usr/share/fonts/inter/Inter-Bold.otf",
        "medium":  "/usr/share/fonts/inter/Inter-Medium.otf",
        "regular": "/usr/share/fonts/inter/Inter-Regular.otf",
        "mono":    "/usr/share/fonts/jetbrains-mono/JetBrainsMono-Regular.ttf",
    }
    return ImageFont.truetype(paths[name], size)
```

See `references/design-system.md` for alternative font stacks (Source Sans, Noto, Outfit).

| Element | Font | Size |
|---------|------|------|
| Main title (hero) | bold | 96px |
| Slide headline | bold | 52-64px |
| Sub-headline | bold | 36-48px |
| Body text | regular | 28-36px |
| Card title | bold | 28-32px |
| Card body | regular | 22-26px |
| Small labels | mono | 16-20px |
| Tag pills | mono | 16px |
| Terminal text | mono | 20px |
| Annotations | regular | 20-24px |

### Icons

See `references/icons.md` for icon set options (Lucide, Tabler, Feather, etc.).

```python
ICON_DIR = os.path.join(SCRIPT_DIR, "icons")
_icon_cache = {}

def load_icon(name, size=96, color=WHITE):
    key = (name, size, color)
    if key in _icon_cache:
        return _icon_cache[key]

    svg_path = os.path.join(ICON_DIR, f"{name}.svg")
    if not os.path.exists(svg_path):
        img = Image.new("RGBA", (size, size), (0, 0, 0, 0))
        _icon_cache[key] = img
        return img

    cmd = [
        "ffmpeg", "-y", "-loglevel", "error",
        "-i", svg_path,
        "-vf", f"scale={size}:{size}",
        "-f", "image2pipe", "-vcodec", "png", "-"
    ]
    result = subprocess.run(cmd, capture_output=True)
    img = Image.open(BytesIO(result.stdout)).convert("RGBA")

    pixels = img.load()
    for y in range(img.height):
        for x in range(img.width):
            r, g, b, a = pixels[x, y]
            if a > 0:
                pixels[x, y] = (color[0], color[1], color[2], a)

    _icon_cache[key] = img
    return img
```

**Rules:** Don't guess names — list the directory. Cache icons. Sizes: 36-48px inline, 64-80px cards, 96-180px hero.

## Animation System

### Easing Functions

```python
def ease_out_cubic(t):
    return 1 - (1 - t) ** 3

def ease_in_out(t):
    return 3 * t * t - 2 * t * t * t

def clamp01(v):
    return max(0.0, min(1.0, v))

def anim_progress(frame, start_frame, duration_frames):
    if frame < start_frame:
        return 0.0
    t = clamp01((frame - start_frame) / max(1, duration_frames))
    return ease_out_cubic(t)
```

### Animation Patterns

**Staggered reveal:**
```python
items = [("Title", 0.5), ("Body", 1.0), ("Card", 2.0)]
for text, delay_seconds in items:
    p = anim_progress(frame, int(delay_seconds * FPS), 15)
    if p > 0:
        draw.text((100, y), text, font=f, fill=WHITE)
```

**Slide-in from left:**
```python
p = anim_progress(frame, start, duration)
x = -300 + int(400 * p)
```

**Fade-in (use for body text — avoids partial text artifacts):**
```python
p = anim_progress(frame, start, duration)
if p > 0:
    draw.text((x, y), text, font=f, fill=color)
```

**Scale-up + fade-in (for icons):**
```python
def paste_icon_animated(img, icon_img, x, y, progress):
    if progress <= 0:
        return
    p = clamp01(progress)
    scale = 0.5 + 0.5 * p
    new_size = (int(icon_img.width * scale), int(icon_img.height * scale))
    scaled = icon_img.resize(new_size, Image.LANCZOS)
    r, g, b, a = scaled.split()
    a = a.point(lambda v: int(v * p))
    scaled = Image.merge("RGBA", (r, g, b, a))
    offset_x = x + (icon_img.width - new_size[0]) // 2
    offset_y = y + (icon_img.height - new_size[1]) // 2
    img.paste(scaled, (offset_x, offset_y), scaled)
```

**Typing effect (for terminal windows):**
```python
chars = int(len(text) * clamp01((frame - start_frame) / (FPS * 0.6)))
visible = text[:chars]
draw.text((x, y), visible, font=f_mono, fill=color)
```

### Crossfade Transitions

Between slides, render 20 frames (~0.67s) of crossfade:

```python
transition_frames = 20
last_frame = current_slide_func(slide_frames - 1, slide_frames)
for t in range(transition_frames):
    alpha = ease_in_out(t / transition_frames)
    first_frame = next_slide_func(0, next_total)
    blended = Image.blend(last_frame, first_frame, alpha)
    proc.stdin.write(blended.tobytes())
```

### Ambient Glow

```python
def draw_ambient_glow(img, cx, cy, radius, color=ACCENT, intensity=0.08):
    overlay = Image.new("RGBA", img.size, (0, 0, 0, 0))
    draw = ImageDraw.Draw(overlay)
    steps = 40
    for i in range(steps):
        r = int(radius * (1 - i / steps))
        alpha = int(255 * intensity * (i / steps))
        c = (*color, alpha)
        draw.ellipse([cx - r, cy - r, cx + r, cy + r], fill=c)
    img.paste(Image.alpha_composite(img.convert("RGBA"), overlay).convert("RGB"))
```

Breathing effect:
```python
breathe = 1.0 + 0.15 * math.sin(frame / (FPS * 1.5))
draw_ambient_glow(img, glow_x, glow_y, 500, ACCENT, intensity * breathe)
```

## Visual Components

### Progress Bar
```python
def draw_progress_bar(img, current_slide, total_slides, frame, total_frames):
    draw = ImageDraw.Draw(img)
    progress = (current_slide + frame / total_frames) / total_slides
    bar_w = int(W * progress)
    draw.rectangle([0, H - 4, bar_w, H], fill=ACCENT)
    draw.rectangle([bar_w, H - 4, W, H], fill=GRAY_70)
```

### Accent Bars
```python
p_bar = anim_progress(frame, start, duration)
draw.rectangle([100, y, 100 + int(300 * p_bar), y + 4], fill=accent_color)
```

### Info Cards
```python
draw.rounded_rectangle([80, y, 900, y + 200], radius=6, fill=GRAY_90)
draw.rectangle([80, y, 86, y + 200], fill=TEAL)  # left accent
```

### Terminal Mockup
```python
def draw_terminal(draw, img, x, y, w, h, lines, frame, start_delay):
    draw.rounded_rectangle([x, y, x + w, y + h], radius=8, fill=(20, 20, 25))
    draw.rounded_rectangle([x, y, x + w, y + 30], radius=8, fill=(35, 35, 42))
    draw.rectangle([x, y + 20, x + w, y + 30], fill=(35, 35, 42))
    for j, c in enumerate([(255, 95, 86), (255, 189, 46), (39, 201, 63)]):
        draw.ellipse([x + 12 + j * 20, y + 8, x + 24 + j * 20, y + 20], fill=c)
    f_mono = font("mono", 20)
    for i, (text, color) in enumerate(lines):
        line_delay = start_delay + i * 0.4
        p = anim_progress(frame, int(line_delay * FPS), 10)
        if p <= 0:
            continue
        chars = int(len(text) * clamp01((frame - int(line_delay * FPS)) / (FPS * 0.6)))
        draw.text((x + 15, y + 40 + i * 28), text[:chars], font=f_mono, fill=color)
```

### Tag Pills
```python
tw = text_width(draw, tag, f_tag) + 24
draw.rounded_rectangle([x, y, x + tw, y + 30], radius=20, outline=GRAY_70, width=1)
draw.text((x + 12, y + 5), tag, font=f_tag, fill=GRAY_40)
```

## Custom Branding

For a complete guide on setting up branded video production — including brand directories,
config files, fonts, logos, and icons — see `references/branding-guide.md`.

The quick version: create a `brand/` directory with a `brand.md` config file and your
logo. The skill reads it automatically and applies your brand to every video.

**Important — Color Dominance**: When a brand is active, the Primary brand color must
dominate. Do NOT assign rainbow colors to feature/benefit cards. Use the Primary color
for all cards and differentiate with icons instead. See the "Color Dominance Rules"
section in `references/branding-guide.md` for the full rules.

### Generated Script Variables

Override these values in the generated script to apply any brand:

```python
# === BRAND CONFIGURATION ===
BRAND_NAME = "My Project"
ACCENT       = (59, 130, 246)
ACCENT_LIGHT = (96, 165, 250)
ACCENT_DARK  = (29, 78, 216)

FONT_PATHS = {
    "bold":    "/path/to/YourFont-Bold.otf",
    "medium":  "/path/to/YourFont-Medium.otf",
    "regular": "/path/to/YourFont-Regular.otf",
    "mono":    "/path/to/MonoFont-Regular.ttf",
}

ICON_DIR = os.path.join(SCRIPT_DIR, "icons")
LOGO_PATH = None  # e.g., "/path/to/logo.png"
```

## Media Assets (Optional)

Embed user-provided screenshots, images, and screen recordings directly into slides.
See `references/media-assets.md` for the full reference.

### Pre-generation: Examining Assets

Before generating the script, Claude must examine the provided assets:

```bash
# For video clips: extract a contact sheet to understand the content
ffmpeg -i assets/demo.mp4 -vf "fps=1,scale=480:-1,tile=4x3" /tmp/contact_sheet.png

# Get clip duration
ffprobe -v error -show_entries format=duration -of csv=p=0 assets/demo.mp4
```

Read the extracted frames or images to understand what they show, then write
narration and auto-assign to the best slide.

### Slide Definition with Assets

```python
slides = [
    (slide_title, "Narration...", 7, None),
    (slide_demo, "Here we see the install...", 10, {
        "path": "assets/demo.mp4",
        "style": "terminal",   # browser | terminal | monitor | phone | plain
    }),
    (slide_ui, "The dashboard shows...", 9, {
        "path": "assets/screenshot.png",
        "style": "browser",
    }),
]
```

### Duration with Video Assets

```python
clip_duration = get_clip_duration(asset["path"])
narration_duration = len(narration_audio) / TTS_SAMPLE_RATE
slide_duration = max(clip_duration, narration_duration) + NARRATION_PADDING
```

### Embedding in Slide Functions

Video clips: extract frames once, then composite the right frame per slide frame.
Images: load once, composite with fade-in animation.

```python
# Pre-load clip frames
clip_frames = load_clip_frames("assets/demo.mp4", target_fps=FPS)

# In the slide render function, for each frame:
clip_idx = max(0, frame - int(delay * FPS))
embed_video_asset(img, clip_frames, clip_idx, x, y, w, h, style="terminal")
```

## Narration with TTS (Optional)

Add AI-generated voice narration using **Kokoro TTS** — an open-weight, Apache 2.0 licensed
text-to-speech model with 82M parameters and 54 voices across 8 languages.

### Prerequisites

```bash
pip install kokoro soundfile
# espeak-ng required as phonemizer backend
sudo dnf install espeak-ng        # Fedora
sudo apt install espeak-ng        # Ubuntu
```

On first run, Kokoro downloads model weights from Hugging Face (~300MB).

### Architecture

Each slide gets a `narration_text` string. The rendering pipeline becomes:

1. **Generate narration**: Kokoro generates a WAV per slide
2. **Auto-calculate durations**: Slide duration = max(speech_length + padding, min_duration)
3. **Render video**: Frames rendered using the calculated durations
4. **Mix audio**: Narration + background music mixed together (music auto-ducks during speech)
5. **Mux**: Final audio muxed with video via ffmpeg

### Slide Definition with Narration

```python
slides = [
    (slide_title, "In this video, you'll learn how to...", 7),
    (slide_problem, "The challenge many teams face is...", 9),
    # (render_function, narration_text, min_duration_seconds)
]
```

### TTS Configuration

```python
TTS_VOICE = "af_heart"       # Kokoro voice ID (see list below)
TTS_LANG = "a"               # 'a' = American English, 'b' = British, etc.
TTS_SPEED = 1.0              # Speech speed multiplier
TTS_SAMPLE_RATE = 24000      # Kokoro native sample rate
NARRATION_PADDING = 1.5      # Extra seconds per slide (before + after speech)
MIN_SLIDE_DURATION = 5       # Minimum slide duration in seconds
```

### Generating Narration

```python
from kokoro import KPipeline

pipeline = KPipeline(lang_code='a')  # American English
audio_chunks = []
for gs, ps, audio in pipeline(narration_text, voice='af_heart', speed=1.0):
    audio_chunks.append(audio)
full_audio = np.concatenate(audio_chunks)  # numpy array at 24kHz
```

### Audio Ducking

Background music automatically reduces volume when narration is active:

```python
# Compute speech envelope
envelope = np.abs(narration_track)
# Smooth with running average
kernel = np.ones(window) / window
envelope = np.convolve(envelope, kernel, mode='same')
# Duck: reduce music to 30% during speech
duck_factor = 1.0 - 0.7 * np.clip(envelope * 5, 0, 1)
music_ducked = music_track * duck_factor
# Mix
final = narration_track * 0.95 + music_ducked
```

### Voice Cloning

To use a custom voice (e.g., your own), provide a `.pt` voice tensor:

```python
import torch
voice_tensor = torch.load('my_voice.pt', weights_only=True)
for gs, ps, audio in pipeline(text, voice=voice_tensor):
    ...
```

### Supported Languages

| Code | Language |
|------|----------|
| `a` | American English |
| `b` | British English |
| `e` | Spanish |
| `f` | French |
| `h` | Hindi |
| `i` | Italian |
| `j` | Japanese (requires `pip install misaki[ja]`) |
| `p` | Brazilian Portuguese |
| `z` | Mandarin Chinese (requires `pip install misaki[zh]`) |

### Popular Voices

Use any of Kokoro's 54 built-in voices. Some popular ones:
- `af_heart` — warm, clear female voice (default)
- `af_bella` — expressive female voice
- `am_adam` — neutral male voice
- `am_michael` — authoritative male voice
- `bf_emma` — British female voice
- `bm_george` — British male voice

## Background Music (Optional)

The music system dynamically generates different soundscapes based on the content type
and mood of the video. See `references/music.md` for the full reference with all layer
implementations, chord progressions, and mood presets.

### Mood Presets

| Mood | BPM | Key | Feel | Content Type |
|------|-----|-----|------|-------------|
| `upbeat` | 108–115 | C major | Confident, forward-moving | demo |
| `ambient` | 90–100 | A minor | Thoughtful, contemplative | explainer |
| `energetic` | 122–130 | E minor | Exciting, driving | promo |
| `calm` | 84–92 | D major | Relaxed, focused | tutorial |
| `cinematic` | 100–110 | C major | Uplifting, building | showcase |
| `dark` | 95–105 | D minor | Serious, tension | problem-focused |
| `lofi` | 75–85 | F major | Warm, chill, nostalgic | casual/informal |

### Usage

```python
MUSIC_MOOD = "calm"  # Auto-set by content type, or overridden by user

music = generate_music(duration_seconds, mood=MUSIC_MOOD)
```

Each mood defines its own:
- **Chord progression** — different harmonic feel (major vs minor, resolved vs tense)
- **Active layers** — which instruments play (pads, bass, kick, hi-hat, arp, etc.)
- **Energy level** (0.0–1.0) — controls intensity and layer volume
- **BPM** — tempo matching the content pacing

### Available Layers

| Layer | Description | Used By |
|-------|-------------|---------|
| `pad` | Sustained detuned sine chords | All moods |
| `pad_wide` | Extra detuning + octave doubling | cinematic |
| `sub_bass` | Low sine following chord roots | All moods |
| `arp` | Plucky repeating note pattern | upbeat, ambient, cinematic |
| `arp_mellow` | Soft attack, slow decay arp | lofi |
| `kick` | Half-time punchy kick (beats 1, 3) | upbeat |
| `kick_four` | Four-on-the-floor kick | energetic |
| `kick_gentle` | Soft, felt-not-heard kick | cinematic |
| `hihat` | 8th note pattern with velocity variation | upbeat |
| `hihat_active` | 16th notes with open hat accents | energetic |
| `pluck` | Accent hits at chord changes | upbeat, energetic |
| `shaker` | Very light 16th note filtered noise | energetic |
| `rumble` | Sub-frequency LFO tension | dark |
| `crackle` | Sparse vinyl crackle texture | lofi |

### Dynamic Energy Curves

For moods like `cinematic`, energy builds over the video:

```python
def get_energy(base_energy, t, duration, curve="flat"):
    progress = t / duration
    if curve == "flat":
        return base_energy
    elif curve == "build":
        return base_energy * (0.5 + 0.5 * progress)
    elif curve == "arc":
        return base_energy * (0.6 + 0.4 * math.sin(progress * math.pi * 0.9))
```

### Volume by Mood

| Mood | Volume | Why |
|------|--------|-----|
| `calm` | 0.15 | Barely there — viewer is learning |
| `ambient` | 0.18 | Present but unobtrusive |
| `lofi` | 0.20 | Warm presence |
| `cinematic` | 0.22 | Emotional but not distracting |
| `upbeat` | 0.22 | Confident presence |
| `energetic` | 0.25 | Felt, not overwhelming |
| `dark` | 0.20 | Tension without distraction |

### Muxing

```python
cmd = [
    "ffmpeg", "-y", "-loglevel", "warning",
    "-i", VIDEO_IN, "-i", AUDIO_WAV,
    "-c:v", "copy", "-c:a", "aac", "-b:a", "192k",
    "-shortest", VIDEO_OUT,
]
```

## Quality Checklist

- [ ] All text uses configured fonts (graceful fallback if missing)
- [ ] ACCENT color appears on every slide
- [ ] Color palette is consistent throughout
- [ ] Background is dark with ambient glow
- [ ] Glow position varies between slides
- [ ] Headlines tell a complete story in sequence
- [ ] Body text fades in (not slides in)
- [ ] All animations use eased timing, no linear motion
- [ ] Staggered reveals have 0.5-1.5s delays
- [ ] Crossfade transitions between all slides
- [ ] Progress bar visible at bottom
- [ ] Icon filenames verified (don't guess)
- [ ] Icons are cached
- [ ] Callout boxes sized to content
- [ ] Video is 1920x1080 at 30fps
- [ ] Music (if included) at 25% volume
- [ ] Narrative follows a clear story arc

## Common Pitfalls

1. **Sliding text shows partial characters**: Use fade-in for body text
2. **Icons overlap text**: Calculate text width first
3. **Callout boxes too large**: Size to content
4. **libx264 not available**: Use `libopenh264`
5. **Icon names wrong**: List the directory, don't guess
6. **Music too loud**: Keep at 25%
7. **Font not found**: Fall back to system default
