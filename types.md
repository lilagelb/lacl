# Types

## String

A `string` is a standard UTF-8 string of characters. It is the default type of values.

### In Values

The string passed as a value will be interpreted as a raw string.

### In Expressions

A string is denoted by being wrapped in double quotes `""`. The following escapes are supported:
- `\n` - newline
- `\r` - carriage return
- `\t` - tab
- `\"` - double quote

#### Operations

`string`s only have the `+` operation defined, where both operands are `string`s. It serves to concatenate two strings.

---

## Integer

An `int` is stored internally as a 64-bit integer, so can hold any integer value between -9223372036854775808 and 9223372036854775807.

### In Values

Valid forms:
- Optionally begin with a minus sign
- Are a string of purely decimal numerals
- The numerals may be separated by underscores, though cannot have leading or trailing underscores (note that there is no restriction on where in the number they appear)

**Examples:** `42`, `-3`, `-9_223_372_036_854_775_808`, `92_23372_03_6_854775807`

### In Expressions

Anything conforming to the requirements for values will be taken to be an `int` when parsing expressions.

#### Operations

| Operation | RHS type         | Return type | Description                                |
| --------- | ---------------- | ----------- | ------------------------------------------ |
| `+`       | `int`            | `int`       | Arithmetic integer addition                |
| `+`       | `float`          | `float`     | Arithmetic float addition                  |
| `-`       | `int`            | `int`       | Arithmetic integer subtraction             |
| `-`       | `float`          | `float`     | Arithmetic float subtraction               |
| `*`       | `int`            | `int`       | Integer scalar multiplication              |
| `*`       | `float`          | `float`     | Floating-point scalar multiplication       |
| `*`       | `duration`       | `duration`  | Integer scalar multiplication of durations |
| `*`       | `size`           | `size`      | Integer scalar multiplication of sizes                                           |
| `/`       | `int` or `float` | `float`     | Floating-point division                    |

---

## Float

A `float` is stored internally as a 64-bit float (the binary64 type from IEEE 754-2008, i.e. the `f64` type in Rust).

### In Values

Valid forms:
- Optionally begin with a minus sign
- Contain a decimal point with at least one decimal numeral on each side
- May contain underscores as a separator, though cannot have leading or trailing underscores on either the integer or fractional part

**Examples:** `3.141_592`, `-2.718`, `0.2`, `-2.0`

### In Expressions

Anything conforming to the requirements for values will be taken to be an `float` when parsing expressions.

#### Operations

| Operation | RHS type         | Return type | Description                                                                             |
| --------- | ---------------- | ----------- | --------------------------------------------------------------------------------------- |
| `+`       | `float` or `int` | `float`     | Floating-point addition                                                                 |
| `-`       | `float` or `int` | `float`     | Floating-point subtraction                                                              |
| `*`       | `float` or `int` | `float`     | Floating-point scalar multiplication                                                    |
| `*`       | `duration`       | `duration`  | Floating-point scalar multiplication of durations. Any fractional remainder is dropped. |
| `*`       | `size`           | `size`      | Floating-point scalar multiplication of sizes. Any fractional remainder is dropped.     |
| `/`       | `float`          | `float`     | Floating-point division                                                                                        |

---

## Boolean

A `bool` can have one of two values: `true` and `false`.

### In Values

Valid forms:
- `true` for true
- `false` for false

### In Expressions

`bool` values cannot be used in expressions (variables holding `bool`s can).

#### Operations

| Operation | RHS type | Return type | Description |
| --------- | -------- | ----------- | ----------- |
| `+`       | `bool`   | `bool`      | Boolean OR  |
| `*`       | `bool`   | `bool`      | Boolean AND            |

---

### Version

A `version` stores a semantic versioning string.

### In Values

Valid forms:
- Optionally begin with a lowercase `v`
- Only feature up to three groups of decimal numerals
- Each group is separated by a full stop `.`

**Examples:** `3`, `4.2`, `1.85.0`

### In Expressions

In expressions, the leading `v` is not optional - it denotes a `version` (to differentiate from `int`s and `float`s)

#### Operations

| Operation | RHS type  | Return type | Description                                                                                                        |
| --------- | --------- | ----------- | ------------------------------------------------------------------------------------------------------------------ |
| `+`       | `version` | `version`   | Performs integer addition on each version part, i.e. `major_1 + major_2`, `minor_1 + minor_2`, `patch_1 + patch_2` |
| `-`       | `version` | `version`   | Performs integer subtraction on each version part, i.e. `major_1 - major_2`, `minor_1 - minor_2`, `patch_1 - patch_2`                                                    |

---

## Datetime

A `datetime` stores a date or time.

### In Values

Valid forms have, in this order:
- A date in ISO-8601 format, with or without the separating dashes
- A separator, which can be either a single space ` ` or a capital `T`
- A time in ISO-8601 format, with or without the separating colons
- A time offset, either:
	- `Z` for UTC
	- A plus or minus, followed immediately by an hour and minute value, with or without the separating colon

