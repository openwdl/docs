# Other Functions

- [`defined`](#defined)
- [`length`](#length)

## `defined`

Tests whether the given optional value is defined, i.e., has a non-`None` value.

**Signatures**

```wdl
Boolean defined(X?)
```

**Parameters**

1. **`X?`**: optional value of any type.

**Returns**

1. `false` if the input value is `None`, otherwise `true`.

**Example**

```wdl
String? name = "Alice"
String? missing = None

Boolean has_name = defined(name)
# `has_name` now contains `true`.

Boolean has_missing = defined(missing)
# `has_missing` now contains `false`.

if (defined(name)) {
  String greeting = "Hello, " + select_first([name])
  # This code will execute because `name` is defined.
}
```

## `length`

Returns the length of the input argument as an `Int`:

- For an `Array[X]` argument: the number of elements in the array.
- For a `Map[X, Y]` argument: the number of items in the map.
- For an `Object` argument: the number of key-value pairs in the object.
- For a `String` argument: the number of characters in the string.

**Signatures**

```wdl
Int length(Array[X]|Map[X, Y]|Object|String)
```

**Parameters**

1. **`Array[X]|Map[X, Y]|Object|String`**: A collection or string whose elements are to
   be counted.

**Returns**

1. The length of the collection/string as an `Int`.

**Example**

```wdl
Array[Int] xs = [1, 2, 3]
Map[String, Int] m = {"a": 1, "b": 2}
String s = "ABCDE"

Int xlen = length(xs)
# `xlen` now contains `3`.

Int mlen = length(m)
# `mlen` now contains `2`.

Int slen = length(s)
# `slen` now contains `5`.
```
