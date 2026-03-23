# The LLVM Compiler Infrastructure

[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/llvm/llvm-project/badge)](https://securityscorecards.dev/viewer/?uri=github.com/llvm/llvm-project)
[![OpenSSF Best Practices](https://www.bestpractices.dev/projects/8273/badge)](https://www.bestpractices.dev/projects/8273)
[![libc++](https://github.com/llvm/llvm-project/actions/workflows/libcxx-build-and-test.yaml/badge.svg?branch=main&event=schedule)](https://github.com/llvm/llvm-project/actions/workflows/libcxx-build-and-test.yaml?query=event%3Aschedule)

Welcome to the LLVM project!

This repository contains the source code for LLVM, a toolkit for the
construction of highly optimized compilers, optimizers, and run-time
environments.

The LLVM project has multiple components. The core of the project is
itself called "LLVM". This contains all of the tools, libraries, and header
files needed to process intermediate representations and convert them into
object files. Tools include an assembler, disassembler, bitcode analyzer, and
bitcode optimizer.

C-like languages use the [Clang](https://clang.llvm.org/) frontend. This
component compiles C, C++, Objective-C, and Objective-C++ code into LLVM bitcode
-- and from there into object files, using LLVM.

Other components include:
the [libc++ C++ standard library](https://libcxx.llvm.org),
the [LLD linker](https://lld.llvm.org), and more.

## Getting Started for Beginners

New to LLVM? Here is a suggested learning path:

1. **Understand what LLVM is** – Read the
   [LLVM Overview](https://llvm.org/docs/index.html) to learn about the
   project's goals, components, and architecture.

2. **Set up your build environment** – Follow the
   [Getting Started with LLVM](https://llvm.org/docs/GettingStarted.html)
   guide to check out the source code and build the project for the first time.

3. **Learn the core concepts** – Work through the
   [LLVM Language Reference Manual](https://llvm.org/docs/LangRef.html) to
   understand LLVM IR, and read the
   [Writing an LLVM Pass](https://llvm.org/docs/WritingAnLLVMNewPMPass.html)
   tutorial for a hands-on introduction.

4. **Explore Clang** – If you are interested in the C/C++ frontend, start with
   the [Clang Getting Started](https://clang.llvm.org/get_started.html) page.

5. **Find beginner-friendly tasks** – Look for issues labeled
   `good first issue` on the
   [GitHub issue tracker](https://github.com/llvm/llvm-project/issues?q=is%3Aopen+label%3A%22good+first+issue%22)
   or browse the
   [LLVM beginner resources](https://llvm.org/docs/GettingInvolved.html).

6. **Join the community** – Ask questions on the
   [LLVM Discourse forums](https://discourse.llvm.org/) or the
   [Discord chat](https://discord.gg/xS7Z362). The community is welcoming to
   newcomers of all backgrounds.

## Getting the Source Code and Building LLVM

Consult the
[Getting Started with LLVM](https://llvm.org/docs/GettingStarted.html#getting-the-source-code-and-building-llvm)
page for information on building and running LLVM.

For information on how to contribute to the LLVM project, please take a look at
the [Contributing to LLVM](https://llvm.org/docs/Contributing.html) guide.

## Getting in touch

Join the [LLVM Discourse forums](https://discourse.llvm.org/), [Discord
chat](https://discord.gg/xS7Z362),
[LLVM Office Hours](https://llvm.org/docs/GettingInvolved.html#office-hours) or
[Regular sync-ups](https://llvm.org/docs/GettingInvolved.html#online-sync-ups).

The LLVM project has adopted a [code of conduct](https://llvm.org/docs/CodeOfConduct.html) for
participants to all modes of communication within the project.
