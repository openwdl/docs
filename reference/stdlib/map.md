# Map Functions

- [`as_pairs`](#as-pairs)
- [`as_map`](#as-map)
- [`keys`](#keys)
- [`contains_key`](#contains-key) <Badge type="tip" text="v1.2" />
- [`values`](#values) <Badge type="tip" text="v1.2" />
- [`collect_by_key`](#collect-by-key)

## `as_pairs`

Converts a `Map` into an `Array` of `Pair`s. Since `Map`s are ordered, the output array
will always have elements in the same order they were added to the `Map`.

**Signatures**

```wdl
Array[Pair[P, Y]] as_pairs(Map[P, Y])
```

**Parameters**

1. **`Map[P, Y]`**: `Map` to convert to `Pair`s.

**Returns**

1. Ordered `Array` of `Pair`s, where each pair contains the key (left) and value (right)
   of a `Map` element.

**Example**

```wdl
Map[String, Int] x = {"a": 1, "c": 3, "b": 2}

Array[Pair[String, Int]] result = as_pairs(x)
# `result` now contains [("a", 1), ("c", 3), ("b", 2)].
```

## `as_map`

Converts an `Array` of `Pair`s into a `Map` in which the left elements of the `Pair`s
are the keys and the right elements the values. All the keys must be unique, or an error
is raised. The order of the key/value pairs in the output `Map` is the same as the order
of the `Pair`s in the `Array`.

**Signatures**

```wdl
Map[P, Y] as_map(Array[Pair[P, Y]])
```

**Parameters**

1. **`Array[Pair[P, Y]]`**: Array of `Pair`s to convert to a `Map`.

**Returns**

1. `Map[P, Y]` of the elements in the input array.

**Example**

```wdl
Array[Pair[String, Int]] x = [("a", 1), ("c", 3), ("b", 2)]

Map[String, Int] result = as_map(x)
# `result` now contains {"a": 1, "c": 3, "b": 2}.
```

## `keys`

Given a key-value type collection (`Map`, `Struct`, or `Object`), returns an `Array` of
the keys from the input collection, in the same order as the elements in the collection.

When the argument is a `Struct`, the returned array will contain the keys in the same
order they appear in the struct definition. When the argument is an `Object`, the
returned array has no guaranteed order.

When the input `Map` or `Object` is empty, an empty array is returned.

**Signatures**

```wdl
Array[P] keys(Map[P, Y])
Array[String] keys(Struct|Object)
```

**Parameters**

1. **`Map[P, Y]|Struct|Object`**: Collection from which to extract keys.

**Returns**

1. `Array[P]` of the input collection's keys. If the input is a `Struct` or `Object`,
   then the returned array will be of type `Array[String]`.

**Example**

```wdl
Map[String, Int] x = {"a": 1, "b": 2, "c": 3}

Array[String] result = keys(x)
# `result` now contains ["a", "b", "c"].
```

## `contains_key` <Badge type="tip" text="Requires WDL v1.2" />

Given a key-value type collection (`Map`, `Struct`, or `Object`) and a key, tests whether
the collection contains an entry with the given key.

This function has three variants:

1. `Boolean contains_key(Map[P, Y], P)`: Tests whether the `Map` has an entry with the
   given key. If `P` is an optional type (e.g., `String?`), then the second argument may
   be `None`.
2. `Boolean contains_key(Object, String)`: Tests whether the `Object` has an entry with
   the given name.
3. `Boolean contains_key(Map[String, Y]|Struct|Object, Array[String])`: Tests recursively
   for the presence of a compound key within a nested collection.

For the third variant, the first argument is a collection that may be nested to any
level, i.e., contain values that are collections, which themselves may contain
collections, and so on. The second argument is an array of keys that are resolved
recursively. If the value associated with any except the last key in the array is `None`
or not a collection type, this function returns `false`.

For example, if the first argument is a `Map[String, Map[String, Int]]` and the second
argument is `["foo", "bar"]`, then the outer `Map` is tested for the presence of key
"foo", and if it is present, then its value is tested for the presence of key "bar". This
only tests for the presence of the named element, *not* whether or not it is `defined`.

**Signatures**

```wdl
Boolean contains_key(Map[P, Y], P)
Boolean contains_key(Object, String)
Boolean contains_key(Map[String, Y]|Struct|Object, Array[String])
```

**Parameters**

1. **`Map[P, Y]|Struct|Object`**: Collection to search for the key.
2. **`P|Array[String]`**: The key to search. If the first argument is a `Map`, then the
   key must be of the same type as the `Map`'s key type. If the `Map`'s key type is
   optional then the key may also be optional. If the first argument is a
   `Map[String, Y]`, `Struct`, or `Object`, then the key may be either a `String` or
   `Array[String]`.

**Returns**

1. `true` if the collection contains the key, otherwise `false`.

**Example**

```wdl
Map[String, Int] m = {"a": 1, "b": 2}

Boolean has_a = contains_key(m, "a")
# `has_a` now contains `true`.

Boolean has_c = contains_key(m, "c")
# `has_c` now contains `false`.
```

## `values` <Badge type="tip" text="Requires WDL v1.2" />

Returns an `Array` of the values from the input `Map`, in the same order as the elements
in the map. If the map is empty, an empty array is returned.

**Signatures**

```wdl
Array[Y] values(Map[P, Y])
```

**Parameters**

1. **`Map[P, Y]`**: `Map` from which to extract values.

**Returns**

1. `Array[Y]` of the input `Map`'s values.

**Example**

```wdl
Map[String, Int] x = {"a": 1, "b": 2, "c": 3}

Array[Int] result = values(x)
# `result` now contains [1, 2, 3].
```

## `collect_by_key`

Given an `Array` of `Pair`s, creates a `Map` in which the right elements of the `Pair`s
are grouped by the left elements. In other words, the input `Array` may have multiple
`Pair`s with the same key. Rather than causing an error (as would happen with
[`as_map`](#as-map)), all the values with the same key are grouped together into an
`Array`.

The order of the keys in the output `Map` is the same as the order of their first
occurrence in the input `Array`. The order of the elements in the `Map` values is the
same as their order of occurrence in the input `Array`.

**Signatures**

```wdl
Map[P, Array[Y]] collect_by_key(Array[Pair[P, Y]])
```

**Parameters**

1. **`Array[Pair[P, Y]]`**: `Array` of `Pair`s to group.

**Returns**

1. `Map` of keys to `Array`s of values.

**Example**

```wdl
Array[Pair[String, Int]] x = [("a", 1), ("b", 2), ("a", 3)]

Map[String, Array[Int]] result = collect_by_key(x)
# `result` now contains {"a": [1, 3], "b": [2]}.
```
