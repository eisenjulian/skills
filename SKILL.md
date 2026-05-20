# Reveal.js Dark Academic Presentation — Standalone Skill

> A complete, self-contained guide for building polished dark-mode academic
> presentations with **Reveal.js 5.x**, **Chart.js 4.x**, and **KaTeX**.
> All starter files are embedded below — copy them out to bootstrap a new project.

---

## Tech Stack

| Layer | Tool | Version | Loaded via |
|-------|------|---------|------------|
| Slides | Reveal.js | 5.1.0 | CDN |
| Math | KaTeX | 0.16.11 | CDN (Reveal plugin) |
| Charts | Chart.js | 4.4.4 | CDN |
| Code highlighting | highlight.js | (bundled) | Reveal plugin |
| Typography | Inter + JetBrains Mono | latest | Google Fonts |
| Styling | Vanilla CSS | — | Local file |
| Build | Bash (`sed`) | — | Local script |

No `npm install` required — everything is CDN-loaded.

---

## Project Structure

```
my-presentation/
  template.html          ← HTML shell (CDN links, Reveal.init, <!-- SLIDES --> placeholder)
  style.css              ← full design system
  charts.js              ← Chart.js configs with per-slide animation
  build.sh               ← combines template + partials → index.html
  index.html             ← generated output (never edit directly)
  imgs/                  ← figures, logos, diagrams
  slides/
    00-intro.html        ← title slide, motivation, objectives, roadmap
    01-background.html   ← background / related work
    02-chapter-a.html    ← first contribution chapter
    03-chapter-b.html    ← second contribution chapter
    ...
    NN-conclusion.html   ← conclusions, future work, thank-you
```

### Rules
- **`index.html` is generated** — always edit `template.html` or `slides/*.html`, then run `./build.sh`.
- **Partial files contain only `<section>` blocks** — no `<html>`, `<head>`, or `<body>`.
- **One partial per logical chapter** — keeps files at ~100–250 lines each.

---

## Starter Files

### `template.html`

Copy this as your HTML shell. Customise the `<title>`.

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title><!-- YOUR PRESENTATION TITLE --></title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/dist/reveal.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/dist/theme/black.css" id="theme">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/plugin/highlight/monokai.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.11/dist/katex.min.css">
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.4/dist/chart.umd.min.js"></script>
  <link
    href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700;800&family=JetBrains+Mono:wght@400;600&display=swap"
    rel="stylesheet">
  <!-- Optional: QR code generation -->
  <!-- <script src="https://cdn.jsdelivr.net/npm/qrcodejs/qrcode.min.js"></script> -->
  <link rel="stylesheet" href="style.css">
</head>

<body>
  <div class="reveal">
    <div class="slides">

      <!-- SLIDES -->

    </div><!-- end slides -->
  </div><!-- end reveal -->

  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/dist/reveal.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/plugin/math/math.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/plugin/highlight/highlight.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/plugin/notes/notes.js"></script>
  <script src="charts.js"></script>
  <script>
    Reveal.initialize({
      hash: true, width: 1280, height: 720, margin: 0.04,
      transition: 'slide', backgroundTransition: 'fade',
      slideNumber: 'c/t',
      plugins: [RevealMath.KaTeX, RevealHighlight, RevealNotes],
      katex: { local: 'https://cdn.jsdelivr.net/npm/katex@0.16.11' }
    });
    Reveal.on('slidechanged', event => { if (typeof initChartsOnSlide === 'function') initChartsOnSlide(event.currentSlide); });
    Reveal.on('ready', event => { if (typeof initChartsOnSlide === 'function') initChartsOnSlide(event.currentSlide); });
  </script>
</body>

</html>
```

**Reveal.js config explained:**
- `1280×720` — standard 16:9 aspect ratio
- `margin: 0.04` — thin border around slides
- `hash: true` — enables URL-based navigation (`#/3/2`)
- `slideNumber: 'c/t'` — shows "current / total" in the corner
- KaTeX, highlight.js, and speaker notes plugins enabled

### `build.sh`

Auto-discovers all partials in `slides/` sorted by filename. Make executable with `chmod +x build.sh`.

