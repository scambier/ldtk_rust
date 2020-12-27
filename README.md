# LDtk Rust Library

This library enables access to [LDtk](https://ldtk.io) data for use in Rust.
LDtk is a 2D level editor for games that supports multiple tile layers, powerful
auto-tiling rules, entity placement and more.

ldtk_rust parses the JSON format created by LDtk into a typed Rust object.
You should be able to use this to generate game levels in any Rust game framework.

## Status

Currently all sample .ldtk files included in the LDtk 0.6.1 release load without
any errors. You can use the [basic example](examples/basic.rs) to check your own
files.

Most of the JSON data from the LDtk files is supported, except fields that seem
to be only used by the editor itself. Open an issue if you find a useful field
not included. Most projects will likley focus on the data in the "Levels" section
of the JSON.

## Getting Started

Calling the new() method on the LdtkFile struct with the path to a LDtk file will
populate a struct that closely resembles the [LDtk JSON format](https://ldtk.io/json/).

```rust
use ldtk_rust::LdtkFile;

fn main() {
    let file_path = "assets/AutoLayers_4_Advanced.ldtk".to_string();
    let ldtk = LdtkFile::new(file_path);
    println!("First level pxHei is {}!", ldtk.levels[0].px_hei);
}
```

CamelCase field naming used in JSON is converted to the snake_case style used in Rust.
A few other field names are altered as needed, for instance the field "type" cannot be
used in Rust since that is a reserved word.

Your editor's autocomplete should help you visualize your options, or you can generate
API docs with "cargo doc --open".

## Run the Examples

You can build and/or run the programs in the example folder using cargo:

```bash
> cargo run --example basic
```

Example dependencies do not load when compliling the library for production.

## Using with [Bevy Engine](https://bevyengine.org/)

An example running in Bevy 0.4 is included in the [examples](examples/) directory. If the LDtk file makes reference to more than one tileset, you may have intermittent issues due to [issue 1056](https://github.com/bevyengine/bevy/issues/1056)


## Implementation Details

This library uses [Serde] to parse the JSON file, so most of the code simply defines structs
that match what is expected in the file. However, a few decisions were made:

* CamelCase names (preferred in JSON) are changed to snake_case names (preferred in Rust)
* JSON types of String, Int and Float become Rust types of String, i32 and f32
* "Type" is a reserved word in Rust, so JSON fields named "type" are prefixed with the struct's
name. For example: "layer_type"
* Fields that allow null as a value are wrapped in an Option (example: "bgColor: Option<String>")
* There is one dynamic type in the source file: the "__value" field in the FieldInstance struct.
In Rust this brought in as a "serde::Value" type. This may not be the best way to hanlde, but
projects consuming this library probably know what is coming in this field and can match/convert
it accordingly.
* The "tileMode" field of the auto-layer "rule" definition node is documented as an enum but is
presented as a String in the JSON and is typed as a String in Rust.
