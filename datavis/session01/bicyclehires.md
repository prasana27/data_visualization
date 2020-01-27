---
elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

```elm {l=hidden}
import VegaLite exposing (..)
```

```elm {v}
myBarchart : Spec
myBarchart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv"

        enc =
            encoding
                << position X [ pName "Month", pTemporal ]
                << position Y [ pName "NumberOfHires", pQuant ]
                << color [ mName "AvHireTime", mQuant ]
    in
    toVegaLite [ width 540, data [], enc [], bar [] ]
```

```elm {v}
myBarchart2 : Spec
myBarchart2 =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv"

        enc =
            encoding
                << position X [ pName "Month", pTemporal ]
                << position Y [ pName "NumberOfHires", pQuant ]
                << color [ mName "AvHireTime", mQuant ]
    in
    toVegaLite [ width 540, data [], enc [], area [] ]
```

```elm {v}
myBarchart3 : Spec
myBarchart3 =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv"

        enc =
            encoding
                << position X [ pName "NumberOfHires", pQuant ]
                << position Y [ pName "AvHireTime", pQuant ]
    in
    toVegaLite [ width 540, height 540, data [], enc [], circle [] ]
```

```elm {v}
myBarchart4 : Spec
myBarchart4 =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/euPolls.json"

        enc =
            encoding
                << position X [ pName "Date", pTemporal ]
                << position Y [ pName "Percent", pQuant ]
                << color [ mName "Answer", mNominal ]
    in
    toVegaLite [ width 540, data [], enc [], bar [] ]
```