```bash
#!/bin/bash
# Combines template.html + slide partials into index.html
# Usage:
#   ./build.sh         → build index.html
#   ./build.sh pdf     → build + export PDF via DeckTape
set -e

# Auto-discover partials in sorted order
PARTIALS=$(ls slides/*.html 2>/dev/null | sort)

if [ -z "$PARTIALS" ]; then
  echo "⚠️  No slide partials found in slides/"
  exit 1
fi

# Build the sed command to insert all partials at the <!-- SLIDES --> marker
SED_CMD='/<!-- SLIDES -->/{
'
for f in $PARTIALS; do
  SED_CMD+="r $f
"
done
SED_CMD+='d
}'

sed -e "$SED_CMD" template.html > index.html
echo "✅ Built index.html ($(wc -l < index.html | tr -d ' ') lines) from: $PARTIALS"

# Optional PDF export
if [ "$1" = "pdf" ]; then
  echo "📄 Exporting PDF..."
  npx -y serve . &
  SERVER_PID=$!
  sleep 2
  npx -y decktape reveal --size 1280x720 http://localhost:3000/index.html slides.pdf
  kill $SERVER_PID
  echo "✅ Exported slides.pdf"
fi
```

### `charts.js`

Skeleton with the per-slide animation pattern. Add one block per chart.

```javascript
/* ============================================= */
/*  Chart.js — Per-Slide Animation Pattern        */
/*  Charts are destroyed and re-created on each   */
/*  slide transition so they animate on entrance.  */
/* ============================================= */

const chartInstances = {};

/**
 * Destroy all existing chart instances so they can be
 * recreated with a fresh entrance animation.
 */
function destroyAllCharts() {
  for (const key in chartInstances) {
    if (chartInstances[key]) {
      chartInstances[key].destroy();
    }
  }
  Object.keys(chartInstances).forEach(key => delete chartInstances[key]);
}

/** Shared animation options — tweak globally here. */
const ANIM = { duration: 900, easing: 'easeOutQuart' };

/** Dark-theme color tokens (match style.css variables). */
const C = {
  tick:       '#94a3b8',
  grid:       'rgba(255,255,255,0.05)',
  title:      '#e2e8f0',
  legend:     '#94a3b8',
  // Semantic bar colors — use consistently:
  baseline:   'rgba(148,163,184,0.6)',   // slate  — prior work / baselines
  secondary:  'rgba(245,158,11,0.6)',    // amber  — secondary method
  comparison: 'rgba(167,139,250,0.5)',   // violet — comparison model
  ours:       'rgba(16,185,129,0.8)',    // green  — "our" method (most opaque)
  best:       'rgba(56,189,248,0.85)',   // cyan   — best result highlight
  human:      'rgba(52,211,153,0.6)',    // teal   — human performance
};

/**
 * Called on every slide transition.  Only the chart(s)
 * whose <canvas> lives inside `slide` are created;
 * everything else is destroyed so it re-animates later.
 *
 * Add one block per chart:
 *   const el = slide.querySelector('#myChartId');
 *   if (el) { chartInstances.myChart = new Chart(el, { ... }); }
 */
function initChartsOnSlide(slide) {
  if (!slide) return;
  destroyAllCharts();

  // ---- Example: Vertical Bar Chart ----
  // const el = slide.querySelector('#exampleChart');
  // if (el) {
  //   chartInstances.example = new Chart(el, {
  //     type: 'bar',
  //     data: {
  //       labels: ['Model A', 'Model B', 'Ours'],
  //       datasets: [{
  //         label: 'Accuracy (%)',
  //         data: [72.3, 78.1, 85.4],
  //         backgroundColor: [C.baseline, C.comparison, C.ours],
  //         borderRadius: 6
  //       }]
  //     },
  //     options: {
  //       responsive: true,
  //       maintainAspectRatio: false,
  //       animation: ANIM,
  //       scales: {
  //         y: { beginAtZero: false, min: 60, max: 90,
  //              ticks: { color: C.tick }, grid: { color: C.grid } },
  //         x: { ticks: { color: C.tick, font: { size: 11 } },
  //              grid: { display: false } }
  //       },
  //       plugins: {
  //         legend: { display: false },
  //         title: { display: true, text: 'Results',
  //                  color: C.title, font: { size: 14 } }
  //       }
  //     }
  //   });
  // }

  // ---- Example: Horizontal Bar Chart ----
  // const el2 = slide.querySelector('#ablationChart');
  // if (el2) {
  //   chartInstances.ablation = new Chart(el2, {
  //     type: 'bar',
  //     data: { ... },
  //     options: {
  //       responsive: true, maintainAspectRatio: false,
  //       animation: ANIM, indexAxis: 'y',
  //       scales: { ... }, plugins: { ... }
  //     }
  //   });
  // }

  // ---- Example: Grouped Bar Chart ----
  // const el3 = slide.querySelector('#comparisonChart');
  // if (el3) {
  //   chartInstances.comparison = new Chart(el3, {
  //     type: 'bar',
  //     data: {
  //       labels: ['Benchmark A', 'Benchmark B', 'Benchmark C'],
  //       datasets: [
  //         { label: 'Baseline', data: [41, 64, 23],
  //           backgroundColor: C.baseline, borderRadius: 4 },
  //         { label: 'Prior Work', data: [56, 73, 24],
  //           backgroundColor: C.secondary, borderRadius: 4 },
  //         { label: 'Ours', data: [64, 92, 26],
  //           backgroundColor: C.ours, borderRadius: 4 }
  //       ]
  //     },
  //     options: {
  //       responsive: true, maintainAspectRatio: false,
  //       animation: ANIM,
  //       scales: {
  //         y: { beginAtZero: true, max: 100,
  //              ticks: { color: C.tick }, grid: { color: C.grid } },
  //         x: { ticks: { color: C.tick }, grid: { display: false } }
  //       },
  //       plugins: {
  //         legend: { position: 'top',
  //                   labels: { color: C.legend, font: { size: 11 } } },
  //         title: { display: true, text: 'Model Comparison',
  //                  color: C.title, font: { size: 14 } }
  //       }
  //     }
  //   });
  // }
}
```

