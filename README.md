# 🚀 RISC-V Week 1 Tasks: Toolchain Setup, Compilation & Debugging

This README documents the hands-on tasks completed for Week 1 of the RISC-V training. It includes installation, compilation, disassembly, debugging, and emulation of RISC-V programs using GCC and QEMU.

---

## 📦 1. Toolchain Setup & Sanity Check

### 🛠️ Extract the Toolchain

```bash
cd ~/Downloads
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz
```

### 🌐 Add to PATH

```bash
echo 'export PATH=$HOME/Downloads/riscv/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### ✅ Verify Installation

```bash
riscv32-unknown-elf-gcc --version
riscv32-unknown-elf-objdump --version
riscv32-unknown-elf-gdb --version
```

---

## 🧾 2. Compiling “Hello, RISC-V!”

### `hello.c` Source

```c
#include <stdio.h>

int main() {
    printf("Hello, RISC-V!\n");
    return 0;
}
```

### 🔧 Compile to ELF

```bash
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -o hello.elf hello.c
file hello.elf
```

---

## 🧠 3. C to Assembly

### 🛠 Generate Assembly

```bash
riscv32-unknown-elf-gcc -S -O0 -march=rv32imac -mabi=ilp32 hello.c
```

### 🧩 Prologue & Epilogue (from `hello.s`)

```asm
main:
    addi sp, sp, -16
    sw   ra, 12(sp)
    sw   s0, 8(sp)
    addi s0, sp, 16
    ...
    lw   ra, 12(sp)
    lw   s0, 8(sp)
    addi sp, sp, 16
    ret
```

---

## 🧮 4. Disassembly & Hex Dump

### 🔍 Disassemble ELF

```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.dump
```

### 🔄 Convert to Intel HEX

```bash
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

---

## 🧪 5. RISC-V Register Cheat Sheet

| Register | ABI Name | Role              | Saved By   |
|----------|----------|-------------------|------------|
| x0       | zero     | Constant 0        | —          |
| x1       | ra       | Return Address    | Caller     |
| x2       | sp       | Stack Pointer     | Callee     |
| x8       | s0/fp    | Frame Pointer     | Callee     |
| x10–x17  | a0–a7    | Args/Return Val   | Caller     |
| x5–x7, x28–x31 | t0–t6 | Temporaries   | Caller     |
| x18–x27  | s2–s11   | Saved Registers   | Callee     |

---

## 🐞 6. GDB Debugging

```bash
riscv32-unknown-elf-gdb hello1.elf
(gdb) target sim
(gdb) break main
(gdb) info reg a0
```

---

## 💻 7. Emulation Using QEMU

### 🧾 Bare-metal Program (`hello2.c`)

```c
#define UART0 0x10000000

void uart_putchar(char c) {
    *(volatile char *)UART0 = c;
}

void uart_puts(const char *s) {
    while (*s) uart_putchar(*s++);
}

void _start() {
    uart_puts("Hello RISC-V from UART!\n");
    while (1) {}
}
```

### 🧰 Linker Script: `link1.ld`

```ld
ENTRY(_start)
SECTIONS {
  . = 0x80010000;
  .text : { *(.text*) }
  .data : { *(.data*) }
  .bss  : { *(.bss*)  }
}
```

### 🏗 Compile

```bash
riscv32-unknown-elf-gcc -g -march=rv32imac -mabi=ilp32 -T link1.ld -nostartfiles -nostdlib -o hello.elf hello2.c
```

### ▶️ Run with QEMU + OpenSBI

```bash
qemu-system-riscv32 -nographic -machine virt \
  -bios build/platform/generic/firmware/fw_jump.bin \
  -kernel hello.elf
```

**Output:**

```
Hello RISC-V from UART!
```

---

## ⚙️ 8. GCC Optimization Demo

```c
int square(int x) {
    int y = x * x;
    return y;
}
```

### Compile without and with optimization

```bash
riscv32-unknown-elf-gcc -S -O0 -o test_O0.s test.c
riscv32-unknown-elf-gcc -S -O2 -o test_O2.s test.c
```

Compare how stack, variables, and code get optimized at `-O2`.


