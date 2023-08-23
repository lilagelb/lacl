# Lilagelb's Application Configuration Language - Specification

## Naming

LACL field and section names can only contain the standard charset of `[A-Za-z0-9_\]`, and cannot start with a number.

## Datatypes

LACL has 4 datatypes:
- Strings (this is the default)
- Integers
- Floats
- Booleans

LACL does *not* have a concept of null. There are many great explanations on the web as to why null is generally a bad idea, so I won't include one here.

## Fields

**Fields** are the basic storage units in LACL. They are individual, named values that hold a piece of data. The basic syntax for field definition is `<field> = <value>`.

However, there is additional syntax for *explicitly* declaring the type of a field: `<field> [<type>] = <value>`, where `<type>` must be one of:
- `string` (this is the default)
- `int`
- `float`
- `bool`

```lacl
height [int] = 164
hair_colour = black
// hair_colour [string] = black  <-- this is the fully classified declaration
//                                   but since `string` is the default type, it is superfluous
```

Note that explicit type declaration is often unnecessary, since the file will be deserialised into a specific schema, which will know the correct types. It is a feature of the language primarily for the sake of [expressions](#Expressions).

## Sections

LACL files are split into **sections**, each of which can contain its own sections, each of which can contain its own sections, etc. To start a section, use a hash `#`, followed by the section name. It is recommended, but not necessary, to insert a space after the hash before the name for the sake of readability (whitespace is trimmed from both ends during parsing).

The below starts a section called `apple` and defines fields `price` and `colour` within it:

```lacl
# apple
price: 1.20
colour: red
```

### Subsections

Each section may be split into **subsections**, denoted by two hashes `##`. For example, the below defines a section called `fruits` containing subsections `apple` and `orange`:

```lacl
# fruits

## apple
price: 1.20
colour: red

## orange
price: 1.80
colour: orange
```

You can add more hashes for ever-smaller sections (e.g. `####`). You can add as many as you like (though if you find that you've gone above three or four, you should probably consider moving to a more structured markup language).

### The Root Section

The base section is called the **root** section, and is held in the reserved name `root`.
It encompasses the entirety of the file.

## Comments

Comments are denoted by a double-slash `//`.
Everything past this will be completely ignored in the parsing of the file.

```lacl
// this is a comment
```

## Lists

A **list** is written as a set of bullet points on newlines after the field name,
using `- ` for the bullets (the whitespace is *technically* unnecessary, but is recommended for readability). It can only contain *exactly one* datatype (that declared by the field definition). For example, the below defines a field containing a list of fruits:

```lacl
fruits:
- apple
- orange
- banana
```

Or, with an explicit type declaration:

```lacl
fruits [string]:
- apple
- orange
- banana
```

## Expressions

**Expressions** are what make LACL somewhat unique: they are a system allowing for fields to reference other fields, and perform basic operations on those values. To use an expression to declare a field, use the walrus `:=` rather than just the equals for field assignment.

```lacl
// a standard field
just_a_field = // a value

// an expression
expression := // an expression
```

The type defaults change for expressions:
- a string of characters beginning with `[A-Za-z_]` is taken to be a [reference](#References), which evaluates to the value of the field to which it refers
	- e.g. `colour`, `fruits.apple.price`
- a string of numerals is taken to be an `int`
	- e.g. `42`, `-103`
- a string of numerals also containing either a full stop `.` or a comma `,` is taken to be a `float`
	- e.g. `3.141` (or `3,141`), `-2.718` (or `-2,718`)
- characters enclosed by double quotes `""` are taken to be a `string`
	- e.g. `"Hello, LACL!"`

This is illustrated by this example:

```lacl
value = blue        // this will take the string value "blue"
expression := blue  // this will look for and take the value of the field `blue`, throwing an error if `blue` doesn't exist
```

### References

A **reference** refers to another field in the file. Names are resolved from the section of the reference outwards (like variable scoping in most programming languages). To explicitly refer to a field outside of the current section where a field of the same name exists in the current section, use dot notation, e.g. `section.field`, to unambiguously specify the intended field. The field does not have to have been defined above the name reference; the parser will search the whole file for the field, erroring if it cannot find one fitting the specifier or if it cannot unambiguously pick a single field.

```lacl
# section_1
## section_1_1
field_1 = one
field_2 = two

## section_1_2
field_1 = eins
field_2 := field_1                // 'eins'
field_3 := section_1_1_1.field_1  // 'one'
							      // `section_1.section_1_1.field_1` would also be fine, but excessive
							      // `root.section_1.section_1_1.field_1` is the full identifier, but very rarely would that need to be used
field_4 := section_1_3.field_1    // 'un' - note that section_1_3.field_1 is defined *after* this reference

## section_1_3
field_1 = un
field_2 = deux
```

The parser will error if the inheriting field explicitly states a type other than that of the name reference. The parser will *not* perform type inference if the referenced field does not declare a type, but the inheriting field does - the referenced field will be taken to have the default string type.

```lacl
pi [float] = 3.1415926
pi_2 [int] := pi        // error - `pi` is of type float, but `pi_2` requires an int
```

```lacl
pi = 3.1415926
pi_2 [float] := pi  // error - `pi` is of type string, but `pi_2` requires a float
```

### Expression Evaluation

LACL expressions hinge around some very basic operators: `+`, `-`, `*`, `/`. They are only defined where intuitive:
- `int` and `float` types have them defined as the standard arithmetic operations.
	- The operators allow a mix of `int`s and `float`s in the operands, but the result will *only* have type `int` if *both* the operands were `int`s (even if the result is integer)
- The `string` type has only `+`, where it serves the purpose of concatenation.
- The `bool` type has only `+`, for boolean OR, and `*`, for boolean AND.

```lacl
a [int] = 3
b [int] = 4
c := a * b   // 12 [int]

pi [float] = 3.141
d := 4 * pi         // 12.564 [float]

hello = Hello
greeting = hello + ", LACL!"  // "Hello, LACL!"

here [bool] = true
looking [bool] = false
should_greet = here * looking  // false [bool]
```

These examples are of course unlikely to come up in application configuration, but hey, writing examples is hard.

A note of caution: expressions require strict type correctness. This is most likely to come up with a field of *intended* type `int` or `float`, but one where the type has not been explicitly specified. If this field is then used for arithmetic calculation, the parser will error. This is better demonstrated by an example:

```lacl
border_x = 3.2            // "3.2"
border_y := 2 * border_x  // this will error - multiplication is not defined between an int (2) and a string (border_x)
```

What should be written is:

```lacl
border_x [float] = 3.2    // 3.2
border_y := 2 * border_x  // 6.4
```