**Chart.js + Reveal.js rules:**
1. `maintainAspectRatio: false` — let CSS `.chart-container` control height
2. `.chart-container` must use a fixed `height` (not `max-height`)
3. Side-by-side charts: `.columns .chart-container` uses 260px
4. `.col { min-width: 0 }` — prevents flex overflow with canvas elements
5. Use `slide.querySelector()` not `document.getElementById()` — scopes to current slide
6. Destroy all charts on every slide change — triggers fresh entrance animation

### `style.css`

Full design system. Customise the `:root` variables first.

```css
/* =========================================== */
/*  Reveal.js Dark Academic — Design System     */
/* =========================================== */

:root {
  /* ---- Core palette (customise per presentation) ---- */
  --accent:       #38bdf8;                /* primary accent (sky-400)     */
  --accent2:      #a78bfa;                /* secondary accent (violet-400)*/
  --success:      #34d399;                /* positive results (emerald)   */
  --warn:         #fb923c;                /* warnings (orange-400)        */

  /* ---- Background & text (dark theme) ---- */
  --bg:           #0f172a;                /* slate-900                    */
  --bg-card:      rgba(255,255,255,0.06);
  --bg-card-accent: rgba(56,189,248,0.10);
  --text:         #e2e8f0;                /* slate-200                    */
  --text-dim:     #94a3b8;                /* slate-400                    */
  --text-bright:  #f8fafc;                /* slate-50                     */

  /* ---- Branding (optional — university/org colors) ---- */
  --brand-primary:   #004b87;             /* e.g. university blue         */
  --brand-secondary: #cfb87c;             /* e.g. university gold         */

  /* ---- Chapter / topic colors (add/remove as needed) ---- */
  --ch-a: #3b82f6;    /* blue   */
  --ch-b: #10b981;    /* green  */
  --ch-c: #f59e0b;    /* amber  */
  --ch-d: #a78bfa;    /* violet */
  --ch-e: #ec4899;    /* pink   */
}

/* ---- Base ---- */
.reveal {
  font-family: 'Inter', sans-serif;
  font-size: 22px;
  color: var(--text);
}

.reveal h1, .reveal h2, .reveal h3, .reveal h4 {
  font-family: 'Inter', sans-serif;
  font-weight: 700;
  color: var(--text-bright);
  text-transform: none;
  letter-spacing: -0.02em;
}

.reveal h2 { font-size: 1.6em; margin-bottom: 0.5em; }
.reveal h4 { font-size: 1em;   margin-bottom: 0.3em; }

/* ----------- Title Slide ----------- */
.title-slide {
  text-align: center; display: flex; flex-direction: column;
  align-items: center; justify-content: center; height: 100%;
}

.title-logo { width: 80px; margin-bottom: 1em; }

.title-main {
  font-size: 2em !important; line-height: 1.2;
  background: linear-gradient(135deg, var(--accent), var(--accent2));
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  background-clip: text; margin-bottom: 0.3em;
}

.thank-you        { font-size: 3em !important; }
.title-subtitle   { font-size: 1.1em; color: var(--text-dim); margin: 0.2em 0; }
.title-author     { font-size: 1.3em; color: var(--text-bright); font-weight: 600; margin: 0.5em 0 0.1em; }
.title-affiliation { color: var(--text-dim); font-size: 0.95em; }
.title-date       { color: var(--brand-secondary); font-size: 0.9em; margin-top: 0.5em; }

/* ----------- Section Titles ----------- */
.section-title {
  font-size: 2.2em !important;
  background: linear-gradient(135deg, var(--accent), var(--accent2));
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  background-clip: text;
}
.section-chapter { color: var(--text-dim);        font-size: 1.1em; }
.section-papers  { color: var(--brand-secondary); font-size: 0.85em; margin-top: 0.3em; }

/* ----------- Subtitle ----------- */
.subtitle { color: var(--text-dim); font-size: 0.95em; margin-top: -0.3em; margin-bottom: 0.8em; }

/* ----------- Columns ----------- */
.columns { display: flex; gap: 1.5em; align-items: flex-start; margin-top: 0.5em; }
.col     { flex: 1; min-width: 0; }

/* ----------- Card Boxes ----------- */
.info-box, .method-box, .approach-box, .example-box {
  background: var(--bg-card); border-radius: 12px;
  padding: 0.8em 1em; margin-bottom: 0.6em;
  border: 1px solid rgba(255,255,255,0.08);
}
.info-box.accent, .method-box.accent {
  background: var(--bg-card-accent);
  border-color: rgba(56,189,248,0.3);
}
.info-box h4    { margin-top: 0; }
.info-box p,
.method-box p   { font-size: 0.85em; color: var(--text-dim); margin: 0.2em 0; }
.approach-box   { text-align: center; padding: 1.2em; }
.example-box    { background: rgba(167,139,250,0.08); border-color: rgba(167,139,250,0.3); font-size: 0.85em; }

/* ----------- Objectives ----------- */
.objective-list { display: flex; flex-direction: column; gap: 0.6em; margin-top: 0.5em; }
.objective {
  display: flex; align-items: flex-start; gap: 0.8em;
  background: var(--bg-card); border-radius: 12px;
  padding: 0.8em 1em; border: 1px solid rgba(255,255,255,0.08);
}
.obj-num {
  background: linear-gradient(135deg, var(--accent), var(--accent2));
  color: var(--bg); font-weight: 800;
  width: 36px; height: 36px; display: flex;
  align-items: center; justify-content: center;
  border-radius: 50%; flex-shrink: 0; font-size: 1em;
}
.objective h4 { margin: 0; text-align: left; }
.objective p  { font-size: 0.85em; color: var(--text-dim); margin: 0.15em 0 0; }

/* ----------- Roadmap ----------- */
.roadmap {
  display: flex; align-items: center; justify-content: center;
  gap: 0.4em; margin: 1.5em auto; flex-wrap: wrap;
}
.roadmap-item {
  padding: 0.5em 0.8em; border-radius: 8px;
  font-size: 0.8em; text-align: center; min-width: 90px;
}
.roadmap-item span { display: block; font-weight: 700; font-size: 0.8em; opacity: 0.7; }
.roadmap-arrow { color: var(--text-dim); font-size: 1.2em; }
.roadmap-note  { color: var(--text-dim); font-size: 0.85em; text-align: center; margin-top: 1em; }

/* Chapter-colored items (roadmap + mini-roadmap) — add more as needed */
.ch-a, .ch-prelim { background: rgba(148,163,184,0.15); border: 1px solid rgba(148,163,184,0.3); }
.ch-b, .ch-tables { background: rgba(59,130,246,0.15);  border: 1px solid rgba(59,130,246,0.3); }
.ch-c, .ch-charts { background: rgba(16,185,129,0.15);  border: 1px solid rgba(16,185,129,0.3); }
.ch-d, .ch-uncert { background: rgba(245,158,11,0.15);  border: 1px solid rgba(245,158,11,0.3); }
.ch-e, .ch-vocab  { background: rgba(167,139,250,0.15); border: 1px solid rgba(167,139,250,0.3); }
.ch-f             { background: rgba(236,72,153,0.15);  border: 1px solid rgba(236,72,153,0.3); }

/* Mini roadmap (compact chapter progress on section title slides) */
.mini-roadmap { display: flex; gap: 0.5em; justify-content: center; margin-top: 1.5em; }
.mini-roadmap span { padding: 0.2em 0.7em; border-radius: 6px; font-size: 0.65em; opacity: 0.3; }
.mini-roadmap span.active { opacity: 1; font-weight: 700; }

/* ----------- Architecture Grid ----------- */
.arch-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 0.8em; margin-top: 0.8em; }
.arch-card {
  background: var(--bg-card); border-radius: 12px; padding: 0.8em;
  border: 1px solid rgba(255,255,255,0.08); text-align: center;
}
.arch-card.highlight { border-color: var(--accent); background: var(--bg-card-accent); }
.model-name { color: var(--accent); font-weight: 700; font-size: 1.1em; }
.arch-card p { font-size: 0.8em; color: var(--text-dim); }

/* ----------- Recipe / Pipeline Flow ----------- */
.recipe-flow {
  display: flex; align-items: center; justify-content: center;
  gap: 1em; margin: 1em 0;
}
.recipe-step {
  background: var(--bg-card); border-radius: 12px;
  padding: 1em 1.5em; flex: 1;
  border: 1px solid rgba(255,255,255,0.08);
}
.recipe-arrow { font-size: 2em; color: var(--text-dim); }
.recipe-step .small    { font-size: 0.8em; color: var(--text-dim); }
.recipe-step .emphasis { color: var(--accent); font-weight: 600; }

/* ----------- Figures ----------- */
.reveal section img {
  display: block; margin-left: auto; margin-right: auto;
  border: none; box-shadow: none;
}
.reveal section img.fig-small {
  max-width: 45% !important; max-height: 200px !important;
  margin: 0.5em auto; border-radius: 8px; background: #fff;
  padding: 0.4em; box-shadow: 0 2px 12px rgba(0,0,0,0.3) !important;
}
.reveal section img.fig-medium {
  max-width: 50% !important; max-height: 260px !important;
  margin: 0.5em auto; border-radius: 8px; background: #fff;
  padding: 0.5em; box-shadow: 0 2px 12px rgba(0,0,0,0.3) !important;
}
.reveal section img.fig-large {
  max-width: 75% !important; max-height: 300px !important;
  margin: 0.5em auto; border-radius: 8px; background: #fff;
  padding: 0.5em; box-shadow: 0 2px 12px rgba(0,0,0,0.3) !important;
}
.reveal section img.fig-xlarge {
  max-width: 88% !important; max-height: 370px !important;
  margin: 0.5em auto; border-radius: 8px; background: #fff;
  padding: 0.5em; box-shadow: 0 2px 12px rgba(0,0,0,0.3) !important;
}
.reveal section img.fig-fill {
  width: 100% !important; max-height: 240px !important;
  object-fit: contain; border-radius: 8px; background: #fff;
  padding: 0.4em; box-shadow: 0 2px 12px rgba(0,0,0,0.3) !important;
}
.fig-caption { text-align: center; font-size: 0.75em; color: var(--text-dim); margin-top: 0.3em; }
.title-logo  { background: transparent !important; padding: 0 !important; box-shadow: none !important; }

/* ----------- Charts ----------- */
.chart-container { position: relative; max-width: 750px; height: 340px; margin: 0.5em auto; }
.columns .chart-container { height: 260px; }

/* ----------- Results ----------- */
.result-highlight {
  text-align: center; font-weight: 700;
  color: var(--success); font-size: 1em; margin-top: 0.5em;
}

/* ----------- Takeaway Box ----------- */
.takeaway {
  background: var(--bg-card); border-radius: 12px;
  padding: 1em 1.5em; border-left: 4px solid var(--accent);
  margin-top: 0.8em;
}
.takeaway ul { margin: 0; }
.takeaway li { margin-bottom: 0.3em; }

/* ----------- Blockquote ----------- */
.reveal blockquote {
  background: rgba(167,139,250,0.08);
  border-left: 4px solid var(--accent2);
  padding: 0.6em 1em; border-radius: 0 8px 8px 0;
  font-style: italic; color: var(--text);
  width: auto; margin: 0.8em auto; font-size: 0.9em;
}

/* ----------- Learning Grid ----------- */
.learning-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 0.8em; margin-top: 0.5em; }
.learning {
  background: var(--bg-card); border-radius: 12px;
  padding: 0.8em 1em; border: 1px solid rgba(255,255,255,0.08);
}
.learning h4 { font-size: 0.9em; margin: 0 0 0.2em; }
.learning p  { font-size: 0.8em; color: var(--text-dim); margin: 0; }

/* ----------- Big Picture ----------- */
.big-picture { display: flex; flex-direction: column; align-items: center; gap: 0.5em; margin: 1em 0; }
.bp-row { display: flex; gap: 1em; }
.bp-box {
  padding: 0.8em 1.2em; border-radius: 12px;
  text-align: center; min-width: 240px; font-weight: 600;
}
.bp-box span { font-size: 0.75em; font-weight: 400; color: var(--text-dim); display: block; margin-top: 0.2em; }
.bp-center {
  font-size: 1.2em; font-weight: 800;
  background: linear-gradient(135deg, var(--accent), var(--accent2));
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; padding: 0.3em;
}

/* ----------- Papers List ----------- */
.papers-list { display: flex; flex-direction: column; gap: 0.35em; margin-top: 0.5em; }
.paper {
  background: var(--bg-card); border-radius: 8px;
  padding: 0.4em 0.8em; font-size: 0.78em;
  display: flex; align-items: center; gap: 0.8em;
  border: 1px solid rgba(255,255,255,0.05);
}
.venue {
  background: linear-gradient(135deg, var(--accent), var(--accent2));
  color: var(--bg); font-weight: 700; font-size: 0.8em;
  padding: 0.15em 0.5em; border-radius: 4px; white-space: nowrap;
}
.paper a       { color: var(--text); text-decoration: none; transition: color 0.2s; }
.paper a:hover { color: var(--accent); text-decoration: underline; }

/* ----------- Lists ----------- */
.reveal ul       { margin-left: 0.8em; }
.reveal li       { margin-bottom: 0.3em; font-size: 0.9em; }
.reveal li.emphasis { color: var(--accent); }
.small           { font-size: 0.8em; color: var(--text-dim); }

/* ----------- Award Badge ----------- */
.award-badge {
  display: inline-block;
  background: linear-gradient(135deg, var(--brand-secondary), #e6c84a);
  color: var(--bg); font-weight: 700; font-size: 0.8em;
  padding: 0.2em 0.7em; border-radius: 6px; margin-top: 0.3em;
}

/* ----------- Slide Numbers ----------- */
.reveal .slide-number { font-size: 14px; color: var(--text-dim); }
```

