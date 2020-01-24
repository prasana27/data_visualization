---
follows: gallery
id: litvis
---

@import "../css/datavis.less"

# Repeats and Concatenations

## Weather comparisons (1)

Monthly weather information for individual years and overall average for Seattle and New York.
Uses [repeat](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#repeat) as the encoding is the same for all charts, but with a different data field. The encoding for each chart itself comprises two layered specifications – one for the yearly data and one for the mean over all years.

```elm {v l}
weatherComparison1 : Spec
weatherComparison1 =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/weather.csv"

        enc1 =
            encoding
                << position X [ pName "date", pOrdinal, pTimeUnit month, pTitle "" ]
                << position Y [ pRepeat arColumn, pQuant, pAggregate opMean ]
                << detail [ dName "date", dTemporal, dTimeUnit year ]
                << color [ mName "location", mNominal ]
                << opacity [ mNum 0.2 ]

        spec1 =
            asSpec [ line [], enc1 [] ]

        enc2 =
            encoding
                << position X [ pName "date", pOrdinal, pTimeUnit month ]
                << position Y [ pRepeat arColumn, pQuant, pAggregate opMean ]
                << color [ mName "location", mNominal ]

        spec2 =
            asSpec [ line [ maInterpolate miMonotone ], enc2 [] ]
    in
    toVegaLite
        [ data []
        , repeat [ columnFields [ "temp_max", "precipitation", "wind" ] ]
        , specification (asSpec [ layer [ spec1, spec2 ] ])
        ]
```

---

## Weather comparisons (2)

Two concatenated charts that show a histogram of precipitation in Seattle and the relationship between min and max temperature. As the encodings for each are different, we use [concat](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#concat) to juxtapose them.

```elm {v l}
weatherComparison2 : Spec
weatherComparison2 =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/weather.csv"

        trans =
            transform
                << filter (fiExpr "datum.location === 'Seattle'")

        enc1 =
            encoding
                << position X [ pName "date", pTimeUnit month, pOrdinal, pTitle "" ]
                << position Y [ pName "precipitation", pQuant, pAggregate opMean ]

        spec1 =
            asSpec [ bar [], enc1 [] ]

        enc2 =
            encoding
                << position X [ pName "temp_min", pQuant, pBin [] ]
                << position Y [ pName "temp_max", pQuant, pBin [] ]
                << size [ mAggregate opCount, mQuant ]

        spec2 =
            asSpec [ circle [], enc2 [] ]
    in
    toVegaLite [ trans [], data [], concat [ spec1, spec2 ] ]
```

---

## Weather comparisons (3)

We can summarise each year's wind speed distribution compactly with a boxplot and use faceting (via the `row` function) to order the boxplots by the median wind speed.

```elm {v l}
weatherComparison3 : Spec
weatherComparison3 =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coHeader [ hdLabelAngle 0 ])
                << configuration (coFacet [ facoSpacing 0 ])

        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/weather.csv"

        trans =
            transform
                << filter (fiExpr "datum.location === 'Seattle'")
                << calculateAs "year(datum.date)" "year"

        enc =
            encoding
                << row
                    [ fName "year"
                    , fOrdinal
                    , fSort [ soByField "wind" opMedian ]
                    , fHeader [ hdTitle "" ]
                    ]
                << position X [ pName "wind", pQuant, pTitle "Wind speed" ]
    in
    toVegaLite [ cfg [], data [], trans [], enc [], boxplot [] ]
```

---

## Country-comparison of Car Characteristics

Repeated stacked histograms of various car engine characteristics.

```elm {v l}
carsByOrigin : Spec
carsByOrigin =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        trans =
            transform
                -- Convert pounds to kilograms
                << calculateAs "datum.Weight_in_lbs * 0.4536" "Weight"

        enc =
            encoding
                << position X [ pRepeat arFlow, pQuant, pBin [] ]
                << position Y [ pQuant, pAggregate opCount ]
                << color [ mName "Origin", mNominal ]
    in
    toVegaLite
        [ data []
        , trans []
        , columns 2
        , repeatFlow [ "Horsepower", "Miles_per_Gallon", "Acceleration", "Weight" ]
        , specification (asSpec [ enc [], bar [] ])
        ]
```

---

## Scatterplot Matrix

Interactive scatterplot. Drag to rescale any pair of data variables. Shift-drag to select a set of points to highlight.

```elm {v l interactive}
splom : Spec
splom =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        sel =
            selection
                << select "myBrush"
                    seInterval
                    [ seOn "[mousedown[event.shiftKey], window:mouseup] > window:mousemove!"
                    , seTranslate "[mousedown[event.shiftKey], window:mouseup] > window:mousemove!"
                    , seZoom "wheel![event.shiftKey]"
                    , seResolve seUnion
                    ]
                << select "grid"
                    seInterval
                    [ seBindScales
                    , seTranslate "[mousedown[!event.shiftKey], window:mouseup] > window:mousemove!"
                    , seZoom "wheel![event.shiftKey]"
                    , seResolve seGlobal
                    ]

        enc =
            encoding
                << position X [ pRepeat arColumn, pQuant ]
                << position Y [ pRepeat arRow, pQuant ]
                << color
                    [ mSelectionCondition (selectionName "myBrush")
                        [ mName "Origin", mNominal ]
                        [ mStr "black" ]
                    ]
                << opacity
                    [ mSelectionCondition (selectionName "myBrush")
                        [ mNum 0.8 ]
                        [ mNum 0.1 ]
                    ]
    in
    toVegaLite
        [ data []
        , repeat
            [ rowFields [ "Horsepower", "Acceleration", "Miles_per_Gallon" ]
            , columnFields [ "Miles_per_Gallon", "Acceleration", "Horsepower" ]
            ]
        , specification (asSpec [ sel [], enc [], circle [] ])
        ]
```

---

## Marginal Histograms

A composite view is used to create marginal histograms around a 2d binned histogram.

```elm {v l}
marginalHisto : Spec
marginalHisto =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/movies.json"

        cfg =
            configure
                << configuration (coRange [ racoHeatmap "greenblue" ])
                << configuration (coView [ vicoStroke Nothing ])

        encPosition =
            encoding
                << position X [ pName "IMDB_Rating", pQuant, pBin [ biMaxBins 10 ] ]
                << position Y [ pName "Rotten_Tomatoes_Rating", pQuant, pBin [ biMaxBins 10 ] ]

        enc1 =
            encoding
                << position X [ pName "IMDB_Rating", pQuant, pAxis [], pBin [] ]
                << position Y
                    [ pAggregate opCount
                    , pQuant
                    , pScale [ scDomain (doNums [ 0, 1000 ]) ]
                    , pAxis []
                    ]

        spec1 =
            asSpec [ height 60, bar [], enc1 [] ]

        spec2 =
            asSpec [ spacing 15, bounds boFlush, hConcat [ spec2_1, spec2_2 ] ]

        enc2_1 =
            encoding
                << position X [ pName "IMDB_Rating", pQuant, pBin [] ]
                << position Y [ pName "Rotten_Tomatoes_Rating", pQuant, pBin [] ]
                << color [ mAggregate opCount, mQuant ]

        spec2_1 =
            asSpec [ rect [], enc2_1 [] ]

        enc2_2 =
            encoding
                << position Y
                    [ pName "Rotten_Tomatoes_Rating"
                    , pQuant
                    , pBin []
                    , pAxis []
                    ]
                << position X
                    [ pAggregate opCount
                    , pQuant
                    , pScale [ scDomain (doNums [ 0, 1000 ]) ]
                    , pAxis []
                    ]

        spec2_2 =
            asSpec [ width 60, enc2_2 [], bar [] ]
    in
    toVegaLite [ cfg [], data [], vConcat [ spec1, spec2 ] ]
```
