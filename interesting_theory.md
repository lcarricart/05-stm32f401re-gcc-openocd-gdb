# Steps to Upload an ELF File and Debug an STM32F401RE

This note summarizes the manual build, flash, and debug workflow for an STM32F401RE microcontroller using a command-line toolchain, OpenOCD, and GDB.

---

## Debugging Workflow

- **OpenOCD (Open On-Chip Debugger)** acts as the hardware interface / debugging bridge between the PC and the microcontroller.

- The actual target being debugged is the **CPU (Central Processing Unit)** inside the microcontroller.  
  In this case, the target is the **STM32F401RE**.

- The evaluation board layout is not the main focus for CPU debugging.  
  What matters most is:
  - the target microcontroller: STM32F401RE
  - the debug interface: usually ST-LINK over USB
  - the debug protocol: usually SWD (Serial Wire Debug)

- An STM32 board is a good starting point because STM32 has strong OpenOCD support and many existing examples.

- Connect the board via USB and check that OpenOCD recognizes it.

- Launch **GDB (GNU Debugger)** and connect it to OpenOCD.

- Load the compiled firmware ELF file into GDB.

- By compiled firmware, this means firmware built using a CLI (Command-Line Interface) toolchain such as:

```bash
arm-none-eabi-gcc
```

This is an Arm cross-compiler used to build firmware for Arm Cortex-M microcontrollers.

- After GDB is connected and the ELF file is loaded, the debug session can be controlled through the CLI, similarly to how Python debugging was controlled with `pdbpp`.

- Alternatively, the same debugging backend can be used through the Visual Studio Code UI, while still relying on OpenOCD and GDB underneath.

---

## Compilation

To build the firmware manually, a build description file is needed.

Common options are:

- `Makefile` for `make`
- `CMakeLists.txt` for CMake

The build file describes how the firmware should be built.

It usually contains:

- source files
- include directories
- compiler flags
- linker flags
- architecture options, for example Cortex-M4
- linker script path
- output file names
- rebuild rules

Example compiler architecture information:

```text
-mcpu=cortex-m4
-mthumb
```

The build system also handles dependency logic, for example:

> if a source file changes, recompile the affected file.

---

## Build = Compile + Link

A build process usually includes at least two main stages:

1. **Compile**
2. **Link**

### Compile

Compilation turns each source file into a machine-level object file.

Example:

```text
main.c -> main.o
gpio.c -> gpio.o
startup.s -> startup.o
```

The output of compilation is usually:

```text
.o
```

These are object files.

### Link

Linking combines all `.o` object files and libraries into the final executable firmware image.

Example:

```text
main.o + gpio.o + startup.o + libraries + linker script -> firmware.elf
```

The linker is also responsible for placing code and data into the correct memory regions, such as Flash and RAM (Random Access Memory).

---

## Common Firmware Output Formats

The build process can produce different output files.

### ELF

```text
firmware.elf
```

ELF means **Executable and Linkable Format**.

The ELF file is the most useful file for debugging because it can contain:

- machine code
- function names
- variable names
- line-number information
- section information
- debug metadata

This allows GDB to connect the running machine code back to the original C source code.

### BIN

```text
firmware.bin
```

BIN means raw binary.

It is a raw representation of the firmware image and is often useful for flashing, but it does not contain the same debug metadata as an ELF file.

### HEX

```text
firmware.hex
```

HEX usually means Intel HEX format.

It is another flashable representation of the firmware image. Like `.bin`, it is useful for programming the device, but it is not ideal for source-level debugging.

---

## Why ELF Is Important for Debugging

The `.bin` and `.hex` files are more raw representations of the firmware.

The `.elf` file keeps debugging symbols and metadata.

Debugging symbols include information such as:

- function names
- variable names
- source file names
- line numbers

This is what allows a proper source-level debugging session.

Without the ELF file, the debugger may still interact with the CPU, but it loses the convenient mapping between machine instructions and the original C code.

---

## Makefile Practicality

A Makefile needs to contain a lot of build information.

For a larger embedded project, it is usually not practical to manually write every single source file name one by one.

A common Makefile technique is to collect source files automatically.

Example:

```make
SOURCES := $(wildcard src/*.c)
```

This collects all `.c` files inside the `src` directory.

For larger projects, the Makefile can also collect files from multiple folders.

However, in this learning workflow, the chosen approach is not to create the full Makefile from scratch.

Instead, the safer approach is:

1. Start from an existing working template.
2. Build it unchanged.
3. Confirm that the ELF file is produced.
4. Flash/debug it.
5. Modify the application code only after the toolchain works.

---

## Summary

The overall workflow is:

```text
C source files
    ↓
Makefile / CMakeLists.txt
    ↓
arm-none-eabi-gcc cross-compilation
    ↓
object files (.o)
    ↓
linking
    ↓
firmware.elf
    ↓
OpenOCD
    ↓
GDB
    ↓
STM32F401RE debug session
```

The key idea is to understand the complete path from source code to a debuggable firmware image without relying entirely on an IDE (Integrated Development Environment).
