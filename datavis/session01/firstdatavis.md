---
elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

# My first litvis document

```elm {l v}
import VegaLite exposing (..)


hellolitvis : Spec
hellolitvis =
    let
        data =
            dataFromColumns []
                << dataColumn "col1" (nums [ 1, 2, 3, 6 ])
                << dataColumn "col2" (nums [ 45, 32, 2, 100 ])

        enc =
            encoding
                << position X [ pName "col1", pQuant ]
                << position Y [ pName "col2", pQuant ]
    in
    toVegaLite [ data [], enc [], square [] ]
```
