---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml
---

@import "../css/datavis.less"

<!-- Everything above this line should probably be left untouched. -->

# Coding Gym 4 : Piping and Anonymous Functions

_These exercises are designed to help you build confidence programming with Elm. They assume no prior experience of coding in any language other than that arising from previous weeks of this module. This document is designed to be viewed in Atom with litvis so you can add your own code blocks to answer each question._

## 4.1. Pipes

So far you have used brackets to control the order of expression evaluation. For example,

```elm
cos (sqrt (7 + 8))
```

mean first evaluate 7+8, then find its square root, then finally calculate the cosine of the that value.

An alternative to using brackets is the [pipe operator](https://package.elm-lang.org/packages/elm/core/latest/Basics#always), symbolised by `|>`. This allows you to control the order of evaluation in a more natural left-to-right reading order without the need to nest brackets. Using the right pipe operator, the expression above could be expressed as

```elm
7 + 8 |> sqrt |> cos
```

More generally, `b (a)` is equivalent `a |> b`. This process of 'chaining' functions is particularly useful when expressions become more complicated.

{(task|}

Modify each of the functions q1-q5 below so they use the right pipe operator in place of brackets. After doing so, the result of the evaluation should remain unchanged.

{|task)}

```elm {l r}
q1 : Float
q1 =
    sqrt (2 + 2)
```

```elm {l r}
q2 : List String
q2 =
    List.reverse (String.words (String.toUpper "Father Charles goes down and ends battle"))
```

```elm {l r}
q3 : List Float
q3 =
    List.map toFloat (List.range 1 10)
```

```elm {l r}
q4 : Float
q4 =
    let
        half x =
            x / 2
    in
    List.sum (List.map half (List.map toFloat (List.range 1 10)))
```

```elm {l}
q5 : Int -> Int -> Int
q5 minVal maxVal =
    let
        cube x =
            x * x * x
    in
    List.sum (List.map cube (List.range minVal maxVal))
```

_To see the result of function q5, we can test it with an example pair of numbers:_

```elm {l r}
testQ5 : Int
testQ5 =
    q5 4 15
```

## 4.2 Anonymous functions

So far we have created functions by giving them a name. This is useful especially when we need to re-use functions on more than one occasion. But sometimes, we might only wish to use a function once. We can create more compact _anonymous functions_ by listing parameters after the `\` symbol and the function itself after the `->` symbol. For example:

```elm
inc : Int -> Int
inc n =
    n + 1
```

can be expressed as the anonymous function:

```elm
\n -> n + 1
```

This is especially common when providing a function to `List.map`:

```elm {l}
incByOne : List Int -> List Int
incByOne myList =
    List.map (\n -> n + 1) myList
```

```elm {l raw}
newList : List Int
newList =
    incByOne [ 10, 20, 30, 40 ]
```

{(task|}

Re-express the map functions in the following (from last week) as anonymous functions (that is, you should no longer need a separate `let...in` but instead place the anonymous function directly after `List.map`)

{|task)}

```elm {l}
inc10 : List Int -> List Int
inc10 myList =
    let
        add10 a =
            a + 10
    in
    List.map add10 myList
```

```elm {l}
double : List Int -> List Int
double myList =
    let
        times2 a =
            a * 2
    in
    List.map times2 myList
```

```elm {l}
sumSq : List Float -> Float
sumSq myList =
    let
        sq num =
            num * num
    in
    List.sum (List.map sq myList)
```

```elm {l}
potato : List String -> List String
potato myList =
    let
        addSpud word =
            word ++ " potato"
    in
    List.map addSpud myList
```
