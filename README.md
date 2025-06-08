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
output

---
<img width="452" alt="image" src="https://github.com/user-attachments/assets/ca5acbf6-34e3-4bc2-a98e-5edc628b6d71" />
<img width="452" alt="image" src="https://github.com/user-attachments/assets/d9005642-f942-4ea1-854e-48041b66adcb" />


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
<img width="452" alt="image" src="https://github.com/user-attachments/assets/9201089d-a72c-4b13-b11f-4a736fe8842a" />
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
output
---
<img width="452" alt="image" src="https://github.com/user-attachments/assets/2d2e3a1a-6a89-4486-bf12-a306797e1fbf" />
<img width="452" alt="image" src="https://github.com/user-attachments/assets/c2b172f6-19dc-4b50-817f-b8d2e95e7948" />
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

<img width="452" alt="image" src="https://github.com/user-attachments/assets/2564d00c-a232-4995-a38c-c568d4d4482a" />
<img width="452" alt="image" src="https://github.com/user-attachments/assets/1b49ab38-c4af-46e1-8c40-b9cc96d495dd" />


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
<img width="452" alt="image" src="https://github.com/user-attachments/assets/101b3930-4a90-475a-9e15-74e74dd2ce58" />
<img width="452" alt="image" src="https://github.com/user-attachments/assets/7b410773-dee5-4a6a-b22f-b15e4ad3e9d6" />
<img width="452" alt="image" src="https://github.com/user-attachments/assets/0894a675-a9a6-4069-8ea2-43e5cea4d292" />
<img width="126" alt="image" src="https://github.com/user-attachments/assets/f03bbb1c-c0c5-487a-9f91-cdd5ae946680" />

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
<img width="452" alt="image" src="https://github.com/user-attachments/assets/c08cd346-28a3-4100-a482-d3b59fd09009" />

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
<img width="452" alt="image" src="https://github.com/user-attachments/assets/8449172c-6301-4795-aef7-70986f3dcb27" />
<img width="452" alt="image" src="https://github.com/user-attachments/assets/fb0a986b-0675-4814-8c7f-095cd0a5f17e" />
<img width="452" alt="image" src="https://github.com/user-attachments/assets/6395d3a9-5e5a-4af6-a186-e887206bd0bb" />
