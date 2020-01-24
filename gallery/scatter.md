---
follows: gallery
id: litvis
---

@import "../css/datavis.less"

# Scatter and Strip Plots

## Simple strip plot

Strip plot showing the distribution of a single variable (precipitation) using tick marks. The [tick](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#tick) mark generates a short line suitable for representing single values.

```elm {v l}
strip : Spec
strip =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/seattle-weather.csv"

        enc =
            encoding
                << position X [ pName "precipitation", pQuant ]
    in
    toVegaLite [ data [], enc [], tick [] ]
```

---

## Multiple strip plots

Multiple distributions can be shown by displaying tick marks for several categories, here by the number of engine cylinders in a motor vehicle.

```elm {v l}
tickDistributions : Spec
tickDistributions =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Cylinders", pOrdinal ]
    in
    toVegaLite [ data [], enc [], tick [] ]
```

---

## Simple scatterplot

Positioning two quantitative variables on orthogonal axes and representing the position with a point mark generates a scatterplot.

```elm {v l}
scatter1 : Spec
scatter1 =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
    in
    toVegaLite [ data [], enc [], point [] ]
```

---

## Colour and shape encoded scatterplot

We can additionally encode other variables with the colour and shape of each point on a scatterplot. Here we double-encode country of origin with both shape and colour.

```elm {v l}
scatter2 : Spec
scatter2 =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << color [ mName "Origin", mNominal ]
                << shape [ mName "Origin", mNominal ]
    in
    toVegaLite [ width 400, height 400, data [], enc [], point [] ]
```

---

## Text-encoded scatterplot

Text can be used in place of a shape symbol to show categorical variables. Here a [transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#transform) is applied with [calculateAs](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#calculateAs) to create a new data field comprising the first letter of country of origin, double-encoded with colour and symbolised as text with [textMark](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#textMark).

```elm {v l}
textScatter : Spec
textScatter =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        trans =
            transform
                << calculateAs "datum.Origin[0]" "OriginInitial"

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << color [ mName "Origin", mNominal ]
                << text [ tName "OriginInitial", tNominal ]
    in
    toVegaLite [ width 400, height 400, data [], trans [], enc [], textMark [] ]
```

---

## Bubble plot

Quantitative or ordinal data can be encoded with symbol size to give a so-called "bubble plot".

```elm {v l}
bubbleplot : Spec
bubbleplot =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << size [ mName "Acceleration", mQuant ]
    in
    toVegaLite [ width 400, height 400, data [], enc [], point [] ]
```

---

## Bubble plot with non-linear scale

Reflecting the influence of [Hans Rosling](https://www.gapminder.org/videos/the-joy-of-stats/) we can use non-linear scales in scatterplots. Here a bubble plot shows the correlation between health and income for 187 countries in the world. This is modified from an example in [One Chart, Twelve Charting Libraries](http://lisacharlotterost.github.io/2016/05/17/one-chart-code/) by Lisa Charlotte Rost.

To focus the plot on the data points, the y-axis is not required to start at zero by setting its [pScale](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pScale) to [`scZero False`](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scZero). The x-axis uses a log scale by setting its position scale to be [`scType`](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scType) [`scLog`](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scLog).

```elm {v l}
logBubble : Spec
logBubble =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/gapminder-health-income.csv"

        enc =
            encoding
                << position X [ pName "income", pQuant, pScale [ scType scLog ] ]
                << position Y [ pName "health", pQuant, pScale [ scZero False ] ]
                << size [ mName "population", mQuant ]
                << color [ mStr "#000" ]
    in
    toVegaLite [ width 500, height 300, data [], enc [], circle [] ]
```

---

## Binned bubble plot

By placing variables into bins, it can sometimes be easier to show patterns in the variable encoded with size. Here, movie ratings from [IMDB](https://www.imdb.com) and [Rotten Tomatoes](https://www.rottentomatoes.com) are compared with ratings placed in bins and symbols sized according to the number of reviews. This shows ratings that differ greatly between the two sites tend to have fewer reviews, suggesting a [regression to the mean](https://en.wikipedia.org/wiki/Regression_toward_the_mean) effect.

Note also the use of the [circle](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#circle) rather than [point](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#point) mark, to specify a filled disc symbol.

```elm {v l}
binnedScatter : Spec
binnedScatter =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/movies.json"

        enc =
            encoding
                << position X [ pName "IMDB_Rating", pQuant, pBin [ biMaxBins 10 ] ]
                << position Y [ pName "Rotten_Tomatoes_Rating", pQuant, pBin [ biMaxBins 10 ] ]
                << size [ mAggregate opCount, mQuant ]
    in
    toVegaLite [ data [], enc [], circle [] ]
```

---

## Scatterplot with categorical variable

Based on this [natural catastrophes chart](https://ourworldindata.org/natural-catastrophes), we can create a bubble/scatter plot where one of the variables is categorical (type of disaster). Double encoding disaster type with colour and position allows the magnitude to be encoded with size even when symbols partially overlap.

```elm {v l}
categoricalScatter : Spec
categoricalScatter =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/disasters.csv"

        trans =
            -- Filter out the total from the categories shown
            transform
                << filter (fiExpr "datum.Entity !== 'All natural disasters'")

        enc =
            encoding
                << position X [ pName "Year", pOrdinal, pAxis [ axLabelAngle 0, axTitle "" ] ]
                << position Y [ pName "Entity", pNominal, pTitle "" ]
                << color [ mName "Entity", mNominal, mLegend [] ]
                << size
                    [ mName "Deaths"
                    , mQuant
                    , mTitle "Annual Global Deaths"
                    , mScale [ scRange (raNums [ 0, 5000 ]) ] -- Size range of circles
                    ]
    in
    toVegaLite
        [ width 600
        , height 400
        , data []
        , trans []
        , enc []
        , circle [ maOpacity 0.8 ]
        ]
```

---

## Scatterplot with null values

Sometimes showing absent values explicitly can be useful. In this example, by encoding a circle of null values with a grey colour we see the distribution of both absent IMDB and Rotten Tomatoes values. Data that have missing values for both variables are shown in the bottom left corner, but note it is not possible to visually assess how many fall into this category.

By default, null values are not shown, so we explicitly include them with a [configuration](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#configuration) setting [`maRemoveInvalid False`](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#coRemoveInvalid). We use [mDataCondition](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#mDataCondition) to encode points with colour conditionally depending on whether ratings are null.

```elm {v l}
scatterWithNulls : Spec
scatterWithNulls =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/movies.json"

        cfg =
            configure
                << configuration (coMark [ maRemoveInvalid False ])

        enc =
            encoding
                << position X [ pName "IMDB_Rating", pQuant ]
                << position Y [ pName "Rotten_Tomatoes_Rating", pQuant ]
                << color
                    [ mDataCondition
                        -- If either rating is null encode with grey
                        [ ( expr "datum.IMDB_Rating === null || datum.Rotten_Tomatoes_Rating === null"
                          , [ mStr "#ddd" ]
                          )
                        ]
                        -- otherwise encode in blue
                        [ mStr "rgb(76,120,168)" ]
                    ]
    in
    toVegaLite [ width 400, height 400, cfg [], data [], enc [], point [] ]
```
