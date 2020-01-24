---
follows: gallery
id: litvis
---

@import "../css/datavis.less"

# Labelling and Annotation

## Labelled bars

Text annotations can be added by creating a separate layer containing text marks.
Note that because bar position and text position match (subject to text alignment), we only need to specify the position encoding (`enc`) once.

```elm {v l}
barLabels : Spec
barLabels =
    let
        data =
            dataFromColumns []
                << dataColumn "pet" (strs [ "Cats", "Dogs", "Guinea pigs" ])
                << dataColumn "value" (nums [ 28, 55, 43 ])

        enc =
            encoding
                << position X [ pName "value", pQuant, pTitle "" ]
                << position Y [ pName "pet", pNominal, pTitle "" ]

        encText =
            encoding
                << text [ tName "value", tQuant ]

        specText =
            asSpec
                [ textMark
                    [ maAlign haRight -- Align text right
                    , maBaseline vaMiddle -- Centre vertical alignment in each bar
                    , maDx -3 -- Shift left 3 pixels
                    , maColor "white" -- Contrast with bar colour
                    ]
                , encText []
                ]
    in
    toVegaLite [ data [], enc [], layer [ asSpec [ bar [] ], specText ] ]
```

---

## Overlaid mean

Monthly precipitation in Seattle with the annual mean precipitation overlaid using a [rule](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#rule) mark. This is created by specifying two [layers](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#layer) – one for the bars and one for the summary line.

```elm {v l}
meanOverlay : Spec
meanOverlay =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/seattle-weather.csv"

        encBar =
            encoding
                << position X
                    [ pName "date"
                    , pTimeUnit month
                    , pAxis [ axTitle "", axLabelAngle 0 ]
                    , pOrdinal
                    ]
                << position Y [ pName "precipitation", pAggregate opMean, pQuant ]

        specBar =
            asSpec [ encBar [], bar [] ]

        encLine =
            encoding
                << position Y [ pName "precipitation", pAggregate opMean, pQuant ]

        specLine =
            asSpec [ rule [ maSize 2, maColor "hsl(0,30%,50%)" ], encLine [] ]
    in
    toVegaLite [ width 300, data [], layer [ specBar, specLine ] ]
```

---

## Threshold overlay

Bar chart that highlights values beyond a threshold. The [PM2.5 value](https://laqm.defra.gov.uk/public-health/pm25.html) (a measure of air pollution) for Beijing observed over 15 days, highlighting the days when PM2.5 level is hazardous to human health.

This makes use of multiple layers. A layer containing the blue bars is combined with one containing just the red bar segments that exceed the threshold (together they are `layer0` below). Further layers with the threshold line and the text annotation are combined as `layer1`. Both `layer0` and `layer1` are combined in the final specification.

```elm {v l highlight=[38,39,61,62,64]}
thresholdBars : Spec
thresholdBars =
    let
        data =
            dataFromColumns []
                << dataColumn "day" (nums (List.map toFloat (List.range 1 15)))
                << dataColumn "pm25" (nums [ 54.8, 112.1, 63.6, 37.6, 79.7, 137.9, 120.1, 103.3, 394.8, 199.5, 72.3, 51.1, 112.0, 174.5, 130.5 ])

        thresholdData =
            dataFromRows []
                << dataRow [ ( "ThresholdValue", num 300 ), ( "Threshold", str "hazardous" ) ]

        encBar =
            encoding
                << position X [ pName "day", pAxis [ axLabelAngle 0 ], pOrdinal ]
                << position Y [ pName "pm25", pQuant ]

        specBar =
            asSpec [ bar [], encBar [] ]

        trans =
            transform
                << filter (fiExpr "datum.pm25 >= 300")
                << calculateAs "300" "baseline"

        encUpperBar =
            encoding
                -- Bars that exceed threshold have a bottom (baseline) and top (pm25) position
                << position X [ pName "day", pAxis [ axLabelAngle 0 ], pOrdinal ]
                << position Y [ pName "baseline", pQuant ]
                << position Y2 [ pName "pm25" ]

        specUpperBar =
            asSpec [ trans [], bar [ maColor "hsl(0,40%,50%)" ], encUpperBar [] ]

        layer0 =
            asSpec [ data [], layer [ specBar, specUpperBar ] ]

        specRule =
            asSpec [ rule [], encRule [] ]

        encRule =
            encoding
                << position Y [ pName "ThresholdValue", pQuant ]

        specText =
            asSpec [ textMark [ maAlign haRight, maDx -2, maDy -4 ], encText [] ]

        encText =
            encoding
                << position X [ pWidth ]
                << position Y [ pName "ThresholdValue", pTitle "PM2.5 concentration", pQuant ]
                << text [ tName "Threshold", tOrdinal ]

        layer1 =
            asSpec [ thresholdData [], layer [ specRule, specText ] ]
    in
    toVegaLite [ layer [ layer0, layer1 ] ]
```

---

## Labelled lines

Atmospheric carbon dioxide by decade. As we are positioning each decade line according to its carbon dioxide values, adding a text label for each allows us to identify which line corresponds to which decade.

```elm {v l}
label3 : Spec
label3 =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/co2-concentration.csv"

        trans =
            transform
                << calculateAs "year(datum.Date)" "year"
                << calculateAs "month(datum.Date)" "month"
                << calculateAs "floor(datum.year / 10)" "decade"
                << calculateAs "(datum.year % 10) + (datum.month / 12)" "scaledDate"

        encPosition =
            encoding
                << position X
                    [ pName "scaledDate"
                    , pAxis [ axTitle "Year into decade", axTickCount 10 ]
                    , pQuant
                    ]
                << position Y
                    [ pName "CO2"
                    , pScale [ scZero False ]
                    , pTitle "CO2 concentration in ppm"
                    , pQuant
                    ]

        encLine =
            encoding
                << detail [ dName "decade", dOrdinal ]

        specLine =
            asSpec [ line [ maOrient moVertical ], encLine [] ]

        transText =
            transform
                << aggregate [ opAs (opArgMin Nothing) "scaledDate" "aggregated" ] [ "decade" ]
                << calculateAs "datum.aggregated.scaledDate" "scaledDate"
                << calculateAs "datum.aggregated.CO2" "CO2"

        encText =
            encoding
                << text [ tName "aggregated.year", tNominal ]

        specText =
            asSpec
                [ transText []
                , textMark [ maAlign haLeft, maBaseline vaTop, maDx 2, maDy 3 ]
                , encText []
                ]
    in
    toVegaLite
        [ width 600
        , height 300
        , data []
        , trans []
        , encPosition []
        , layer [ specLine, specText ]
        ]
```

---

## Annotated time periods

The population of the German city of Falkensee over time with annotated time periods highlighted.

```elm {v l}
falkensee : Spec
falkensee =
    let
        data =
            dataFromColumns []
                << dataColumn "year" (strs [ "1875", "1890", "1910", "1925", "1933", "1939", "1946", "1950", "1964", "1971", "1981", "1985", "1989", "1990", "1991", "1992", "1993", "1994", "1995", "1996", "1997", "1998", "1999", "2000", "2001", "2002", "2003", "2004", "2005", "2006", "2007", "2008", "2009", "2010", "2011", "2012", "2013", "2014" ])
                << dataColumn "population" (nums [ 1309, 1558, 4512, 8180, 15915, 24824, 28275, 29189, 29881, 26007, 24029, 23340, 22307, 22087, 22139, 22105, 22242, 22801, 24273, 25640, 27393, 29505, 32124, 33791, 35297, 36179, 36829, 37493, 38376, 39008, 39366, 39821, 40179, 40511, 40465, 40905, 41258, 41777 ])

        highlightData =
            dataFromColumns []
                << dataColumn "start" (strs [ "1933", "1948" ])
                << dataColumn "end" (strs [ "1945", "1989" ])
                << dataColumn "event" (strs [ "Nazi Rule", "GDR (East Germany)" ])

        encRects =
            encoding
                << position X [ pName "start", pTimeUnit year, pAxis [], pTemporal ]
                << position X2 [ pName "end", pTimeUnit year, pTemporal ]
                << color [ mName "event", mNominal ]

        specRects =
            asSpec [ highlightData [], rect [ maOpacity 0.5 ], encRects [] ]

        encPopulation =
            encoding
                << position X
                    [ pName "year"
                    , pTimeUnit year
                    , pAxis [ axTitle "", axGrid False, axDomain False ]
                    , pTemporal
                    ]
                << position Y
                    [ pName "population", pQuant ]

        specLine =
            asSpec
                [ encPopulation []
                , line
                    [ maColor "#333"
                    , maInterpolate miMonotone
                    , maPoint (pmMarker [ maColor "black", maOpacity 0.5 ])
                    ]
                ]
    in
    toVegaLite
        [ width 500
        , data []
        , layer [ specRects, specLine ]
        ]
```
