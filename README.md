# elm-firebase

An adapter to use Firebase with Elm 0.18

> Warning: This library makes heavy use of native code, and therefore has a high likelyhood of causing runtime errors. Use at your own risk.

## A note about this fork

This repository, ucode/elm-firebase, is a fork of pairshaped/elm-firebase. The upstream repository has several pending pull requests that fix bugs we were running into, so we merged them in this repository.

We do not plan to update this library to 0.19, but it is likely that a 0.19 elm-firebase library will be created eventually.

## Alternatives

- [ports](https://guide.elm-lang.org/interop/ports.html) (all Elm versions)
- [ThomasWeiser/elmfire](https://github.com/ThomasWeiser/elmfire) (Elm 0.17)

## Getting started

1. Add this library to your `elm-package.json`
2. Use [elm-github-install](https://github.com/gdotdesign/elm-github-install) to download the code
3. Load the Firebase JS SDK in your HTML

### `elm-package.json` example

```json
{
  "dependencies": {
    "ucode/elm-firebase": "x.y.z <= v < x.y.z"
  },
  "dependency-sources": {
    "ucode/elm-firebase": {
      "url": "https://github.com/ucode/elm-firebase",
      "ref": "x.y.z"
    }
  }
}
```

Of course, replace `x.y.z` with the appropriate version number. [See all versions here](https://github.com/ucode/elm-firebase/releases).

### elm-github-install

```sh
$ npx -p elm-github-install elm-install
# or
$ npm i -g elm-github-install
$ elm-install
```

## Key differences from JavaScript library

 - `snapshot.val()` maps to `Firebase.Database.Snapshot.value snapshot` rather than `Firebase.Database.Snapshot.val snapshot`. I chose to be more explicit because I thought `val` wasn't as meaningful as it could be.
 - `reference.on`, `reference.off`, `query.on`, and `query.off` map to singular subscription methods: `Firebase.Database.Reference.on` and `Firebase.Database.Query.on` respectively. When you're done, just remove your subscription from `Sub.batch` and elm-firebase will do the rest!

## Examples

### Connecting to your firebase database

```elm
import Html
import Firebase
import Firebase.Database.Types
import Firebase.Database.Reference
import Firebase.Database.Snapshot



main =
  Html.program
      { init = init
      , update = update
      , subscriptions = \_ -> Sub.none
      , view = -- ...
      }

type alias Model =
    { app : Firebase.App
    , db : Firebase.Database.Types.Database
    }


init : ( Model, Cmd Msg )
init =
  let
      app : Firebase.App
      app =
          Firebase.init
              { apiKey = "your firebase api key"
              , databaseURL = "https://your-firebase-app.firebaseio.com"
              , -- These aren't necessary for just connecting to your database
                authDomain = ""
              , storageBucket = ""
              , messagingSenderId = ""
              , projectId = ""
              }

      {-
          It's not necessary to store the database, but it will make it easier
          since all your database interactions are going to either be in `update`
          or `subscriptions`, and both have access to your model.
      -}
      db : Firebase.Database.Types.Database
      db =
          Firebase.Database.init app

      initialModel : Model
      initialModel =
          { app = app
          , db = db
          }
  in
      ( initialModel
      , Cmd.none
      )
```

### Getting the value of a reference

```elm
import Firebase
import Firebase.Database.Types
import Firebase.Database.Reference
import Firebase.Database.Snapshot



-- Same main/model/init stuff as above...


type Msg
    = GetValueOfFoo
    | FooValue Firebase.Database.Types.Snapshot


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    GetValueOfFoo ->
        let
            fooRef : Firebase.Database.Types.Reference
            fooRef =
                model.db
                    |> Firebase.Database.ref (Just "foo")
        in
            ( model
            , Task.perform FooValue (Firebase.Database.Reference.once "value" fooRef)
            )

    FooValue snapshot ->
        let
            {-
              This decodes the value of "/foo" as a string.
            -}
            value : Result String String
            value =
                snapshot
                    |> Firebase.Database.Snapshot.value           -- Gives us a Json.Decode.Value
                    |> Json.Decode.decodeValue Json.Decode.string -- Convert into a Result String a (where a is a String)
                    |> Debug.log "FooValue.value.result"          -- Output the result (either `Err decodeMessage` or `Ok value`)
        in
            ( model
            , Cmd.none
            )
```


### Subscribing to changes of a reference

```elm
import Firebase
import Firebase.Database.Types
import Firebase.Database.Reference
import Firebase.Database.Snapshot



main =
  Html.program
      { init = init
      , update = update
      , subscriptions = subscriptions
      , view = -- ...
      }



-- Same model/init as above...


type Msg
    = FooValue Firebase.Database.Types.Snapshot


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    FooValue snapshot ->
        let
            {-
              This decodes the value of "/foo" as a string.
            -}
            value : Result String String
            value =
                snapshot
                    |> Firebase.Database.Snapshot.value           -- Gives us a Json.Decode.Value
                    |> Json.Decode.decodeValue Json.Decode.string -- Convert into a Result String a (where a is a String)
                    |> Debug.log "FooValue.value.result"          -- Output the result (either `Err decodeMessage` or `Ok value`)
        in
            ( model
            , Cmd.none
            )


subscriptions : Model -> Sub Msg
subscriptions model =
    let
        fooRef : Firebase.Database.Types.Reference
        fooRef =
            model.db
                |> Firebase.Database.ref (Just "foo")
    in
      Sub.batch
          [ Firebase.Database.Reference.on "value" fooRef FooValue
          ]
```

## More examples

Based on excellent advice from [@pdamoc](https://github.com/pdamoc), here is [elm-firebase-todomvc](https://github.com/mrozbarry/elm-firebase-todomvc), and a live demo [here](https://elm-firebase-todomvc.firebaseapp.com/).

Check out the [kitchen sink](./examples/kitchenSink/src/Main.elm) or [writer](./examples/writer/src/Main.elm) examples for information.

# Special Thanks

See [credits](./CREDITS.md) for a list of people who have contributed code, ideas, and support.
