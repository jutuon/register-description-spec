# VGA register extension for register description files

Extension name: "vga"

## New fields in the `[[register]]` section

### The `index` field

Type of this field is Integer.

## Changes to existing `[[register]]` fields

Register address field can now optionally be a String value
containing a hexadecimal number with
one question mark. Question mark represents hexadecimal `B` or `D`. For example
setting flag `read_address = "0x3?4"` means that read address is `0x3B4` or `0x3D4`.
