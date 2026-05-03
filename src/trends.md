---
title: Trends
toc: false
---

```js
import {QUARTERLY, ALL_UNITS, COLORS, sh, fK} from "./components/data.js";
import * as Plot from "npm:@observablehq/plot";
import * as Inputs from "npm:@observablehq/inputs";
```

# Quarterly Trends

<div style="font-size:13px;color:var(--theme-foreground-muted);margin:-0.25rem 0 1.5rem;">
  Created with Claude by <strong>Michael Giberson</strong> &nbsp;·&nbsp;
  <a href="https://x.com/MichaelGiberso3" target="_blank">X/Twitter</a> &nbsp;·&nbsp;
  <a href="https://www.linkedin.com/in/michaelgiberson/" target="_blank">LinkedIn</a>
</div>

Quarterly operating data for all 10 ordered units, November 2022 through March 2026. Dashed vertical lines mark the two waves of 202(c) order dates: **May 2025** (Campbell, Eddystone) and **December 2025** (Schahfer, Culley, Craig, Centralia).

---

```js
const metric = view(Inputs.radio(
  new Map([
    ["Capacity factor (%)",      "cf"],
    ["Generation (MWh)",         "mwh"],
    ["Operating hours",          "oh"],
    ["CO₂ emissions (tons)",    "co2"],
  ]),
  {label: "Metric", value: "cf"}
));
```

```js
const selectedUnits = view(Inputs.checkbox(ALL_UNITS, {
  label: "Units",
  value: ALL_UNITS,
  format: u => sh(u),
}));
```

```js
// Build trend data for selected units + metric
const trendData = QUARTERLY.filter(d => selectedUnits.includes(d.u));
const metricLabel = new Map([
  ["cf","Capacity factor (%)"],["mwh","Generation (MWh)"],
  ["oh","Operating hours"],["co2","CO₂ emissions (tons)"],
]).get(metric);

display(Plot.plot({
  marginBottom: 60, marginLeft: 60, marginRight: 20,
  width: 860, height: 440,
  x: {label: null, tickRotate: -35},
  y: {
    label: metricLabel, grid: true,
    tickFormat: metric === "cf" ? d => d + "%" : fK,
  },
  color: {
    domain: ALL_UNITS,
    range:  ALL_UNITS.map(u => COLORS[u]),
    legend: true,
  },
  marks: [
    Plot.line(trendData, {
      x: "q", y: metric, stroke: "u", strokeWidth: 2,
      tip: true,
      title: d => `${sh(d.u)}\n${d.q}: ${metric === "cf" ? d[metric].toFixed(1)+"%" : fK(d[metric])}`,
    }),
    Plot.dot(trendData, {x: "q", y: metric, fill: "u", r: 3}),
    // Order date reference lines
    Plot.ruleX(["2025Q2"], {stroke: "#ef4444", strokeDasharray: "6,3", strokeWidth: 1.5}),
    Plot.ruleX(["2025Q4"], {stroke: "#f97316", strokeDasharray: "6,3", strokeWidth: 1.5}),
    Plot.text(
      [{q:"2025Q2", label:"← Campbell/Eddystone orders"}],
      {x:"q", y: metric === "cf" ? 100 : null,
       text:"label", fill:"#ef4444", fontSize:10, textAnchor:"start", dx:4, dy:-6}
    ),
    Plot.text(
      [{q:"2025Q4", label:"← Schahfer/Culley/Craig/Centralia orders"}],
      {x:"q", y: metric === "cf" ? 90 : null,
       text:"label", fill:"#f97316", fontSize:10, textAnchor:"start", dx:4, dy:-6}
    ),
  ],
}));
```

<div style="font-size:11px;color:var(--theme-foreground-muted);font-style:italic;margin:.25rem 0 1.5rem;">
Note: Some quarters show near-zero values reflecting failed startup attempts, not sustained shutdowns — notably Centralia 2024 Q2 (31 generating hours from an aborted startup) and Campbell 2 2024 Q1 (9 meaningful generating hours). These visible dips are real operational events, not data artifacts.
</div>

---

## Key findings

```js
display(html`<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));gap:12px;margin:1rem 0 1.5rem;">
${[
  {title:"Pre-existing operational fragility",
   text:"Several units show pre-order quarters of near-zero generation from failed startups, equipment outages, or extended maintenance. These visible dips pre-date the orders and reflect the condition of aging equipment already approaching planned retirement."},
  {title:"Distinct dispatch patterns by region",
   text:"MISO units (Campbell, Schahfer, Culley) show clear seasonal cycles. WECC units (Craig, Centralia) operated more steadily pre-order. Eddystone (PJM) functioned almost entirely as a winter/heat-wave peaker at around 1% capacity factor throughout the entire period."},
  {title:"Order timing maps onto observed shifts",
   text:"The two reference lines align with visible behavior changes for several units: Campbell 2 collapse, Schahfer 18 cessation, Craig C1 going to zero in 2026 Q1. Other units show little visible change post-order, consistent with continued economic dispatch for units that remained price-competitive."},
].map(f => html`<div style="background:var(--theme-background-alt);border-radius:6px;padding:14px 16px;border-left:3px solid var(--theme-foreground-faintest);">
  <div style="font-size:13px;font-weight:700;margin-bottom:6px;">${f.title}</div>
  <div style="font-size:12px;line-height:1.55;color:var(--theme-foreground-muted);">${f.text}</div>
</div>`)}
</div>`);
```

---

### References

```js
display(html`<div style="background:var(--theme-background-alt);border-radius:6px;padding:16px 20px;margin:.5rem 0 1rem;">
<div style="font-size:11px;font-weight:700;text-transform:uppercase;letter-spacing:.1em;color:var(--theme-foreground-muted);margin-bottom:10px;">References — Trends</div>
${[
  ["https://www.energy.gov/ceser/2025-doe-202c-orders","DOE CESER — official 2025 202(c) orders index with effective dates for each order"],
  ["https://www.powermag.com/doe-has-issued-more-than-40-section-202c-emergency-orders-since-may-2025-heres-an-updated-log/","Power Magazine — log of all orders with order timelines and context"],
  ["https://campd.epa.gov/","EPA CAMPD — Clean Air Markets Program Data (primary data source)"],
].map(([url, label]) => html`<div style="display:flex;gap:10px;align-items:flex-start;font-size:12px;line-height:1.5;margin-bottom:6px;">
  <span style="width:8px;height:8px;border-radius:2px;background:#94a3b8;flex-shrink:0;margin-top:4px;display:inline-block;"></span>
  <a href="${url}" target="_blank" style="color:var(--theme-foreground-muted);border-bottom:1px dotted var(--theme-foreground-faintest);">${label}</a>
</div>`)}
</div>`);
```

<div style="padding:12px 16px;background:var(--theme-background-alt);border:1px solid var(--theme-foreground-faintest);border-radius:6px;font-size:11px;color:var(--theme-foreground-muted);line-height:1.6;font-style:italic;">
<strong style="font-style:normal;">Note on AI-generated content:</strong> Key Findings narrative interpretations were drafted with AI assistance. Readers are encouraged to verify specific factual claims against the cited sources.
</div>
