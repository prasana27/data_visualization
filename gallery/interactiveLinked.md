---
follows: gallery
id: litvis
---

@import "../css/datavis.less"

# Interactive Linked Views

_For all interactive examples on this page, the code block header must include the `interactive` keyword._

## Zoom and Overview

We can concatenate a zoomed view and an overview together and use interaction with the overview to select the zoomed scale. _Try dragging a rectangle in the lower grey area chart in order to zoom and pan the main chart._

```elm {v l interactive}
globalTemperatures : Spec
globalTemperatures =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/temperatureAnomalies.json"
                [ parse [ ( "Anomaly", foNum ) ] ]

        sel =
            selection
                << select "myBrush"
                    seInterval
                    [ seEncodings [ chX ]
                    , seSelectionMark [ smFill "hsl(320,100%,40%)" ]
                    ]

        enc1 =
            encoding
                << position X
                    [ pName "Date"
                    , pTemporal
                    , pScale [ scDomain (doSelection "myBrush") ]
                    , pTitle ""
                    ]
                << position Y [ pName "Anomaly", pQuant ]
                << color
                    [ mName "Anomaly"
                    , mQuant
                    , mScale [ scScheme "redblue" [ 1 - (1.35 - 0.78) / (2 * 1.35), 0 ] ]
                    ]

        spec1 =
            asSpec [ width 500, bar [], enc1 [] ]

        enc2 =
            encoding
                << position X [ pName "Date", pTemporal, pAxis [ axFormat "%Y", axTitle "" ] ]
                << position Y [ pName "Anomaly", pQuant, pAxis [] ]

        spec2 =
            asSpec [ width 500, height 60, sel [], enc2 [], bar [ maColor "lightgrey" ] ]
    in
    toVegaLite [ data, vConcat [ spec1, spec2 ] ]
```

---

## Bar chart with minimap

The same idea is useful for large charts, where we can show a 'minimap' of the entire chart and use a selection to show only part of it at full size.

_Drag a selection on the minimap to show part of it._

```elm {v l interactive}
minimap : Spec
minimap =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        sel =
            selection
                << select "brush" seInterval [ seEncodings [ chY ] ]

        trans =
            transform
                << joinAggregate [ opAs opCount "Name" "numModels" ] [ wiGroupBy [ "Name" ] ]
                << window [ ( [ wiOp woRowNumber ], "rowNumber" ) ] [ wiSort [ wiDescending "numCars" ] ]

        transFilter =
            transform
                << filter (fiSelection "brush")

        encMain =
            encoding
                << position Y
                    [ pName "Name"
                    , pOrdinal
                    , pAxis [ axMinExtent 200, axTitle "" ]
                    , pSort [ soByChannel chX, soDescending ]
                    ]
                << position X
                    [ pName "numModels"
                    , pQuant
                    , pScale [ scDomain (doNums [ 0, 6 ]) ]
                    , pAxis [ axOrient siTop, axTitle "Number of models" ]
                    ]

        specMain =
            asSpec [ height 600, transFilter [], encMain [], bar [] ]

        encMini =
            encoding
                << position Y
                    [ pName "Name"
                    , pNominal
                    , pSort [ soByChannel chX, soDescending ]
                    , pAxis []
                    ]
                << position X [ pAggregate opCount, pQuant, pAxis [] ]

        specMini =
            asSpec [ width 50, height 200, sel [], encMini [], bar [] ]
    in
    toVegaLite [ data [], trans [], hConcat [ specMain, specMini ] ]
```

---

## Responsive Table

