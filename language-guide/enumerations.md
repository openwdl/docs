# Enumerations

**Enumerations** (or "enums") ([spec link][spec-enums]) define a closed set of
named choices that are considered valid in a specific context. Unlike `String`
inputs that accept any value, enums restrict allowed values so that the
execution engine can validate inputs before running a workflow.

Consider a workflow that processes data files in different formats with a
`format` parameter. Using `String` as the type means invalid values won't be
caught until runtime—possibly after hours of execution. Using an `enum` catches
these errors immediately.

```wdl
version 1.3

enum DataFormat {
  CSV,
  TSV,
  JSON
}

task parse_data {
  input {
    File infile
    DataFormat format = DataFormat.CSV
  }

  command <<<
    echo "Parsing ~{format} file: ~{infile}"
  >>>
}

workflow process_data {
  input {
    Array[File] files
    DataFormat format
  }

  scatter (file in files) {
    call parse_data {
      input:
        infile = file,
        format = format
    }
  }
}
```

## Enum values

Each choice in an enum has an associated value. By default, choices without
explicit values are assigned a `String` value matching the choice name. In the
`DataFormat` example above, `DataFormat.CSV` has the value `"CSV"` and
`DataFormat.JSON` has the value `"JSON"`.

You can assign explicit values to choices using `=`. Values can be any WDL
type—primitives, compound types, or even structs.

```wdl
version 1.3

# String values (explicit)
enum Color {
  Red = "#FF0000",
  Green = "#00FF00",
  Blue = "#0000FF"
}

# Map values for configuration presets
enum ResourcePreset {
  Small = {"cpu": 2, "memory_gb": 4},
  Medium = {"cpu": 8, "memory_gb": 16},
  Large = {"cpu": 32, "memory_gb": 64}
}

# Array values
enum AnalysisMode {
  Quick = ["--fast", "--no-validation"],
  Standard = ["--validate"],
  Thorough = ["--validate", "--deep-check", "--full-report"]
}
```

## Extracting values with `value()`

To get the underlying value from an enum choice, use the `value()` standard
library function.

```wdl
version 1.3

enum Color {
  Red = "#FF0000",
  Green = "#00FF00",
  Blue = "#0000FF"
}

workflow example {
  Color favorite = Color.Red
  String hex_code = value(favorite)  # "#FF0000"
}
```

## Explicit vs implicit typing

Enums can be explicitly typed by adding a type in square brackets after the
name, or implicitly typed by letting WDL infer the type from the values.

```wdl
version 1.3

# Explicitly typed as Float—the Int value 3 coerces to Float
enum ExplicitFloat[Float] {
  Three = 3,
  Pi = 3.14159
}

# Implicitly typed—WDL infers Float from the values
enum ImplicitFloat {
  Three = 3.0,
  Pi = 3.14159
}

# Implicitly typed as String (the default when no values are provided)
enum Status {
  Pending,
  Running,
  Complete
}
```

If values don't coerce to a common type, WDL will report an error:

```wdl
# ERROR: Int and String don't coerce to a common type
enum Invalid {
  Number = 42,
  Text = "hello"
}
```

## Comparing enums

Two enum values can be compared for equality using `==` or `!=`. To be equal,
they must be the same choice of the same enum type.

```wdl
version 1.3

enum Priority {
  Low,
  Medium,
  High
}

workflow example {
  Priority level = Priority.High

  Boolean is_high = (level == Priority.High)   # true
  Boolean is_low = (level == Priority.Low)     # false
}
```

Enum values cannot be compared with ordinal operators (`<`, `>`, `<=`, `>=`).

## String interpolation

When an enum is used in string interpolation, it serializes to its choice name
(not its underlying value).

```wdl
version 1.3

enum Color {
  Red = "#FF0000"
}

workflow example {
  Color c = Color.Red
  String message = "Selected: ~{c}"        # "Selected: Red"
  String hex = "Hex code: ~{value(c)}"     # "Hex code: #FF0000"
}
```

[spec-enums]: https://github.com/openwdl/wdl/blob/wdl-1.3/SPEC.md#enumeration-types-enums