---

## Slide Patterns

Every slide partial is a sequence of `<section>` blocks. Start each partial file with an identifying comment:

```html
<!-- ============================================================ -->
<!-- CHAPTER 2: Background & Related Work                          -->
<!-- ============================================================ -->
```

### Title Slide

```html
<section>
  <div class="title-slide">
    <img src="imgs/logo.png" class="title-logo" alt="Logo">
    <h1 class="title-main">Presentation Title<br>Second Line</h1>
    <p class="title-subtitle">Conference / Defense / Seminar</p>
    <p class="title-author">Author Name</p>
    <p class="title-affiliation">University / Lab</p>
    <p class="title-date">Date</p>
  </div>
  <aside class="notes">Welcome. Today I'll present...</aside>
</section>
```

### Section Title (chapter divider)

```html
<section>
  <h2 class="section-title">Chapter Title<br>Second Line</h2>
  <p class="section-chapter">Chapter N</p>
  <div class="mini-roadmap">
    <span class="ch-a">Topic A</span>
    <span class="ch-b active">Topic B</span>
    <span class="ch-c">Topic C</span>
  </div>
  <aside class="notes">Brief chapter intro.</aside>
</section>
```

The `active` class highlights the current chapter. Define chapter colors in CSS.

### Two-Column Content

