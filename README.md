# PureScript For Haskellers

Some info that supposed to help
to understand [PureScript](http://www.purescript.org/)
from [Haskell](https://www.haskell.org/) perspective.

If you already know js it will be even simplier.

## About PureScript

PureScript written in Haskell but usually distributed as binaries via NPM.

It uses **Bower** instead of **NPM** because **Bower**
have flat dependencies and better dependency resolution.<br>
This explained better here: http://harry.garrood.me/blog/purescript-why-bower/

PureScript is **strict** by default!

### Useful links

- https://pursuit.purescript.org
  (kinda like https://stackage.org for Haskell)
- http://try.purescript.org (online REPL)

## From Haskell perspective

1. **About `Prelude`**:

   PureScript acts like `{-# LANGUAGE NoImplicitPrelude #-}` in Haskell,
   and `Prelude` also isn't distributed with PureScript compiler.

   You need to install dependency `purescript-prelude` and to import it:

   ```purescript
   import Prelude
   ```

2. **About `forall`**:

   PureScript acts like `{-# LANGUAGE ExplicitForAll #-}` in Haskell.

   You need to explicitly declare `forall` for every polymorphic type variable.

3. **About unicode**:

   In PureScript using unicode is allowed by default.

   Basic unicode symbols also included
   (such as `∷`, `←`, `→`, `⇐`, `⇒`, `∀`, etc.)

4. **Basic operators equivalents** (those which differ):

   | PureScript                      | Haskell                                            |
   | ---                             | ---                                                |
   | `(<<<)`                         | `(.)`                                              |
   | `(>>>)`                         | `flip (.)` or `(.>)` from `flow` package           |
   | `(#)`                           | `(Data.Function.&)` from `base` package            |
   | `(<#>)`                         | `(Control.Lens.Operators.<&>)` from `lens` package |
   | `(<>)` (`Semigroup` type class) | `(++)`                                             |
   | `a >>= const b`                 | `a >> b`                                           |
   | `const b =<< a`                 | `b << a`                                           |

5. **Basic functions equivalents** (those which differ):

   | PureScript                                                                                         | Haskell                             |
   | ---                                                                                                | ---                                 |
   | `map` (`Functor` type class)                                                                       | `fmap` (works like `map` for lists) |
   | `unsafeThrow`<br>from `Control.Monad.Eff.Exception.Unsafe`<br>from `purescript-exceptions` package | `error`                             |
   | `forkAff`<br>from `Control.Monad.Aff`<br>from `purescript-aff` package                             | `forkIO`                            |

   If you're looking for **Haskell**'s `Control.Concurrent.MVar` look at
   **PureScript**'s `Control.Monad.Aff.AVar` from `purescript-aff` package.

6. **About point-free style**:

   For partially applied operators you must specify *ghost* place for a value:

   | PureScript | Haskell |
   | ---        | ---     |
   | `(_ + 2)`  | `(+ 2)` |
   | `(2 + _)`  | `(2 +)` |

   You're defenitely familiar with `{-# LANGUAGE LambdaCase #-}` in **Haskell**:

   In **PureScript** you have kinda the same, but again,
   you need to explicitly set *ghost* place for a value:

   ```purescript
   case _ of
     Just x  -> 34
     Nothing -> 42
   ```

   That in **Haskell** would be:

   ```haskell
   \case
     Just x  -> 34
     Nothing -> 42
   ```

   To update a record in **PureScript** you also use a *ghost* place marker:

   ```purescript
   _ { foo = 42 }
   ```

   But in **PureScript** you also able to easily modify nested records without
   even using stuff like lenses:

   ```purescript
   _ { foo { bar { baz = 42 } } }
   ```

   You able create a function that fills record values this way:

   ```purescript
   { foo: _, bar: _ }
   ```

   Which is equivalent to:

   ```purescript
   \foo bar -> { foo: foo, bar: bar }
   ```

   Or even to (as in js):

   ```purescript
   \foo bar -> { foo, bar }
   ```

7. **About Unit**:

   | PureScript   | Haskell    |
   | ---          | ---        |
   | Type `Unit`  | Type `()`  |
   | Value `unit` | Value `()` |

8. **About IO**:

   If you're looking what would be equivalent to `IO ()` in **Haskell** or just
   wondering what the heck is `Eff (foo :: FOO) Unit`:

   **PureScript** have improved implementation of `IO` monad in **Haskell**, the
   main difference is that `Eff` monad (in **PureScript**) have additional
   parameter to specify limitation of possible side-effects (such as `CONSOLE`,
   `DOM`, `REF`, etc.) so you can have more precise control of `IO` stuff.

   You defenitely should read official docs about this, the story couldn't be
   told in few sentences.

   Few tips about **Eff** (*Eff* means *effects*):

   - `IO ()` is kinda `forall eff. Eff eff Unit`;
   - You must type your own `Eff` monads providing type of side-effects which it
     going to make (e.g. `Eff (console :: CONSOLE) Unit`);
   - But usually it's better to allow to use your monad inside more complex ones
     by making it polymorphic (e.g.
     `forall eff. Eff (console :: CONSOLE | eff)`, that means it can do
     `CONSOLE` stuff but not limited to be used in context of others);
   - `|` could be read as `as` (this aliases whole block inside parentheses).

   For async stuff (kinda threading, but remember you're in js world, it's not
   really threads) you have similar `Aff` monad. You also should read docs about
   this too.

   Few tips about **Aff**:

   - Doing `Aff` is kinda doing `forkIO` in **Haskell** I believe;
   - Use `launchAff` or `launchAff_` to run `Aff` from `Eff` monad
     asynchronously;
   - Use `forkAff` to run another `Aff` from `Aff` monad asynchronously;
   - Use `liftEff` (`Control.Monad.Eff.Class` from `purescript-eff`)
     to execute `Eff` from `Aff` monad;
   - Use `liftEff'` (`Control.Monad.Aff` from `purescript-aff`)
     to execute `Eff` from `Aff` monad if `Eff` monad has `EXCEPTION` effect.

   Keep in mind that **PureScript** is strict by default, so using:

   ```purescript
   if condition
      then someMonad foo bar
      else pure unit
   ```

   could be better than:

   ```purescript
   when condition $ someMonad foo bar
   ```

   in sense of efficiency, because `if` condition compiles to native js `if`
   condition while `when` constructs function reference with possible partial
   application.

   See also:
   - https://pursuit.purescript.org/packages/purescript-eff
   - https://pursuit.purescript.org/packages/purescript-aff

9. **About booleans**

    | PureScript     | Haskell       |
    | ---            | ---           |
    | Type `Boolean` | Type `Bool`   |
    | Value `true`   | Value `True`  |
    | Value `false`  | Value `False` |

10. **About tuples**

    In **PureScript** there's no special syntax for tuples.

    You also need to install `purescript-tuples`.

    | PureScript            | Haskell            |
    | ---                   | ---                |
    | Type `Tuple Bool Int` | Type `(Bool, Int)` |
    | Value `Tuple true 42` | Value `(True, 42)` |
    | Pattern `(Tuple x y)` | Pattern `(x, y)`   |

11. **About lists**

    **PureScript** has builtin `Array`s.
    Functional `List`s are provided by `purescript-lists` package.

    `[1,2,3]` will produce an `Array Int`
    (not `[Int]` because in **PureScript**
    there's no sugar for typing `Array`s/`List`s).

    **PureScript** doesn't have special syntax for `Array` comprehensions.<br>
    Here is an example of doing comprehension using monads:

    ```purescript
    factors :: Int -> Array (Tuple Int Int)
    factors n = do
      a <- 1 .. n
      b <- 1 .. a
      guard $ a * b == n
      pure $ Tuple a b
    ```

    An example of `Array` patterns:

    ```purescript
    f []     = -1
    f [x]    = x
    f [x, y] = x * y
    f _      = 0
    ```

    There's no builtin *cons* for `Array`s for pattern-matching
    (some performance issues)
    but some helpers are provided by `purescript-arrays` package.

    See also about this:<br>
    https://stackoverflow.com/questions/42450347/purescript-pattern-match-arrays-of-unknown-length#42450443

    Pattern-matching on `List`s:

    | PureScript    | Haskell    |
    | ---           | ---        |
    | `(Cons x xs)` | `(x : xs)` |

12. **About records**:

    Records in **PureScript** isn't limited to be used in context of `data`,
    they're independent, you don't need (but may) have a wrapper for a record.

    Here is an example of a function that works with records:

    ```purescript
    foo :: { foo :: String, bar :: Int } -> Int
    foo x = x.bar
    ```

    Type of `foo` is equivalent to:

    ```purescript
    foo :: Record (foo :: String, bar :: Int) -> Int
    ```

    `foo` also can deal with any record that have `bar :: Int`
    if it's typed like this:

    ```purescript
    foo :: forall r. { bar :: Int | r } -> Int
    ```

    You can read about `|` above, it acts here the same way.

    Constructing new records is simple:

    ```purescript
    bar = { foo: "Foo", bar: 42, baz: true }
    ```

    But keep in mind that when you construct new record you use `:` but when you
    update a record you use `=`:

    ```purescript
    bar { bar = 34 }
    ```

    An example how to update a nested record:

    ```purescript
    foo { bar { baz { bzz = 42 } } }
    ```

    Destructuring also works as in js:

    1.
       ```purescript
       foo x = log x.bar
       ```
       ```purescript
       foo { bar } = <- log bar
       ```
       ```purescript
       foo { bar: baz } = <- log baz
       ```
    2.
       ```purescript
       foo = do
         x <- bar
         log x.baz
       ```
       ```purescript
       foo = do
         { baz } <- bar
         log baz
       ```
       ```purescript
       foo = do
         { baz: bzz } <- bar
         log bzz
       ```

13. **About deriving type class instance**:

    Deriving instances separated from `data`, here's an example:

    ```purescript
    derive instance eqLocation :: Eq Location
    derive instance genericLocation :: Generic Location
    instance showLocation :: Show Location where show = gShow
    ```

    Names `eqLocation`, `genericLocation` and `showLocation` is just for
    produced js code, they're named like this just by convention but they can be
    named differently.

14. **About imports**:

    In **PureScript** you don't have `qualified` keyword for imports,
    if an import have `as` alias it **is** `qualified`.

    In **PureScript** `as` keyword must be places after everything
    (even after explicit imports).

    | PureScript                     | Haskell                                  |
    | ---                            | ---                                      |
    | `import Data.Foo as Foo`       | `import qualified Data.Foo as Foo`       |
    | `import Data.Foo as Foo (foo)` | `import qualified Data.Foo (foo) as Foo` |

15. **About some packages**:

    - `Maybe` isn't included, install `purescript-maybe`
    - `purescript-console` for writing to the console
    - `purescript-nullable` to deal with js `null`s (when you really need it)
    - `purescript-generics` to deal with `Generic` stuff
    - `purescript-lens` if you're looking for Kmett's lenses

This is pretty short list that supposed to get basic stuff as fast as possible,
read articles by this links to go deeper:

- https://github.com/purescript/documentation/blob/master/language/Differences-from-Haskell.md
- https://github.com/purescript/documentation/blob/master/guides/Getting-Started.md
- https://github.com/purescript/documentation/tree/master/language
