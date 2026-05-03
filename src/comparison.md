---
title: Pre vs Post
toc: false
---

```js
import {PREPOST, ALL_UNITS, sh, fN, fK} from "./components/data.js";
import * as Plot from "npm:@observablehq/plot";
```

# Pre-Order vs Post-Order Comparison

<div style="font-size:13px;color:var(--theme-foreground-muted);margin:-0.25rem 0 1.5rem;">
  Created with Claude by <strong>Michael Giberson</strong> &nbsp;·&nbsp;
  <a href="https://x.com/MichaelGiberso3" target="_blank">X/Twitter</a> &nbsp;·&nbsp;
  <a href="https://www.linkedin.com/in/michaelgiberson/" target="_blank">LinkedIn</a>
</div>

Generation-weighted averages across all hours in each period. Pre-order period: November 2022 through each unit's order effective date. Post-order period: order date through March 31, 2026 (1–4 quarters depending on unit). Campbell and Eddystone ordered May 2025; Schahfer, Culley, Craig, and Centralia ordered December 2025.

---

## Average capacity factor: pre-order vs post-order

```js
const cfData = ALL_UNITS.flatMap(u => {
  const pre  = PREPOST.find(d => d.u === u && d.p === "pre");
  const post = PREPOST.find(d => d.u === u && d.p === "post");
  return [
    {unit: sh(u), period: "Pre-order",  cf: pre?.cf  ?? 0},
    {unit: sh(u), period: "Post-order", cf: post?.cf ?? 0},
  ];
});

display(Plot.plot({
  marginBottom: 80, marginLeft: 50,
  width: 820, height: 380,
  x: {domain: ["Pre-order", "Post-order"], label: null, tickRotate: -4},
  y: {label: "Capacity factor (%)", grid: true},
  fx: {label: null, padding: 0.05},
  color: {domain: ["Pre-order","Post-order"], range: ["#3b82f6","#ef4444"], legend: true},
  marks: [
    Plot.barY(cfData, {
      fx: "unit", x: "period", y: "cf", fill: "period",
      tip: true, title: d => `${d.unit}\n${d.period}: ${d.cf.toFixed(1)}%`,
    }),
    Plot.ruleY([0]),
  ],
}));
```

---

## Average quarterly generation: pre-order vs post-order

MWh per quarter — total period generation divided by quarters in period. Note several units have just one quarter of post-order data. Interpret with caution.

```js
const genData = ALL_UNITS.flatMap(u => {
  const pre  = PREPOST.find(d => d.u === u && d.p === "pre");
  const post = PREPOST.find(d => d.u === u && d.p === "post");
  return [
    {unit: sh(u), period: "Pre-order",  mwh: Math.round(pre.mwh / pre.n)},
    {unit: sh(u), period: "Post-order", mwh: Math.round(post.mwh / post.n)},
  ];
});

display(Plot.plot({
  marginBottom: 80, marginLeft: 60,
  width: 820, height: 380,
  x: {domain: ["Pre-order", "Post-order"], label: null, tickRotate: -40},
  y: {label: "Avg MWh per quarter", grid: true, tickFormat: fK},
  fx: {label: null, padding: 0.05},
  color: {domain: ["Pre-order","Post-order"], range: ["#3b82f6","#ef4444"], legend: true},
  marks: [
    Plot.barY(genData, {
      fx: "unit", x: "period", y: "mwh", fill: "period",
      tip: true, title: d => `${d.unit}\n${d.period}: ${fK(d.mwh)} MWh/quarter avg`,
    }),
    Plot.ruleY([0]),
  ],
}));
```

---

## Key findings by unit

```js
const narratives = [
  {title:"Craig C1", color:"#dc2626",
   text:"Zero post-order generation. Tri-State confirmed a mechanical valve failure on December 19, 2025 — eleven days before the unit's planned retirement and twelve days before the DOE order took effect. The unit was offline before the order was issued; Tri-State stated repairs would be required to comply."},
  {title:"Centralia BW22", color:"#2563eb",
   text:"73 generating hours post-order, 9.0% capacity factor. TransAlta CEO John Kousinioris told analysts the company did not expect the plant to run given that the Pacific Northwest is \"flush\" with hydropower. TransAlta is proceeding with a coal-to-gas conversion planned for late 2028."},
  {title:"Schahfer 18", color:"#67e8f9",
   text:"Zero post-order generation. Schahfer Unit 18 had been offline since July 9, 2025 — nearly six months before the 202(c) order — due to turbine problems. NIPSCO's parent NiSource estimated total continued-operation costs for Schahfer in excess of $1 billion."},
  {title:"Campbell Unit 2", color:"#a855f7",
   text:"Capacity factor dropped from 39.3% pre-order to 4.4% post-order while Units 1 (71.3%) and 3 (54.9%) continued running. Consumers Energy reported $254 million in compliance costs through December, seeking $135 million in cost recovery from MISO ratepayers."},
  {title:"Culley 2", color:"#d97706",
   text:"Capacity factor fell from 29.2% to 15.3%. CenterPoint Energy formally asked DOE not to renew the order in a February 2026 letter, citing $14–18 million in required turbine overhaul costs and describing Culley 2 as \"an inefficient and increasingly unreliable asset.\" DOE renewed over CenterPoint's objection."},
  {title:"Eddystone 3 & 4", color:"#059669",
   text:"Capacity factors of ~1% both pre- and post-order. These oil/gas-fired peakers ran 124 cumulative hours during Winter Storm Fern (January 26–29, 2026), which DOE cited as justification for the February 24, 2026 order extension. State Power Project notes the units produced energy during fewer than 1% of all hours in each of the four years prior to the order."},
];

display(html`<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));gap:12px;margin:1rem 0 1.5rem;">
${narratives.map(f => html`<div style="background:var(--theme-background-alt);border-radius:6px;padding:14px 16px;border-left:3px solid ${f.color};">
  <div style="font-size:13px;font-weight:700;margin-bottom:6px;color:${f.color};">${f.title}</div>
  <div style="font-size:12px;line-height:1.55;color:var(--theme-foreground-muted);">${f.text}</div>
</div>`)}
</div>`);
```

