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

Frames are captured into a fixed-length buffer at the chosen rate. The composite is the accumulation of frames between the two handles on the density strip. Moving a handle adds or subtracts individual frames from the running accumulator rather than recomputing from scratch, so scrubbing stays responsive even on long clips.

**Blend modes**

- **Mean** — averages the range. Motion smears, static parts stay sharp. Long enough, and moving people disappear.
- **Sum** — adds the range. Light builds on light; anything dark contributes nothing. Light painting.
- **Lighten** — keeps the brightest value each pixel ever reached. Traffic trails, sparklers, star trails.
- **Darken** — keeps the darkest. The inverse; good for pulling silhouettes out of a moving bright field.

Exposure is in stops and applies after accumulation, so you can push Sum hard without re-recording.

**Trail the last** anchors the start handle a fixed number of frames behind the end, giving a rolling window instead of a growing one. During recording that reads as a decaying trail; on playback it moves both handles together.

**Rewinding** is just the end handle moving backwards — press rewind, or drag the handle. Mean and Sum unwind frame by frame. Lighten and Darken can't be un-maxed, so shrinking the range recomputes that span; it's still fast, just doing more work.

## Notes on limits

Frames are held in RAM as raw RGB. At Mid detail (480px wide) each frame is around 380 KB, so a 30-second buffer at 12/s is roughly 140 MB. Raise **Detail** or **Length** and that climbs quickly — if the tab dies mid-record, that's why. Low detail plus a long buffer is the safe combination for multi-minute exposures.

Recording stops when the buffer fills rather than dropping old frames, so a trimmed start always means what you think it means.

Export build writes a video of the exposure assembling frame by frame. Chrome gives WebM; Safari may refuse entirely, in which case save a still instead.
