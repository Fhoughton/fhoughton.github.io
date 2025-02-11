---
layout: post
title:  "Trying Elm, A Functional Web Application Language"
date:   2024-12-06 00:19:13 +0100
categories: Elm Functional Web
---

# What Is Elm?
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Elm_logo.svg/1024px-Elm_logo.svg.png" style="width:25%">

Elm is a functional programming language for creating web applications. It compiles to JavaScript, but provides a functional and typesafe layer that eliminates most runtime errors in JS, allowing more confident and safe programming.

Elm can render webpages, describing html through Elm DOM, allowing the majority of a webpage to be pure Elm.

# Installing Elm
Installing Elm was a breeze; Often when trying new programming languages they only offer source builds, and only for Linux, with Elm the website provides an executable installer for all platforms, as well as packages for certain Linux distros.

# Trying Elm: A Basic Word Counter Webapp
I decided that a simple word counter app would be a good test of Elm's capabilities, so I followed the [official docs](https://elm-lang.org/docs) and worked my way along:

*Starting a project*
```bash
elm init
```

*Running the webserver*
```bash
elm reactor
```

The Elm Architecture divides an application into three parts: Model, Update, and View.

**Model**

The Model holds the application’s state. For the word counter, the state includes the user input and the resulting word count.
```elm
type alias Model =
    { userInput : String }

init : Model
init =
    { userInput = "" }
```

**Update**

The Update function handles all state changes. For this app, it just processes changes to the application input, as the word count isn't stored as a state.
```elm
type Msg
    = UpdateInput String

update : Msg -> Model -> Model
update msg model =
    case msg of
        UpdateInput newInput ->
            { model | userInput = newInput }
```

**View**

The View function defines how the application is rendered as HTML.

```elm
view : Model -> Html Msg
view model =
    let
        wordCount =
            model.userInput
                |> String.trim -- Remove trailing whitespace
                |> String.split "\n" -- Split on newlines, to count newlines with words as a new word
                |> List.concatMap (String.split " ") -- Split spaces within the lines to count space seperated words
                |> List.filter (\word -> word /= "") -- Remove empty items, no word
                |> List.length -- Now count the total, which is our wordcount

        -- Embedded CSS styles for use in the webpage
        embeddedStyles =
            [ ( "body", "font-family: Arial, sans-serif; margin: 20px; background-color: #f9f9f9; color: #333;" )
            , ( "textarea", "width: 100%; max-width: 800px; height: auto; resize: vertical; padding: 10px; border: 1px solid #ccc; border-radius: 4px; font-size: 16px; line-height: 1.5; box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.1);" )
            , ( ".word-count", "font-weight: bold; font-size: 18px;" )
            ]
    in
    -- Construct the html to be displayed to the user
    div []
        [
          -- Embed the css in a style node
          node "style" [] [ text (String.join "\n" (List.map (\(selector, rules) -> selector ++ " { " ++ rules ++ " }") embeddedStyles)) ]
        , textarea
            [ placeholder "Type or paste text here..."
            , value model.userInput
            , rows 10
            , onInput UpdateInput
            , style "margin-bottom" "20px"
            ]
            []
        , div [ Html.Attributes.class "word-count" ] [ text ("Word count: " ++ String.fromInt wordCount) ]
        ]
```

# The Resulting App
The final app was responsive, worked well and was generally pleasant. I enjoyed using Elm and despite some verbosity in its syntax it was very pleasant to work with. The Elm compiler was very useful, catching many classes of errors at compile-time and giving me confidence in the application's stability.
![](/images/elm-wordcounter.png)

<br>
<br>
<br>
Full Code listing:
```elm
module Main exposing (..)

import Browser
import Html exposing (Html, div, text, textarea, node)
import Html.Attributes exposing (placeholder, rows, value, style)
import Html.Events exposing (onInput)

-- MAIN
main =
    Browser.sandbox { init = init, update = update, view = view }

-- MODEL
type alias Model =
    { userInput : String }

init : Model
init =
    { userInput = "" }

-- UPDATE
type Msg
    = UpdateInput String

update : Msg -> Model -> Model
update msg model =
    case msg of
        UpdateInput newInput ->
            { model | userInput = newInput }


-- VIEW
view : Model -> Html Msg
view model =
    let
        wordCount =
            model.userInput
                |> String.trim -- Remove trailing whitespace
                |> String.split "\n" -- Split on newlines, to count newlines with words as a new word
                |> List.concatMap (String.split " ") -- Split spaces within the lines to count space seperated words
                |> List.filter (\word -> word /= "") -- Remove empty items, no word
                |> List.length -- Now count the total, which is our wordcount

        -- Embedded CSS styles for use in the webpage
        embeddedStyles =
            [ ( "body", "font-family: Arial, sans-serif; margin: 20px; background-color: #f9f9f9; color: #333;" )
            , ( "textarea", "width: 100%; max-width: 800px; height: auto; resize: vertical; padding: 10px; border: 1px solid #ccc; border-radius: 4px; font-size: 16px; line-height: 1.5; box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.1);" )
            , ( ".word-count", "font-weight: bold; font-size: 18px;" )
            ]
    in
    -- Construct the html to be displayed to the user
    div []
        [
          -- Embed the css in a style node
          node "style" [] [ text (String.join "\n" (List.map (\(selector, rules) -> selector ++ " { " ++ rules ++ " }") embeddedStyles)) ]
        , textarea
            [ placeholder "Type or paste text here..."
            , value model.userInput
            , rows 10
            , onInput UpdateInput
            , style "margin-bottom" "20px"
            ]
            []
        , div [ Html.Attributes.class "word-count" ] [ text ("Word count: " ++ String.fromInt wordCount) ]
        ]
```