---
sidebar_position: 2
---

# Composite Macros

Rustler provides a bunch of macros for generating encoder and decoder implementations for composite data structures. These proc macros are implemented in the `rustler_codegen` crate.

## Maps

The `NifMap` attribute will generate encoders and decoders from Elixir maps. Atoms are used as keys. It is used as follows:

```rust
#[derive(NifMap)]
struct MyMap {
    a: u32,
    b: u32,
}
```

## Elixir Structs

The `NifStruct` attribute will generate encoders and decoders from an Elixir struct. It is mostly identical to the `NifMap` attribute, but also manages the `:__struct__` field. It is used as follows:

```rust
#[derive(NifStruct)]
#[module = "Elixir.MyStruct"]
struct MyStruct {
    a: u32,
    b: u32,
}
```