```html
<section>
  <h2>Slide Title</h2>
  <div class="columns">
    <div class="col">
      <!-- Left: image, list, method-box, etc. -->
    </div>
    <div class="col">
      <!-- Right: complementary content -->
    </div>
  </div>
  <aside class="notes">On the left... On the right...</aside>
</section>
```

### Method / Info Box

```html
<div class="method-box">
  <h4>Method Name</h4>
  <p>Description of the method.</p>
</div>

<div class="method-box accent">     <!-- highlighted variant -->
  <h4>Our Approach</h4>
  <p>What makes it different.</p>
</div>
```

Other box variants: `.info-box`, `.approach-box`, `.example-box`.

### Results Slide with Chart

```html
<section>
  <h2>Results Title</h2>
  <div class="chart-container">
    <canvas id="uniqueChartId"></canvas>
  </div>
  <p class="fragment result-highlight">Key takeaway: our method achieves X%</p>
  <aside class="notes">
    Looking at the chart, [describe specific bars/values]...
    [>>>]
    The takeaway is...
  </aside>
</section>
```

### Figure Slide

```html
<section>
  <h2>Architecture Overview</h2>
  <img src="imgs/architecture.png" class="fig-large" alt="Architecture diagram">
  <p class="fig-caption">Figure 1: System architecture</p>
  <aside class="notes">This figure shows...</aside>
</section>
```

