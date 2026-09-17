---
name: kokoro-tts-voiceover
description: Local AI voice generation and precision-timed video dubbing using Kokoro ONNX. Generates natural, studio-quality speech locally with zero external APIs, zero cost, and microsecond timing alignment for canvas and procedural animation films.
---

# Kokoro TTS & Precision Video Dubbing

A zero-latency, private, local text-to-speech skill built on [Kokoro-ONNX](https://github.com/thewh1teagle/kokoro-onnx). It synthesizes human-grade voiceovers, formats timed narration scripts, guarantees **zero audio overlap**, and cleanly ducks background music in animated films.

---

## 1. Environment & Setup

The local model files and virtual environment are installed at:
```bash
~/kokoro-tts/
├── .venv/                 # Dedicated Python environment with kokoro-onnx, soundfile, onnxruntime
├── kokoro-v1.0.onnx       # Full 82M parameter English ONNX weights (~326 MB)
├── voices-v1.0.bin        # Voice embeddings pack (~28 MB)
├── tts.py                 # Core synthesis program
└── kokoro                 # CLI launcher script
```

### Verification & Quick Test
```bash
~/kokoro-tts/kokoro --text "System check. Local voice generation is active." --voice af_heart --out test.wav
```

---

## 2. Voice Selection Guide

Kokoro provides multiple high-quality voices categorized by gender and accent:

| Voice ID | Accent | Gender | Vibe & Ideal Use Case |
|---|---|---|---|
| `af_heart` | American | Female | Warm, balanced, engaging. **Default for educational/scientific explainers.** |
| `af_bella` | American | Female | Bright, expressive, fast-paced. Great for dynamic shorts. |
| `af_nicole` | American | Female | Calm, whisper-soft, introspective. Ideal for philosophical essays. |
| `af_sarah` | American | Female | Clear, authoritative, documentary-style. |
| `am_michael` | American | Male | Deep, reassuring, narrative baritone. Great for historical/philosophical films. |
| `am_fenrir` | American | Male | Gritty, dramatic, high-energy. |
| `bf_emma` | British | Female | Refined, academic, articulate. Ideal for BBC-style science explainers. |
| `bf_isabella` | British | Female | Elegant, literary, cinematic. |
| `bm_george` | British | Male | Warm British intellectual narrator. |
| `bm_fable` | British | Male | Storybook, rhythmic, classical. |

List all available voices programmatically:
```bash
~/kokoro-tts/kokoro --list-voices
```

---

## 3. The Zero-Overlap Timing Algorithm

When dubbing short-form video (such as 30s–60s vertical or square animations), cues must synchronize with visual scene cuts without **talking over each other**.

### The Strict Sequential Placement Rule
Given a list of cues `[(target_time, text, base_speed), ...]`:
1. Synthesize audio for cue $i$.
2. Strip leading and trailing silence (`numpy` threshold).
3. Compute duration $D_i = \text{samples} / \text{sample\_rate}$.
4. If $T_{\text{target}} < T_{\text{prev\_end}} + \Delta_{\text{breath}}$ (where $\Delta_{\text{breath}} \ge 0.20\text{s}$):
   - **Either**: Recalculate speech rate: $\text{speed} = \max(\text{base\_speed}, \frac{D_i}{\text{available\_window}})$.
   - **Or**: Push $T_i = T_{\text{prev\_end}} + \Delta_{\text{breath}}$ so a natural breath is preserved.
5. Place the non-overlapping slice into the master audio buffer.

```python
# Reference Implementation
import numpy as np

def place_cues_safely(cues, kokoro, voice="af_heart", min_gap=0.22, sr=48000, total_sec=60.0):
    total_samples = int(total_sec * sr)
    master = np.zeros(total_samples, dtype=np.float32)
    last_end_time = 0.0

    for idx, (target_t, text, speed) in enumerate(cues):
        samples, orig_sr = kokoro.create(text, voice=voice, speed=speed, lang="en-us")
        # Resample to 48kHz if needed
        if orig_sr != sr:
            samples = np.repeat(samples, sr // orig_sr)
        
        # Trim digital silence (< 0.005 peak)
        non_silent = np.where(np.abs(samples) > 0.005)[0]
        if len(non_silent) > 0:
            samples = samples[non_silent[0]:non_silent[-1] + 1]

        dur = len(samples) / sr

        # ENFORCE ZERO OVERLAP
        actual_start = max(target_t, last_end_time + min_gap)
        start_idx = int(actual_start * sr)
        end_idx = start_idx + len(samples)

        if start_idx < total_samples:
            fit_end = min(end_idx, total_samples)
            master[start_idx:fit_end] += samples[:fit_end - start_idx]
            last_end_time = actual_start + dur
            
    return master
```

---

## 4. Audio Ducking & Final Muxing Pipeline

Never place raw speech over full-volume music. Always mix with balanced sidechain ducking via FFmpeg:

```bash
ffmpeg -y \
  -i video.mp4 \
  -i score.wav \
  -i voiceover.wav \
  -filter_complex "\
    [1:a]volume=0.30[bg]; \
    [2:a]volume=1.15[vo]; \
    [bg][vo]amix=inputs=2:duration=first:dropout_transition=2[aout]" \
  -map 0:v -map "[aout]" \
  -c:v copy -c:a aac -b:a 192k -shortest final.mp4
```

- `volume=0.30[bg]`: Ducks background music by ~10.5 dB under the voice.
- `volume=1.15[vo]`: Gives speech clarity and punch.
- `amix=inputs=2`: Smooth summing without digital clipping.
