# got-milk-data 🍼

A single-file web app for visualizing a baby's milk **consumption** and
**pumping** over time, from an exported care log.

**Live:** https://nhdaly.github.io/got-milk-data/

Everything lives in [`baby-feed-rate.html`](baby-feed-rate.html) — no build step,
no dependencies to install. Open the file in a browser (it loads Chart.js from a
CDN) or visit the live page above.

## What it does

It estimates the **average milk rate (ml/hour)** over time using a moving window,
and plots two stacked, time-aligned charts:

- **Consumption (feeds)** — Combined by default, with toggleable **Bottle** and
  **Breast** lines (click the legend).
- **Pumping (production)** — total pumped per session (left + right).

Each chart shows a **summary of the totals over the currently viewed range**, and
nights (18:00–06:00) are shaded grey so the day/night rhythm is easy to read.

## Controls

- **Window duration** — how many hours each average spans.
- **Smoothing** — morphs the averaging window from a hard box (0%) to a full
  cosine bell (100%). It's volume-conserving, so smoothing changes the shape, not
  the area under the curve.
- **Assumed breastfeeding milk rate** — ml/min used to estimate volume for breast
  feeds (which record only a duration, not a measured volume).
- **Zoom & pan** — the zoom slider sets the visible window size (labeled as a
  duration, e.g. "3 days"); drag or scroll either chart to pan. Both charts stay
  in sync. Opens on the most recent 3 days.

## Data

Load data with the **Upload CSV/TSV** button, or try the built-in samples
(**Full log**, **Day / night demo**, **Single day**).

Input is a tab- or comma-separated export with a header row. The relevant columns:

| Column | Used for |
| --- | --- |
| `Type` | `Feed` or `Pump` (other rows are ignored) |
| `Start` | timestamp, `YYYY-MM-DD H:MM` (local time) |
| `Duration` | breast-feed length, `H:MM` |
| `Start Location` | `Bottle` or `Breast` (feeds) |
| `End Condition` | bottle volume (feeds) / right-side volume (pumps), e.g. `70ml` |
| `Start Condition` | left-side volume (pumps), e.g. `40ml` |

- **Bottle feeds** use the recorded volume.
- **Breast feeds** estimate volume as `duration × assumed rate`.
- **Pumps** sum the left and right volumes.

## How the rate is computed

Each feed/pump is a point event. For every 15-minute sample, nearby events are
summed with a window weight and divided by the window duration to get an average
ml/hour. The window's effective width is held constant as you change smoothing, so
the area under the curve always equals the total volume consumed, and an isolated
feed reads `volume ÷ window`.
