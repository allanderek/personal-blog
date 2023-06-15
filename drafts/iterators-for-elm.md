
# Python iterators

# Python iterators are kind of laziness for a strict language

# We do a lot of 'list' processing in Elm

# Iterators for Elm

Iterators can be made with zero new syntax.

```
type Iterator a
    Empty
  | Current a (() -> Iterator a)

integers : Iterator Int
integers =
    Current 0 (\() -> 
  
type alias Iterator a =
  { current : a 
  , next : () -> Maybe a
  }
  
integers : Iterator Int
integers =
  { current = 0
  , next : \() -> Just 1
  }

next : Int -> 
```
  


