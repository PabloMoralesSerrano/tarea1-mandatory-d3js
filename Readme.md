# Task 1: Spain Covid-19

We have to represent the cases of Covid-19 in Spain on a map using d3js. The more cases there are in a community, the larger the circles.

## Initial Covid Representation in Spain
![map affected coronavirus](./content/captura-covid-inicial.png"initial covid map")

## Last 14 days Covid Representation in Spain
![map affected coronavirus](./content/captura-covid-final.png"lastcovid map")

# Steps

```bash
npm install
```

- You must install this packages to work with map formats GeoJSON or TopoJSON, topo JSON:

```bash
npm install topojson-client --save
```

```bash
npm install @types/topojson-client --save-dev
```

- Install the _composite projections_ project to display the Canary Island just below spain.

```bash
npm install d3-composite-projections --save
```

- Import Spanish topojson. This file that contains latitude and longitude  of the Spain's communities. Also the covid stats must to be imported.

_./src/index.ts_
```typescript
import * as d3 from "d3";
import { interpolateMagma } from "d3";
import { on } from "node:events";
import * as topojson from "topojson-client";
const spainjson = require("./spain.json");
const d3Composite = require("d3-composite-projections");
import { latLongCommunities } from "./communities";

import { statsIni, statsLast, ResultEntry } from "./covid";
```
- Using this function we obtain the number of cases that the most affected community has.
```typescript
const maxAffected = (s: ResultEntry[]) => {
  const max = s.reduce((max, item) => (item.value > max ? item.value : max), 0);
  return max;
};
```

- The next function is used to calculate the radius that every community must have. The most affected community is going to have the biggest circle and the less affected community the smallest.

``` typescript
const calculateRadiusBasedOnAffectedCases = (
  comunidad: string,
  s: ResultEntry[]
) => {
  const entry = s.find((item) => item.name === comunidad);

  const affectedRadiusScale = d3
    .scaleLinear()
    .domain([0, maxAffected(s)])
    .range([0, 50]);

  return entry ? affectedRadiusScale(entry.value) : 0;
};
```
- Setting the spain projection, translation and scale:

```typescript
const aProjection = d3Composite
  .geoConicConformalSpain()
  .scale(3300)
  .translate([500, 400]);
const geoPath = d3.geoPath().projection(aProjection);

const geojson = topojson.feature(spainjson, spainjson.objects.ESP_adm1);
```

- Setting a background color and rendering map:

```typescript
const svg = d3
  .select("body")
  .append("svg")
  .attr("width", 1024)
  .attr("height", 800)
  .attr("style", "background-color: #FBFAF0");

svg
  .selectAll("path")
  .data(geojson["features"])
  .enter()
  .append("path")
  .attr("class", "country")
  .attr("d", geoPath as any);

```

- Setting the initial state of the map with stats of the covid beginnings:

```typescript
svg
  .selectAll("circle")
  .data(latLongCommunities)
  .enter()
  .append("circle")
  .attr("class", "selected-country")
  .attr("r", (d) => calculateRadiusBasedOnAffectedCases(d.name, statsIni))
  .attr("cx", (d) => aProjection([d.long, d.lat])[0])
  .attr("cy", (d) => aProjection([d.long, d.lat])[1])
  ;
```
- Function for updating the circles radius when clicking a button:

```typescript
const updateRadius = (data: ResultEntry[]) => {
  d3.selectAll("circle")
    .data(latLongCommunities)
    .transition()
    .duration(500)
    .attr("class", "selected-country")
    .attr("r", (d) => calculateRadiusBasedOnAffectedCases(d.name, data))
    .attr("cx", (d) => aProjection([d.long, d.lat])[0])
    .attr("cy", (d) => aProjection([d.long, d.lat])[1])
};
```
- Getting the id of the buttons and listening to an event, then update the radius depending of the button selected.
```typescript
document
  .getElementById("Inicio")
  .addEventListener("click", function handlResultsIni() {
    updateRadius(statsIni);
  });

document
  .getElementById("Final")
  .addEventListener("click", function handlResultsFinal() {
    updateRadius(statsLast);
  });
  ```
  _./src/map.css_
  - Color for the country and the circles.
  ```css
  .country {
  stroke-width: 1;
  stroke: #2f4858;
  fill: #008c86;
}
```
```css
  .selected-country {
  stroke-width: 1;
  stroke: #bc5b40;
  fill: #f88f70;
  fill-opacity: 0.7;
}
```

  _./src/index.html_
  - Defining the buttons ids and importing the css and index.ts
  
  ```
  <html>
  <head>
    <link rel="stylesheet" type="text/css" href="./map.css" />
  </head>
  <div>
    <button id="Inicio">Initial Cases</button>
    <button id="Final">Last 14 days cases</button>
  </div>
  <body>
    <script src="./index.ts"></script>
  </body>
</html>
  ```
