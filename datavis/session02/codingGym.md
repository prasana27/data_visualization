---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml
---

@import "../css/datavis.less"

<!-- Everything above this line should probably be left untouched. -->

# Coding Gym 2 : Lists

_These exercises are designed to help you build confidence programming with Elm. They assume no prior experience of coding in any language other than that arising from previous weeks of this module. This document is designed to be viewed in Atom with litvis so you can add your own code blocks to answer each question._

## 2.1. Lists of items

In the previous gym you created functions that generated single values such as an integer, a floating point number or a String of text. We move on now to consider [lists of items](https://package.elm-lang.org/packages/elm/core/latest/List). Each item can again be a number, some text etc. but now we can create an ordered collection of them.

Lists are indicated with square brackets and items separated with commas. Lists can also be empty. For example:

```elm {l siding}
people : List String
people =
    [ "Ada Lovelace", "Alan Turing", "Jacques Bertin" ]


birthYears : List Int
birthYears =
    [ 1815, 1912, 1918 ]


todoList : List String
todoList =
    []
```

{(task|}

Create separate functions, each in their own code block, that display the following. Remember to include `elm {l raw}` in each code block header so you can see the results. If created successfully, you should see the evaluation of each function appear in the preview window.

- the names of your four favourite animals
- the whole numbers 1 to 10 inclusive
- the first 37 [triangular numbers](https://en.wikipedia.org/wiki/Triangular_number)
- The first 10 numbers in the sequence 1/1, 1/2, 1/3, 1/4 etc.
- The colours of the rainbow
- A list of birds that have four legs

{|task)}

```elm {l raw}
animalList : List String
animalList =
    [ "Eagle", "Butterfly", "Blue whale", "Cheetah" ]


numbers1to10 : List Int
numbers1to10 =
    [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ]


triangularNum : List Int
triangularNum =
    [ 0, 1, 3, 6, 10, 15, 21, 28, 36, 45, 55, 66, 78, 91, 105, 120, 136, 153, 171, 190, 210, 231, 253, 276, 300, 325, 351, 378, 406, 435, 465, 496, 528, 561, 595, 630, 666 ]


numSeq : List Float
numSeq =
    [ 1 / 1, 1 / 2, 1 / 3, 1 / 4, 1 / 5 ]
```

## 2.2. In-built functions to generate lists

Elm has a number of functions that make it easier to create lists. In particular [List.range](https://package.elm-lang.org/packages/elm/core/latest/List#range) for creating a range of whole numbers; [List.repeat](https://package.elm-lang.org/packages/elm/core/latest/List#repeat) for creating a list of repeated items, the [cons operator `::`](<https://package.elm-lang.org/packages/elm/core/latest/List#(::)>) for adding an item to a list and the [++](https://package.elm-lang.org/packages/elm/core/latest/Basics#++) operator for combining two lists. For example:

```elm {l siding raw}
firstTen : List Int
firstTen =
    List.range 1 10


fiveGoldRings : List String
fiveGoldRings =
    List.repeat 5 "gold ring"


newKidOnTheBlock : List String
newKidOnTheBlock =
    "New kid" :: [ "old kid", "another old kid" ]


oddsAndEvens : List Int
oddsAndEvens =
    [ 1, 3, 5, 7, 9 ] ++ [ 2, 4, 6, 8 ]
```

{(task|}

Choosing from the list-generating functions above, create separate functions to do the following:

- List the whole numbers 1 to 61 inclusive
- List the whole numbers between -1000 and -980 inclusive
- Add your own name to this group of musicians: `["John", "Paul", "George", "Ringo"]`
- Create a list of 8 "pieces of eight"
- Create a single list made up by combining a list of three animals, three vegetables and three minerals.

{|task)}

```elm{l raw}
num1to61 : List Int
num1to61 =
    List.range 1 61


numNeg : List Int
numNeg =
    List.range -1000 -980


addName : List String
addName =
    "Prasana" :: [ "John", "Paul", "George", "Ringo" ]


eightPieces : List String
eightPieces =
    List.repeat 8 "pieces of 8"


comboList : List String
comboList =
    [ "lion", "rhino" ] ++ [ "banana", "grapes" ] ++ [ "granite" ]
```

## 2.3. Summarising Lists

Elm has some in-built functions that can summarise the contents of a list including [List.length](https://package.elm-lang.org/packages/elm/core/latest/List#length), [List.sum](https://package.elm-lang.org/packages/elm/core/latest/List#sum) and [List.product](https://package.elm-lang.org/packages/elm/core/latest/List#product). For example:

```elm {l siding raw}
firstTenTotal : Int
firstTenTotal =
    List.sum (List.range 1 10)


numberOfAnimals : Int
numberOfAnimals =
    List.length [ "cat", "dog", "giraffe" ]


power : Int
power =
    List.product [ 2, 3, 4 ]
```

{(task|}

Using the list-summarising functions above, create separate functions to do the following:

- Calculate the total of the numbers 1 to 61
- Calculate the number of items in this combined list `[] ++ ["John", "Paul", "George", "Ringo"]`
- Calculate the mean (average) of all integers from 1 to 61
- Calculate 6 factorial (equivalent to `6 * 5 * 4 * 3 * 2 * 1`)

{|task)}

```elm{l raw}
sumNum : Int
sumNum =
    List.sum (List.range 1 61)


numList : Int
numList =
    List.length ([] ++ [ "John", "Paul", "George", "Ringo" ])


avgNum : Int
avgNum =
    sumNum // List.length num1to61
```
