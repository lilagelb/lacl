# Lilagelb's Application Configuration Language - Specification v0.3.0

## Naming

LACL field and section names can only contain the standard charset of `[A-Za-z0-9_]`, and cannot start with a number. They also cannot be a single underscore, as this has reserved meaning.

## Datatypes

LACL has 7 datatypes (see the [type specification](types.md) for more details):

| Type     | LACL Identifier | What does it hold?           | Examples                                 |
| -------- | --------------- | ---------------------------- | ---------------------------------------- |
| String   | `string`        | A raw character string       | `Hello, world!`                          |
| Integer  | `int`           | An integer value             | `42`, `-3`                               |
| Float    | `float`         | A floating-point value       | `3.14159`, `-2.718`                      |
| Boolean  | `bool`          | A boolean value              | `true`, `false`                          |
| Version  | `version`       | A semantic versioning number | `1.5.3`, `v1.34`                         |
| Datetime | `datetime`      | A date or time               | `2023-08-23`, `0815`, `20230823T081500Z` |
| Duration | `duration`      | A duration of time           | `23s`, `5min`, `8hr`, `2day`, `9week`                                         |
| Size     | `size`          | A digital file size          | `8KiB`, `7TB`,`256b`                     |

LACL does *not* have a concept of null. There are many great explanations on the web as to why null is generally a bad idea, so I won't include one here.

## Fields

**Fields** are the basic storage units in LACL. They are individual, named values that hold a piece of data. The basic syntax for field definition is `<field> = <value>`.

