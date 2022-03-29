---
title: Use Show and Read Judiciously
author: Ilia Rodionov
---

## Motivating examples

### Read instances: typechecks, but doesn't work

One of the commonly admitted upsides of Haskell and its powerful type-system is
ability to refactor programs in a safe manner. Indeed, by using thoroughly defined
types and just by leveraging the easiness of introducing zero run-time cost
`newtype`s, we can eliminate a whole class of errors caused by representing many
different things by one and the same type.

That being said, we should exercise extra attention when it comes to munging data
at application's boundaries where everything likely gets a representation based on
very basic types, most often `String`.

Consider the following example of reading an environment variable from a real codebase:

```Haskell
pgPort :: Word16
pgPort = fromMaybe 5432 $ readMaybe =<< lookupEnv "PG_PORT"

lookupEnv :: String -> Maybe String
lookupEnv = ...
```

Someone had to move from `Word16` to `String` here for some reason and came up with
these lines having done mere mechanical changes:

```Haskell
pgPort :: String
pgPort = fromMaybe "5432" $ readMaybe =<< lookupEnv "PG_PORT"
```

While this code compiles, it doesn't work as expected. Reading a `String` will fail
unless the value is properly quoted, but no one will expect this being the case when
setting a value of an environment variable. As a result, you most probably will get the
default value regardless what has been set:

```Haskell
>>> readMaybe "5432" :: Maybe String
Nothing
>>> readMaybe "\"5432\"" :: Maybe String
Just "5432"
```

Remembering that `String` is effectively a list of `Char` we also might specify the
value as following:

```
λ> readMaybe "['5','4','3','2']" :: Maybe String
Just "5432"
```

Albeit it can be quite frustrating, such an implementation of `Read` makes sense since
it holds a *social contract* of being a counterpart to `Show`. But let sleeping dogs lie,
and let's take how `Show` has been implemented without further questions. The lesson we
learned from that example is that it's intrinsically unsafe to use `Read` standard
instances for parsing values.

### Show instances: escaping strings

TODO: Example with Text-based logging.

## What to do?

### Use newtype wrappers
### Skip showing of Strings/Texts

## Additional quirks

### Octal and hexadecimal literals

```
>>> readMaybe " 10 " :: Maybe Int
Just 10
>>> readMaybe "0xA" :: Maybe Int
Just 10
>>> readMaybe "0o12" :: Maybe Int
Just 10
```

### NumericUnderscores

`NumericUnderscores` brings readability to long numbers, but doesn't affect `Read`/`Show`
instances, which might again brings up a question why these instances are implemented
as they are (since initially `Show`-ed string representations match literals):

```
>>> :set -XNumericUnderscores
>>> readMaybe "1_000" :: Maybe Int
Nothing
```