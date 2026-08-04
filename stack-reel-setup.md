# My Reel Setup

*You commented "stack" so here it is. From @theprocrastihacker*

---

## Before anything else

Nothing here costs money. It all runs on the laptop I already had.

It also took me about 3 months to build. So if you just want to post videos, honestly go learn to
edit. That is faster. I built this because I wanted the pipeline more than I wanted the videos.

---

## The 4 pieces

### 1. The script: Claude

It researches the topic against the real docs, then writes the script.

One rule I stick to. Every fact needs a source. Getting something wrong in front of developers
costs way more than the reel is worth.

### 2. The captions: Whisper, running on my laptop

[faster-whisper](https://github.com/SYSTRAN/faster-whisper), the `small.en` model, int8 on CPU.

It gives you word level timing, so each caption lands on the exact word.

```bash
pip install faster-whisper
```

```python
from faster_whisper import WhisperModel
m = WhisperModel("small.en", device="cpu", compute_type="int8")
segments, info = m.transcribe("audio.wav", word_timestamps=True)
```

No API, nothing uploaded, no monthly fee. Takes about 20 seconds for a 60 second video.

Heads up: give it a WAV file, not an MP4. That one saved me an hour of weird errors.

### 3. The edit: HyperFrames

[HyperFrames](https://hyperframes.heygen.com) is open source (Apache 2.0) from HeyGen. It turns
HTML and GSAP into video.

So editing is just writing code. Every card, animation and transition is written, not dragged
around a timeline.

```bash
npx hyperframes init my-reel
npx hyperframes lint      # catches problems before you render
npx hyperframes render    # gives you an MP4
```

The real win is that it is repeatable. Change one line, render again, and you get the exact same
video with just that one change. Nothing to rebuild.

### 4. The footage: my phone

Shot on a Pixel in a community office room 20 minutes from my apartment. No studio, no lights,
no camera.

---

## The thing that broke my footage

Phones record in HDR. If you convert that to normal video without tone mapping, you get grey milky
walls, blown out windows, and flat everything. I actually shipped a batch like this before I caught it.

Check your file first:

```bash
ffprobe -v error -select_streams v:0 \
  -show_entries stream=color_transfer,color_primaries,pix_fmt \
  -of default=nw=1 input.mp4
```

If `color_transfer` says `arib-std-b67` or `smpte2084`, you have HDR and you need this:

```bash
TM="zscale=t=linear:npl=100,format=gbrpf32le,zscale=p=bt709,tonemap=tonemap=hable:desat=0,zscale=t=bt709:m=bt709:r=tv,format=yuv420p"

ffmpeg -i in.mp4 -vf "$TM,scale=1080:1920:flags=lanczos,unsharp=5:5:0.6:5:5:0.0" \
  -an -r 30 -c:v libx264 -preset slow -crf 16 -pix_fmt yuv420p \
  -color_primaries bt709 -color_trc bt709 -colorspace bt709 out.mp4
```

Two more things I learned the hard way:

**Never crop into phone footage.** Shrinking a full 4K frame down to 1080p squeezes 4 pixels into 1,
and that is what makes it look sharp. Crop in and you throw that away. If you want a close up, you
have to actually shoot a close up.

**Always check your output at full size.** Both of these problems are invisible in a small preview
and really obvious at full resolution.

---

## What it costs

| Thing | Cost |
|---|---|
| Claude | subscription I already pay for |
| Whisper | free, runs locally |
| HyperFrames | free, open source |
| ffmpeg | free |
| Camera | my phone |
| **Every month** | **nothing** |

---

*More stuff like this at @theprocrastihacker. Daily tech and AI, minus the jargon.*
