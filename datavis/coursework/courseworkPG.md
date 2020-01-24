---
id: litvis

narrative-schemas:
  - ../narrative-schemas/courseworkPG.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

# Postgraduate Coursework Template

{(questions|}

- Add your question here
- Add your question here
- etc.

{|questions)}

{(visualization|}

This is my vis.

{|visualization)}

{(insights|}

{|insights)}

{(designJustification|}

{|designJustification)}

{(validation|}

{|validation)}

{(references|}

**Haroz, S., and Whitney, D.** (2012) How capacity limits of attention influence information visualization effectiveness. _IEEE Transactions on Visualization and Computer Graphics_, 18(12) pp.2402-2410.

{|references)}
