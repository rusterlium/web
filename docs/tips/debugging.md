---
sidebar_position: 1
---

# Debugging hard crashes

Although the Rust language is memory safe by default, it also provides various escape hatches that allow you to work with pointers directly, and to interact with C code.

As such, if you ever encounter any hard crashes of the BEAM, it is useful to know how to debug these.

## Common steps

### 1. Launch normal ERTS in LLDB/Valgrind

If your problem is simple and within your NIF, you could attempt launching the normal ERTS within a debugger. This can give you helpful errors if the error manifests within the code of the NIF itself.

You should build your NIF in debug mode, or at the very least make sure debug symbols are enabled.

### 2. Build your NIF with sanitizers

Building your NIF with the various sanitizers from LLVM can often reveal issues quickly and easily.

The following sanitizers are probably the most useful:
* [AddressSanitizer](https://en.wikipedia.org/wiki/AddressSanitizer) - Detects various memory corruption bugs.
* [MemorySanitizer](https://clang.llvm.org/docs/MemorySanitizer.html) - Detects reads of uninitialized memory.
* [ThreadSanitizer](https://clang.llvm.org/docs/ThreadSanitizer.html) - Detects data races/threading issues.

You will need Unstable Rust installed, how to enable the sanitizers is documented in the [rust unstable book](https://doc.rust-lang.org/unstable-book/compiler-flags/sanitizer.html).

#### You can start your app

If you can start your app successfully without things crashing, the easiest thing you can do is attach LLDB to the ERTS process once started.

```bash
$ lldb
(lldb) process attach --pid <ERTS_PID>
```

You can then perform the actions that make your app crash. LLDB should trap the crash and enable you to inspect the crashed process.

#### Your app crashes on boot

You can hack LLDB into the launch scripts.

TODO document this better.

### 3. Use a debug emulator

Start off by [building a debug emulator](#building-a-debug-emulator).

The main command we will use here is the `cerl` command. It enables you to automatically launch ERTS with a wire variety of debug tools. In order to launch your app through `cerl`, do the following:

1. Set `ELIXIR_CLI_DRY_RUN` in your terminal and launch your app. This should print out a long command prefixed with `erl`.
2. Replace the `erl` prefix with `cerl`. Right after the `cerl` command you should put a launch flag (`cerl --lldb`, `cerl --valgrind`).

## Appendix

### Building a debug emulator

To build ERTS with debug symbols and more, you can follow the instructions listed [here](https://github.com/erlang/otp/blob/master/HOWTO/INSTALL.md#how-to-build-a-debug-enabled-erlang-runtime-system).

The short version is:
```bash
$ git clone https://github.com/erlang/otp.git
$ cd otp
$ ./configure
$ export ERL_TOP=`pwd`
$ make
$ cd $ERL_TOP/erts/emulator
$ make debug
```

In order to run stuff with the ERTS you just built, you need to add it to your path:
```bash
$ export PATH="$(pwd):$PATH"
```

After this, the following should be true:
* `iex` should print `Erlang/OTP <version> [DEVELOPMENT]`. This means iex is running on the debug emulator.
* `cerl` should be runnable.
