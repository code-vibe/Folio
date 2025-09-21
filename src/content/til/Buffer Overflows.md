---
pubDatetime: 2025-09-21
title: Exploring Buffer Overflows in C
postSlug: buffer-overflow-c
tags:
  - TIL
  - C
  - Security
  - Rust
description: "A practical demo of stack-based buffer overflows in C, memory corruption, and GDB monitoring, inspired by Rust bounds check study."
---

I was studying **Rust’s bounds checks** and their performance cost. To understand the safety trade offs, I experimented with **stack based buffer overflows in C**.

---

## Minimal Example

```c
#include <stdio.h>

int main() {
    char buffer[16];
    printf("Enter some text: ");
    gets(buffer);  // unsafe
    printf("You entered: %s\n", buffer);
    return 0;
}
```

#### Compile 

>**c -fno-stack-protector -z execstack -g overflow.c -o overflow**


- `-fno-stack-protector` → disables stack canaries
    
- `-z execstack` → makes stack executable
    
- `-g` → debug symbols for GDB
    
- `gets()` → unsafe; no bounds check

### Run & Crash
```
Enter some text: AAAAAAAAAAAAAAAAAA
You entered: AAAAAAAAAAAAAAAAAA
Segmentation fault (core dumped)

```

- Input longer than 16 bytes **overwrites stack memory**, including the return address.
    
### **Stack layout:**
```
[ higher ]
| return address | <- overwritten
| saved frame    |
| buffer[16]     | <- input
[ lower ]

```

### Monitor with GDB
```
gdb ./overflow
(gdb) break main
(gdb) run
(gdb) next

```
- Enter input longer than buffer.
- Inspect memory:
```
**(gdb) x/32x $rsp**

```
- `0x41` (ASCII 'A') overwrites stack, segfault occurs at `0x41414141`.

### Safe Function Pointer Experiment
```
void (*func_ptr)() = NULL;

```
- Overflow buffer with **address of a function**.
    
- Calling `func_ptr()` executes the function, simulating execution hijack safely.

### Memory layout:
```
+------------+----------+
| buffer[16] | func_ptr |
+------------+----------+
Input: "AAAAAAAA" + function_address

```

### Key Takeaways
- Buffer overflows = memory corruption & arbitrary execution.
    
- GDB helps **visualize memory changes live**.
    
- Rust prevents this at compile time but adds bounds check overhead.
    
- Experiments reinforce the importance of **memory safety** and **input validation**.