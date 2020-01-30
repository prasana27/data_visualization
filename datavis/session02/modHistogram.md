---
follows: imdbRoot
---

A mod histogram:

```elm {l}
goldenRatio : Float
goldenRatio =
    1.618
```

```elm {l v}
histogram : Spec
histogram =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        enc =
            encoding
                << position X
                    [ pName "IMDB_Rating"
                    , pOrdinal
                    , pScale [ scPaddingInner 0.25 ]
                    , pBin [ biStep 1 ]
                    , pAxis [ axTitle "" ]
                    ]
                << position Y [ pAggregate opCount, pQuant, pAxis [ axTitle "", axValues (nums [ 500 ]) ] ]
                << color [ mName "IMDB_Rating", mOrdinal ]
    in
    toVegaLite [ height (300 / goldenRatio), width 300, cfg [], data [], enc [], bar [], bar [ maFillOpacity 0.5 ] ]
```
