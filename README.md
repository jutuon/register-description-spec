# Register description file specification.

Under construction.

Version: 0.1

## Objectives

* Machine readable file format for storing register
descriptions.
* Simple to edit and validate with a text editor.
* Provide enough details about the registers to
allow binding generation for different programming
languages.

## Table of contents

* [Motivation](#Motivation)
* [Specification](#Specification)

## Motivation

Hardware device manufactures often release reference manuals in
PDF file format. It is tricky to extract information about the registers
automatically from PDF file so often manual copy and pasting is required for
writing device drivers.

## Specification

Register description files are written in [TOML v0.5.0](https://github.com/toml-lang/toml/blob/master/versions/en/toml-v0.5.0.md).

### The `[register_description]` section

```toml
[register_description]
version = "0.1"
device_name = "my_device"
default_register_size_in_bits = 8
```

#### The `version` field

Version of the register description file version.

#### The `device_name` field

Device name for which the register description file is written to.

#### The `device_description` field (optional)

Description about the device for which the register description file is written to.

#### The `default_register_size_in_bits` field (optional)

When this is set you can omit field `size_in_bits` when specifying
a register.

#### The `extension` field (optional)

Enable extension which can add and modify register flags. Useful for adding
device specific information about the registers. Created extensions are
documented in separate files in the `extensions` directory. Value
of this field is String containing the name of the extension.

### The `[[register]]` sections

You can set register to a specific category by modifying
section name to `[[register.category_name]]`

```toml
[[registers.general]]
name = "LED Register"
read_address = 0x123
size_in_bits = 8
bit_fields = [
    { bit = "7:3",  reserved = true },
    { bit = "2",    name = "LED Enabled", description = "Status of the LED." },
    { bit = "1:0",  name = "LED Color Setting" },
]
enums = [
    {
        name = "LED Color",
        bit = "1:0",
        values = [
            { value = 0, name = "Random" },
            { value = 1, name = "Red" },
            { value = 2, name = "Green" },
            { value = 3, name = "Blue" },
        ]
    }
]
```

#### The `name` field

Set register name.

#### Register address fields

One register address field is required for `[[register]]` section.

* `read_address`
* `write_address`
* `read_write_address`

Type of register address field is integer. Specifying multiple register address fields
is not supported.

#### The `size_in_bits` field (optional if global default is set)

Set register size in bits.

#### Declaring bit fields

Field `bit_fields` contains the list of bit fields in the register.
Specified bit fields must not overlap and
enough bit fields must be listed so that every bit in the register is used.

##### Bit field object fields

* `bit` - String which contains number of the bit. Bit number zero is the least significant
bit (LSB). This String can also contain inclusive bit range using most significant bit (MSB) and LSB like this: "msb:lsb".
* `name` (optional if `reserved = true`)
* `description` (optional)
* `reserved` (optional, defaults to false) - Set this to set current bit field as reserved.

#### Declaring enumerations (optional)

Field `enums` contains the list of enumerations specific to the register.
Register bit field might have a set of valid values and this feature allows
listing of those values.

##### Enumeration object fields

* `name`   - Name of the enumeration.
* `bit`    - Register bit field location String which must be also present in the bit field list.
* `values` - List of enumeration value objects.
* `description` (optional)

##### Enumeration value object fields

* `value` - Integer
* `name`  - String
* `description` (optional)

#### The `description` field (optional)

Description about the register.
