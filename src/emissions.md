---
title: Emissions Intensity
toc: false
---

```js
import {EPP, EQ, ALL_UNITS, COLORS, sh, fN, fK} from "./components/data.js";
import * as Plot from "npm:@observablehq/plot";
import * as Inputs from "npm:@observablehq/inputs";
```

# Emissions Intensity

<div style="font-size:13px;color:var(--theme-foreground-muted);margin:-0.25rem 0 1.5rem;">
  Created with Claude by <strong>Michael Giberson</strong> &nbsp;·&nbsp;
  <a href="https://x.com/MichaelGiberso3" target="_blank">X/Twitter</a> &nbsp;·&nbsp;
  <a href="https://www.linkedin.com/in/michaelgiberson/" target="_blank">LinkedIn</a>
</div>

Per-MWh emissions rates before and after 202(c) orders, across three pollutants. Rates are generation-weighted across all hours the unit was producing electricity. **No control technology changes were observed for any unit during the data period** — rate shifts reflect operational patterns (output level, cycling frequency), not equipment changes.

---

```js
const pollutant = view(Inputs.radio(
  new Map([
    ["CO₂ (lb/MWh)", "co2r"],
    ["SO₂ (lb/MWh)", "so2r"],
    ["NOx (lb/MWh)", "noxr"],
  ]),
  {label: "Pollutant", value: "co2r"}
));
```

```js
const pLabel = new Map([["co2r","CO₂ (lb/MWh)"],["so2r","SO₂ (lb/MWh)"],["noxr","NOx (lb/MWh)"]]).get(pollutant);
const pKeyP  = pollutant.replace("r","rP");
const fmt3   = pollutant === "co2r" ? d => fN(d) : d => d.toFixed(3);

// Build bar data
const emisData = EPP.flatMap(d => {
  const rows = [{unit: sh(d.u), period: "Pre-order", val: d[pollutant]}];
  if (d[pKeyP] !== null) rows.push({unit: sh(d.u), period: "Post-order", val: d[pKeyP]});
  return rows;
});

display(Plot.plot({
  marginBottom: 80, marginLeft: 60,
  width: 820, height: 380,
  x: {domain: ["Pre-order", "Post-order"], label: null, tickRotate: -40},
  y: {label: pLabel, grid: true, tickFormat: pollutant === "co2r" ? d => fN(d) : d => d.toFixed(1)},
  fx: {label: null, padding: 0.05},
  color: {domain: ["Pre-order","Post-order"], range: ["#3b82f6","#ef4444"], legend: true},
  marks: [
    Plot.barY(emisData, {
      fx: "unit", x: "period", y: "val", fill: "period",
      tip: true, title: d => `${d.unit}\n${d.period}: ${fmt3(d.val)} ${pLabel}`,
    }),
    Plot.ruleY([0]),
  ],
}));
```

<div style="font-size:11px;color:var(--theme-foreground-muted);font-style:italic;margin:.25rem 0 1.5rem;">
Units with zero post-order generation (Craig C1, Schahfer 18) appear with pre-order bars only. Centralia post-order is based on only 73 generating hours — interpret with caution.
</div>

---

## Operating output and emissions rate changes

Median gross output (MW) when generating, and per-MWh emissions rates. No control technology changes observed for any unit.

```js
const tableRows = EPP.filter(d => d.co2rP !== null);

display(html`<div style="overflow-x:auto;margin:1rem 0;">
<table style="width:100%;border-collapse:collapse;font-size:12px;">
<thead><tr style="border-bottom:2px solid var(--theme-foreground-faintest);">
  ${["Unit","Pre MW","Post MW","Output Δ","CO₂ Δ","SO₂ Δ","NOx Δ","Post hrs","Controls"].map(h =>
    html`<th style="text-align:left;padding:7px 9px;color:var(--theme-foreground-muted);font-weight:600;white-space:nowrap;">${h}</th>`)}
