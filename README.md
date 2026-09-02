# Accumulator

A camera PWA that builds a single exposure out of many frames, and lets you scrub back through the build or trim where the exposure starts and ends.

## Deploying to GitHub Pages

1. Create a repo and drop all six files in the root (or in `/docs`).
2. Settings → Pages → Deploy from a branch → `main` / root.
3. Open the URL on your phone. Pages is HTTPS, which `getUserMedia` requires.
4. Chrome menu → Add to home screen. It runs offline after the first load.

All paths are relative, so it works from `username.github.io/repo-name/` without changes.

Files:

```
index.html                 whole app, no build step
manifest.webmanifest
sw.js                      cache-first app shell
icon-192.png
icon-512.png
icon-maskable-512.png
```

## How it works

Frames are captured into a fixed-length buffer at the chosen rate. The composite is the accumulation of frames between the two handles on the strip. Moving a handle adds or subtracts individual frames from a running accumulator rather than recomputing from scratch, so scrubbing stays responsive on long clips.

The strip shows thumbnails of the clip along the top and a frame-to-frame change trace along the bottom, auto-scaled to the busiest moment. Peaks are where something moved or a light passed through.

## The exposure model

Accumulation happens in **linear light**. Each frame is converted out of sRGB before being added, and the result is encoded back on the way to the screen. This is what a sensor actually does, and it is the difference between highlights that bloom and highlights that go muddy. Turn it off to accumulate in gamma space if you want the flatter, more digital look.

**Normalize** controls the divisor: the accumulated total is divided by `n^p`, where `n` is the frame count and `p` is the slider.

- `1.00` (Average) — brightness stays constant however long you expose. Silky water, ghosted people, the classic ND-filter look.
- `0.00` (Sum) — nothing is divided out. Light genuinely builds up, and a long enough exposure blows out, exactly like leaving the shutter open. For light painting.
- Between — partial normalization. Around `0.5` the image brightens with time but slowly enough to stay usable, which is usually the sweet spot for anything longer than a few seconds.

**Exposure** is a clean gain in stops, applied after accumulation. Nothing is baked in at capture, so you can rebalance any take at any time. **Fit** picks the exposure that puts the brightest half-percent of pixels just under white.

**Soft highlights** replaces the hard clip at white with a shoulder that rolls off toward it. Accumulations bloom instead of flattening into solid white patches. It only affects the top of the range.

**Lighten** and **Darken** are not exposure physics — they keep the brightest or darkest value each pixel ever reached. Lighten is the right tool for traffic trails and star trails, where you want the trails to build without the sky building with them. Darken pulls silhouettes out of a moving bright field.

**Trail the last** anchors the start handle a fixed number of frames behind the end, giving a rolling window instead of a growing one — a decaying trail rather than a total.

**Rewinding** is just the end handle moving backwards. Expose unwinds frame by frame. Lighten and Darken cannot be un-maxed, so shrinking the range recomputes that span.

## Notes on limits

Frames are held in RAM as raw RGB. At Mid detail (480px wide) each frame is around 380 KB, so a 30-second buffer at 12/s is roughly 140 MB. Raise **Detail** or **Length** and that climbs quickly — if the tab dies mid-record, that's why. Low detail plus a long buffer is the safe combination for multi-minute exposures.

Recording stops when the buffer fills rather than dropping old frames, so a trimmed start always means what you think it means.

Export build writes a video of the exposure assembling frame by frame. Chrome gives WebM; Safari may refuse entirely, in which case save a still instead.
