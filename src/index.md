---
title: Overview
toc: false
---

```js
import {PREPOST, ALL_UNITS, sh, fN, fK, unitStatus} from "./components/data.js";
import * as Plot from "npm:@observablehq/plot";
```

# FPA Section 202(c) Unit Operations Tracker

<div style="font-size:13px;color:var(--theme-foreground-muted);margin:-0.25rem 0 1.5rem;">
  Created with Claude by <strong>Michael Giberson</strong> &nbsp;·&nbsp;
  <a href="https://x.com/MichaelGiberso3" target="_blank">X/Twitter</a> &nbsp;·&nbsp;
  <a href="https://www.linkedin.com/in/michaelgiberson/" target="_blank">LinkedIn</a>
</div>

Beginning in May 2025, the U.S. Department of Energy invoked Section 202(c) of the Federal Power Act to issue emergency orders preventing the planned retirements of coal and gas-fired generating units at six facilities across five states. This tracker analyzes hourly EPA CAMPD data for all 10 ordered units — November 2022 through March 2026 — to examine how these units operated before and after the orders took effect.

---

```js
// ── Summary stat cards ──────────────────────────────────────────────────────
const postRows = PREPOST.filter(d => d.p === "post");
const stats = [
  {label:"Zero-generation units post-order", value: postRows.filter(d => d.mwh === 0).length,          sub:"of 10 ordered units"},
  {label:"Low CF units (<10%) post-order",   value: postRows.filter(d => d.cf < 10 && d.mwh > 0).length, sub:"of 10 ordered units"},
  {label:"Total post-order generation",      value: fK(postRows.reduce((s,d) => s+d.mwh, 0))+" MWh",  sub:"across all ordered units"},
  {label:"Total post-order CO₂",            value: fK(postRows.reduce((s,d) => s+d.co2, 0))+" tons",  sub:"across all ordered units"},
];

display(html`<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(155px,1fr));gap:12px;margin:1rem 0 1.5rem;">
${stats.map(c => html`<div style="background:var(--theme-background-alt);border-left:3px solid #3b82f6;border-radius:6px;padding:14px 16px;">
  <div style="font-size:11px;text-transform:uppercase;letter-spacing:.08em;color:var(--theme-foreground-muted);margin-bottom:6px;">${c.label}</div>
  <div style="font-size:24px;font-weight:800;">${c.value}</div>
  <div style="font-size:11px;color:var(--theme-foreground-faint);margin-top:3px;">${c.sub}</div>
</div>`)}
</div>`);
```

---

## Capacity factor: pre-order vs post-order

Generation-weighted averages across all hours in each period. Craig C1 and Schahfer 18 have zero post-order generation.

