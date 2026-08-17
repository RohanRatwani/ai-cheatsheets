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

## The thing that broke my audio (and this one is worse)

I shipped a reel where you genuinely could not hear me. I had a YouTube video playing at half volume
right before it, heard that fine, then my own reel was inaudible at the same setting.

Phone voice memos come out **far quieter than you think**. Mine measured **-38 LUFS**. Instagram wants
about **-14 LUFS**. That is roughly 24 decibels down, which is somewhere near a sixteenth of normal
loudness. My true peak was -21 dBFS, so there was a mountain of unused headroom just sitting there.

Measure yours first. If the integrated number is not close to -14, this is your problem too:

```bash
ffmpeg -i voice.m4a -af ebur128 -f null -
```

Then fix it in two passes. Pass one measures, pass two applies:

```bash
# 1. read the numbers out of this
ffmpeg -i voice.m4a -af "highpass=f=80,afftdn=nf=-28,loudnorm=I=-14:TP=-1.5:print_format=json" -f null -

# 2. paste them back in as measured_*
ffmpeg -i voice.m4a -af "highpass=f=80,afftdn=nf=-28,\
acompressor=threshold=-18dB:ratio=3:attack=8:release=180,\
loudnorm=I=-14:TP=-1.5:LRA=11:measured_I=-39.9:measured_TP=-22.8:measured_LRA=3.5:measured_thresh=-50.4:linear=true,\
alimiter=limit=0.94" -ar 48000 -c:a aac -b:a 192k voice-clean.m4a
```

**Why each piece is there:**

- **highpass** kills desk rumble and the thump of picking the phone up
- **afftdn** removes room hiss. You need this, because turning the voice up 24 dB turns the room up
  24 dB too
- **acompressor** evens out your loud and quiet words so nothing disappears
- **loudnorm** does the actual loudness match. Two passes because one pass drifts
- **alimiter** catches stray peaks so Instagram never clips you
- **`linear=true` matters a lot.** It applies one fixed gain instead of riding the level up and down.
  That means your word timings do not move, so if you already synced captions or animation to the
  transcript, everything stays in sync. Without it, things drift

Two gotchas if you script this:

1. **ffmpeg prints its analysis to stderr and still exits with code 0.** So if you are reading the
   output in Node or Python, the "success" path gives you nothing. Read stderr, not stdout.
2. **`ebur128` prints an `I:` line for every frame as it goes.** If you regex for the first `I:` you
   will grab a silent moment at the start and see something like -70. Only read the number under the
   final `Integrated loudness:` heading.

And the free fix that beats all of this: **hold the phone 15 to 20 cm from your mouth**, not across
the desk. Distance is the whole problem. Record one test line and play it back before you record
everything.

---

## What to actually make (this matters more than the tools)

The tools are the easy part. Picking the wrong thing to make is what wastes your week.

Two formats do almost all the work for me:

**One clip, one line, all the detail in the caption.** A 6 to 8 second clip holding a single hook
line, with the real explanation written in the caption. A short clip gets watched 5 or 10 times
through, which reads as very high retention, while a 20 second multi-card video has a place to drop
off every few seconds. Use this whenever the content is *reference material*: file paths, tool names,
config keys. Nobody absorbs a file path at 3 seconds a card.

**The red and green list.** Four to six `don't do this / do this instead` pairs. Build it as an
accumulating two column list where the rows you have not reached yet are solid colour bars that
resolve into words on their beat. Being able to *count* how many answers are still hidden holds
people better than a progress bar, and the finished list is the frame people screenshot.

And the hook rule I wish I had learned first: **write the hook as the viewer's own sentence, in
quotes.** Not "6 things to stop doing", which is a table of contents. Something like *"I want better
answers from AI but I don't know what I'm doing wrong."* If someone has had that exact thought they
are already watching, and if they have not, they were never your audience.

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

## I packaged the whole thing up

Everything above, as actual working code:
**[github.com/RohanRatwani/reel-engine](https://github.com/RohanRatwani/reel-engine)**

It is a Claude Code skill. Clone it into your skills folder, ask for a reel, and it runs the
pipeline. MIT licensed, take whatever you want from it.

```bash
git clone https://github.com/RohanRatwani/reel-engine .claude/skills/reel-engine
```

What is in there:

- **The audio fix as one command.** `vo-clean.mjs` does the whole two pass chain above for you
- **`conform-footage.mjs`** detects HDR, tone maps it, and refuses to crop. It also tells you when
  your footage cannot carry a close up
- **Two render ready templates.** A single clip loop and the red green checklist, both lint clean
- **`clip-finder.mjs`** takes a ten minute recording and hands you ranked clip candidates with
  timestamps, so you stop scrubbing a timeline looking for the good part
- **Local Whisper** for word level caption timing
- **Every gotcha written down**, including the ones that shipped broken videos before I caught them

No API keys, nothing to sign up for, no footage of mine in there.

Still want just the copy paste version? The [Reel Starter Kit](reel-starter-kit.md) has the prompt
that writes the script, the voiceover word budget, and a template.

---

*More stuff like this at @theprocrastihacker. Daily tech and AI, minus the jargon.*
