# Array Functions

- [`range`](#range)
- [`transpose`](#transpose)
- [`cross`](#cross)
- [`zip`](#zip)
- [`unzip`](#unzip)
- [`contains`](#contains) <Badge type="tip" text="v1.2" />
- [`chunk`](#chunk) <Badge type="tip" text="v1.2" />
- [`flatten`](#flatten)
- [`select_first`](#select_first)
- [`select_all`](#select_all)

## `range`

Creates an array of the given length containing sequential integers starting from 0.
The length must be >= `0`. If the length is `0`, an empty array is returned.

**Signatures**

```wdl
Array[Int] range(Int)
```

**Parameters**

1. **`Int`**: The length of array to create.

**Returns**

1. An `Array[Int]` containing integers `0..(N-1)`.

**Example**

```wdl
Array[Int] indexes = range(5)
# `indexes` now contains `[0, 1, 2, 3, 4]`.
```

## `transpose`

Transposes a two-dimensional array according to the standard matrix transposition rules,
i.e. each row of the input array becomes a column of the output array. The input array
must be square - i.e., every row must have the same number of elements - or an error is
raised. If either the inner or the outer array is empty, an empty array is returned.

**Signatures**

```wdl
Array[Array[X]] transpose(Array[Array[X]])
```

**Parameters**

1. **`Array[Array[X]]`**: A `M*N` two-dimensional array.

**Returns**

1. A `N*M` two-dimensional array (`Array[Array[X]]`) containing the transposed input
   array.

**Example**

```wdl
# input array is 2 rows * 3 columns
Array[Array[Int]] input_array = [[0, 1, 2], [3, 4, 5]]

Array[Array[Int]] output_array = transpose(input_array)
# output array is 3 rows * 2 columns: [[0, 3], [1, 4], [2, 5]]
```

## `cross`

Creates an array of `Pair`s containing the cross product of two input arrays, i.e., each
element in the first array is paired with each element in the second array.

Given `Array[X]` of length `M`, and `Array[Y]` of length `N`, the cross product is
`Array[Pair[X, Y]]` of length `M*N` with the following elements:
`[(X0, Y0), (X0, Y1), ..., (X0, Yn-1), (X1, Y0), ..., (X1, Yn-1), ..., (Xm-1, Yn-1)]`.
If either of the input arrays is empty, an empty array is returned.

**Signatures**

```wdl
Array[Pair[X,Y]] cross(Array[X], Array[Y])
```

**Parameters**

1. **`Array[X]`**: The first array of length `M`.
2. **`Array[Y]`**: The second array of length `N`.

**Returns**

1. An `Array[Pair[X, Y]]` of length `M*N`.

**Example**

```wdl
Array[Int] xs = [1, 2, 3]
Array[String] ys = ["a", "b"]

Array[Pair[Int, String]] result = cross(xs, ys)
# `result` now contains [(1, "a"), (1, "b"), (2, "a"), (2, "b"), (3, "a"), (3, "b")].
```

## `zip`

Creates an array of `Pair`s containing the dot product of two input arrays, i.e., the
elements at the same indices in each array `X[i]` and `Y[i]` are combined together into
(`X[i]`, `Y[i]`) for each `i` in `range(length(X))`. The input arrays must have the same
lengths or an error is raised. If the input arrays are empty, an empty array is returned.

**Signatures**

```wdl
Array[Pair[X,Y]] zip(Array[X], Array[Y])
```

**Parameters**

1. **`Array[X]`**: The first array of length `N`.
2. **`Array[Y]`**: The second array of length `N`.

**Returns**

1. An `Array[Pair[X, Y]]` of length `N`.

**Example**

```wdl
Array[Int] xs = [1, 2, 3]
Array[String] ys = ["a", "b", "c"]

Array[Pair[Int, String]] result = zip(xs, ys)
# `result` now contains [(1, "a"), (2, "b"), (3, "c")].
```

## `unzip`

Creates a `Pair` of `Arrays`, the first containing the elements from the `left` members
of an `Array` of `Pair`s, and the second containing the `right` members. If the array is
empty, a pair of empty arrays is returned. This is the inverse of the `zip` function.

**Signatures**

```wdl
Pair[Array[X], Array[Y]] unzip(Array[Pair[X, Y]])
```

**Parameters**

1. **`Array[Pair[X, Y]]`**: The `Array` of `Pair`s of length `N` to unzip.

**Returns**

1. A `Pair[Array[X], Array[Y]]` where each `Array` is of length `N`.

**Example**

```wdl
Array[Pair[Int, String]] int_str_arr = [(0, "hello"), (42, "goodbye")]

Pair[Array[Int], Array[String]] result = unzip(int_str_arr)
# `result` now contains ([0, 42], ["hello", "goodbye"]).
```

## `contains` <Badge type="tip" text="Requires WDL v1.2" />

Tests whether the given array contains at least one occurrence of the given value.

**Signatures**

```wdl
Boolean contains(Array[P], P)
Boolean contains(Array[P?], P?)
```

**Parameters**

1. **`Array[P]` or `Array[P?]`**: an array of any primitive type.
2. **`P` or `P?`**: a primitive value of the same type as the array. If the array's type
   is optional, then the value may also be optional.

**Returns**

1. `true` if the array contains at least one occurrence of the value, otherwise `false`.

**Example**

```wdl
Array[String?] samples = [None, "foo", "bar"]
String name = "bar"

Boolean has_null = contains(samples, None)
# `has_null` now contains `true`.

Boolean has_name = contains(samples, name)
# `has_name` now contains `true`.

Boolean has_missing = contains(samples, "baz")
# `has_missing` now contains `false`.
```

## `chunk` <Badge type="tip" text="Requires WDL v1.2" />

Given an array and a length *n*, splits the array into consecutive, non-overlapping
arrays of *n* elements. If the length of the array is not a multiple of *n* then the
final sub-array will have `length(array) % n` elements.

**Signatures**

```wdl
Array[Array[X]] chunk(Array[X], Int)
```

**Parameters**

1. **`Array[X]`**: The array to split. May be empty.
2. **`Int`**: The desired length of the sub-arrays. Must be > 0.

**Returns**

1. An array of sub-arrays, where each sub-array is of length `N` except possibly the last
   one.

**Example**

```wdl
Array[String] s1 = ["a", "b", "c", "d", "e", "f"]

Array[Array[String]] result = chunk(s1, 3)
# `result` now contains [["a", "b", "c"], ["d", "e", "f"]].

Array[String] s2 = ["a", "b", "c", "d", "e"]
Array[Array[String]] result2 = chunk(s2, 3)
# `result2` now contains [["a", "b", "c"], ["d", "e"]].
```

## `flatten`

Flattens a nested `Array[Array[X]]` by concatenating all of the element arrays, in order,
into a single array. The function is not recursive - e.g. if the input is
`Array[Array[Array[Int]]]` then the output will be `Array[Array[Int]]`. The elements in
the concatenated array are *not* deduplicated.

**Signatures**

```wdl
Array[X] flatten(Array[Array[X]])
```

**Parameters**

1. **`Array[Array[X]]`**: A nested array to flatten.

**Returns**

1. An `Array[X]` containing the concatenated elements of the input array.

**Example**

```wdl
Array[Array[Int]] nested = [[1, 2, 3], [1], [21, 22]]

Array[Int] result = flatten(nested)
# `result` now contains [1, 2, 3, 1, 21, 22].
```

## `select_first`

Selects the first - i.e., left-most - non-`None` value from an `Array` of optional
values. The optional second parameter provides a default value that is returned if the
array is empty or contains only `None` values. If the default value is not provided and
the array is empty or contains only `None` values, then an error is raised.

**Signatures**

```wdl
X select_first(Array[X?]+)
X select_first(Array[X?], X)
```

**Parameters**

1. **`Array[X?]+`**: Non-empty `Array` of optional values.
2. **`X`**: (Optional) The default value.

**Returns**

1. The first non-`None` value in the input array, or the default value if it is provided
   and the array does not contain any non-`None` values.

**Example**

```wdl
Int? maybe_five = 5
Int? maybe_none = None
Int? maybe_three = 3

Int result1 = select_first([maybe_five, maybe_none, maybe_three])
# `result1` now contains `5`.

Int result2 = select_first([maybe_none, maybe_five, maybe_three])
# `result2` now contains `5`.

Int result3 = select_first([], 42)
# `result3` now contains `42`.
```

## `select_all`

Filters the input `Array` of optional values by removing all `None` values. The elements
in the output `Array` are in the same order as the input `Array`. If the input array is
empty or contains only `None` values, an empty array is returned.

**Signatures**

```wdl
Array[X] select_all(Array[X?])
```

**Parameters**

1. **`Array[X?]`**: `Array` of optional values.

**Returns**

1. An `Array` of all non-`None` values in the input array.

**Example**

```wdl
Int? maybe_five = 5
Int? maybe_none = None
Int? maybe_three = 3

Array[Int] result = select_all([maybe_five, maybe_none, maybe_three])
# `result` now contains [5, 3].
```
