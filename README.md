# Middleham Castle — scroll timeline

An interactive scroll-driven reconstruction of Middleham Castle, North Yorkshire,
showing the site's development from the timber motte-and-bailey on William's Hill
through to the ruined "Windsor of the North".

Built by [Tower & Keep](https://www.towerandkeep.org/), a 501(c)(3) non-profit
bringing historical sites and stories to life through immersive media.

## What's here

| File | Purpose |
|---|---|
| `index.html` | The whole page — markup, styles and scroll engine in one file |
| `middleham-timeline.mp4` | The background reconstruction, scrubbed by scroll |
| `middleham-3d-flyaround.mp4` | Looping clip in the "Explore in 3D" panel |
| `tk-logo.png` | Header logo |
| `.nojekyll` | Tells GitHub Pages to serve the files as-is |

No build step, no dependencies. Open `index.html` in a browser and it runs.

## How the scroll works

The page holds a full-screen `<video>` fixed in place while a tall invisible
element below it provides the scroll distance. As you scroll, the video's
`currentTime` is driven directly from the scroll position — the video never
plays, it is scrubbed.

Three numbers control the feel, all near the top of the `<script>`:

- **`SCRUB_SMOOTHING`** (0.10) — how fast the video catches up to the scrollbar.
  Low values glide between mouse-wheel clicks instead of mirroring each one.
- **`VIDEO_ENDS_AT`** (0.94) — the scroll fraction where the video reaches its
  last frame. Everything after it is a still frame before the closing panel.
- **`#scroll-driver { height }`** (2200vh) — total scroll distance.

The driver height matters more than it looks. Each video frame has to cover a
certain number of scroll pixels, and below roughly 12–15 px per frame the motion
reads as continuous. A **longer** page is steppier, not smoother, because the
frames run out. The arithmetic is written out in a comment above the height.

The video is encoded with a keyframe every 8 frames. This matters: scrubbing
seeks constantly, and with the usual one-keyframe-every-few-seconds the browser
shows an approximated frame instead of the real one, which looks like blur.

## The cards

Chapter cards are positioned by `data-position` — a fraction of the total scroll
distance, not a timestamp. To convert:

```
position = seconds / 43.03 * VIDEO_ENDS_AT
```

They are placed to land inside the era of the footage they describe, inset from
the cuts so none appears during a transition.

Sides alternate automatically (odd left, even right). To pin one card against
that pattern, add `data-side="left"` or `data-side="right"`. Note the
alternation counts *live* cards, so commenting one out flips every card below
it — `data-side` is how you hold one in place.

Some neighbouring cards overlap briefly. Those pairs must not share a side or
they will collide on desktop; the list is in a comment above the cards. On
screens under 640px only the nearest card is shown, since cards are centred
there and two at once would stack.

## Performance mode

The `<html>` tag carries `perf-lite` or `perf-high`. `perf-high` gives the cards
a frosted-glass backdrop blur; `perf-lite` makes them more opaque instead. The
blur is re-computed by the GPU every frame while the video repaints beneath it,
so `perf-lite` is noticeably smoother on integrated graphics.

## Publishing

This folder is ready to serve as-is. On GitHub Pages, push it to a repository
and enable Pages in **Settings → Pages** with the branch set to `main` and the
folder to `/ (root)`.

Note that GitHub Pages serves from a case-sensitive filesystem while Windows is
not, so filenames here are deliberately lowercase — a file that loads locally
can 404 once published if the case doesn't match.

Do not move the video to Git LFS. GitHub Pages does not serve LFS content; it
returns the pointer file and the video silently fails to play.
