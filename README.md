# XenonRecomp

**XenonRecomp** is a specialized tool designed to bridge the gap between legacy Xbox 360 software and modern hardware. By converting Xbox 360 PowerPC (PPC) executables into C++ source code, it facilitates the Ahead-Of-Time (AOT) recompilation of game logic for native execution on x86 platforms.

Inspired by the methodology used in [N64Recomp](https://github.com/N64Recomp/N64Recomp), this project focuses on static analysis and translation of machine code into a high-level intermediate representation.

> **⚠️ Disclaimer:** XenonRecomp provides *code translation only*. It does not include a runtime environment, hardware abstraction layer (HAL), or API wrappers. Successfully running the generated code requires a custom-built runtime that implements the Xbox 360 hardware interfaces (e.g., GPU, I/O, Audio).

## Core Architecture

### Instruction Translation
The recompiler analyzes PowerPC instructions and translates them directly into C++ operations. To maintain execution context, generated functions accept a `CpuState` struct containing the values of all PPC registers, alongside a pointer to the base address of the emulated memory space.

*   **Endianness:** The Xbox 360 is a big-endian system, whereas modern x86 is little-endian. XenonRecomp handles this by:
    *   Swapping byte order for standard memory loads and stores.
    *   Marking these operations as `volatile` to prevent compiler reordering.
    *   Reversing the 16-byte layout of vector registers (VMX/AltiVec) to match SIMD processing requirements.

### Floating Point & Vector Processing
*   **FPU vs. VMX:** The tool differentiates between standard FPU operations (which preserve denormalized numbers) and VMX operations (which flush denormals to zero). The generated code manages the CPU's floating-point control flags dynamically to ensure accurate emulation.
*   **SIMD Implementation:** Most VMX instructions are mapped directly to x86 intrinsics (SSE/AVX). While currently optimized for x86, the logic could theoretically be adapted for other architectures using libraries like [SIMD Everywhere](https://github.com/simd-everywhere/simde).

### Memory & Indirect Calls
*   **MMIO (Memory-Mapped I/O):** Hardware-specific operations (such as XMA audio decoding) are currently unimplemented and require manual runtime support.
*   **Virtual Function Calls:** The tool handles C++ vtables by generating "perfect hash tables" at runtime. This allows for the dynamic resolution of virtual calls without requiring a full dynamic linker.

## Workflow Overview

1.  **Analysis:** XenonRecomp parses the input executable to identify code segments and control flow.
2.  **Conversion:** It generates C++ source files representing the game's logic, wrapping instructions in function calls to the `CpuState`.
3.  **Compilation:** The generated C++ code is compiled using a standard C++ compiler (e.g., MSVC, Clang, GCC) targeting the host architecture.
4.  **Runtime Integration:** The resulting object files are linked against a custom runtime that implements memory management and hardware callbacks.

## Building

*(Note: Build instructions should be added here based on the specific build system used, typically CMake or Make.)*

## Contributing

Contributions are welcome, particularly in the areas of:
*   Implementing missing PowerPC instruction variants.
*   Expanding support for MMIO operations.
*   Optimizing the generated C++ output for better compiler optimization.