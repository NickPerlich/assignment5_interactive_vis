# Data Centers and County Income

```js
import * as d3 from "npm:d3";
import * as aq from "npm:arquero";
const { op } = aq;
```

```js
const datacenters3 = aq.fromCSV(await FileAttachment("data/datacenters3.csv").text());
const datacenters4 = aq.fromCSV(await FileAttachment("data/datacenters4.csv").text());
const incomePerCapita = aq.fromCSV(await FileAttachment("data/IncomePerCapitaByYear.csv").text());
const countyGDP = aq.fromCSV(await FileAttachment("data/county_gdp_summary.csv").text());
```

```js
const datacenters43 = datacenters3.join_left(
  datacenters4.select('Name', 'GeoFIPS', 'GeoName', 'Longitude', 'Latitude'),
  [['Data center'], ['Name']]
);

const datacenterFipsByYear = (() => {
  const m = new Map();
  for (let y = 2018; y <= 2030; y++) {
    const activeFips = new Set();
    for (const row of Array.from(datacenters43)) {
      if (new Date(row.Date).getFullYear() <= y && row["Power (MW)"] > 0) {
        const fips = String(row.GeoFIPS).padStart(5, "0");
        if (fips !== "00000" && !fips.includes("null")) activeFips.add(fips);
      }
    }
    m.set(y, activeFips);
  }
  return m;
})();

const incomeByFipsYear = (() => {
  const m = new Map();
  const gdpByFips = new Map(countyGDP.objects().map(d => [String(d.fips).padStart(5, "0"), d]));
  for (const row of incomePerCapita) {
    const fips = String(row.GeoFIPS).padStart(5, "0");
    const byYear = new Map();
    for (const key of Object.keys(row)) {
      const y = +key;
      if (y >= 2018 && y <= 2024 && row[key] != null) byYear.set(y, +row[key]);
    }
    const base = byYear.get(2024);
    const gdpRow = gdpByFips.get(fips);
    if (base != null && gdpRow != null) {
      const rate = (y) => {
        if (y <= 2025) return Math.pow(1 + gdpRow.gdp_growth_1y_pct / 100, 1/1) - 1;
        if (y <= 2029) return Math.pow(1 + gdpRow.gdp_growth_5y_pct / 100, 1/5) - 1;
        return Math.pow(1 + gdpRow.gdp_growth_10y_pct / 100, 1/10) - 1;
      };
      for (let y = 2025; y <= 2030; y++) {
        byYear.set(y, base * Math.pow(1 + rate(y), y - 2024));
      }
    }
    m.set(fips, byYear);
  }
  return m;
})();

const barData = (() => {
  const m = new Map();
  for (let y = 2018; y <= 2030; y++) {
    const activeFips = datacenterFipsByYear.get(y);
    const withDC = [], withoutDC = [];
    for (const [fips, byYear] of incomeByFipsYear) {
      const v = byYear.get(y);
      if (v == null || isNaN(v)) continue;
      if (activeFips.has(fips)) withDC.push({ fips, v });
      else withoutDC.push({ fips, v });
    }
    const avg = arr => arr.length ? arr.reduce((s, d) => s + d.v, 0) / arr.length : null;
    const topCounty = arr => arr.length ? arr.reduce((a, b) => a.v > b.v ? a : b) : null;
    m.set(y, {
      with: { avg: avg(withDC), count: withDC.length, top: topCounty(withDC) },
      without: { avg: avg(withoutDC), count: withoutDC.length, top: topCounty(withoutDC) }
    });
  }
  return m;
})();

const fipsToName = (() => {
  const m = new Map();
  for (const row of incomePerCapita) {
    const fips = String(row.GeoFIPS).padStart(5, "0");
    m.set(fips, row.GeoName);
  }
  return m;
})();
```

