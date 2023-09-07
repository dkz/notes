# Haskell

## Extensible records

_Extensible record_ is a collection of fields with an operation to add more dynamically.
Extensible records are akin to type-safe maps, where field types can take the role of a key.
With modern Haskell features it's easy to define such a type:
```haskell
{-# language Safe #-}
{-# language DataKinds #-}

import Data.Typeable

data Record :: [*] -> * where
  Empty :: Record '[]
  Extend :: Typeable t => t -> Record ts -> Record (t ': ts)

rempty = Empty
(^+) :: Typeable t => Record ts -> t -> Record (t:ts)
rec ^+ t = Extend t rec
```

This definition is pretty much a carbon copy of `HList`.
`Typeable` enables a type-safe escape hatch for a value of a user provided type.
Contrary to expectations, safe Haskell doesn't prohibit this:
```haskell
unsafeGet :: forall t ts. Typeable t => Record ts -> t
unsafeGet (Extend (v :: vt) rest) =
  case eqT @t @vt of
    Just Refl -> v
    Nothing -> unsafeGet rest

unsafeUpd :: forall t ts. Typeable t => (t -> t) -> (Record ts -> Record ts)
unsafeUpd f Empty = Empty
unsafeUpd f (Extend (v :: vt) record) =
  case eqT @t @vt of
    Just Refl -> record ^+ f v
    Nothing -> unsafeUpd f record ^+ v
```

However, these functions are unsafe since type system can't guarantee a value of a type is present.
Type families and constraint kinds help to enforce the type checking:
```haskell
{-# language TypeFamilies #-}
{-# language ConstraintKinds #-}

import Data.Kind (Constraint)

type family Has t (fields :: [*]) :: Constraint where
  Has t (t:fields) = ()
  Has t (_:fields) = Has t fields
```

`Has t fields` collapses into an empty constraint `()` if type `t` is present in the list `ts`.
Otherwise, Haskell complaints that it's impossible to resolve the constraint.
This is more friendly than type equality constraints, primarily because of a nice compiler error.
Now, actual `get` and `upd` functions are just:
```haskell
get :: forall t ts. (Typeable t, Has t ts) => Record ts -> t
get = unsafeGet
upd :: forall t ts. (Typeable t, Has t ts) => (t -> t) -> (Record ts -> Record ts)
upd = unsafeUpd
```

`HList`-based records use labels to distinguish between fields of a similar type.
Arguably it's less ergonomic than simply using new types.
Consider using 2d vector as an example record type:
```haskell
newtype X = X Double
newtype Y = Y Double

norm :: (Has X t, Has Y t) => Record t -> Double
norm rec = let (X x, Y y) = (get rec, get rec) in sqrt (x^^2 + y^^2)
```

Listing constraints for every field in the record becomes annoying.
However, type families support constraint kinds and enable chaining of constraints.
One can define a constraint family and a custom constraint type to reduce repetition:
```haskell
type family SubtOf (super :: [*]) (fields :: [*]) :: Constraint where
  SubtOf '[] fields = ()
  SubtOf (t:super) fields = (Has t fields, SubtOf super fields)

type Vec2d t = SubtOf '[X, Y] t
```
With that, type of `norm` is just `Vec2d t => Record t -> Double`.