Figure size classes: `.fig-small` (45%), `.fig-medium` (50%), `.fig-large` (75%), `.fig-xlarge` (88%), `.fig-fill` (100%).

### Objectives / Numbered List

```html
<div class="objective-list">
  <div class="objective fragment">
    <div class="obj-num">1</div>
    <div>
      <h4>First Objective</h4>
      <p>Description of the objective.</p>
    </div>
  </div>
  <div class="objective fragment">
    <div class="obj-num">2</div>
    <div>
      <h4>Second Objective</h4>
      <p>Description of the objective.</p>
    </div>
  </div>
</div>
```

### Roadmap / Progress Bar

```html
<div class="roadmap">
  <div class="roadmap-item ch-a"><span>Ch. 2</span>Topic A</div>
  <div class="roadmap-arrow">→</div>
  <div class="roadmap-item ch-b"><span>Ch. 3</span>Topic B</div>
  <div class="roadmap-arrow">→</div>
  <div class="roadmap-item ch-c"><span>Ch. 4</span>Topic C</div>
</div>
<p class="roadmap-note">Each chapter addresses a different aspect of the problem.</p>
```

### Takeaway Box

```html
<div class="takeaway fragment">
  <strong>Key Takeaways:</strong>
  <ul>
    <li><strong>Finding 1:</strong> description.</li>
    <li><strong>Finding 2:</strong> description.</li>
  </ul>
</div>
```