</tr></thead>
<tbody>
${tableRows.map(d => {
  const od = d.mlP - d.ml;
  const pctCell = (pct) => `<td style="padding:7px 9px;font-weight:600;color:${pct > 0 ? "#ef4444" : "#22c55e"};">${pct > 0 ? "+" : ""}${pct.toFixed(1)}%</td>`;
  return html`<tr style="border-bottom:1px solid var(--theme-foreground-faintest);">
    <td style="padding:7px 9px;font-weight:600;white-space:nowrap;">${sh(d.u)}</td>
    <td style="padding:7px 9px;">${d.ml}</td>
    <td style="padding:7px 9px;">${d.mlP}</td>
    <td style="padding:7px 9px;font-weight:600;color:${od < 0 ? "#ef4444" : "#22c55e"};">${od > 0 ? "+" : ""}${od} MW</td>
    <td style="padding:7px 9px;font-weight:600;color:${d.co2pct > 0 ? "#ef4444" : "#22c55e"};">${d.co2pct > 0 ? "+" : ""}${d.co2pct.toFixed(1)}%</td>
    <td style="padding:7px 9px;font-weight:600;color:${d.so2pct > 0 ? "#ef4444" : "#22c55e"};">${d.so2pct > 0 ? "+" : ""}${d.so2pct.toFixed(1)}%</td>
    <td style="padding:7px 9px;font-weight:600;color:${d.noxpct > 0 ? "#ef4444" : "#22c55e"};">${d.noxpct > 0 ? "+" : ""}${d.noxpct.toFixed(1)}%</td>
    <td style="padding:7px 9px;color:${d.gh < 200 ? "#f97316" : "inherit"};">${fN(d.gh)}${d.gh < 200 ? " ⚠" : ""}</td>
    <td style="padding:7px 9px;font-size:11px;color:var(--theme-foreground-muted);">${d.ctrl}</td>
  </tr>`;
})}
</tbody></table></div>`);
```

<div style="font-size:11px;color:var(--theme-foreground-muted);font-style:italic;margin:.25rem 0 1.5rem;">
⚠ = fewer than 200 generating hours post-order (small sample — interpret with caution). Output = median gross generation in MW during generating hours.
</div>

---

## CO₂ rate quarterly trend

```js
const selUnits = view(Inputs.checkbox(
  ALL_UNITS.filter(u => !u.includes("Eddystone")),
  {
    label: "Units (Eddystone excluded — gas peakers with volatile rates at <3% CF)",
    value: ALL_UNITS.filter(u => !u.includes("Eddystone")),
    format: u => sh(u),
  }
));
```

```js
const eqFiltered = EQ.filter(d => selUnits.includes(d.u));

display(Plot.plot({
  marginBottom: 60, marginLeft: 70, marginRight: 20,
  width: 860, height: 420,
  x: {label: null, tickRotate: -35},
  y: {label: "CO₂ (lb/MWh)", grid: true, tickFormat: d => fN(d)},
  color: {domain: ALL_UNITS, range: ALL_UNITS.map(u => COLORS[u]), legend: true},
  marks: [
    Plot.line(eqFiltered, {
      x: "q", y: "co2r", stroke: "u", strokeWidth: 2,
      tip: true, title: d => `${sh(d.u)}\n${d.q}: ${fN(d.co2r)} lb/MWh`,
    }),
    Plot.dot(eqFiltered, {x: "q", y: "co2r", fill: "u", r: 3}),
    Plot.ruleX(["2025Q2"], {stroke: "#ef4444", strokeDasharray: "6,3", strokeWidth: 1.5}),
    Plot.ruleX(["2025Q4"], {stroke: "#f97316", strokeDasharray: "6,3", strokeWidth: 1.5}),
  ],
}));
```

<div style="font-size:11px;color:var(--theme-foreground-muted);font-style:italic;margin:.25rem 0 1.5rem;">
11 quarter-unit observations excluded where total quarterly generation was below 2,000 MWh — these were failed-startup attempts where the boiler burned fuel at near-zero output, producing per-MWh rates 5–10× above steady-state. Excluded: Centralia 2024 Q2; Campbell 2 2024 Q1; Eddystone 3 and 4 in several low-generation quarters.
</div>

---

## Key findings