Not all datetimes have to have all parts:
- If a date is specified, the time is optional
- If a time is specified, the date is optional
- If only *one* of a date and time are specified, the separator should be omitted
- Times do not have to have seconds specified
- Times do not have to have an offset specified
	- An offset cannot be given without a time

Note: any part of the datetime left unpopulated will be taken to be zero.

**Examples:** `2023-09-08`, `1702`, `17:02:49+01:00`, `16:02:00Z`, `20230908T160200Z`, `17:02`

### In Expressions

`datetime` values cannot be used in expressions (variables holding `datetime`s can).

#### Operations

| Operation | RHS type   | Return type | Description                                                                                          |
| --------- | ---------- | ----------- | ---------------------------------------------------------------------------------------------------- |
| `+`       | `duration` | `datetime`  | Returns the datetime that is RHS after the LHS. Any unit of time smaller than the second left after the operation is dropped. |
| `-`       | `duration` | `datetime`  | Returns the datetime that is RHS before the LHS. Any unit of time smaller than the second left after the operation is dropped.                                                                                                     |

---

## Duration

A `duration` stores a length of time. It is stored as a 64-bit unsigned integer number of nanoseconds, so can represent a duration of up to 30500.57 7-day weeks, or roughly 83 years.

## In Values

Valid forms:
- Start with a string of decimal digits, with optional underscores for separation (though there cannot be a leading underscore)
- Optionally, a decimal point may be used to provide a fractional part, with another string of decimal digits (again, underscores may be used as a separator)
- Conclude with one of the following suffixes, optionally with an underscore separating the quantity from the suffix
	- `ns`: nanoseconds
	- `us`: microseconds
	- `ms`: milliseconds
	- `s`: seconds
	- `min`: minutes
	- `hr`: hours
	- `wk`: weeks

**Examples:** `364ns`, `10232us`, `34ms`, `23.4s`, `5min`, `8_hr`, `2day`, `9.5week`

### In Expressions

Anything conforming to the requirements for values will be taken to be a `duration` when parsing expressions.

#### Operations

| Operation | RHS type         | Return type | Description                                                                                                                                  |
| --------- | ---------------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `+`       | `duration`       | `duration`  | Addition of durations                                                                                                                        |
| `-`       | `duration`       | `duration`  | Subtraction of durations                                                                                                                     |
| `*`       | `int` or `float` | `duration`  | Scalar multiplication of durations. Any fractional part (time units smaller than the nanosecond) left after the operation is dropped |
| `/`       | `duration`       | `float`     | Division of durations, i.e. how many of one duration fits in another                                                                         |
| `/`       | `int` or `float` | `duration`  | Division of durations. Any fractional part (time units smaller than the nanosecond) left after the operation is dropped                                                                                                                                             |

---

## Size

A `size` holds a digital size. It is stored internally as a 64-bit unsigned integer number of bits, so can represent a size of up to 2 exbibytes.

### In Values

Valid forms are formed as follows:
- A string of decimal digits, with optional underscores for separation (though there cannot be a leading underscore)
- Optionally, a decimal point may be used to provide a fractional part, with another string of decimal digits (again, underscores may be used as a separator)
- An optional separating underscore
- An optional multiplier specifier, one of:
	- `k` - kilo, x1000
	- `Ki` - kibi, x1024
	- `M` - mega, x1000000
	- `Mi` - mebi, x1048576
	- `G` - giga, x1000000000
	- `Gi` - gibi, x1073741824
	- `T` - tera, x1000000000000
	- `Ti` - tebi, x1099511627776
	- `P` - petta, x1000000000000000
	- `Pi` - pebi, x1125899906842624
	- `E` - exa, x1000000000000000000
	- `Ei` - exbi, x1152921504606846976
- Either `b` for bits, or `B` for bytes

**Examples:** `2.4kB`, `4_TiB`, `3.2Mib`

### In Expressions

Anything conforming to the requirements for values will be taken to be a `size` when parsing expressions.

#### Operations

| Operation | RHS type         | Return type | Description                                                                                                               |     |     |     |     |
| --------- | ---------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------- | --- | --- | --- | --- |
| `+`       | `size`           | `size`      | Addition of sizes                                                                                                         |     |     |     |     |
| `-`       | `size`           | `size`      | Subtraction of sizes                                                                                                      |     |     |     |     |
| `*`       | `int` or `float` | `size`      | Scalar multiplication of sizes. Any fractional part (size units smaller than the bit) left after the operation is dropped |     |     |     |     |
| `/`       | `size`           | `float`     | Division of sizes, i.e. how many of one size fits in another                                                              |     |     |     |     |
| `/`       | `int` or `float` | `size`      | Division of sizes. Any fractional part (size units smaller than the bit) left after the operation is dropped              |     |     |     |     |
