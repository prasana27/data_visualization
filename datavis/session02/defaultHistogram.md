---
follows: imdbRoot
---

A default histogram:

```elm {l v}
histogram : Spec
histogram =
    let
        enc =
            encoding
                << position X [ pName "IMDB_Rating", pOrdinal, pBin [] ]
                << position Y [ pAggregate opCount, pQuant ]
                << color [ mName "IMDB_Rating", mOrdinal ]
    in
    toVegaLite [ data [], enc [], bar [] ]
```
