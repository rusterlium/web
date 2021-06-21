---
sidebar_position: 2
---

# Cheat Sheet

## Type mapping
This is a quick overview of the `Encoder`/`Decoder` implementation Rustler provides out of the box.

:::note
This is incomplete, contributions are welcome.
:::

| Rust Type | Elixir Type | Notes |
| --------- | ----------- | ----- |
| unsigned integer primitives (`u8`, `u16`, `u32`, `u64`) | number `0..2^n` | Decoding will fail if number is out of range of primitive. Encoding will never fail. |
| signed integer primitives (`i8`, `i16`, `i32`, `i64`) | number `(n^2/2-1)..(n^2/2)` | Same as unsigned. |
| `String` | `String.t()` | Decoding will perform UTF-8 validation of the full binary, will fail if binary is not valid UTF-8. Encoding will never fail. If more control is desired, you can work with the string as a `Binary` |
| `Vec<T>` | `list()` | List must be proper. |
| `Vec<u8>` | `[0..256]` | This is a special case of `Vec<T>`. If you want `Vec<u8>` to encode into a binary, you can either copy it into a `OwnedBinary` or make a wrapper struct with the correct encoder implementation. |
| `rustler::Binary`/`rustler::OwnedBinary` | `binary()` | `OwnedBinary` is owned and mutable, `Binary` can be shared and is immutable. |

## Scenarios

* I want to pass data between Elixir and Rust.
  * I want my data encoded and readable as a normal term.
    * For simple types, [encoded and decoder implementations](#type-mapping).
    * For composite types like Elixir structs, use the [macros we provide for generating `Encoder`/`Decoder`s](/docs/concepts/composite-macros).
    * For iterating maps or lists, use the `MapIterator` or `ListIterator` structs.
  * I want my data to stay in it's Rust representation.
    * Use [resources](/docs/concepts/resources)
