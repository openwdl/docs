# Enum Functions <Badge type="tip" text="v1.3" />

- [`value`](#value)

## `value`

```wdl
T value(Enum)
```

Returns the underlying value associated with an enum choice.

**Parameters**

1. **`Enum`**: an enum choice of any enum type.

**Returns**

1. The choice's associated value.

**Example**

```wdl
enum Color {
  Red = "#FF0000",
  Green = "#00FF00",
  Blue = "#0000FF"
}

enum Priority {
  Low = 1,
  Medium = 5,
  High = 10
}

workflow example {
  Color color = Color.Red
  Priority priority = Priority.High

  String choice_name = "~{color}"    # "Red"
  String hex_value = value(color)    # "#FF0000"
  Int priority_num = value(priority) # 10
}
```