```js
const margin = { top: 60, right: 40, bottom: 80, left: 90 };
const width = 700 - margin.left - margin.right;
const height = 500 - margin.top - margin.bottom;

const container = d3.create("div").style("font-family", "sans-serif").style("position", "relative");

const sliderDiv = container.append("div")
  .style("margin-bottom", "10px")
  .style("display", "flex")
  .style("align-items", "center")
  .style("gap", "12px");

sliderDiv.append("label")
  .style("font-size", "13px")
  .style("font-weight", "bold")
  .text("Year:");

const slider = sliderDiv.append("input")
  .attr("type", "range")
  .attr("min", 2018)
  .attr("max", 2030)
  .attr("step", 1)
  .attr("value", 2024)
  .style("width", "300px");

const yearLabel = sliderDiv.append("span")
  .style("font-size", "13px")
  .style("font-weight", "bold")
  .text("2024");

const svg = container.append("svg")
  .attr("width", width + margin.left + margin.right)
  .attr("height", height + margin.top + margin.bottom);

const g = svg.append("g")
  .attr("transform", `translate(${margin.left},${margin.top})`);

const allAvgs = [];
for (const [y, d] of barData) allAvgs.push(d.with.avg, d.without.avg);
const yMax = d3.max(allAvgs) * 1.1;

const x = d3.scaleBand()
  .domain(["With Data Center", "Without Data Center"])
  .range([0, width])
  .padding(0.4);

const y = d3.scaleLinear()
  .domain([0, yMax])
  .range([height, 0]);

g.append("g")
  .attr("transform", `translate(0,${height})`)
  .call(d3.axisBottom(x).tickSize(0))
  .select(".domain").remove();

g.append("g")
  .call(d3.axisLeft(y).tickFormat(d => `$${d3.format(",.0f")(d)}`));

g.append("text")
  .attr("transform", "rotate(-90)")
  .attr("x", -height / 2)
  .attr("y", -70)
  .attr("text-anchor", "middle")
  .attr("font-size", "12px")
  .attr("fill", "currentColor")
  .text("Average Income per Capita");

svg.append("text")
  .attr("x", (width + margin.left + margin.right) / 2)
  .attr("y", 20)
  .attr("text-anchor", "middle")
  .attr("font-size", "15px")
  .attr("fill", "currentColor")
  .attr("font-weight", "bold")
  .text("Average County Income: With vs Without Data Centers");

svg.append("text")
  .attr("x", (width + margin.left + margin.right) / 2)
  .attr("y", 38)
  .attr("text-anchor", "middle")
  .attr("font-size", "11px")
  .attr("fill", "#666")
  .text("Historical 2018–2024 | Projected 2025–2030");

const tooltip = container.append("div")
  .style("position", "absolute")
  .style("background", "white")
  .style("border", "1px solid #ccc")
  .style("border-radius", "6px")
  .style("padding", "10px")
  .style("font-size", "12px")
  .style("color", "black") 
  .style("pointer-events", "none")
  .style("opacity", 0);

const groups = ["with", "without"];
const labels = ["With Data Center", "Without Data Center"];
const colors = ["#0369a1", "#d97706"];

function update(year) {
  const d = barData.get(year);
  const isProjected = year > 2024;
  yearLabel.text(year + (isProjected ? " (Projected)" : ""));

  const barData2 = groups.map((grp, i) => ({
    grp, label: labels[i], color: colors[i], avg: d[grp].avg ?? 0
  }));

  g.selectAll("rect")
    .data(barData2, d => d.grp)
    .join(
      enter => enter.append("rect")
        .attr("x", d => x(d.label))
        .attr("width", x.bandwidth())
        .attr("rx", 4)
        .attr("y", height)
        .attr("height", 0)
        .attr("fill", d => d.color)
        .call(e => e.transition().duration(500)
          .attr("y", d => y(d.avg))
          .attr("height", d => height - y(d.avg))
          .attr("opacity", isProjected ? 0.7 : 1)),
      update => update
        .call(u => u.transition().duration(500)
          .attr("y", d => y(d.avg))
          .attr("height", d => height - y(d.avg))
          .attr("fill", d => isProjected ? d3.color(d.color).copy({opacity: 0.6}) : d.color)
          .attr("opacity", isProjected ? 0.7 : 1)),
      exit => exit
        .call(e => e.transition().duration(500)
          .attr("y", height).attr("height", 0).remove())
    );

  g.selectAll(".bar-label")
    .data(barData2, d => d.grp)
    .join(
      enter => enter.append("text")
        .attr("class", "bar-label")
        .attr("text-anchor", "middle")
        .attr("font-size", "12px")
        .attr("font-weight", "bold")
        .attr("fill", "white")
        .attr("x", d => x(d.label) + x.bandwidth() / 2)
        .attr("y", height)
        .call(e => e.transition().duration(500)
          .attr("y", d => y(d.avg) - 6)
          .text(d => `$${d3.format(",.0f")(d.avg)}`)),
      update => update
        .call(u => u.transition().duration(500)
          .attr("x", d => x(d.label) + x.bandwidth() / 2)
          .attr("y", d => y(d.avg) - 6)
          .text(d => `$${d3.format(",.0f")(d.avg)}`)),
      exit => exit
        .call(e => e.transition().duration(500).attr("y", height).remove())
    );

  g.selectAll("rect")
    .on("mouseover", function(event, d) {
      const info = barData.get(year)[d.grp];
      const topName = fipsToName.get(info.top.fips) ?? info.top.fips;
      tooltip.style("opacity", 1)
        .html(`
          <strong>${d.label}</strong><br/>
          Average Income: <b>$${d3.format(",.0f")(info.avg)}</b><br/>
          Number of Counties: <b>${info.count}</b><br/>
          Top County: <b>${topName}</b><br/>
          Top County Income: <b>$${d3.format(",.0f")(info.top.v)}</b>
        `);
    })
    .on("mousemove", function(event) {
      const rect = container.node().getBoundingClientRect();
      tooltip
        .style("left", (event.clientX - rect.left + 12) + "px")
        .style("top", (event.clientY - rect.top - 28) + "px");
    })
    .on("mouseout", function() {
      tooltip.style("opacity", 0);
    });
}

update(2024);
slider.on("input", function() { update(+this.value); });

display(container.node());
```