### Blockquote

```html
<blockquote class="fragment">
  "What I cannot create, I do not understand."
  <br><span class="small">— Richard Feynman</span>
</blockquote>
```

### Fragments (progressive reveal)

```html
<li class="fragment">Appears on click</li>
<div class="columns fragment">...</div>
<p class="fragment result-highlight">Key result</p>
```

### QR Codes

Uncomment the QRCode.js CDN in `template.html`, then:

```html
<div id="qr-demo" style="display:inline-block;"></div>
```

Generate in `initChartsOnSlide`:

```javascript
const qrEl = slide.querySelector('#qr-demo');
if (qrEl && !qrEl.hasChildNodes()) {
  new QRCode(qrEl, {
    text: 'https://example.com',
    width: 180, height: 180,
    colorDark: '#e2e8f0',       // light dots on dark background
    colorLight: '#00000000',    // transparent background
    correctLevel: QRCode.CorrectLevel.M
  });
}
```

### Papers List

```html
<div class="papers-list">
  <div class="paper">
    <span class="venue">EMNLP 2024</span>
    <a href="#">Paper title here</a>
  </div>
  <div class="paper">
    <span class="venue">ACL 2023</span>
    <a href="#">Another paper title</a>
  </div>
</div>
```

### Pipeline / Recipe Flow

```html
<div class="recipe-flow">
  <div class="recipe-step">
    <h4>Step 1</h4>
    <p class="small">Description</p>
  </div>
  <div class="recipe-arrow">→</div>
  <div class="recipe-step">
    <h4>Step 2</h4>
    <p class="small">Description</p>
  </div>
</div>
```

---

## Speaker Notes

### Format

```html
<aside class="notes">
  Main talking points for this slide.
  Reference visual elements: "As you can see in the chart on the left..."

  [>>>]
  Text after advancing to the next fragment.

  [>>>]
  Text after the second fragment advance.
</aside>
```

### Rules