```js
// ── Grouped bar chart — pre vs post CF ──────────────────────────────────────
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
  x: {domain: ["Pre-order", "Post-order"], label: null, tickRotate: -40},
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

## Unit status summary

```js
// ── Status table ─────────────────────────────────────────────────────────────
display(html`<div style="overflow-x:auto;margin:1rem 0;">
<table style="width:100%;border-collapse:collapse;font-size:13px;">
<thead><tr style="border-bottom:2px solid var(--theme-foreground-faintest);">
  ${["Unit","Nameplate MW","Pre CF","Post CF","Change","Post MWh","Status"].map(h =>
    html`<th style="text-align:left;padding:8px 10px;color:var(--theme-foreground-muted);font-weight:600;white-space:nowrap;">${h}</th>`)}
</tr></thead>
<tbody>
${ALL_UNITS.map(u => {
  const pre  = PREPOST.find(d => d.u === u && d.p === "pre");
  const post = PREPOST.find(d => d.u === u && d.p === "post");
  const chg  = post.cf - pre.cf;
  const st   = unitStatus(post.cf, post.mwh);
  return html`<tr style="border-bottom:1px solid var(--theme-foreground-faintest);">
    <td style="padding:8px 10px;font-weight:600;white-space:nowrap;">${sh(u)}</td>
    <td style="padding:8px 10px;">${pre.np}</td>
    <td style="padding:8px 10px;">${pre.cf.toFixed(1)}%</td>
    <td style="padding:8px 10px;font-weight:700;">${post.cf.toFixed(1)}%</td>
    <td style="padding:8px 10px;font-weight:600;color:${chg < 0 ? "#ef4444" : "#22c55e"};">${chg > 0 ? "+" : ""}${chg.toFixed(1)}pp</td>
    <td style="padding:8px 10px;">${fN(post.mwh)}</td>
    <td style="padding:8px 10px;"><span style="background:${st.color}22;color:${st.color};padding:2px 8px;border-radius:4px;font-size:11px;font-weight:700;">${st.label}</span></td>
  </tr>`;
})}
</tbody></table></div>`);
```

---

## Key findings

```js
display(html`<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));gap:12px;margin:1rem 0 1.5rem;">
${[
  {title:"Most ordered units are barely running",
   text:"Of 10 units under 202(c) emergency orders, 2 had zero post-order generation as of Q1 2026 (Craig C1, Schahfer 18). Several others operate at small fractions of their pre-order capacity factors. The orders maintain capacity availability on paper, but the units are largely not dispatching."},
  {title:"Campbell continues, but at reduced output",
   text:"Campbell Units 1 and 3 maintained relatively high post-order generation; Unit 2 effectively idled. Across the plant, post-order generation through December was 39% below the prior-year period. Consumers Energy reported $254M in compliance costs, seeking $135M from MISO ratepayers."},
  {title:"Cost recovery and ratepayer concerns are central",
   text:"Independent analyses by Grid Strategies and Synapse Energy Economics estimate compliance costs in the tens of millions per quarter for several plants. Tri-State (Craig), CenterPoint (Culley), and NIPSCO (Schahfer) have each raised cost and reliability concerns publicly."},
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
<div style="font-size:11px;font-weight:700;text-transform:uppercase;letter-spacing:.1em;color:var(--theme-foreground-muted);margin-bottom:10px;">References — Overview</div>
${[
  ["https://www.powermag.com/doe-has-issued-more-than-40-section-202c-emergency-orders-since-may-2025-heres-an-updated-log/","Power Magazine — comprehensive log of all Section 202(c) orders since May 2025"],
  ["https://www.energy.gov/ceser/2025-doe-202c-orders","DOE CESER — official 2025 202(c) orders index"],
  ["https://statepowerproject.org/challenges-to-doe-202c-orders/","State Power Project — log of legal challenges to DOE 202(c) orders"],
  ["https://www.utilitydive.com/news/doe-emergency-order-campbell-coal-power-plant-appeal/815387/","Utility Dive (March 2026) — Campbell generation 39% below prior year, $254M cost"],
].map(([url, label]) => html`<div style="display:flex;gap:10px;align-items:flex-start;font-size:12px;line-height:1.5;margin-bottom:6px;">
  <span style="width:8px;height:8px;border-radius:2px;background:#94a3b8;flex-shrink:0;margin-top:4px;display:inline-block;"></span>
  <a href="${url}" target="_blank" style="color:var(--theme-foreground-muted);border-bottom:1px dotted var(--theme-foreground-faintest);">${label}</a>
</div>`)}
</div>`);
```

<div style="padding:12px 16px;background:var(--theme-background-alt);border:1px solid var(--theme-foreground-faintest);border-radius:6px;font-size:11px;color:var(--theme-foreground-muted);line-height:1.6;font-style:italic;">
<strong style="font-style:normal;">Note on AI-generated content:</strong> Key Findings narrative interpretations were drafted with AI assistance based on the underlying CAMPD data and the references above. Readers are encouraged to verify specific factual claims against the cited sources. Quantitative values are derived directly from the EPA CAMPD hourly dataset; interpretive framing reflects judgments that should be independently checked.
</div>
