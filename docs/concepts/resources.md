---
sidebar_position: 2
---

# Resources

Resources are often used when you want to return a Rust resource into Elixir without needing to convert the data into a term. This can be useful in a wire variety of cases, some examples:

* Wrapping a native library. Native libraries often contain structs that methods are called on to perform manipulations. Those structs can be exposed as a Resource, and the methods can be exposed as separate NIFs.
* Implementing an efficient data structure natively, for use in Elixir. You would wrap your data structure in a resource, and write NIFs to manipulate the structure.

## How to write a Resource

There are a couple of steps involved in writing a Resource:

1. Determine/write the type you want to expose as a resource. Because we implement a trait for this type automatically, this type needs to be defined in your NIF crate. Read [here](#only-locally-defined-types-can-be-registered-as-resources) for more information about this limitation.
2. Register the resource when the NIF is loaded. This is done by specifying a `load` function in the `rustler::init!` macro, and calling `rustler::resource!` for your type within that init function.
3. Create an instance of your resource by calling `ResourceArc::new` and returning it from a NIF.
4. Write other NIFs that accept your resource as an argument.

### Simple example

This is a simple example of a Resource. It wraps a simple `u32` integer, exposing NIFs for incrementing and getting the inner value.

```rust
// 1. We define the type we want to expose as a resource. In this 
//    case it is a `struct`.
struct MyStruct {
    num: Mutex<u32>,
}

#[rustler::nif(name = "new")]
fn new() -> ResourceArc<MyStruct> {
    let my_struct = MyStruct {
        num: Mutex::new(0),
    };

    // 3. We wrap our struct in a `ResourceArc` and return it.
    ResourceArc::new(my_struct)
}

#[rustler::nif(name = "increment")]
fn increment(resource: ResourceArc<MyStruct>) {
    // 4. Here we accept the resource as an argument, lock the inner 
    //    mutex and increment the inner number.
    let locked = resource.num.lock().unwrap();
    *locked += 1;
}

#[rustler::nif(name = "get")]
fn get(resource: ResourceArc<MyStruct>) -> u32 {
    // 4. Here we accept the resource as an argument, lock the inner 
    //    mutex, and return the value within.
    let locked = resource.num.lock().unwrap();
    *locked
}

// 2. Define a load function and call `rustler::resource!` on the 
//    struct we defined in step 1.
pub fn load(env: Env) -> bool {
    rustler::resource!(MyStruct, env);
    true
}

rustler::init!(
    "Elixir.MyNif",
    [
        new,
        increment,
        get,
    ],
    // 2. ... load function will be called on load.
    load = load
);
```

## Examples

* The [franz](https://github.com/scrogson/franz/) Kafka client library. See `producer` and `consumer`.

## Limitations

### Only locally defined types can be registered as resources

### Resources must be `Sync`

### Resources must be `+ 'static`