---

### References

```js
const refs = [
  {color:"#dc2626", url:"https://tristate.coop/us-doe-orders-tri-state-keep-craig-generating-station-unit-operating-next-90-days", label:"Tri-State Generation press release (Dec 31, 2025) — confirms broken valve outage Dec 19, 2025"},
  {color:"#dc2626", url:"https://coloradosun.com/2025/12/31/craig-station-emergency-order-coal-power-colorado-trump/", label:"Colorado Sun (Dec 31, 2025) — Craig Unit 1 mechanical valve failure and DOE order details"},
  {color:"#2563eb", url:"https://www.utilitydive.com/news/washington-earthjustice-sue-doe-centralia-emergency-order-transalta/813754/", label:"Utility Dive (March 4, 2026) — TransAlta CEO on Centralia: region \"flush\" with hydropower"},
  {color:"#2563eb", url:"https://washingtonstatestandard.com/2026/03/11/new-targeted-tax-in-wa-aims-to-keep-coal-power-plant-shut-down/", label:"Washington State Standard — TransAlta Centralia 2028 coal-to-gas conversion"},
  {color:"#67e8f9", url:"https://www.canarymedia.com/articles/clean-energy/indiana-retiring-two-coal-plants", label:"Canary Media (Dec 15, 2025) — Schahfer unit offline since July 9, 2025 due to turbine problems"},
  {color:"#67e8f9", url:"https://earthjustice.org/press/2026/groups-challenge-illegal-order-halting-the-retirement-of-indiana-power-plants", label:"Earthjustice (March 16, 2026) — Schahfer/Culley legal challenge, NIPSCO billion-dollar repair estimate"},
  {color:"#a855f7", url:"https://www.utilitydive.com/news/doe-emergency-order-campbell-coal-power-plant-appeal/815387/", label:"Utility Dive (March 23, 2026) — Campbell 39% below prior year, $254M cost"},
  {color:"#a855f7", url:"https://www.circleofblue.org/2026/water-energy/executive-order-puts-oldest-polluting-coal-plants-back-in-action/", label:"Circle of Blue (April 2026) — Campbell operating cost ~$615,000/day"},
  {color:"#d97706", url:"https://www.citact.org/news/centerpoint-urged-federal-government-not-reissue-emergency-order-costly-coal-plant", label:"Citizens Action Coalition — CenterPoint letter opposing Culley Unit 2 renewal, $14–18M repair estimate"},
  {color:"#059669", url:"https://www.energy.gov/documents/eddystone-202c-order-no-202-26-17", label:"DOE Order 202-26-17 (Feb 24, 2026) — Eddystone Winter Storm Fern: 124 cumulative hours"},
  {color:"#059669", url:"https://statepowerproject.org/challenges-to-doe-202c-orders/", label:"State Power Project — Eddystone produced energy <1% of all hours in each of the four prior years"},
];

display(html`<div style="background:var(--theme-background-alt);border-radius:6px;padding:16px 20px;margin:.5rem 0 1rem;">
<div style="font-size:11px;font-weight:700;text-transform:uppercase;letter-spacing:.1em;color:var(--theme-foreground-muted);margin-bottom:10px;">References — Pre vs Post</div>
${refs.map(r => html`<div style="display:flex;gap:10px;align-items:flex-start;font-size:12px;line-height:1.5;margin-bottom:6px;">
  <span style="width:8px;height:8px;border-radius:2px;background:${r.color};flex-shrink:0;margin-top:4px;display:inline-block;"></span>
  <a href="${r.url}" target="_blank" style="color:var(--theme-foreground-muted);border-bottom:1px dotted var(--theme-foreground-faintest);">${r.label}</a>
</div>`)}
</div>`);
```

<div style="padding:12px 16px;background:var(--theme-background-alt);border:1px solid var(--theme-foreground-faintest);border-radius:6px;font-size:11px;color:var(--theme-foreground-muted);line-height:1.6;font-style:italic;">
<strong style="font-style:normal;">Note on AI-generated content:</strong> Key Findings narrative interpretations were drafted with AI assistance based on the underlying CAMPD data and the references above. Readers are encouraged to verify specific factual claims against the cited sources.
</div>