```js
display(html`<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));gap:12px;margin:1rem 0 1.5rem;">
${[
  {title:"CO₂: partial-output penalty", color:"#ef4444",
   text:"Campbell 3 (+10.2%) and Campbell 2 (+7.1%) ran at lower median output post-order (626 vs 851 MW; 200 vs 279 MW). Coal boilers achieve peak thermal efficiency near rated output — at partial output they burn more fuel per MWh. Control technology was unchanged for both units."},
  {title:"NOx: SCR cycling degradation", color:"#f97316",
   text:"Campbell 2's NOx rate jumped 39.8% post-order despite unchanged Selective Catalytic Reduction equipment. SCR catalysts require sustained high temperatures to function efficiently. With only 713 post-order generating hours vs 13,334 pre-order, frequent starts and stops likely cooled the catalyst between runs."},
  {title:"Higher output = cleaner per MWh", color:"#22c55e",
   text:"Schahfer 17's CO₂ rate fell 3.8% as median output rose from 197 to 255 MW — fewer hours but more concentrated dispatch yielded better thermal efficiency. SO₂ and NOx rose modestly, possibly reflecting scrubber/burner performance at the different operating point."},
  {title:"Culley 2: control system instability", color:"#d97706",
   text:"SO₂ up 71% while NOx down 55% post-order — but the quarterly chart shows wild oscillations in both pollutants throughout the entire period, pre- and post-order. CenterPoint described Culley 2 as \"inefficient and increasingly unreliable\" with $14–18M in pending overhaul costs."},
  {title:"Policy implication", color:"#3b82f6",
   text:"When 202(c) orders force units into an \"available but reluctantly dispatched\" mode, they cycle more and run at partial output. The data suggest this can worsen per-MWh emissions intensity even as total output declines — the grid receives less energy from these plants, but that energy is somewhat dirtier per MWh."},
].map(f => html`<div style="background:var(--theme-background-alt);border-radius:6px;padding:14px 16px;border-left:3px solid ${f.color};">
  <div style="font-size:13px;font-weight:700;margin-bottom:6px;color:${f.color};">${f.title}</div>
  <div style="font-size:12px;line-height:1.55;color:var(--theme-foreground-muted);">${f.text}</div>
</div>`)}
</div>`);
```

---

### References

```js
display(html`<div style="background:var(--theme-background-alt);border-radius:6px;padding:16px 20px;margin:.5rem 0 1rem;">
<div style="font-size:11px;font-weight:700;text-transform:uppercase;letter-spacing:.1em;color:var(--theme-foreground-muted);margin-bottom:10px;">References — Emissions Intensity</div>
${[
  {color:"#94a3b8", url:"https://campd.epa.gov/", label:"EPA CAMPD — Clean Air Markets Program Data (source for all hourly CEMS emissions)"},
  {color:"#f97316", url:"https://www.sierraclub.org/sites/default/files/2026-01/burning-money-synapse-report-on-indiana-orders.pdf", label:"Synapse Energy Economics (Jan 2026) — Schahfer/Culley deferred maintenance and operational unreliability"},
  {color:"#f97316", url:"https://earthjustice.org/press/2026/groups-challenge-illegal-order-halting-the-retirement-of-indiana-power-plants", label:"Earthjustice — NIPSCO $1B+ Schahfer repair estimate, units \"plagued\" by deferred maintenance"},
  {color:"#d97706", url:"https://www.citact.org/news/centerpoint-urged-federal-government-not-reissue-emergency-order-costly-coal-plant", label:"Citizens Action Coalition — Culley described by CenterPoint as \"inefficient and increasingly unreliable\""},
  {color:"#94a3b8", url:"https://www.energy.gov/documents/order-number-202-25-12-schahfer", label:"DOE Schahfer Order 202-25-12 (Dec 23, 2025) — unit nameplate capacities, commissioning dates 1983/1986"},
].map(r => html`<div style="display:flex;gap:10px;align-items:flex-start;font-size:12px;line-height:1.5;margin-bottom:6px;">
  <span style="width:8px;height:8px;border-radius:2px;background:${r.color};flex-shrink:0;margin-top:4px;display:inline-block;"></span>
  <a href="${r.url}" target="_blank" style="color:var(--theme-foreground-muted);border-bottom:1px dotted var(--theme-foreground-faintest);">${r.label}</a>
</div>`)}
</div>`);
```

<div style="padding:12px 16px;background:var(--theme-background-alt);border:1px solid var(--theme-foreground-faintest);border-radius:6px;font-size:11px;color:var(--theme-foreground-muted);line-height:1.6;font-style:italic;">
<strong style="font-style:normal;">Note on AI-generated content:</strong> Key Findings narrative interpretations were drafted with AI assistance based on the underlying CAMPD data and the references above. Readers are encouraged to verify specific factual claims against the cited sources. Quantitative values are derived directly from the EPA CAMPD hourly dataset; interpretive framing reflects judgments that should be independently checked.
</div>