However, there is additional syntax for *explicitly* declaring the type of a field: `<field> [<type>] = <value>`, where `<type>` must be one of the identifiers listed in the table [above](#Datatypes). This is called a *type annotation*.

```lacl
height [int] = 164
hair_colour = black
// hair_colour [string] = black  <-- this is the fully classified declaration
//                                   but since `string` is the default type, it is superfluous
```

Note that type annotation is often unnecessary, since the file will be deserialised into a specific [schema](#Schemas), which will declare the correct types. It is a feature of the language primarily for the sake of [expressions](#Expressions).

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

You can add more hashes for ever-smaller sections (e.g. `####`). The number of hashes a section has is known as its **section level**. You can add as many as you like (though if you find that you've gone above a level of three or four, you should probably consider moving to a more structured markup language). Subsections must *strictly* have a section level of one more than the section that contains them (i.e. you cannot increase by more than one level at a time).

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
expression := // an expression, note the walrus `:=`
```

The type defaults change for expressions. A string of characters beginning with `[A-Za-z_]` is taken to be a [reference](#References), which evaluates to the value of the field to which it refers (e.g. `colour`, `fruits.apple.price`). This is illustrated by the below example:

```lacl
value = blue        // this will take the string value "blue"
expression := blue  // this will look for and take the value of the field `blue`, throwing an error if `blue` doesn't exist
```

See the [type specification](types.md) for the parsing details on types.

### References

A **reference** refers to another field in the file. Names are resolved from the section of the reference outwards (like variable scoping in most programming languages). To explicitly refer to a field outside of the current section where a field of the same name exists in the current section, use dot notation, e.g. `section.field`, to unambiguously specify the intended field. The field must have been defined above the name reference; the parser will not search the whole file for the field.It will error if it cannot find one fitting the specifier or if it cannot unambiguously pick a single field.

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
field_4 := section_1_3.field_1    // error - section_1_3.field_1 is defined *after* this reference

## section_1_3
field_1 = un
field_2 = deux
```

The parser will error if the inheriting field explicitly states a type other than that of the name reference. The parser will *not* perform type inference if the referenced field does not declare a type, but the inheriting field does - the referenced field will have been taken to have the default string type.

```lacl
pi [float] = 3.1415926
pi_2 [int] := pi        // error - `pi` is of type float, but `pi_2` requires an int
```

```lacl
pi = 3.1415926
pi_2 [float] := pi  // error - `pi` is of type string, but `pi_2` requires a float
```

### Expression Evaluation

LACL expressions hinge around some very basic operators: `+`, `-`, `*`, `/`, which have their standard precedences. They are only defined where intuitive, and details can be found in the [type specification](types.md). Parentheses `()` may also be used to alter evaluation order.

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

## Templates

Templates allow for further elimination of unnecessary duplication: they define patterns of sections and fields that the parser can insert into other parts of the document. They are particularly useful when writing [schemas](#Schemas).

### Defining a Template

Template definitions start with `@template`, followed by the name of the template. Template names follow the same rules as all other LACL names. To end a template block, use `@end`. The block below defines an empty template called `my_template`:

```lacl
@template my_template
@end
```

Then, simply use standard LACL syntax to define a hierarchy of sections and fields:

```
@template my_template
# my_template_section
my_template_field = a value
@end
```

Note that templates form their own 'virtual file' - section levels should start from 1. Upon template insertion, the section levels will be 'bumped' by the section level of the section in which the template was inserted. That is, a section of level 2 in a template inserted into a section of level 1 will have a section level of 3.

## Using a Template

To bring in a template, use `@insert`, followed by the template name.

```lacl
@template steel_hand_tool
type = hand tool
material = steel
@end

# products
## screwdriver
@insert steel_hand_tool

## hammer
@insert steel_hand_tool
```

This will expand to:

```lacl
# products
## screwdriver
type = hand tool
material = steel

## hammer
type = hand tool
material = steel
```

## Schemas

Schemas declare the types of fields for the parser. A LACL schema is a LACL file with no type annotations, where the value of each field is the type of that field. The first line of the file should be `@schema`, so the parser knows to interpret it as such. Take the example below:

```lacl
@schema

name = string
age = int

# appearance
height = float
eye_colour = string
```

This says that in the `root` section, the field `name` has type `string` and the field `age` has type `int`, whilst in the `appearance` section, the field `height` is a `float` and `eye_colour` a `string`.

### Parse Modes

Whilst LACL does not have to be parsed against a schema, doing so allows the data returned by the parser to be used with confidence that it is in the format and types that you expect, rather than performing the checks yourself.

When parsing LACL against a schema, the parser has three possible modes: strict, tolerant, and lax. These modes determine the behaviour of the parser when it comes across a field whose type it cannot determine from the schema.

| Parse mode | Behaviour on unreconcilable field type                                                     |
| ---------- | ------------------------------------------------------------------------------------------ |
| Strict     | Error on fields and sections not in schema                                                 |
| Tolerant   | Ignore fields and sections not in schema                                                   |
| Lax        | Accepts fields not declared in the schema, assuming them to have the default `string` type |

### Special Schema Syntax

In schemas, sections are permitted to have undeclared names, in order to match against any section with that name. Such sections with undeclared names should be given the name of '`_`', i.e. a single underscore. The schema below:

```lacl
@schema

# _
foo = int
bar = string
```

Will match the following LACL:

```lacl
# fizz
foo = 3
bar = buzz
```

### Schemas and Templates

[Templates](#Templates) are particularly useful when writing schemas, and schema templates come with a small amount of additional syntax: repeat specifiers. These are enclosed in square brackets `[]` directly after the `@insert` declaration, and consist of either a single number, or two numbers separated by a colon. A single number specifies that the source should have *exactly* that many matches of the template. Consider the schema below.

```lacl
@schema

@template product
# _
price = float
colour = string
@end

# products
@insert[2] product
```

This would match

```lacl
# products
## apple
price = 1.80
colour = red

## orange
price = 1.20
colour = orange
```

But if we removed either `apple` or `orange` there would not be enough - this would be an error. If more were specified, the behaviour would depend on the [parse mode](#Parse Modes), since it would be a section not defined in the schema.

If the repeat specifier is two numbers separated by a colon (i.e. `[<start>:<end>]`), then they specify a range of potential repeats. The range is fully inclusive, i.e. `[3:5]` would match 3, 4, or 5 repeats.
- If the first number is unspecified, it is taken to be zero (i.e. `[:3]`) would match 0, 1, 2, or 3 repeats.
- If the second number is unspecified, it is taken to be unbounded (i.e. `[2:]`) would match any number of repeats more than 1, but would require at least two.

Repeat specifiers are quite powerful when combined with [undeclared section names](#Special Schema Syntax). Revisiting the example above, we could change the schema to be:

```lacl
@schema

@template product
# _
price = float
colour = string
@end

# products
@insert[:] product
```

This allows us to match any number of products, so long as they conform to the template.
