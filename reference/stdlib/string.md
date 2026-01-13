# String Functions

- [`find`](#find) <Badge type="tip" text="v1.2" />
- [`matches`](#matches) <Badge type="tip" text="v1.2" />
- [`split`](#split) <Badge type="tip" text="v1.3" />
- [`sub`](#sub)

## `find` <Badge type="tip" text="Requires WDL v1.2" />

Given two `String` parameters `input` and `pattern`, searches for the occurrence of
`pattern` within `input` and returns the first match or `None` if there are no matches.
`pattern` is a [regular expression](https://en.wikipedia.org/wiki/Regular_expression)
and is evaluated as a [POSIX Extended Regular Expression
(ERE)](https://en.wikipedia.org/wiki/Regular_expression#POSIX_basic_and_extended).


Note that regular expressions are written using regular WDL strings, so backslash
characters need to be double-escaped. For example:

```wdl
String? first_match = find("hello\tBob", "\\t")
```

**Signatures**

```wdl
String? find(String, String)
```

**Parameters**

1. **`String`**: the input string to search.
2. **`String`**: the pattern to search for.

**Returns**

1. The contents of the first match, or `None` if `pattern` does not match `input`.

**Example**

```wdl
String? match = find("Hello, world!", "e..o");
# `match` now contains `ello`.
```

## `matches` <Badge type="tip" text="Requires WDL v1.2" />

Given two `String` parameters `input` and `pattern`, tests whether `pattern` matches
`input` at least once. `pattern` is a [regular
expression](https://en.wikipedia.org/wiki/Regular_expression) and is evaluated as a
[POSIX Extended Regular Expression
(ERE)](https://en.wikipedia.org/wiki/Regular_expression#POSIX_basic_and_extended).

To test whether `pattern` matches the entire `input`, make sure to begin and end the
pattern with anchors. For example:

```wdl
Boolean full_match = matches("abc123", "^a.+3$")
```

Note that regular expressions are written using regular WDL strings, so backslash
characters need to be double-escaped. For example:

```wdl
Boolean has_tab = matches("hello\tBob", "\\t")
```

**Signatures**

```wdl
Boolean matches(String, String)
```

**Parameters**

1. **`String`**: the input string to search.
2. **`String`**: the pattern to search for.

**Returns**

1. `true` if `pattern` matches `input` at least once, otherwise `false`.

**Example**

```wdl
Boolean matches = matches("sample1234_R1.fastq", "_R1");
# `matches` now contains `true`.
```

## `split` <Badge type="tip" text="Requires WDL v1.3" />

Given the two `String` parameters `input` and `delimiter`, this function splits the input
string on the provided delimiter and stores the results in a `Array[String]`. `delimiter`
is a [regular expression](https://en.wikipedia.org/wiki/Regular_expression) and is
evaluated as a [POSIX Extended Regular Expression
(ERE)](https://en.wikipedia.org/wiki/Regular_expression#POSIX_basic_and_extended).
Regular expressions are written using regular WDL strings, so backslash characters need
to be double-escaped (e.g., `"\\t"`).

**Signatures**

```wdl
Array[String] split(String, String)
```

**Parameters**

1. **`String`**: the input string.
2. **`String`**: the delimiter to split on as a regular expression.

**Returns**

1. The parts of the input string split by the delimiter. If the delimiter does not match
   anything in the input string, an array containing a single entry of the input string
   is returned.

**Example**

```wdl
String input = "a,b,c"
Array[String] parts = split(input, ",")
# `parts` now contains `["a", "b", "c"]`.

String multiline = "line1\nline2\nline3"
Array[String] lines = split(multiline, "\\n")
# `lines` now contains `["line1", "line2", "line3"]`.
```

## `sub`

Given three String parameters `input`, `pattern`, `replace`, this function replaces all
non-overlapping occurrences of `pattern` in `input` by `replace`. `pattern` is a
[regular expression](https://en.wikipedia.org/wiki/Regular_expression) and is evaluated
as a [POSIX Extended Regular Expression
(ERE)](https://en.wikipedia.org/wiki/Regular_expression#POSIX_basic_and_extended).
Regular expressions are written using regular WDL strings, so backslash characters need
to be double-escaped (e.g., `"\\t"`).


**Signatures**

```wdl
String sub(String, String, String)
```

**Parameters**

1. **`String`**: the input string.
2. **`String`**: the pattern to search for.
3. **`String`**: the replacement string.

**Returns**

1. the input string, with all occurrences of the pattern replaced by the replacement
   string.

**Example**

```wdl
String chocolike = "I like chocolate when\nit's late"
String chocolove = sub(chocolike, "like", "love") #

# `chocolove` now contains `"I love chocolate when\nit's late"`.
```
