# LLVM Project – Codebase Guide

This document explains the structure, technologies, and organization of the
LLVM monorepo so that new contributors can orient themselves quickly.

---

## Table of Contents

1. [What Is This Repository?](#what-is-this-repository)
2. [Top-Level Directories](#top-level-directories)
3. [Key Technologies and Languages](#key-technologies-and-languages)
4. [Code Organization](#code-organization)
   - [LLVM Core](#llvm-core)
   - [Clang Front-End](#clang-front-end)
   - [LLDB Debugger](#lldb-debugger)
   - [LLD Linker](#lld-linker)
   - [MLIR](#mlir)
   - [Runtime Libraries](#runtime-libraries)
5. [Component Relationships](#component-relationships)
6. [Build System](#build-system)
7. [Testing Infrastructure](#testing-infrastructure)

---

## What Is This Repository?

The **LLVM Project** is an open-source (Apache-2.0 with LLVM exceptions)
compiler infrastructure toolkit.  Its primary goals are:

* Provide a reusable, modular **compiler middle-end and back-end** (the core
  "LLVM" libraries) that translates a well-defined intermediate representation
  (LLVM IR) into efficient machine code.
* Host a family of **language front-ends** (Clang for C/C++/Objective-C,
  Flang for Fortran, etc.) that lower source code to LLVM IR.
* Bundle the matching **developer toolchain** components: linker (LLD),
  debugger (LLDB), standard libraries (libc++, LLVM-libc), and dozens of
  ancillary tools.

---

## Top-Level Directories

| Directory | What it contains |
|-----------|-----------------|
| `llvm/` | The core LLVM libraries and tools: IR, analysis/transform passes, code-generation back-ends for 27+ architectures, assembler/disassembler, `opt`, `llc`, `lli`, and ~100 other tools. |
| `clang/` | C / C++ / Objective-C / Objective-C++ front-end compiler.  Contains the lexer, parser, AST, semantic analyser, code generator, and the `clang` driver. |
| `clang-tools-extra/` | Additional Clang-based tools: `clang-tidy`, `clang-format`, `clangd` (language-server), `clang-doc`, and others. |
| `lldb/` | The LLVM debugger.  Supports a broad set of OS and architecture plug-ins. |
| `lld/` | The LLVM linker.  Separate sub-linkers for ELF, COFF/PE, Mach-O, WebAssembly, and XCOFF. |
| `mlir/` | Multi-Level Intermediate Representation framework.  Enables building domain-specific compilers (e.g., for ML, GPUs) as a stack of reusable dialects. |
| `flang/` | Modern Fortran front-end built entirely in C++17. |
| `flang-rt/` | Fortran runtime library companion to Flang. |
| `compiler-rt/` | Low-level compiler support: builtins, AddressSanitizer, UBSan, ThreadSanitizer, coverage, profiling. |
| `libcxx/` | LLVM's implementation of the C++ Standard Library (libc++). |
| `libcxxabi/` | C++ ABI support library (Itanium ABI, `__cxa_*`, `std::exception`). |
| `libc/` | A modern, modular LLVM C standard library implementation. |
| `libunwind/` | Stack unwinding library (C++ exception propagation, `_Unwind_*`). |
| `libclc/` | OpenCL C built-in library implementation. |
| `libsycl/` | SYCL (heterogeneous C++) programming model library. |
| `openmp/` | OpenMP runtime (CPU threading and GPU offload). |
| `offload/` | Low-level GPU offload runtime (used by OpenMP and HIP/CUDA). |
| `bolt/` | Post-link binary optimizer that uses profile data to reorder code for better I-cache efficiency. |
| `polly/` | Polyhedral loop optimizer (loop tiling, vectorisation, parallelisation). |
| `orc-rt/` | ORC (On-Request Compilation) JIT runtime library. |
| `llvm-libgcc/` | Shim that maps `libgcc` ABI onto `compiler-rt`+`libunwind`. |
| `cross-project-tests/` | Integration tests that span multiple sub-projects. |
| `cmake/` | Shared CMake modules used by all sub-projects. |
| `runtimes/` | Thin CMake wrapper for building runtime libraries (libc++, libc, etc.) against an already-built LLVM/Clang. |
| `third-party/` | Vendored third-party code (e.g., `unittest/` contains Google Test). |
| `utils/` | Miscellaneous scripts and utilities for project maintenance. |
| `.github/` | GitHub Actions CI/CD workflow definitions. |

---

## Key Technologies and Languages

### Programming Languages

* **C++17** – the required minimum standard for all LLVM C++ code.
* **C** – used in a few libraries (parts of `libc`, `compiler-rt`, `libunwind`)
  and for C-compatible public APIs.
* **LLVM IR / LLVM assembly** – the internal intermediate representation,
  written in `.ll` files and manipulated via the `llvm/IR/` APIs.
* **TableGen** (`.td` files) – a declarative language used by LLVM's
  `tblgen` tool to generate large amounts of boilerplate C++ code (instruction
  descriptions, register definitions, intrinsic tables, etc.).
* **Python** – test scripts, utility tools, the `lit` test runner, and the
  LLDB Python API.
* **Fortran** – the source language supported by Flang.
* **Bash / CMake** – build scripts and CI helpers.

### Key Libraries and Frameworks (within LLVM)

* **LLVM IR** (`llvm/include/llvm/IR/`) – The central typed, SSA-based
  intermediate representation shared by all front-ends and passes.
* **TableGen** (`llvm/utils/TableGen/`, `llvm/include/llvm/TableGen/`) –
  code-generation from declarative target descriptions.
* **LLVM Support** (`llvm/lib/Support/`) – platform-agnostic ADTs (vectors,
  maps, strings), file I/O, threading, error handling, and more.
* **LLVM Analysis** (`llvm/lib/Analysis/`) – program analysis passes (alias
  analysis, loop analysis, scalar evolution, etc.).
* **LLVM Transforms** (`llvm/lib/Transforms/`) – optimization passes
  (inlining, constant folding, loop unrolling, vectorisation, etc.).
* **MC layer** (`llvm/lib/MC/`) – low-level machine-code abstraction shared by
  the assembler and code generator.

---

## Code Organization

### LLVM Core

```
llvm/
├── include/llvm/        # Public headers, mirroring lib/ structure
├── lib/
│   ├── IR/              # Core IR types: Value, Function, Module, BasicBlock …
│   ├── Bitcode/         # Binary serialization of LLVM IR
│   ├── AsmParser/       # Parse textual LLVM IR (.ll files)
│   ├── Analysis/        # Analysis passes (alias analysis, loop info, …)
│   ├── Transforms/      # Optimization passes
│   │   ├── IPO/         # Interprocedural (inlining, LTO)
│   │   ├── Scalar/      # Scalar optimizations (mem2reg, GVN, …)
│   │   ├── Vectorize/   # Auto-vectorization (Loop, SLP)
│   │   └── Utils/       # Pass utilities
│   ├── CodeGen/         # Target-independent code generator
│   ├── Target/          # Per-architecture back-ends (AArch64, X86, RISCV, …)
│   ├── MC/              # Machine-code layer (MCInst, MCStreamer, …)
│   ├── MCA/             # Machine Code Analyzer (performance modelling)
│   ├── LTO/             # Link-time optimization orchestration
│   ├── Linker/          # IR-level module linker
│   ├── ExecutionEngine/ # Interpreter (lli) and ORC/MCJIT JIT engines
│   ├── Object/          # Object-file reading/writing (ELF, COFF, Mach-O)
│   ├── DebugInfo/       # DWARF, PDB, CodeView debug info
│   ├── Support/         # Platform-agnostic utilities (ADT, threading, …)
│   └── TableGen/        # TableGen front-end library
├── tools/               # Standalone command-line tools (opt, llc, llvm-*, …)
├── test/                # FileCheck-driven regression tests
├── unittests/           # Google Test unit tests
└── utils/               # Developer scripts and TableGen back-ends
```

### Clang Front-End

```
clang/
├── include/clang/       # Public headers
├── lib/
│   ├── Basic/           # Diagnostics, source locations, built-in types
│   ├── Lex/             # Preprocessor and lexer
│   ├── Parse/           # Recursive-descent parser → AST
│   ├── Sema/            # Semantic analysis, overload resolution, type-checking
│   ├── AST/             # AST node types and visitors
│   ├── CodeGen/         # Lower AST → LLVM IR
│   ├── Driver/          # Compilation pipeline, flag handling
│   ├── StaticAnalyzer/  # Clang Static Analyzer (CSA)
│   ├── Tooling/         # LibTooling: build AST-based tools
│   ├── Format/          # clang-format library
│   └── Interpreter/     # Clang-Repl (incremental C++ interpreter)
├── tools/               # clang, clang-format, clang-repl, …
└── test/                # Regression tests
```

### LLDB Debugger

```
lldb/
├── include/lldb/        # Public C++ and C APIs
├── source/
│   ├── API/             # SBDebugger, SBTarget, SBProcess … (public API)
│   ├── Core/            # Central debugger logic
│   ├── Breakpoint/      # Breakpoint & watchpoint management
│   ├── Commands/        # Built-in LLDB commands
│   ├── Expression/      # Expression evaluation (calls Clang/LLVM JIT)
│   ├── Host/            # OS-specific abstractions
│   ├── Symbol/          # Symbol table loading and querying
│   ├── Target/          # Process/thread/frame model
│   └── Plugins/         # Dynamic loaders, OS plug-ins, language plug-ins, …
├── tools/               # lldb executable, lldb-server, lldb-dap (DAP adapter)
└── test/                # API tests (Python) and lit-based unit tests
```

### LLD Linker

```
lld/
├── Common/              # Shared utilities (symbol tables, error handling)
├── ELF/                 # ELF linker (Linux, *BSD, bare-metal)
├── COFF/                # COFF/PE linker (Windows)
├── MachO/               # Mach-O linker (macOS, iOS)
├── wasm/                # WebAssembly linker
└── XCOFF/               # XCOFF linker (AIX)
```

### MLIR

```
mlir/
├── include/mlir/        # Dialect headers (Affine, Linalg, SCF, GPU, LLVM, …)
├── lib/
│   ├── IR/              # Core MLIR types (Operation, Region, Block, Value)
│   ├── Dialect/         # ~40 built-in dialects
│   ├── Transforms/      # Generic transformations (canonicalization, inlining)
│   ├── Conversion/      # Dialect lowering passes (Linalg → LLVM, etc.)
│   └── ExecutionEngine/ # JIT execution of MLIR modules via LLVM ORC
├── tools/               # mlir-opt, mlir-cpu-runner, mlir-tblgen
└── test/                # Regression tests
```

### Runtime Libraries

| Library | Purpose |
|---------|---------|
| `compiler-rt/lib/builtins/` | Soft-float, integer arithmetic, ABI helpers |
| `compiler-rt/lib/asan/` | AddressSanitizer runtime |
| `compiler-rt/lib/profile/` | Instrumentation-based profiling (PGO) |
| `libcxx/` | `std::` C++ Standard Library (headers + sources) |
| `libcxxabi/` | Itanium C++ ABI: `__cxa_throw`, `std::terminate`, RTTI |
| `libunwind/` | `_Unwind_*` stack unwinding for exception propagation |
| `libc/` | `printf`, `malloc`, POSIX APIs – full LLVM C library |
| `openmp/runtime/` | `libgomp`-compatible OpenMP runtime |
| `offload/` | GPU task-offload runtime (AMDGPU, NVPTX, Intel GPU) |
| `orc-rt/` | Support library for ORC JIT sessions |

---

## Component Relationships

The diagram below shows the high-level data flow from source code to executable:

```
  ┌─────────────────────────────────────────────────────────────┐
  │  Source code   (.c / .cpp / .f90 / …)                       │
  └───────────────────────┬─────────────────────────────────────┘
                          │
          ┌───────────────▼──────────────┐
          │  Front-End  (Clang / Flang)  │
          │  Lex → Parse → Sema → AST   │
          └───────────────┬──────────────┘
                          │  LLVM IR
          ┌───────────────▼──────────────┐
          │  LLVM Middle-End             │
          │  Analysis + Transformation   │
          │  passes  (opt -O2 / -O3)     │
          └───────────────┬──────────────┘
                          │  Optimised LLVM IR
          ┌───────────────▼──────────────┐
          │  LLVM Back-End  (CodeGen)    │
          │  Instruction selection,      │
          │  register allocation,        │
          │  scheduling → Machine IR     │
          └───────────────┬──────────────┘
                          │  Object file (.o)
          ┌───────────────▼──────────────┐
          │  Linker  (LLD)               │
          │  Symbol resolution,          │
          │  relocation, section layout  │
          └───────────────┬──────────────┘
                          │  Executable / shared library
          ┌───────────────▼──────────────┐
          │  (Optional post-processing)  │
          │  BOLT – profile-guided       │
          │  binary optimisation         │
          └──────────────────────────────┘

  At runtime the program links against the runtime libraries:
    compiler-rt  ·  libc++  ·  libc++abi  ·  libunwind  ·  libc
```

LLDB sits beside this pipeline: it drives the execution of the built program
and calls back into LLVM/Clang libraries (for expression evaluation and
debug-info parsing).

MLIR occupies a parallel track: front-ends (e.g., for ML frameworks) lower
source-level operations through a series of MLIR dialects before eventually
lowering to LLVM IR or directly to machine code.

---

## Build System

### Requirements

| Tool | Minimum version |
|------|----------------|
| CMake | 3.20.0 |
| C++ compiler | Must support C++17 |
| Python | 3.8 (for `lit` and LLDB tests) |
| Ninja (recommended) | any recent version |

### Typical configure + build

```bash
# Create a build directory outside the source tree
cmake -G Ninja \
      -S llvm \                            # source root
      -B build \                           # build directory
      -DCMAKE_BUILD_TYPE=Release \
      -DLLVM_ENABLE_PROJECTS="clang;lld;lldb" \
      -DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi;libunwind"

ninja -C build
```

### Key CMake Variables

| Variable | Effect |
|----------|--------|
| `LLVM_ENABLE_PROJECTS` | Comma-separated list of sub-projects to build alongside LLVM (`clang`, `lld`, `lldb`, `mlir`, `flang`, `bolt`, `polly`, `clang-tools-extra`, …). |
| `LLVM_ENABLE_RUNTIMES` | Runtime libraries built in a separate "runtimes" CMake invocation using the just-built compiler (`libc`, `libc++`, `libc++abi`, `libunwind`, `compiler-rt`, `openmp`, `offload`). |
| `LLVM_TARGETS_TO_BUILD` | Architectures to build back-ends for (default: `all`).  Limit to e.g. `X86;AArch64` for faster builds. |
| `CMAKE_BUILD_TYPE` | `Debug`, `Release`, `RelWithDebInfo`, `MinSizeRel`. |
| `LLVM_USE_LINKER` | Use a faster linker (`lld`) for the LLVM build itself. |
| `LLVM_ENABLE_ASSERTIONS` | Enable `assert()` calls (auto-enabled in Debug builds). |

### TableGen

Many source files inside `llvm/lib/Target/*/` are generated from `.td`
(TableGen) files.  CMake rules call `llvm-tblgen` automatically; the generated
`.inc` files are placed in the build tree and `#include`d by the C++ sources.

---

## Testing Infrastructure

### Lit (LLVM Integrated Tester)

`lit` is the primary test driver.  Tests are plain text files with embedded
`RUN:` commands and `CHECK:` patterns processed by `FileCheck`.

```bash
# Run all Clang tests
ninja -C build check-clang

# Run all LLVM tests
ninja -C build check-llvm

# Run a single test file
llvm/utils/lit/lit.py build/test/CodeGen/X86/my-test.ll
```

### Unit Tests (Google Test)

Located in `*/unittests/` directories.  Built and run via:

```bash
ninja -C build check-llvm-unit
ninja -C build check-clang-unit
```

### Sanitizer / Fuzzing Tests

`compiler-rt` contains its own test suite and fuzz targets; sanitiser
test programs are in `compiler-rt/test/`.

### CI / CD

Workflow definitions live in `.github/workflows/`.  The project runs
automated testing on Linux, macOS, and Windows via GitHub Actions, covering
build, unit tests, and integration tests for each major sub-project.
