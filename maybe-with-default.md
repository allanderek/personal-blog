---
title: "Elm's Maybe.withDefault"
tags: elm compilation
date: 2021-01-25
---

In certain circles in the Elm community it is seen as more *'Elmish'*, that is more idiomatic, or more desirable to write code using functions to combine/inspect common datatypes. So this code:

```elm
Maybe.withDefault 0 mInt
```

is more desirable than the following code which uses a case expression to achieve the same end:


```elm
case mInt of
    Nothing ->
        0
    Just x ->
        x
```

One question that arises, is does one compile to more efficient code? Let's test this, we can easily write the following into an elm file and compile with optimisation turned on:


```elm
increment : Int -> Int
increment x = x + 1

mIncrement : Maybe Int -> Int
mIncrement mInt =
    case mInt of
        Nothing ->
            1
        Just x ->
            increment x

decrement  : Int -> Int
decrement x = x - 1

mDecrement : Maybe Int -> Int
mDecrement mInt =
    mInt
        |> Maybe.withDefault 0
        |> decrement

```

I have deliberately written `increment` and `decrement` as separate functions so that this does not affect the compilation of either.

```elm
var $author$project$Main$decrement = function (x) {
	return x - 1;
};
var $elm$core$Maybe$withDefault = F2(
	function (_default, maybe) {
		if (!maybe.$) {
			var value = maybe.a;
			return value;
		} else {
			return _default;
		}
	});
var $author$project$Main$mDecrement = function (mInt) {
	return $author$project$Main$decrement(
		A2($elm$core$Maybe$withDefault, 0, mInt));
};
var $author$project$Main$increment = function (x) {
	return x + 1;
};
var $author$project$Main$mIncrement = function (mInt) {
	if (mInt.$ === 1) {
		return 1;
	} else {
		var x = mInt.a;
		return $author$project$Main$increment(x);
	}
};
```

So as perhaps expected the syntactical `case` expression compiles to more direct code. However it is possible that the Javascript engine can improve the performance of the more indirect code. Some benchmarking is required.
However, there is a further problem, if the `Nothing` case is some expensive expression, then using `Maybe.withDefault` will evaluate that expression even when it is ultimately thrown away. This is one of the advantages of a lazy language.


There exists a special tool [elm-optimize-level-2](https://github.com/mdgriffith/elm-optimize-level-2) which takes the Javascript code produced by the elm-compiler and improves it. I think there exists an opportunity for a tool that operates before the elm-compiler and translates idiomatic into more performant elm. Though of course the first step would be to do some benchmarking.