We can create a table of data using the [textMark](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#textMark) that is dependent on the current selection from a linked scatterplot.

_Drag a rectangular brush to show (first 20) selected points in a table._

```elm {v l interactive}
responsiveTable : Spec
responsiveTable =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        trans =
            transform
                << window [ ( [ wiOp woRowNumber ], "rowNumber" ) ] []

        sel =
            selection
                << select "brush" seInterval []

        encPoint =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << color
                    [ mSelectionCondition (selectionName "brush")
                        [ mName "Cylinders", mOrdinal ]
                        [ mStr "grey" ]
                    ]

        specPoint =
            asSpec [ sel [], point [], encPoint [] ]

        tableTrans =
            transform
                << filter (fiSelection "brush")
                << window [ ( [ wiOp woRank ], "rank" ) ] []
                << filter (fiLessThan "rank" (num 20))

        encHPText =
            encoding
                << position Y [ pName "rowNumber", pOrdinal, pAxis [] ]
                << text [ tName "Horsepower", tNominal ]

        specHPText =
            asSpec
                [ title "Engine power" []
                , tableTrans []
                , textMark []
                , encHPText []
                ]

        encMPGText =
            encoding
                << position Y [ pName "rowNumber", pOrdinal, pAxis [] ]
                << text [ tName "Miles_per_Gallon", tNominal, tFormat ".1f" ]

        specMPGText =
            asSpec
                [ title "Efficiency (mpg)" []
                , tableTrans []
                , encMPGText []
                , textMark []
                ]

        encOriginText =
            encoding
                << position Y [ pName "rowNumber", pOrdinal, pAxis [] ]
                << text [ tName "Origin", tNominal ]

        specOriginText =
            asSpec
                [ title "Country of origin" []
                , tableTrans []
                , encOriginText []
                , textMark []
                ]

        res =
            resolve
                << resolution (reLegend [ ( chColor, reIndependent ) ])

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite
        [ cfg []
        , data []
        , trans []
        , res []
        , hConcat [ specPoint, specHPText, specMPGText, specOriginText ]
        ]
```

---

## Cross Filtering

Comparison of flight distances, delays and times using cross-filtering. _Drag a rectangle in any one of the charts to select items that are highlighted across all three._

```elm {v l interactive}
crossFilter : Spec
crossFilter =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/flights-2k.json"

        trans =
            transform
                << calculateAs "hours(datum.date)" "time"

        sel =
            selection
                << select "myBrush" seInterval [ seEncodings [ chX ], seEmpty ]

        selTrans =
            transform
                << filter (fiSelection "myBrush")

        encPosition =
            encoding
                << position X [ pRepeat arColumn, pQuant, pBin [ biMaxBins 20 ] ]
                << position Y [ pAggregate opCount, pQuant ]

        spec1 =
            asSpec [ sel [], bar [] ]

        spec2 =
            asSpec [ selTrans [], bar [ maColor "goldenrod" ] ]

        spec =
            asSpec [ data [], trans [], encPosition [], layer [ spec1, spec2 ] ]
    in
    toVegaLite
        [ repeat [ columnFields [ "distance", "delay", "time" ] ]
        , specification spec
        ]
```

---

## Linked Scatterplot Matrix

The position scales are bound to an interval selection allowing a scatterplot to be zoomed and panned. By resolving the scaling globally, the scaling in any one plot is applied across all views. _Try dragging the pointer in any scatterplot._

```elm {v l interactive}
linkedSplom : Spec
linkedSplom =
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
                << select "myZoom"
                    seInterval
                    [ seBindScales
                    , seTranslate "[mousedown[!event.shiftKey], window:mouseup] > window:mousemove!"
                    , seZoom "wheel![event.shiftKey]"
                    , seResolve seGlobal
                    ]

        enc =
            encoding
                << position X [ pRepeat arColumn, pMType Quantitative ]
                << position Y [ pRepeat arRow, pMType Quantitative ]
                << color
                    [ mSelectionCondition (selectionName "myBrush")
                        [ mName "Origin", mMType Nominal ]
                        [ mStr "lightgrey" ]
                    ]
    in
    toVegaLite
        [ data []
        , repeat
            [ rowFields [ "Horsepower", "Acceleration", "Miles_per_Gallon" ]
            , columnFields [ "Miles_per_Gallon", "Acceleration", "Horsepower" ]
            ]
        , specification (asSpec [ data [], point [], sel [], enc [] ])
        ]
```

---

## Bar Selection for 2d Histogram Filtering

Selecting film genres in the bar chart allows dynamic update of the 2d histogram.

```elm {v l interactive highlight=[11-13,15-17]}
barSelection : Spec
barSelection =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/movies.json"

        trans =
            transform
                << filter (fiExpr "datum.Major_Genre != null")

        sel =
            selection
                << select "selectedGenre" seSingle [ seEncodings [ chX ] ]

        selTrans =
            transform
                << filter (fiSelection "selectedGenre")

        encPosition =
            encoding
                << position X [ pName "IMDB_Rating", pQuant, pBin [] ]
                << position Y [ pName "Rotten_Tomatoes_Rating", pQuant, pBin [] ]

        enc2dHisto =
            encoding
                << color
                    [ mAggregate opCount
                    , mQuant
                    , mScale [ scScheme "greenblue" [] ]
                    , mLegend []
                    ]

        spec2dHisto =
            asSpec [ width 300, rect [], enc2dHisto [] ]

        encCircles =
            encoding
                << size [ mAggregate opCount, mQuant ]

        specCircles =
            asSpec [ selTrans [], encCircles [], point [ maColor "grey" ] ]

        specHeat =
            asSpec [ encPosition [], layer [ spec2dHisto, specCircles ] ]

        specBar =
            asSpec [ width 420, height 120, sel [], encBar [], bar [] ]

        encBar =
            encoding
                << position X
                    [ pName "Major_Genre"
                    , pNominal
                    , pAxis [ axTitle "Film genre", axLabelAngle -40 ]
                    ]
                << position Y
                    [ pAggregate opCount
                    , pQuant
                    , pAxis [ axGrid False ]
                    ]
                << color
                    [ mSelectionCondition (selectionName "selectedGenre")
                        [ mStr "steelblue" ]
                        [ mStr "lightgrey" ]
                    ]
    in
    toVegaLite [ data [], trans [], vConcat [ specHeat, specBar ] ]
```

---

## Two-way Selection Filtering

Selecting weather types in the bar chart allows dynamic update of the weather time-series. Selecting part of the time series updates the bar chart.

```elm {v l interactive highlight=[7-9,11-13,15-17,19-21,58,71]}
twoWayFiltering : Spec
twoWayFiltering =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/seattle-weather.csv"

        selTime =
            selection
                << select "timeBrush" seInterval [ seEncodings [ chX ] ]

        transTime =
            transform
                << filter (fiSelection "timeBrush")

        selBar =
            selection
                << select "barSelect" seMulti [ seEncodings [ chColor ] ]

        transBar =
            transform
                << filter (fiSelection "barSelect")

        weatherColours =
            categoricalDomainMap
                [ ( "sun", "#e7ba52" )
                , ( "fog", "#c7c7c7" )
                , ( "drizzle", "#aec7ea" )
                , ( "rain", "#1f77b4" )
                , ( "snow", "#9467bd" )
                ]

        encTime =
            encoding
                << position X
                    [ pName "date"
                    , pTemporal
                    , pTimeUnit monthDate
                    , pAxis [ axTitle "", axFormat "%b" ]
                    ]
                << position Y
                    [ pName "temp_max"
                    , pQuant
                    , pScale [ scDomain (doNums [ -5, 40 ]) ]
                    , pAxis [ axTitle "Maximum Daily Temperature (C)" ]
                    ]
                << color
                    [ mSelectionCondition (selectionName "timeBrush")
                        [ mName "weather", mNominal, mScale weatherColours ]
                        [ mStr "#cfdebe" ]
                    ]
                << size
                    [ mName "precipitation"
                    , mQuant
                    , mScale [ scDomain (doNums [ -1, 50 ]) ]
                    ]

        specTime =
            asSpec [ width 600, point [], selTime [], transBar [], encTime [] ]

        encBar =
            encoding
                << position X [ pAggregate opCount, pQuant ]
                << position Y [ pName "weather", pNominal ]
                << color
                    [ mSelectionCondition (selectionName "barSelect")
                        [ mName "weather", mNominal, mScale weatherColours ]
                        [ mStr "#acbf98" ]
                    ]

        specBar =
            asSpec [ width 600, selBar [], transTime [], encBar [], bar [] ]
    in
    toVegaLite [ title "Seattle Weather, 2012-2015" [], data [], vConcat [ specTime, specBar ] ]
```

---

## Smooth Zooming

A similar idea of of presenting both an overview (upper chart) and detailed view (lower chart), this time revealing a greater temporal resolution in the detailed view as the view extent gets smaller.

_Try dragging an interval selection in the upper chart and then changing the zoom extent._

```elm {v l interactive}
smoothZoom : Spec
smoothZoom =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/flights-5k.json"
                [ parse [ ( "date", foDate "" ) ] ]

        trans =
            transform
                << calculateAs "hours(datum.date) + minutes(datum.date) / 60" "time"

        sel =
            selection
                << select "brush" seInterval [ seEncodings [ chX ] ]

        enc1 =
            encoding
                << position X
                    [ pName "time"
                    , pQuant
                    , pBin [ biMaxBins 30 ]
                    , pAxis [ axFormat ".2f" ]
                    ]
                << position Y [ pAggregate opCount, pQuant ]

        spec1 =
            asSpec [ width 700, height 100, sel [], enc1 [], bar [] ]

        enc2 =
            encoding
                << position X
                    [ pName "time"
                    , pQuant
                    , pBin [ biMaxBins 30, biSelectionExtent "brush" ]
                    , pAxis [ axFormat ".2f" ]
                    ]
                << position Y [ pAggregate opCount, pQuant ]

        spec2 =
            asSpec [ width 700, height 100, enc2 [], bar [] ]
    in
    toVegaLite [ data, trans [], vConcat [ spec1, spec2 ] ]
```