1. **Every `[>>>]` must match a `fragment`** on the slide — one per fragment, in order.
2. **Reference visual elements explicitly**: "On the left…", "In the chart…", "As the figure shows…"
3. **For chart slides**: mention specific data values, bar comparisons, or metric names.
4. **For two-column slides**: walk through left then right, referencing positions.
5. **Never add `[>>>]` without a matching fragment**.
6. **Don't reference concepts before they've been introduced** in earlier slides.

Press **`S`** during the presentation to open the speaker notes window.

---

## CSS Class Reference

| Class | Purpose |
|-------|---------|
| `.title-slide` | Centered title page layout |
| `.title-main` | Gradient text for presentation title |
| `.section-title` | Large gradient chapter title |
| `.section-chapter` | "Chapter N" label |
| `.mini-roadmap` | Compact chapter progress bar |
| `.columns` / `.col` | Flexbox two-column layout |
| `.method-box` | Glass-card content box |
| `.method-box.accent` | Highlighted card variant |
| `.info-box` / `.approach-box` / `.example-box` | Other card variants |
| `.objective-list` / `.objective` / `.obj-num` | Numbered objective cards |
| `.roadmap` / `.roadmap-item` / `.roadmap-arrow` | Full progress navigation |
| `.chart-container` | Fixed-height chart wrapper (340px; 260px in columns) |
| `.result-highlight` | Green bold key result text |
| `.takeaway` | Bordered chapter summary box |
| `.fig-small/medium/large/xlarge/fill` | Image sizing with white bg + shadow |
| `.fig-caption` | Image caption |
| `.papers-list` / `.paper` / `.venue` | Publication list with venue badges |
| `.arch-grid` / `.arch-card` | 4-column architecture grid |
| `.learning-grid` / `.learning` | 2-column findings grid |
| `.recipe-flow` / `.recipe-step` / `.recipe-arrow` | Pipeline flow |
| `.big-picture` / `.bp-box` / `.bp-center` | Overview diagram |
| `.award-badge` | Gold gradient achievement badge |
| `.model-name` | Accent-colored model/method name |
| `.small` | Dimmed small text |
| `.subtitle` | Slide subtitle |

---

## PDF Export

### Option 1: Browser Print (built-in)

Append `?print-pdf` to the URL, then **File → Print → Save as PDF**:
```
http://localhost:8000/index.html?print-pdf
```
Settings: Landscape, No margins, Background graphics ✅ enabled.

> **Caveat:** Chart.js canvases all render at once with `?print-pdf`, so only the
> last slide's charts appear. Use DeckTape instead for presentations with charts.

### Option 2: DeckTape (best quality)

```bash
npx -y decktape reveal --size 1280x720 http://localhost:8000/index.html slides.pdf
```

DeckTape navigates sequentially in headless Chrome, so per-slide charts work correctly.

Or use the built-in `./build.sh pdf` command, which starts a server, exports, and cleans up.

---

## Editing Guidelines

### For AI editors
- **Edit one partial at a time** — never load `index.html` for editing.
- **Run `./build.sh`** after editing partials to regenerate `index.html`.
- **When editing notes**: check the slide's visual elements to ensure notes reference them.
- **When adding charts**: add `<canvas>` in the partial **and** the config in `charts.js`.
- **When reordering slides**: check speaker notes for forward references.

### For human editors
- Edit `slides/*.html` directly — each is ~100–250 lines.
- Preview: `./build.sh && open index.html` (or use a local server).
- Speaker notes: press **S** during the presentation.
- `build.sh` auto-discovers partials in sorted order — use numeric prefixes (`00-`, `01-`, …).

---

## Quick-Start Checklist

1. Create a new directory for the presentation.
2. Before starting, iterate on the outline in markdown form, until you are happy with it, and only then generate the slides.
3. Copy `template.html`, `build.sh`, `charts.js`, and `style.css` from the blocks above.
4. Customise `:root` variables in `style.css` (accent colors, branding, chapter colors).
5. Set the `<title>` in `template.html`.
6. Create `slides/00-intro.html` with a title slide.
7. Add chapter partials (`01-*.html`, `02-*.html`, …) using the patterns above.
8. Add any chart configs in `charts.js`.
9. Run `chmod +x build.sh && ./build.sh`, or `./build.sh pdf` if you want a PDF.
10. Open `index.html` in a browser.
