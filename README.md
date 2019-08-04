# Register Description File Specification

Under construction.

Version: 0.1

## Objectives

* Machine readable file format for storing hardware
device register descriptions.
* Simple to edit and validate with a text editor.
* Provide enough details about the registers to
allow binding generation for different programming
languages.

## Motivation

Hardware device manufactures release reference manuals usually in
PDF file format. It is tricky to extract information about the registers
automatically from PDF file so manual copy and pasting is usually required for
writing device drivers.

## Example

```toml
[register_description]
version = "0.1"
name = "My device"
default_register_size = "8"
default_register_access = "rw"

[[register.general]]
name = "LED Register"
absolute_address = 0x123
bit_fields = [
    { bit = "7:3",  reserved = true },
    { bit = "2",    name = "LED Enabled", description = "Status of the LED." },
    { bit = "1:0",  name = "LED Color Setting" },
]

[[register.general.enums]]
name = "LED Color",
bit = "1:0",
values = [
    { value = 0, name = "Random" },
    { value = 1, name = "Red" },
    { value = 2, name = "Green" },
    { value = 3, name = "Blue" },
]
```

## File format

Register description files are written in [TOML v0.5.0](https://github.com/toml-lang/toml/blob/master/versions/en/toml-v0.5.0.md).

## Register bit indexing

Register bit with index zero is the least significant bit (LSB).

## New value types

These types extend the regular TOML value types by adding
some extra requirements.

### `Identifier`

A `String` which matches regex `^[a-zA-Z]{1}([a-zA-Z0-9 ]*[a-zA-Z0-9]{1}|[a-zA-Z0-9]*)$`

### `RegisterSize`

A `String` with value of `8`, `16`, `32` or `64`.

### `RegisterAccess`

A `String` with value of `r`, `w` or `rw`.

### `UnsignedInteger`

An `Integer` which contains only unsigned values.

### `BitField`

A `String` with special syntax.

#### Bit field with length one

The `String` must contain an unsigned decimal number which is register bit index.

Examples: `0`, `1`, `7`, `15`

#### Bit field with length greater than one

The `String` contains two unsigned decimal numbers separated with a double colon.
The first number is the most significant bit (MSB) of the bit field and
the second number is least significant bit (LSB) of the bit field.
The numbers are also register bit indexes and have this property: `msb > lsb`.

Examples: `1:0`, `7:2`, `15:8`

## The `[register_description]` section

```toml
[register_description]
version = "0.1"
name = "My device"
default_register_size = "8"
```

### The `version` field

Type: `String`

Register description file specification version.

### The `name` field

Type: `Identifier`

### The `description` field (optional)

Type: `String`

### The `default_register_size` field (optional)

Type: `RegisterSize`

When this field is set you can omit field `size` when specifying
a register.

### The `default_register_access` field (optional)

Type: `RegisterAccess`

When this field is set you can omit field `access` when specifying
a register.

### The `extension` field (optional)

Type: `String`

Valid value is a specification extension name.

Specification extension can add and modify register flags.
Extensions are useful for adding
device specific information about the registers. Available extensions are
documented in separate files in the `extensions` directory.

## The `[[register]]` sections

You can set register to a specific group by modifying
section name to `[[register.group_name]]`. If group
is specified once, then all registers must belong to some group.

```toml
[[register.general]]
name = "LED Register"
absolute_address = 0x123
bit_fields = [
    { bit = "7:3",  reserved = true },
    { bit = "2",    name = "LED Enabled", description = "Status of the LED." },
    { bit = "1:0",  name = "LED Color Setting" },
]

[[register.general.enum]]
name = "LED Color",
bit = "1:0",
values = [
    { value = 0, name = "Random" },
    { value = 1, name = "Red" },
    { value = 2, name = "Green" },
    { value = 3, name = "Blue" },
]
```

### The `name` field

Type: `Identifier`

### The `description` field (optional)

Type: `String`

### The `size` field (optional if global default is set)

Type: `RegisterSize`

### The `access` field (optional if global default is set)

Type: `RegisterAccess`

### Specifying register location

One register location field is required for the `[[register]]` section.

* `absolute_address` - Port or memory address which doesn't need additional calculations.
* `relative_address` - Port or memory address which needs additional calculations.
* `index` - Register index number. Register is accessed through index register.

Type: `UnsignedInteger`

Specifying multiple register location fields is not supported.

### Register bit fields

Register field `bit_fields` is a list of bit fields in the register.

Type: `Array of Inline Tables`

#### Requirements

* Bit field must not overlap with other bit fields.
* Enough bit fields must be declared so that every bit in the register is used.
* Bit field must stay within the register bounds.

#### The `bit` field

Type: `BitField`

#### The `name` field (optional if `reserved = true`)

Type: `Identifier`

#### The `reserved` field (optional, defaults to `false`)

Type: `Boolean`

Mark bit field to contain reserved bits.

#### The `description` field (optional)

Type: `String`

### The `[[register.enum]]` sections (optional)

It is possible to define enums for previously defined register
with this section. Register bit field might have a set of valid
values and this feature allows listing of those values.

If register groups are used, then enum section is defined
with `[[register.group_name.enum]]`.

#### Requirements

* Enum bit field matches with previously defined register bit field which
  is not marked as reserved.
* Only one register enum can exist per register bit field.

#### The `name` field

Type: `Identifier`

#### The `bit` field

Type: `BitField`

#### The `description` field (optional)

Type: `String`

#### Enum values

Enum field `values` is a list of enum values.

Type: `Array of Inline Tables`

##### Requirements

* No duplicate integer values.
* Enum value must not overflow the enum bit field.

##### The `value` field

Type: `Integer`

##### The `name` field

Type: `Identifier`

##### The `description` field (optional)

Type: `String`
