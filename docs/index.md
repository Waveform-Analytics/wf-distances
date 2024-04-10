---
toc: false
---

<style>

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: var(--sans-serif);
  margin: 4rem 0 6rem;
  text-wrap: balance;
  text-align: center;
}

.hero h1 {
  margin: 2rem 0;
  max-width: none;
  font-size: 14vw;
  font-weight: 900;
  line-height: 1;
  background: linear-gradient(30deg, var(--theme-foreground-focus), currentColor);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero h2 {
  margin: 0;
  max-width: 34em;
  font-size: 20px;
  font-style: initial;
  font-weight: 500;
  line-height: 1.5;
  color: var(--theme-foreground-muted);
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 75px;
  }
}

</style>

<div class="hero">
  <h1>Offshore wind turbine noise effects</h1>
  <h2>Impact distances for marine mammal injury and behavioral disturbance</h2>
</div>

<div class="grid grid-cols-2">
  <div class="card">
  ${clusterPlot1(distancesAll)}
  
  </div>
  <div class="card"></div>
</div>

```js
const distances = aq.fromCSV(await FileAttachment('data/example-wf-impact-ranges.csv').text());
const speciesInfo = aq.fromCSV(await FileAttachment("data/species-info.csv").text());
const distancesAll = distances.join(speciesInfo,["species"])
```

```js
display(distancesAll)

```


```js
function clusterPlot1(data) {
  return Plot.plot({
  title: "Behavioral disturbance distances",
  subtitle: "Impact hammering only",
  width:700,
  y: {
    grid: true, label: "Distance", 
    tickFormat: d => `${d/1000} km`,
     },
  fx: {label: null, domain: ["LF", "MF", "HF", "PW"]},
  color: {legend: true, domain: ["0dB", "6dB", "10dB"]},
  marks: [
    Plot.ruleY([0]),
    Plot.dot(data.filter(d => d.season === "Summer"), 
             Plot.dodgeX("middle", 
                         {fx: "hearing_group", 
                          y: "beh_NOAA-int", 
                          fill: "attenuation",
                          r: 3.5,
                          channels: {
                            species: "species",
                            found: {value: "found", label: "Foundation"},
                            hammerID: {value: "hammerID", label: "Hammer ID"},
                          },
                         tip: {
                           format: {
                             species: true,
                             found: true,
                             y: (d) => `${d/1000}km`,
                             fx: false,
                             fill: false,
                           }
                         }}
                        ),
            )
  ]
});
}

