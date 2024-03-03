# json-decode-pipeline

Build JSON decoders using the pipeline [`(|>)`](http://package.gren-lang.org/packages/gren-lang/core/3.0.0/Basics#|>)
operator.

## Motivation

It's common to decode into a record that has a `type alias`. Here's an example
of this from the [`map3`](http://package.gren-lang.org/packages/gren-lang/core/5.0.0/Json-Decode#map3)
docs:

```gren
type alias Job = { name : String, id : Int, completed : Bool }

point : Decoder Job
point =
  map3 Job
    (field "name" string)
    (field "id" int)
    (field "completed" bool)
```

This works because a record type alias can be called as a normal function. In
that case it accepts one argument for each field (in whatever order the fields
are declared in the type alias) and then returns an appropriate record built
with those arguments.

The `mapN` decoders are straightforward, but require manually changing N
whenever the field count changes. This library provides functions designed to
be used with the `|>` operator, with the goal of having decoders that are both
easy to read and easy to modify.

## Examples

Here is a decoder built with this library.

```gren
import Json.Decode as Decode exposing (Decoder, decodeString, float, int, nullable, string)
import Json.Decode.Pipeline exposing (required, optional, hardcoded)


type alias User =
  { id : Int
  , email : Maybe String
  , name : String
  , percentExcited : Float
  }


userDecoder : Decoder User
userDecoder =
  Decode.succeed User
    |> required "id" int
    |> required "email" (nullable string) -- `null` decodes to `Nothing`
    |> optional "name" string "(fallback if name is `null` or not present)"
    |> hardcoded 1.0
```

In this example:

* `required "id" int` is similar to `(field "id" int)`
* `optional` is like `required`, but if the field is either `null` or not present, decoding does not fail; instead it succeeds with the provided fallback value.
* `hardcoded` does not look at the provided JSON, and instead always decodes to the same value.

You could use this decoder as follows:

```gren
decodeString
  userDecoder
  """
    {"id": 123, "email": "sam@example.com", "name": "Sam Sample"}
  """
```

The result would be:

```gren
{ id = 123
, email = Just "sam@example.com"
, name = "Sam Sample"
, percentExcited = 1.0
}
```

Alternatively, you could use it like so:

```gren
decodeString
  userDecoder
  """
    {"id": 123, "email": null, "percentExcited": "(hardcoded)"}
  """
```

In this case, the result would be:

```gren
{ id = 123
, email = Nothing
, name = "(fallback if name is `null` or not present)"
, percentExcited = 1.0
}
```

---

[![NoRedInk](https://cloud.githubusercontent.com/assets/1094080/9069346/99522418-3a9d-11e5-8175-1c2bfd7a2ffe.png)](http://noredink.com/about/team)
