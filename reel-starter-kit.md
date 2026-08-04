# Reel Starter Kit

*The actual prompts and template, so you can make one yourself. From @theprocrastihacker*

The other file explains what tools I use. This one gives you the stuff to copy.

---

## 1. The prompt that writes the script

Paste this into Claude (or any AI). Swap the topic.

```
You are writing a script for a short vertical video. 8 seconds.

Topic: [YOUR TOPIC HERE]

Rules:
- One hook line, 8 words max. It must create a question in the viewer's head.
- One supporting chip: 3 or 4 words, like a subtitle.
- Then write the full caption separately. That is where all the detail goes.
- No em dashes anywhere.
- High school reading level. Short words, short sentences.
- Do not invent facts. If you are not sure about something, say so instead of guessing.

Give me:
1. Three hook options, so I can pick
2. The chip text
3. The full caption, with the real detail and a clear takeaway
```

**Why it is built this way:** the video's only job is to stop the scroll. The caption does the
teaching. If you try to teach inside 8 seconds you will fail at both.

---

## 2. The prompt for the voiceover

Once you have a script and a rendered video:

```
Write a voiceover for a [X] second video.

Budget 2.3 words per second. That is the number that matters. At 3 words per second
it sounds rushed. So for a 19 second video, write about 42 words. Not more.

Leave a natural pause between each line. The visuals fill those gaps.

The script is: [PASTE YOUR SCRIPT]
```

I got this wrong the first time. Wrote 62 words for a 19 second video and it came out
sounding panicked. 2.3 words per second is the fix.

---

## 3. A working template you can render

Save this as `index.html` inside a HyperFrames project. Drop any vertical clip in as
`assets/broll.mp4`. Then run `npx hyperframes render`.

```html
<!doctype html>
<html>
<head>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.14.2/dist/gsap.min.js"></script>
<style>
  * { margin:0; padding:0; box-sizing:border-box; }
  html, body { width:1080px; height:1920px; overflow:hidden; background:#0e0e0e; }
  #root { container-type:size; font-family:system-ui, sans-serif; }
  video.fill { position:absolute; inset:0; width:100%; height:100%; object-fit:cover; }

  /* dark gradient so text stays readable over any footage */
  #scrim { position:absolute; inset:0; background:linear-gradient(180deg,
    rgba(12,11,10,.8) 0%, rgba(12,11,10,.5) 26%, rgba(12,11,10,0) 55%,
    rgba(12,11,10,.55) 100%); }

  #hook { position:absolute; left:7cqw; right:7cqw; top:11cqh;
    font-weight:600; font-size:8cqw; line-height:1.1; color:#f7f2e6;
    text-shadow:0 .3cqw 1.8cqw rgba(0,0,0,.6); }
  #hook .accent { color:#f0a87a; font-style:italic; }

  #chipwrap { position:absolute; left:0; right:0; top:31cqh; text-align:center; }
  #chip { display:inline-block; font-family:ui-monospace, monospace;
    font-size:2.5cqw; letter-spacing:.16em; text-transform:uppercase;
    color:#161513; background:#f0c860; padding:1.5cqw 3cqw; border-radius:1.2cqw; }

  #ctawrap { position:absolute; left:0; right:0; bottom:15cqh; text-align:center; }
  #cta { display:inline-block; background:#efe6d6; color:#161513;
    font-weight:600; font-size:3.4cqw; padding:2.2cqw 4.4cqw;
    border-radius:1.8cqw; box-shadow:0 .7cqw 0 #161513; }

  #handle { position:absolute; left:0; right:0; bottom:8cqh; text-align:center;
    font-family:ui-monospace, monospace; font-size:2.1cqw; letter-spacing:.12em;
    color:#f3ecd9; text-shadow:0 .2cqw 1cqw rgba(0,0,0,.7); }
</style>
</head>
<body>
  <div id="root" data-composition-id="main" data-start="0" data-duration="8"
       data-width="1080" data-height="1920">

    <video id="v1" class="fill" src="assets/broll.mp4" data-start="0" data-duration="8"
           data-track-index="1" data-media-start="0" muted playsinline
           style="object-position:50% 42%"></video>

    <div id="scrim" class="clip" data-start="0" data-duration="8" data-track-index="5"></div>

    <div id="hook" class="clip" data-start="0" data-duration="8" data-track-index="20">
      Your hook goes <span class="accent">here.</span>
    </div>

    <div id="chipwrap" class="clip" data-start="0" data-duration="8" data-track-index="21">
      <span id="chip">your chip text</span>
    </div>

    <div id="ctawrap" class="clip" data-start="0" data-duration="8" data-track-index="22">
      <span id="cta">full breakdown in the caption</span>
    </div>

    <div id="handle" class="clip" data-start="0" data-duration="8" data-track-index="23">
      @yourhandle
    </div>
  </div>

<script>
  const tl = gsap.timeline({ paused: true });

  // slow push so an 8 second loop never looks like a frozen image
  tl.fromTo("#v1", { scale: 1.0 }, { scale: 1.07, duration: 8, ease: "none" }, 0);

  tl.fromTo("#hook",   { opacity:0, y:26 }, { opacity:1, y:0, duration:.55, ease:"power3.out" }, .15);
  tl.fromTo("#chip",   { opacity:0, scale:.9 }, { opacity:1, scale:1, duration:.5, ease:"back.out(1.6)" }, .55);
  tl.fromTo("#cta",    { opacity:0, y:20 }, { opacity:1, y:0, duration:.5, ease:"power3.out" }, .85);
  tl.fromTo("#handle", { opacity:0 }, { opacity:1, duration:.5 }, 1.0);

  window.__timelines = { main: tl };
</script>
</body>
</html>
```

**Three things in there that took me a while to learn:**

1. The `<video>` tag has to be a direct child of `#root`. Wrap it in a div and it renders black.
2. Every timed element needs `class="clip"` plus the three `data-` attributes, or it will not
   show up.
3. That slow `scale` push matters more than you would think. Without it an 8 second loop reads
   as a still image and people scroll past.

---

## 4. Getting your phone footage ready

```bash
ffmpeg -i yourclip.mp4 \
  -vf "scale=1080:1920:flags=lanczos,unsharp=5:5:0.6:5:5:0.0" \
  -an -r 30 -c:v libx264 -preset slow -crf 16 -pix_fmt yuv420p \
  assets/broll.mp4
```

If your phone shoots HDR (most do now), you need the tone mapping step too. That is in the
other cheatsheet, and skipping it is why your footage looks washed out and grey.

---

## 5. The order I actually work in

```
1. Pick a topic
2. Prompt 1 above  -> hook + caption
3. Shoot or pick one clip, 10 to 15 seconds
4. Convert it with the ffmpeg command
5. Paste the template, swap the text
6. npx hyperframes render
7. Post it, put the real detail in the caption
```

First one will take you an afternoon. After that it is about 25 minutes.

---

*More at @theprocrastihacker. Daily tech and AI, minus the jargon.*
