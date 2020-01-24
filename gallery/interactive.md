---
follows: gallery
id: litvis
---

@import "../css/datavis.less"

# Interactive Charts

_For all interactive examples on this page, the code block header must include the `interactive` keyword._

## Default Tooltip

Adding `maTooltip ttEncoding` to a visible mark will enable tooltips describing the data that have been encoded in that mark.

```elm {v l interactive}
defaultTooltip : Spec
defaultTooltip =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
    in
    toVegaLite [ width 400, height 400, data [], enc [], point [ maTooltip ttEncoding ] ]
```

---

## Customised Tooltips

The data value shown in a tooltip can be customised by encoding any data field with the [tooltip](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#tooltip) channel.

```elm {v l interactive highlight=11}
customTooltip : Spec
customTooltip =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << tooltip [ tName "Origin", tNominal ]
    in
    toVegaLite [ width 400, height 400, data [], enc [], point [] ]
```

To show multiple values, encode with the [tooltips](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#tooltips) (plural) channel. The format of the values shown in the tooltip can be controlled with [tFormat](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#tFormat).

```elm {v l interactive highlight=[16-24]}
topTrumps : Spec
topTrumps =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        trans =
            transform
                -- Convert to Kg
                << calculateAs "datum.Weight_in_lbs  * 0.45359" "Weight"

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << tooltips
                    [ [ tName "Name", tNominal ]
                    , [ tName "Origin", tNominal ]
                    , [ tName "Year", tTemporal, tTimeUnit year ]
                    , [ tName "Cylinders", tQuant ]

                    -- Show weight (in Kgs) without decimal places and a thousands separator
                    , [ tName "Weight", tQuant, tFormat ",.0f" ]
                    ]
    in
    toVegaLite [ width 400, height 400, data [], trans [], enc [], point [] ]
```

---

## Hyperlinked Chart Marks

Marks can be associated with clickable hyperlinks by encoding them with the [hyperlink](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#hyperlink) channel. _Try clicking on any of the circle symbols below:_

```elm {v l interactive highlight=19}
hyperScatter : Spec
hyperScatter =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        trans =
            transform
                << calculateAs "'https://www.google.com/search?q=' + datum.Name" "url"

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << tooltips
                    [ [ tName "Name", tNominal ]
                    , [ tName "Year", tTemporal, tTimeUnit year ]
                    ]
                << hyperlink [ hName "url", hNominal ]
    in
    toVegaLite [ width 400, height 400, data [], trans [], enc [], point [] ]
```

---

## Zoomable Scatterplot

You can make any chart zoomable by binding its positional scales to an interval selection. Additionally, this example sets an initial zoomed-in view by setting the positional x and y scales to be a small part of their respective data domains.
_Drag to pan. Zoom in or out with mouse-wheel/zoom gesture._

```elm {v l interactive highlight=[7,8,17,24]}
zoomScatter : Spec
zoomScatter =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        sel =
            selection << select "myGrid" seInterval [ seBindScales ]

        enc =
            encoding
                << position X
                    [ pName "Horsepower"
                    , pQuant

                    -- Set initial x-axis scaling (will change with zooming)
                    , pScale [ scDomain (doNums [ 75, 150 ]) ]
                    ]
                << position Y
                    [ pName "Miles_per_Gallon"
                    , pQuant

                    -- Set initial y-axis scaling (will change with zooming)
                    , pScale [ scDomain (doNums [ 20, 40 ]) ]
                    ]
                << size [ mName "Cylinders", mQuant ]
    in
    toVegaLite [ data [], sel [], enc [], circle [] ]
```

---

## Interval highlighting

Drag out a rectangular area ("interval selection") to highlight points that fall within the rectangle.

```elm {v l interactive highlight=[7-9,16-21]}
intervalHighlight : Spec
intervalHighlight =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        sel =
            selection
                << select "myBrush" seInterval []

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << color
                    [ mSelectionCondition (selectionName "myBrush")
                        -- Encoding when selected:
                        [ mName "Cylinders", mOrdinal ]
                        -- Encoding when not selected:
                        [ mStr "grey" ]
                    ]
    in
    toVegaLite [ width 400, height 400, data [], sel [], enc [], point [] ]
```

---

## Hover Highlighting

_Mouse over individual points or select multiple points with the shift key_

```elm {v l interactive}
hoverHighlight : Spec
hoverHighlight =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        sel =
            selection
                << select "myPaintbrush"
                    seMulti
                    [ seOn "mouseover", seNearest True, seEmpty ]

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << fillOpacity
                    [ mSelectionCondition (selectionName "myPaintbrush")
                        [ mNum 1 ]
                        [ mNum 0.3 ]
                    ]
                << color
                    [ mSelectionCondition (selectionName "myPaintbrush")
                        [ mStr "rgb(230,110,90)" ]
                        [ mStr "rgb(80,80,80)" ]
                    ]
                << size
                    [ mSelectionCondition (selectionName "myPaintbrush")
                        [ mNum 200 ]
                        [ mNum 50 ]
                    ]
    in
    toVegaLite [ width 400, height 400, data [], sel [], enc [], point [ maFilled True ] ]
```

---

## Selection with Slider

Input GUI elements such as [sliders](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#iRange) and [checkboxes](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#iCheckbox) can be used to select and filter data.

```elm {v l interactive}
sliderSelection : Spec
sliderSelection =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json"

        trans =
            transform
                << calculateAs "year(datum.Year)" "Year"

        transSelection =
            transform
                << filter (fiSelection "CylYr")

        sel =
            selection
                << select "CylYr"
                    seSingle
                    [ seFields [ "Cylinders", "Year" ]
                    , seBind
                        [ iRange "Cylinders" [ inName "Cylinders ", inMin 3, inMax 8, inStep 1 ]
                        , iRange "Year" [ inName "Year ", inMin 1970, inMax 1981, inStep 1 ]
                        ]
                    , seInit [ ( "Cylinders", num 4 ), ( "Year", num 1970 ) ]
                    , seEmpty
                    ]

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << color
                    [ mSelectionCondition (selectionName "CylYr")
                        [ mName "Origin", mNominal ]
                        [ mStr "black" ]
                    ]
                << opacity
                    [ mSelectionCondition (selectionName "CylYr")
                        [ mNum 0.9 ]
                        [ mNum 0.1 ]
                    ]
                << size
                    [ mSelectionCondition (selectionName "CylYr")
                        [ mNum 100 ]
                        [ mNum 50 ]
                    ]
    in
    toVegaLite [ width 400, height 400, data [], trans [], enc [], sel [], circle [] ]
```

---

## Hover and Click Highlight

Move pointer over bars to highlight and click a bar to select. Shift-click allows you to select multiple bars.

```elm {v l interactive highlight=[9-12,19-22,24-31,35]}
hoverAndHighlight : Spec
hoverAndHighlight =
    let
        data =
            dataFromColumns []
                << dataColumn "a" (strs [ "A", "B", "C", "D", "E", "F", "G", "H", "I" ])
                << dataColumn "b" (nums [ 28, 55, 43, 91, 81, 53, 19, 87, 52 ])

        sel =
            selection
                << select "highlight" seSingle [ seOn "mouseover", seEmpty ]
                << select "select" seMulti []

        enc =
            encoding
                << position X [ pName "a", pOrdinal, pTitle "" ]
                << position Y [ pName "b", pQuant, pTitle "" ]
                << fillOpacity
                    [ mSelectionCondition (selectionName "select")
                        [ mNum 1 ]
                        [ mNum 0.3 ]
                    ]
                << strokeWidth
                    [ mDataCondition
                        [ ( and (selected "select") (expr "length(data('select_store')) > 0")
                          , [ mNum 2 ]
                          )
                        , ( selected "highlight", [ mNum 1 ] )
                        ]
                        [ mNum 0 ]
                    ]
    in
    toVegaLite
        [ data []
        , sel []
        , enc []
        , bar [ maFill "#4C78A8", maStroke "black", maCursor cuPointer ]
        ]
```

---

## Dynamic Data Calculation

Aggregate operations such as counts and averages are subject to any data filtering transformations. If the filtering is dependent on a selection, we can create dynamic positioning of marks. Here the average of a selection is represented by a horizontal rule that responds to a dynamic selection. _Drag over bars to update selection average._

```elm {v l interactive highlight=[7-9,13,27-29]}
dynamicAverage : Spec
dynamicAverage =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/seattle-weather.csv"

        sel =
            selection
                << select "myBrush" seInterval [ seEncodings [ chX ] ]

        encPosition =
            encoding
                << position Y [ pName "precipitation", pQuant, pAggregate opMean ]

        enc1 =
            encoding
                << position X [ pName "date", pOrdinal, pTimeUnit month ]
                << opacity
                    [ mSelectionCondition (selectionName "myBrush")
                        [ mNum 1 ]
                        [ mNum 0.7 ]
                    ]

        spec1 =
            asSpec [ sel [], bar [], enc1 [] ]

        trans =
            transform
                << filter (fiSelection "myBrush")

        spec2 =
            asSpec [ trans [], rule [ maColor "firebrick", maSize 3 ] ]
    in
    toVegaLite [ data [], encPosition [], layer [ spec1, spec2 ] ]
```

---

## Filtered Selection

For continuous marks like [area](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#area), [line](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#line) and [trail](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#area) we can highlight parts of it by layering a filtered subset on top of the original data. In the example below we select an interval, but only in the X position channel, allowing a time period to be highlighted (_drag a rectangle to see the effect_).

```elm {v l interactive highlight=[7-9,11-13,25,26,28,29]}
timeBrush : Spec
timeBrush =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/unemployment-across-industries.json"

        trans =
            transform
                << filter (fiSelection "myBrush")

        sel =
            selection
                << select "myBrush" seInterval [ seEncodings [ chX ] ]

        enc =
            encoding
                << position X
                    [ pName "date"
                    , pTemporal
                    , pTimeUnit yearMonth
                    , pAxis [ axTitle "", axFormat "%Y" ]
                    ]
                << position Y [ pName "count", pQuant, pAggregate opSum ]

        specBackground =
            asSpec [ sel [], area [ maOpacity 0.2 ] ]

        specHighlight =
            asSpec [ trans [], area [ maOpacity 1 ] ]
    in
    toVegaLite [ width 400, data [], enc [], layer [ specBackground, specHighlight ] ]
```

---

## Interactive Legend

Using [seBindLegend](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#seBindLegend) we can create two-way interaction between a legend and the data it summarises. Clicking on a data item highlights it in the legend and double-clicking on a legend entry highlights the associated data.

```elm {v l interactive}
interactiveLegend : Spec
interactiveLegend =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/westMidlands/westMidsCrimesAggregated.tsv"

        cScale =
            categoricalDomainMap
                [ ( "Anti-social behaviour", "rgb(59,118,175)" )
                , ( "Burglary", "rgb(81,157,62)" )
                , ( "Criminal damage and arson", "rgb(141,106,184)" )
                , ( "Drugs", "rgb(239,133,55)" )
                , ( "Robbery", "rgb(132,88,78)" )
                , ( "Vehicle crime", "rgb(213,126,190)" )
                ]

        encHighlight =
            encoding
                << position X [ pName "month", pMType Temporal, pTitle "" ]
                << position Y [ pName "reportedCrimes", pMType Quantitative, pTitle "Reported crimes" ]
                << color
                    [ mSelectionCondition (selectionName "mySelection")
                        [ mName "crimeType", mMType Nominal, mScale cScale ]
                        [ mStr "black" ]
                    ]
                << opacity
                    [ mSelectionCondition (selectionName "mySelection")
                        [ mNum 1 ]
                        [ mNum 0.1 ]
                    ]

        sel =
            selection
                << select "mySelection"
                    seMulti
                    [ seOn "click"
                    , seBindLegend [ blField "crimeType", blEvent "dblclick" ]
                    ]
    in
    toVegaLite [ width 540, data [], sel [], encHighlight [], circle [] ]
```

---

## Dynamic Annotations

For more detailed comparison between values, we can position a vertical rule mark by filtering dates with a single valued selection. We can annotate the rule by showing point and text marks for that selection. The original lines, filtered rule and filtered point and text marks are all layered on top of each other. Adapted from [this example by Jake Vanderplas](https://bl.ocks.org/jakevdp/a414950f61e4b224765f2439dd1f09b9)

```elm {v l interactive}
dynamicAnnotation : Spec
dynamicAnnotation =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/stocks.csv"

        enc1 =
            encoding
                << position X [ pName "date", pTemporal, pTitle "" ]
                << position Y [ pName "price", pQuant, pAxis [ axFormat "$.0f", axTitle "" ] ]
                << color [ mName "symbol", mNominal, mTitle "Tech Company" ]

        spec1 =
            asSpec
                [ enc1 []
                , layer
                    [ asSpec [ line [] ]
                    , asSpec [ sel1_2 [], enc1_2 [], point [] ]
                    ]
                ]

        enc1_2 =
            encoding
                << opacity [ mSelectionCondition (expr "myTooltip") [ mNum 1 ] [ mNum 0 ] ]

        sel1_2 =
            selection
                << select "myTooltip"
                    seSingle
                    [ seNearest True, seOn "mouseover", seEncodings [ chX ], seEmpty ]

        spec2 =
            asSpec [ trans2 [], layer [ spec2_1, spec2_2 ] ]

        trans2 =
            transform << filter (fiSelection "myTooltip")

        spec2_1 =
            asSpec [ enc2_1 [], rule [ maColor "gray" ] ]

        enc2_1 =
            encoding << position X [ pName "date", pTemporal ]

        spec2_2 =
            asSpec [ enc2_2 [], textMark [ maAlign haLeft, maDx 5, maDy -5 ] ]

        enc2_2 =
            encoding
                << position X [ pName "date", pTemporal ]
                << position Y [ pName "price", pQuant ]
                << text [ tName "price", tQuant ]
                << color [ mName "symbol", mNominal ]
    in
    toVegaLite [ width 600, height 350, data [], layer [ spec1, spec2 ] ]
```

Instead of labelling the vertical rule directly we can add a tooltip associated with the selection indicated by the rule.

```elm {v l interactive}
dynamicTooltips : Spec
dynamicTooltips =
    let
        desc =
            description "Displays tooltips for all stock prices of the hovered time"

        stockData =
            dataFromUrl "https://vega.github.io/vega-lite/data/stocks.csv"
                [ parse [ ( "date", foDate "" ) ] ]

        enc =
            encoding
                << position X [ pName "date", pTemporal ]

        transSelFilter =
            transform
                << filter (fiSelection "hover")

        enc1 =
            encoding
                << position Y [ pName "price", pQuant ]
                << color [ mName "symbol", mNominal ]

        spec1 =
            asSpec
                [ enc1 []
                , layer
                    [ asSpec [ line [] ]
                    , asSpec [ transSelFilter [], point [] ]
                    ]
                ]

        sel =
            selection
                << select "hover"
                    seSingle
                    [ seFields [ "date" ]
                    , seEmpty
                    , seOn "mouseover"
                    , seClear "mouseout"
                    , seNearest True
                    ]

        transPivot =
            transform
                << pivot "symbol" "price" [ piGroupBy [ "date" ] ]

        enc2 =
            encoding
                << opacity [ mSelectionCondition (expr "hover") [ mNum 0.3 ] [ mNum 0 ] ]
                << tooltips
                    [ [ tName "AAPL", tQuant ]
                    , [ tName "AMZN", tQuant ]
                    , [ tName "GOOG", tQuant ]
                    , [ tName "IBM", tQuant ]
                    , [ tName "MSFT", tQuant ]
                    ]

        spec2 =
            asSpec [ sel [], transPivot [], enc2 [], rule [] ]
    in
    toVegaLite [ width 600, height 300, stockData, enc [], layer [ spec1, spec2 ] ]
```

---

## Dynamic Scaling

Instead of simply reporting the data under the current selection, we can use the selection to rescale the data, so they show values relative to those under the selection. This uses [lookupSelection](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#lookupSelection) to join the data under the selection to the primary data source.

```elm {v l interactive}
dynamicRescaling : Spec
dynamicRescaling =
    let
        stockData =
            dataFromUrl "https://vega.github.io/vega-lite/data/stocks.csv"
                [ csv, parse [ ( "date", foDate "" ) ] ]

        sel =
            selection
                << select "index"
                    seSingle
                    [ seOn "mouseover"
                    , seEncodings [ chX ]
                    , seNearest True
                    , seInit [ ( "x", dt [ dtYear 2005, dtMonthNum Jan, dtDate 1 ] ) ]
                    ]

        trans =
            transform
                << lookupSelection "symbol" "index" "symbol"
                << calculateAs "datum.index && datum.index.price > 0 ? (datum.price - datum.index.price)/datum.index.price : 0"
                    "indexed_price"

        pointEnc =
            encoding
                << position X [ pName "date", pTemporal, pAxis [] ]

        pointSpec =
            asSpec [ sel [], pointEnc [], point [ maOpacity 0 ] ]

        lineEnc =
            encoding
                << position X [ pName "date", pTemporal, pAxis [] ]
                << position Y
                    [ pName "indexed_price"
                    , pQuant
                    , pAxis [ axTitle "Change relative to selected date", axFormat "%" ]
                    ]
                << color [ mName "symbol", mNominal ]

        lineSpec =
            asSpec [ trans [], lineEnc [], line [] ]

        ruleTrans =
            transform
                << filter (fiSelection "index")

        ruleEnc =
            encoding
                << position X [ pName "date", pTemporal, pAxis [] ]
                << color [ mStr "firebrick" ]

        textEnc =
            encoding
                << position Y [ pNum 310 ]
                << text [ tName "date", tTemporal, tTimeUnit yearMonth ]

        labelledRuleSpec =
            asSpec
                [ ruleTrans []
                , ruleEnc []
                , layer
                    [ asSpec [ rule [ maStrokeWidth 0.5 ] ]
                    , asSpec [ textEnc [], textMark [ maAlign haCenter, maFontWeight W100 ] ]
                    ]
                ]
    in
    toVegaLite [ width 650, height 300, stockData, layer [ pointSpec, lineSpec, labelledRuleSpec ] ]
```

---

## Multi-series Tooltips

Here we show tooltips for multiple series under the current selection.

```elm {v l interactive}
multiSeriesTooltip : Spec
multiSeriesTooltip =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/seattle-weather.csv"

        cfg =
            configure
                << configuration (coAxisY [ axcoMinExtent 30 ])

        enc =
            encoding
                << position X [ pName "date", pTemporal, pTimeUnit yearMonthDate, pTitle "" ]
                << tooltips
                    [ [ tName "date", tTemporal, tTimeUnit yearMonthDate ]
                    , [ tName "temp_max", tQuant ]
                    , [ tName "temp_min", tQuant ]
                    ]

        enc1 =
            encoding
                << position Y [ pName "temp_max", pQuant, pTitle "temperature" ]

        spec1 =
            asSpec [ enc1 [], line [ maColor "orange", maStrokeWidth 1 ] ]

        enc2 =
            encoding
                << position Y [ pName "temp_min", pQuant ]

        spec2 =
            asSpec [ enc2 [], line [ maColor "red", maStrokeWidth 1 ] ]

        sel =
            selection
                << select "hover" seSingle [ seOn "mouseover", seEmpty ]

        enc3 =
            encoding
                << color
                    [ mSelectionCondition (VegaLite.not (selectionName "hover"))
                        [ mStr "transparent" ]
                        []
                    ]

        spec3 =
            asSpec [ sel [], enc3 [], rule [] ]
    in
    toVegaLite
        [ width 600
        , cfg []
        , data []
        , enc []
        , layer [ spec1, spec2, spec3 ]
        ]
```

---

## Interactive Map

Moving the mouse over an airport shows its direct connections.

```elm {v l interactive}
connections : Spec
connections =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        backdropSpec =
            asSpec
                [ dataFromUrl "https://vega.github.io/vega-lite/data/us-10m.json" [ topojsonFeature "states" ]
                , geoshape [ maFill "#ddd", maStroke "#fff" ]
                ]

        lineTrans =
            transform
                << filter (fiSelection "mySelection")
                << lookup "origin"
                    (dataFromUrl "https://vega.github.io/vega-lite/data/airports.csv" [])
                    "iata"
                    (luAs "o")
                << lookup "destination"
                    (dataFromUrl "https://vega.github.io/vega-lite/data/airports.csv" [])
                    "iata"
                    (luAs "d")

        lineEnc =
            encoding
                << position Longitude [ pName "o.longitude", pQuant ]
                << position Latitude [ pName "o.latitude", pQuant ]
                << position Longitude2 [ pName "d.longitude" ]
                << position Latitude2 [ pName "d.latitude" ]

        lineSpec =
            asSpec
                [ dataFromUrl "https://vega.github.io/vega-lite/data/flights-airport.csv" []
                , lineTrans []
                , lineEnc []
                , rule [ maColor "black", maOpacity 0.35 ]
                ]

        airportTrans =
            transform
                << aggregate [ opAs opCount "" "routes" ] [ "origin" ]
                << lookup "origin"
                    (dataFromUrl "https://vega.github.io/vega-lite/data/airports.csv" [])
                    "iata"
                    (luFields [ "state", "latitude", "longitude" ])
                << filter (fiExpr "datum.state !== 'PR' && datum.state !== 'VI'")

        airportEnc =
            encoding
                << position Longitude [ pName "longitude", pQuant ]
                << position Latitude [ pName "latitude", pQuant ]
                << size [ mName "routes", mQuant, mScale [ scRange (raNums [ 0, 1000 ]) ], mLegend [] ]
                << order [ oName "routes", oQuant, oSort [ soDescending ] ]

        sel =
            selection
                << select "mySelection" seSingle [ seOn "mouseover", seNearest True, seEmpty, seFields [ "origin" ] ]

        airportSpec =
            asSpec
                [ dataFromUrl "https://vega.github.io/vega-lite/data/flights-airport.csv" []
                , airportTrans []
                , sel []
                , circle []
                , airportEnc []
                ]
    in
    toVegaLite
        [ cfg []
        , width 700
        , height 400
        , projection [ prType albersUsa ]
        , layer [ backdropSpec, lineSpec, airportSpec ]
        ]
```
