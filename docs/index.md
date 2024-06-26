---
toc: false
---

<style>

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: var(--sans-serif);
  margin: 4rem 0 4rem;
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

.hero p {
  color: var(--theme-foreground-muted);
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 75px;
  }
}

</style>



<div class="hero">
    <h1>Offshore wind turbine installation: noise effects</h1>
    <h2>Impact distances for marine mammal injury and behavioral disturbance</h2>
    <br>
    <p>These data are for demonstration purposes only. Dashboard by <a href="https://waveformanalytics.com">Waveform Analytics, LLC</a>.</p>
    
</div>


<div class="grid grid-cols-1">

<div class="card">
  <h1>Subset selections</h1>

  <br>
  <h2>Installation type</h2>
  Wind turbines may be installed using either impact pile driving alone, or else a combination of impact and vibratory pile driving. 
  ${tidySubsetTextPick}

  
  <br>
  <h2>Criteria</h2>
  Impact ranges may be calculated using criteria describing the potential for either injury or behavioral disturbance. 
  ${criteriaInput}

  <br>
  <i>* Unless otherwise indicated, the plots below show a broadband attenuation level of <b>10dB</b>, and assume the <b>summer</b> sound speed profile.</i>

</div>

</div>


<div class="grid grid-cols-2">

  <div class="card grid-rowspan-2">

  <h2>Impact distances for each species</h2>

  ${resize((width) => plotAllSpecies(subsetDistances, {width})  )   }
  
  </div>


  <div class="card">

  <h2>Seasonal comparison</h2>
  <p>The temperature structure of the water column changes from summer to winter, and that affects the sound propagation. This plot shows how impact distances differ between the two seasons.</p>

  ${resize((width) => plotSeasonCompare(subsetDistances, {width}) )}
  
  </div>

  <div class="card  grid-rowspan-2">
  <h2>Attenuation comparison</h2>
  <p>To mitigate the effects of the pile driving sounds, different noise attenuation systems can be used to essentially muffle the sound and reduce the impact ranges. This plot shows how different broadband attenuation levels can reduce the impact ranges.</p>

  ${resize((width) => clusterPlot1(subsetDistances, {width}))}

  </div>

  <div class="card">
  <h2>Foundation comparison</h2>
  ${resize((width) => plotFoundCompare(subsetDistances, {width}) )}

  <i>Note: This plot format is based on <a href="https://observablehq.com/d/e34f190c55d77a97">a very nice example</a>
 by Tanya Shapiro</i>

  </div>

</div>



```js
const distances = aq.fromCSV(await FileAttachment('data/example-wf-impact-ranges.csv').text());
const speciesInfo = aq.fromCSV(await FileAttachment("data/species-info.csv").text());
const distancesAll = distances.join(speciesInfo,["species"])

// all distances to a single column ("distance"), with an identifying column ("criteria")
const distancesTidy = distancesAll
  .fold(['beh_NOAA-cont', 'beh_NOAA-int', 'beh_Wood', 'inj_pk-pts', 'inj_sel-pts-impulsive', 'inj_sel-pts-nonimpulsive','inj_sel-pts-both','beh_NOAA-both'], { as: ['criteria', 'distance'] }).derive({distanceKM: d => d.distance / 1000})
  .filter(d => d.distance != null)

// unique foundations
const unique = distancesTidy.groupby(["species","attenuation","found", "season", "vibpen", "abundance", "hearing_group", "shortName", "criteria", "distance"]).rollup({"mean": aq.op.mean("distance")})
```


```js
// Subset for Vibe+Impact pile driving
const tidyVibe = distancesTidy.filter( 
  d => ((d.criteria === "beh_NOAA-both") | 
        (d.criteria === "inj_sel-pts-both")) & 
  (d.vibpen === "vib_and_impact") 
  // (d.attenuation === "10dB") & 
  // (d.season === "Summer")
  )

// Subset for Impact-only pile driving
const tidyImpact = distancesTidy.filter( 
  d => ((d.criteria === "beh_NOAA-int") | 
        (d.criteria === "inj_sel-pts-impulsive")) & 
  (d.vibpen === "impact_only") 
  // (d.attenuation === "10dB") & 
  // (d.season === "Summer")
  )

```


```js
// User selection
const tidySubsetTextPick = Inputs.radio(["Impact", "Impact + vibe"], {label: "Select a subset:", value: "Impact"});
const tidySubsetPick = Generators.input(tidySubsetTextPick);

const criteriaInput = Inputs.radio(["Behavior", "Injury"], {label: "Select criteria:", value: "Behavior"});
const criteriaPick = Generators.input(criteriaInput);

const tidyObjects = distancesTidy.objects()
// const searchInput = Inputs.search(tidyObjects);


```

```js
const searchInput = Inputs.search(tidyObjects, {placeholder: "Search distance data…"});
// const tableView = Inputs.table(searchInput.objects());


```


```js
// const speciesTablePick = Generators.input(searchInput);

// display(Inputs.table(searchInput))

```


```js
const subsetDistances = getSubsetPick(tidySubsetPick, criteriaPick)


```


```js
function getSubsetPick(instPick, critPick) {
  if (instPick === "Impact") {
    if (critPick === "Behavior") 
      {
        return tidyImpact.filter(d => (d.criteria === "beh_NOAA-int"));   
      } else {
        return tidyImpact.filter(d => (d.criteria === "inj_sel-pts-impulsive"));
      }
  } else {
    if (critPick === "Behavior") 
      {
        return tidyVibe.filter(d => (d.criteria === "beh_NOAA-both"));
      } else {
        return tidyVibe.filter(d => (d.criteria === "inj_sel-pts-both"))
      }
    
  }
}

```


