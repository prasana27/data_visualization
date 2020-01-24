---
follows: gallery
id: litvis
---

@import "../css/datavis.less"

# Faceted Charts

## Faceted bar chart

Bar charts showing the reported crimes over time in the West Midlands, faceted by crime type.
Faceting is simply another form of data encoding, but rather than show different data values with different colours, or shapes or sizes, we use different charts. Here those charts are arranged horizontally with the [column](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#column) faceting operator.

```elm {v l highlight=22}
facetBars : Spec
facetBars =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/westMidlands/westMidsCrimesAggregated.tsv"

        crimeColours =
            categoricalDomainMap
                [ ( "Anti-social behaviour", "rgb(59,118,175)" )
                , ( "Burglary", "rgb(81,157,62)" )
                , ( "Criminal damage and arson", "rgb(141,106,184)" )
                , ( "Drugs", "rgb(239,133,55)" )
                , ( "Robbery", "rgb(132,88,78)" )
                , ( "Vehicle crime", "rgb(213,126,190)" )
                ]

        enc =
            encoding
                << position X [ pName "month", pTemporal, pTitle "" ]
                << position Y [ pName "reportedCrimes", pQuant, pAggregate opSum ]
                << color [ mName "crimeType", mNominal, mLegend [] ]
                << column [ fName "crimeType", fNominal ]
    in
    toVegaLite [ height 100, width 120, data [], enc [], bar [] ]
```

---

## Faceted Stacked Bars

Barley crop yields in 1931 and 1932 (i.e. faceted by year) shown as stacked bar
charts. The title of the collection of faceted views is controlled via [fHeader](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#fHeader), which in this case simply removes the redundant `year` title.

```elm {v l highlight=12}
facetStackedBars : Spec
facetStackedBars =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/barley.json"

        enc =
            encoding
                << position X [ pName "yield", pQuant, pAggregate opSum ]
                << position Y [ pName "variety", pNominal ]
                << color [ mName "site", mNominal ]
                << column [ fName "year", fOrdinal, fHeader [ hdTitle "" ] ]
    in
    toVegaLite [ data [], enc [], bar [] ]
```

---

## Faceted Scatterplot

Scatterplots of US box-office takings vs profits for different MPAA film ratings.

```elm {v l highlight=11}
facetScatterplot : Spec
facetScatterplot =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/movies.json"

        enc =
            encoding
                << position X [ pName "Worldwide_Gross", pQuant ]
                << position Y [ pName "US_DVD_Sales", pQuant ]
                << column [ fName "MPAA_Rating", fOrdinal ]
    in
    toVegaLite [ width 120, height 120, data [], enc [], point [] ]
```

and the same faceted charts laid out in two columns:

```elm {v l highlight=[14,15]}
facetTwoCols : Spec
facetTwoCols =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/movies.json"

        enc =
            encoding
                << position X [ pName "Worldwide_Gross", pQuant ]
                << position Y [ pName "US_DVD_Sales", pQuant ]
    in
    toVegaLite
        [ data []
        , columns 2
        , facetFlow [ fName "MPAA_Rating", fOrdinal ]
        , specification (asSpec [ enc [], point [] ])
        ]
```

---

## Vertical Layout

Distributions of car engine power for different countries of origin. Here arranged vertically with [row](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#row).

```elm {v l highlight=11}
facetBars2 : Spec
facetBars2 =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant, pBin [ biMaxBins 15 ] ]
                << position Y [ pAggregate opCount, pQuant ]
                << row [ fName "Origin", fOrdinal, fHeader [ hdTitle "" ] ]
    in
    toVegaLite [ data [], enc [], bar [] ]
```

---

## Faceting with independent scales

Stock prices of five large companies as a small multiples of area charts. Here the small multiples use independent scales on the y-axis to account for the constrasting absolute prices.

```elm {v l highlight=[30-32]}
stocks : Spec
stocks =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/stocks.csv"

        enc =
            encoding
                << position X
                    [ pName "date"
                    , pTemporal
                    , pAxis [ axFormat "%Y", axTitle "", axGrid False ]
                    ]
                << position Y
                    [ pName "price"
                    , pQuant
                    , pAxis [ axTitle "", axGrid False ]
                    ]
                << color
                    [ mName "symbol"
                    , mNominal
                    , mLegend []
                    ]
                << row
                    [ fName "symbol"
                    , fNominal
                    , fHeader [ hdTitle "Stock price", hdLabelAngle 0 ]
                    ]

        res =
            resolve
                << resolution (reScale [ ( chY, reIndependent ) ])

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite [ width 300, height 50, cfg [], res [], data [], enc [], area [] ]
```

---

## Anscombe's Quartet

[Anscombe's Quartet](https://en.wikipedia.org/wiki/Anscombe%27s_quartet) famously illustrates the power of data visualization in detecting patterns not captured with numerical summaries (all four have the same means, standard deviations and correlation coefficient).

```elm {v l}
anscombe : Spec
anscombe =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/anscombe.json"

        trans =
            transform
                << regression "Y" "X" [ rgExtent (num 0) (num 20), rgAs "rx" "ry" ]

        encScatter =
            encoding
                << position X [ pName "X", pQuant, pAxis [ axTitle "", axGrid False ] ]
                << position Y [ pName "Y", pQuant, pAxis [ axTitle "", axGrid False ] ]

        encLine =
            encoding
                << position X [ pName "rx", pQuant ]
                << position Y [ pName "ry", pQuant ]

        scatter =
            asSpec [ encScatter [], circle [ maOpacity 1, maColor "black" ] ]

        rLine =
            asSpec [ trans [], encLine [], line [ maStrokeWidth 1 ] ]
    in
    toVegaLite
        [ data []
        , columns 2
        , facetFlow [ fName "Series", fOrdinal, fHeader [ hdTitle "" ] ]
        , specification (asSpec [ layer [ scatter, rLine ] ])
        ]
```

---

## Ordered Small Multiples

The 'trellis' display by Becker _et al._ helped establish small multiples as a "powerful mechanism for understanding interactions in studies of how a response depends on explanatory variables". Here reproduced as a set of small multuples of Barley yields from the 1930s, complete with ordering to facilitate comparison.

The arrangement uses [facetFlow](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#facetFlow) to create a flow layout, sorted by the median yield, and [columns](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#columns) to arrange the flow over two columns.

```elm {v l}
barleySmallMultiples : Spec
barleySmallMultiples =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/barley.json"

        enc =
            encoding
                << position X
                    [ pName "yield"
                    , pQuant
                    , pAggregate opMedian
                    , pScale [ scZero False ]
                    ]
                << position Y
                    [ pName "variety"
                    , pOrdinal
                    , pSort [ soByChannel chX, soDescending ]
                    ]
                << color [ mName "year", mNominal ]
    in
    toVegaLite
        [ data []
        , columns 2
        , facetFlow
            [ fName "site"
            , fOrdinal
            , fSort [ soByField "yield" opMedian ]
            , fHeader [ hdTitle "" ]
            ]
        , specification (asSpec [ heightStep 12, enc [], point [] ])
        ]
```
