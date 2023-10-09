# RISC-V RV32I[MACF] emulator
![GitHub Actions](https://github.com/sysprog21/rv32emu/actions/workflows/main.yml/badge.svg)
```
                       /--===============------\
      ______     __    | |⎺⎺⎺⎺⎺⎺⎺⎺⎺⎺⎺⎺⎺⎺⎺|     |
     |  _ \ \   / /    | |               |     |
     | |_) \ \ / /     | |   Emulator!   |     |
     |  _ < \ V /      | |               |     |
     |_| \_\ \_/       | |_______________|     |
      _________        |                   ::::|
     |___ /___ \       '======================='
       |_ \ __) |      //-'-'-'-'-'-'-'-'-'-'-\\
      ___) / __/      //_'_'_'_'_'_'_'_'_'_'_'_\\
     |____/_____|     [-------------------------]
```

`rv32emu` is an emulator for the 32 bit [RISC-V processor model](https://riscv.org/technical/specifications/) (RV32),
faithfully implementing the RISC-V instruction set architecture (ISA).
It serves as an exercise in modeling a modern RISC-based processor, demonstrating
the device's operations without the complexities of a hardware implementation.
The code is designed to be accessible and expandable, making it an ideal educational
tool and starting point for customization. It is primarily written in C99, with
a focus on efficiency and readability.

Features:
* Fast interpreter for executing the RV32 ISA
* Comprehensive support for RV32I and M, A, C extensions
* Partial support for the F extension
* Memory-efficient design
* Built-in ELF loader
* Implementation of commonly used newlib system calls
* Experimental SDL-based display/event/audio system calls for running video games
* Support for remote GDB debugging

## Build and Verify

`rv32emu` relies on certain third-party packages for full functionality and access to all its features.
To ensure proper operation, the target system should have the [SDL2 library](https://www.libsdl.org/) 
and [SDL2_Mixer library](https://wiki.libsdl.org/SDL2_mixer) installed.
* macOS: `brew install sdl2 sdl2_mixer`
* Ubuntu Linux / Debian: `sudo apt install libsdl2-dev libsdl2-mixer-dev`

Build the emulator.
```shell
$ make
```

Run sample RV32I[M] programs:
```shell
$ make check
```

Run [Doom](https://en.wikipedia.org/wiki/Doom_(1993_video_game)), the classical video game, via `rv32emu`:
```shell
$ make doom
```

The build script will then download data file for Doom automatically.
When Doom is loaded and run, an SDL2-based window ought to appear.

If RV32F support is enabled (turned on by default), [Quake](https://en.wikipedia.org/wiki/Quake_(series))
demo program can be launched via:
```shell
$ make quake
```

The usage and limitations of Doom and Quake demo are listed in [docs/demo.md](docs/demo.md).

### Customization

`rv32emu` is configurable, and you can override the below variable(s) to fit your expectations:
* `ENABLE_EXT_M`: Standard Extension for Integer Multiplication and Division
* `ENABLE_EXT_A`: Standard Extension for Atomic Instructions
* `ENABLE_EXT_C`: Standard Extension for Compressed Instructions (RV32C.F excluded)
* `ENABLE_EXT_F`: Standard Extension for Single-Precision Floating Point Instructions
* `ENABLE_Zicsr`: Control and Status Register (CSR)
* `ENABLE_Zifencei`: Instruction-Fetch Fence
* `ENABLE_GDBSTUB` : GDB remote debugging support
* `ENABLE_SDL` : Experimental Display and Event System Calls

e.g., run `make ENABLE_EXT_F=0` for the build without floating-point support.

Alternatively, configure the above items in advance by executing `make config` and
specifying them in a configuration file. Subsequently, run `make` according to the provided
configurations. For example, employ the following commands:
```shell
$ make config ENABLE_SDL=0
$ make
```

### RISCOF

[RISCOF](https://github.com/riscv-software-src/riscof) (RISC-V Compatibility Framework) is
a Python based framework that facilitates testing of a RISC-V target against a golden reference model.

The RISC-V Architectural Tests, also known as [riscv-arch-test](https://github.com/riscv-non-isa/riscv-arch-test),
provide a fundamental set of tests that can be used to verify that the behavior of the
RISC-V model aligns with RISC-V standards while executing specific applications.
These tests are not meant to replace thorough design verification.

Reference signatures are generated by the formal RISC-V model [RISC-V SAIL](https://github.com/riscv/sail-riscv)
in Executable and Linkable Format (ELF) files.
ELF files contain multiple testing instructions, data, and signatures, such as `cadd-01.elf`.
The specific data locations that the testing model (this emulator) must write to during
the test are referred to as test signatures.
These test signatures are written upon completion of the test and are then compared to the reference signature.
Successful tests are indicated by matching signatures.

To install [RISCOF](https://riscof.readthedocs.io/en/stable/installation.html#install-riscof):
```shell
$ python3 -m pip install git+https://github.com/riscv/riscof
```

[RISC-V GNU Compiler Toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain) should be prepared in advance.
You can obtain prebuilt GNU toolchain for `riscv32-elf` from the [Automated Nightly Release](https://github.com/riscv-collab/riscv-gnu-toolchain/releases).
Then, run the following command:
```shell
$ make arch-test
```

For macOS users, installing `sdiff` might be required:
```shell
$ brew install diffutils
```

To run the tests for specific extension, set the environmental variable `RISCV_DEVICE` to one of `I`, `M`, `C`, `Zifencei`, `privilege`.
```shell
$ make arch-test RISCV_DEVICE=I
```

Current progress of this emulator in riscv-arch-test (RV32):
* Passed Tests
    - `I`: Base Integer Instruction Set
    - `M`: Standard Extension for Integer Multiplication and Division
    - `C`: Standard Extension for Compressed Instruction
    - `Zifencei`: Instruction-Fetch Fence
    - `privilege`: RISCV Privileged Specification
* Unsupported tests (runnable but incomplete)
    - `F` Standard Extension for Single-Precision Floating-Point

Detail in riscv-arch-test:
* [RISCOF document](https://riscof.readthedocs.io/en/stable/)
* [riscv-arch-test repository](https://github.com/riscv-non-isa/riscv-arch-test)
* [RISC-V Architectural Testing Framework](https://github.com/riscv-non-isa/riscv-arch-test/blob/master/doc/README.adoc)
* [RISC-V Architecture Test Format Specification](https://github.com/riscv-non-isa/riscv-arch-test/blob/master/spec/TestFormatSpec.adoc)

## GDB Remote Debugging

`rv32emu` is permitted to operate as gdbstub in an experimental manner since it supports
a limited number of [GDB Remote Serial Protocol](https://sourceware.org/gdb/onlinedocs/gdb/Remote-Protocol.html) (GDBRSP).
You must first build the emulator and set `ENABLE_GDBSTUB` to `1` in the `Makefile` in order
to activate this feature. After that, you might execute it using the command below.
```shell
$ build/rv32emu -g <binary>
```

The `<binary>` should be the ELF file in RISC-V 32 bit format. Additionally, it is advised
that you compile programs with the `-g` option in order to produce debug information in
your ELF files.

You can run `riscv-gdb` if the emulator starts up correctly without an error. It takes two
GDB commands to connect to the emulator after giving GDB the supported architecture of the
emulator and any debugging symbols it may have.

```shell
$ riscv32-unknown-elf-gdb
(gdb) file <binary>
(gdb) target remote :1234
```

Congratulate yourself if `riscv-gdb` does not produce an error message. Now that the GDB
command line is available, you can communicate with `rv32emu`.

### Dump registers as JSON

If the `-d [filename]` option is provided, the emulator will output registers in JSON format.
This feature can be utilized for tests involving the emulator, such as compiler tests.

You can also combine this option with `-q` to directly use the output.
For example, if you want to read the register x10 (a0), then run the following command:
```shell
$ build/rv32emu -d - out.elf -q | jq .x10
```

## RISC-V Instructions/Registers Usage Statistics

This is a static analysis tool for assessing the usage of RV32 instructions/registers
in a given target program.
Build this tool by running the following command:
```shell
$ make tool
```

After building, you can launch the tool using the following command:
```shell
$ build/rv_histogram [-ar] [target_program_path]
```

The tool includes two optional options:
* `-a`: output the analysis in ascending order(default is descending order)
* `-r`: output usage of registers(default is usage of instructions)

_Example Instructions Histogram_
![Instructions Hisrogram Example](docs/histogram-insn.png)

_Example Registers Histogram_
![Registers Hisrogram Example](docs/histogram-reg.png)

## Continuous Benchmarking

Continuous benchmarking is integrated into GitHub Actions,
allowing the committer and reviewer to examine the comment on benchmark comparisons between
the pull request commit(s) and the latest commit on the master branch within the conversation.
This comment is generated by the benchmark CI and provides an opportunity for discussion before merging.

The results of the benchmark will be rendered on a [GitHub page](https://sysprog21.github.io/rv32emu-bench/).
Check [benchmark-action/github-action-benchmark](https://github.com/benchmark-action/github-action-benchmark) for the reference of benchmark CI workflow.

There are several files that have the potential to significantly impact the performance of `rv32emu`, including:
* `src/decode.c`
* `src/rv32_template.c`
* `src/emulate.c`

As a result, any modifications made to these files will trigger the benchmark CI.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.

## External sources

In `rv32emu` repository, there are some prebuilt ELF files for testing purpose.
* `aes.elf` : See [tests/aes.c](tests/aes.c)
* `captcha.elf` : See [tests/captcha.c](tests/captcha.c)
* `cc.elf` : See [tests/cc](tests/cc)
* `chacha20.elf` : See [tests/chacha20](tests/chacha20)
* `coremark.elf` : See [eembc/coremark](https://github.com/eembc/coremark) [RV32M]
* `dhrystone.elf` : See [rv8-bench](https://github.com/michaeljclark/rv8-bench)
* `doom.elf` : See [sysprog21/doom_riscv](https://github.com/sysprog21/doom_riscv) [RV32M]
* `ieee754.elf` : See [tests/ieee754.c](tests/ieee754.c) [RV32F]
* `jit-bf.elf` : See [ezaki-k/xkon_beta](https://github.com/ezaki-k/xkon_beta)
* `lena.elf`: See [tests/lena.c](tests/lena.c)
* `line.elf` : See [tests/line.c](tests/line.c) [RV32M]
* `maj2random.elf` : See [tests/maj2random.c](tests/maj2random.c) [RV32F]
* `mandelbrot.elf` : See [tests/mandelbrot.c](tests/mandelbrot.c)
* `nqueens.elf` : See [tests/nqueens.c](tests/nqueens.c)
* `nyancat.elf` : See [tests/nyancat.c](tests/nyancat.c)
* `pi.elf` : See [tests/pi.c](tests/pi.c) [RV32M]
* `qrcode.elf` : See [tests/qrcode.c](tests/qrcode.c)
* `quake.elf` : See [sysprog21/quake-embedded](https://github.com/sysprog21/quake-embedded) [RV32F]
* `readelf.elf` : See [tests/readelf](tests/readelf)
* `richards.elf` : See [tests/richards.c](tests/richards.c)
* `rvsim.elf` : See [tests/rvsim.c](tests/rvsim.c)
* `scimark2.elf` : See [tests/scimark2](tests/scimark2) [RV32MF]
* `stream.elf` : See [tests/stream](tests/stream.c) [RV32MF]

## Reference

* [Writing a simple RISC-V emulator in plain C](https://fmash16.github.io/content/posts/riscv-emulator-in-c.html)
* [Writing a RISC-V Emulator in Rust](https://book.rvemu.app/)
* [Juraj's RISC-V note](https://jborza.com/tags/riscv/)
* [libriscv: RISC-V userspace emulator library](https://github.com/fwsGonzo/libriscv)
* [GUI-VP: RISC-V based Virtual Prototype (VP) for graphical application development](https://github.com/ics-jku/GUI-VP)
* [LupV: an education-friendly RISC-V based system emulator](https://gitlab.com/luplab/lupv)
* [yutongshen/RISC-V-Simulator](https://github.com/yutongshen/RISC-V-Simulator)
* [mini-rv32ima](https://github.com/cnlohr/mini-rv32ima) / [video: Writing a Really Tiny RISC-V Emulator](https://youtu.be/YT5vB3UqU_E)
* [RVVM](https://github.com/LekKit/RVVM)

## License

`rv32emu` is available under a permissive MIT-style license.
Use of this source code is governed by a MIT license that can be found in the [LICENSE](LICENSE) file.