```js
function clusterPlot1(data, {width}) {
  return Plot.plot({
    width,
    height: 500,
    y: {
      grid: true, label: "Distance", 
      tickFormat: d => `${d/1000} km`,
      },
    fx: {label: null, domain: ["LF", "MF", "HF", "PW"]},
    color: {legend: true, domain: ["0dB", "6dB", "10dB"], scheme: "Warm"},
    marks: [
      Plot.ruleY([0]),
      Plot.dot(data.filter(d => (d.season === "Summer") & (d.distance > 0)), 
        Plot.dodgeX("middle", 
        {fx: "hearing_group", 
        y: "distance", 
        fill: "attenuation",
        stroke: "black",
        strokeWidth: 0.5,
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
```

```js
// unique
function plotSeasonCompare(data, {width}) {
  return Plot.plot({
    marginLeft: 80,
    width,
    height: 175,
    color: {legend: true, scheme: "Observable10", reverse: true},
    x: {label: "Distance", tickFormat: d => `${d/1000} km`},
    y: {axis: null},
    fy: {label: null},
    marks: [
      Plot.ruleX([0]),
      Plot.tickX(
        data.filter(d => (d.attenuation === "10dB")),
        {x: "distance", y: "season", 
        fy: "found", stroke: "season",
        strokeOpacity: 0.7}
      ),  
      Plot.tickX(
        data.filter(d => (d.attenuation === "10dB")),
        Plot.groupY(
          {x: "max"},
          {x: "distance", y: "season", 
          fy: "found", 
          stroke: "black", strokeWidth: 2}
        )
      )
    ]
  }
)
}
```

```js
// tidyImpact OR tidyVibe
function plotAllSpecies(data, {width}) {
  return Plot.plot({
    y: {grid: true, label: null},
    x: {label: "Distance", tickFormat: d => `${d/1000} km`},
    marginLeft: 120,
    height: 500,
    width,
    color: {legend: true, scheme: "Tableau10"},
    symbol: {legend: true},
    marks: [
      Plot.dot(
        data.filter(d => (d.season === "Summer") & (d.attenuation === "10dB")),
      {
        x: "distance", y: "shortName", 
        fill: "found", r: 6, symbol: "hammerID", stroke: "black",
        channels: 
        {
          shortName: {value: "shortName", label: "Species"},
          species: {value: "species", label: "Species"},
          found: {value: "found", label: "Foundation"},
          instSch: {value: "instSch", label: "Inst. Sch."},
          hammerID: {value: "hammerID", label: "Hammer ID"},
          distance: {value: "distance", label: "Distance"}
        },
        tip: 
        {
          format: {
            shortName: false,
            species: true,
            found: true,
            instSch: true,
            distance: (d) => `${d/1000} km`,
            x: false,
            fill: false,
            y: false,
            symbol: false,
          }
        },
      })
    ]
  })
}
```

```js
// tidyImpact OR tidyVibe
function plotFoundCompare(data0, {width}) {
  const data = data0.filter(d => (d.season === "Summer") & (d.attenuation === "10dB"));
  const plot = Plot.plot({
      width,
      height:250,
      y: {label:null, tickSize:0, label:null, axis:null},
      x: {grid:true, tickSize:0, label: "Distance (km)"},
      color: {scheme: "Tableau10"},
      facet: {data: data, y: "found", label:null},
      marginLeft:80,
      marginBottom:40,
      marks: [
      //density plot
      Plot.areaY(data,
        Plot.binX({y: 'count'},{x: "distanceKM", fill: 'found', fillOpacity: 0.85, curve: 'basis', 
        thresholds:40, stroke:'found', strokeWidth:2}
      )),
        //custom mark to create point interval
        markPointInterval(data, 
          {y:0, x:"distanceKM", text: "distanceKM", fillText:"white", fill: 'black'}),
        //jitter for each observation
        Plot.dot(data, {x: d => d.distanceKM, y: d=> -5 + Math.random()*2, r:4,
          fill: "found", dy:10, stroke:"white", strokeWidth:0.5,
          //customize tooltip on hover with channels and tip format
          channels: {species: "species", found: "found", distance: "distanceKM"}, 
          tip: {format: {fill: false, x: false, y: false, fy: false}}}),
      ]
  })
  return plot;
}
```




```js
function markPointInterval(data, {y, x, fill, text, fillText}){
   const marks = [
     //range
    Plot.ruleY(data, 
               Plot.groupY({x1: "min", x2: "max"},
                           {y: y, x: x, stroke: fill, strokeWidth:2})
              ),
    //quartiles IQR
    Plot.ruleY(data, 
               Plot.groupY({x1: (D) => d3.quantile(D, 0.25),x2: (D) => d3.quantile(D, 0.75)},
                           {y: y, x: x, stroke: fill, strokeWidth:4.5})),
    //mean dot
    Plot.dot(data, 
             Plot.groupY({x: "mean"},
                         {y: y, x: x, fill: fill, r:9})),
    //mean text
    Plot.text(data, 
             Plot.groupY({x: "mean", text: d=> d3.mean(d).toFixed(1)},
                         {y: y, x: x, fill: fillText, text: text,
                         fontSize:8})),
     ]

  return marks
}


```