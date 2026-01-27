+++
title = "The Stack"
date = 2026-01-27
description = "Understanding the NOP instruction, how it works, and why it is used in x86 assembly"
tags = ["lowlevel"]
+++

# Understanding the Stack in x86-64 Assembly
Different operating systems place the stack at different addresses, and if **ASLR (Address Space Layout Randomization)** is enabled, the stack location may change between program executions.

## Typical Process Memory Layout
```
High Addresses
┌──────────────┐
│    Stack     │ (grows downward)
├──────────────┤
│    Heap      │ (grows upward)
├──────────────┤
│ Code (text)  │
├──────────────┤
│ Global data  │
└──────────────┘
Low Addresses
```

* The **stack grows toward lower addresses**
* The **heap grows toward higher addresses**

If the stack and heap grow too much, they can **collide**, causing the program to crash.

## Stack Pointer (RSP)
On x86-64 systems, the **RSP register** (stack pointer) points to the **top of the stack**.

* The top of the stack is the **lowest address currently in use**
* Memory beyond the top of the stack is **undefined**
* Programs should never rely on data outside the current stack region

## What Kind of Information Is Stored on the Stack?
The stack commonly contains:

* **Return addresses**  
  When a function calls another function, the return address is pushed onto the stack so execution can return to the caller.

* **Local variables**

* **Function arguments**  
  (Sometimes passed via registers, sometimes via the stack, depends on calling conventions.)

* **Saved registers**  
  Registers may be saved to the stack so functions don't overwrite each other's values.

* **Register spilling**  
  If a function is too complex and there aren't enough registers, the compiler temporarily stores values on the stack.

* **Stack allocations**  
  Functions like `alloca()` explicitly allocate memory on the stack instead of the heap.

## Example Program

```c
#include <stdio.h>

int bar(int y)
{
    int a = 3 * y;
    printf("bar returned %d\n", a);
    return a;
}

int foo(int x)
{
    int b = 5 * x;
    printf("foo passed %d\n", b);
    return bar(b);
}

int main()
{
    int c = foo(7);
    printf("main passed %d\n", c);
}
```

Each function call creates a **stack frame**, a region of the stack used by that function.

* `main()` gets a stack frame
* `foo()` gets a stack frame when called by `main`
* `bar()` gets a stack frame when called by `foo`

As functions return, their stack frames are removed.

### Conceptual Stack Layout
```
Stack Bottom (higher addresses)

┌──────────────────┐
│    main() frame  │ 
├──────────────────┤
│    foo()  frame  │ 
├──────────────────┤
│    bar()  frame  │
└──────────────────┘

Stack Top (lower addresses)
```

* New frames are created as the stack **grows downward**
* Frames disappear as functions return

## Important Mental Model
The direction of stack growth is a **convention**, not a rule.

You may see:
* stacks drawn top-to-bottom
* left-to-right
* right-to-left

What matters is understanding:
* **push = move stack pointer**
* **pop = restore stack pointer**
* **last function called returns first**

# Push & Pop Instructions

Let's look at real assembly.

```c
#include <stdio.h>

int main()
{
    printf("Hello world");
    return 0;
}
```

Disassembly:
```assembly
main:
    push rbp
    mov  rbp, rsp
    ...
    pop  rbp
```

## PUSH 
**push** stores a value on the stack.

It pushes an 8-byte value onto the stack. When a push occurs, it automatically decrements the stack pointer RSP by 8 because you've just pushed 8 bytes of data onto the stack. What does that mean?

```asm
push rbp
```

What happens internally?
```
RSP = RSP - 8
[RSP] = RBP
```

### Why 8 bytes?
- x86-64 is a 64-bit architecture
- A register like RBP is 8 bytes (64 bits)
- The stack is 8-byte aligned

Store the value at the new top of the stack:
```
[RSP] = RBP
```

### Visual Example

Before **push rbp**:
```
High addresses

┌──────────────────┐
│       ???        │ 
├──────────────────┤
│       ???        │ 
├──────────────────┤
│   Top of stack   │  ← RSP = 0x7FFE8
└──────────────────┘

Low addresses
```

After **push rbp**:
```
High addresses

┌──────────────────┐
│       ???        │ 
├──────────────────┤
│    saved RBP     │  ← RSP = 0x7FFE0 (moved down by 8)
├──────────────────┤
│    old stack     │  
└──────────────────┘

Low addresses
```

The stack grows downward, so pushing data means moving the stack pointer to a lower address and writing the value there to preserve execution state.

### Why do we do this?
We do this so functions can safely call other functions and then come back without destroying each other's data.

When a function runs, it needs:
- space for local variables
- a place to save registers
- a way to remember where to return

But:
- there are only 16 general-purpose registers
- multiple functions can be active at once 
- function calls can be nested

So we need a structured way to:
- save state 
- restore state
- do it fast

The stack solves this.

### Why PUSH specifically?
Calling a function must:

- save something
- restore it later
- work recursively
- work in any order

A Last-In, First-Out (LIFO) structure does this perfectly.

```c
int main() {
    foo();
}
```

Conceptually:
- **main()** is running
- **main()** calls **foo()**
- **foo()** must:
  - not destroy main's data
  - know where to return

So we do:
```assembly
push rbp        ; save caller's frame
mov  rbp, rsp   ; create new frame
```

Now **foo** has its own space on the stack.

When **foo** finishes:
```asm
pop rbp
```
Everything goes back exactly how it was.

### Why move RSP before writing?
RSP must always point to the top of the stack. The stack grows downward, so pushing means making room first, then storing.

If we didn't move RSP:
- data would overwrite existing values
- return addresses would be corrupted

Think of **push** as:  
*"Save this, I'll need it later."*

and **pop** as:  
*"Ok, give it back."*

### PUSH variants
In 64-bit execution mode, there are different versions of push:
- First: you can push the value from a 64-bit register
- Second: you can push a 64-bit value from memory where the memory is given in a special form called **r/m**

## What is an r/m (rm/x) operand?
An **r/m operand** means:

"This instruction can use **either a register value OR a value from memory**."

That's it.

## Register vs Memory 
In Intel syntax:

* `rbx` → use the **value in the register**
* `[rbx]` → treat the value in `rbx` as a **memory address**, then load the value stored at that address

Square brackets **always mean memory access**.

This is just like pointer dereferencing in C:

```c
x = rbx;      // register value
x = *rbx;     // memory value (conceptually)
```

## Why does x86 need this?
Because sometimes you want to:

* operate on a register
* and sometimes you want to operate on **data in memory**

The same instruction should support both.

Example:

```asm
push rbx     ; push register value
push [rbx]   ; push value stored in memory
```

Same instruction, different operand source.

## The 4 Common r/m Forms 
### 1. Register only

```asm
push rbx
```

Use the value inside `rbx`.

### 2. Memory: base only
```asm
push [rbx]
```

Meaning:
* `rbx` holds an address
* read the value at that address
* push that value

### 3. Memory: base + index × scale
```asm
push [rbx + rcx*4]
```

Meaning:

* start at address in `rbx`
* add `rcx * 4`
* read the value from that computed address

Used for **arrays**.

### 4. Memory: base + index × scale + displacement
```asm
push [rbx + rcx*8 + 0x20]
```

Meaning:
* start at `rbx`
* add `rcx * 8`
* add constant offset `0x20`
* read value from that address

Used for **structs**, **arrays of structs**, etc.

## Why only scale = 1, 2, 4, 8?
Because these match common data sizes:

* 1 → byte
* 2 → short
* 4 → int
* 8 → pointer / long

This lets the CPU calculate addresses efficiently.

## When are these "weird" forms used?
They show up when accessing:

* arrays
* structs
* stack variables
* objects
* function arguments

Example in C:
```c
arr[i]
```

Becomes:
```asm
[base + index*element_size]
```

### r/m examples with PUSH
```asm
push rbx                  ; push register value
push [rbx]                ; push value from memory
push [rbx + rcx*4]        ; push arr[rcx]
push [rbx + rcx*8 + 0x20] ; push struct field
```

### Important Note: Size Prefixes
When pushing from memory, you need to specify the size:

```asm
push qword [rbx + rcx*4]    ; push 64-bit value
```
You need `qword` because the assembler needs to know you're pushing 64 bits, not 32 (`dword`) or 16 (`word`). In 64-bit mode, `push` always works with 8-byte values, but the assembler needs the explicit size for memory operands.

## Concrete PUSH Example
Let's see what actually happens when this instruction runs:
```asm
push rax
```

### Initial State
Registers:
```
RAX = 0x0000000000000003
RSP = 0x00007FFF0FE8   ; top of the stack
```

Stack (before push rax):
```
Address         Value
-----------------------
0x00007FFF0FF0  0x0001
0x00007FFF0FE8  0x0002   ← RSP (top of stack)
0x00007FFF0FE0  undefined
0x00007FFF0FD8  undefined
```

### What push rax does
1. Move the stack pointer down:
```
RSP = RSP - 8
```

2. Store the value:
```
[RSP] = RAX
```

### Final State
Registers:
```
RAX = 0x0000000000000003
RSP = 0x00007FFF0FE0   ; new top of the stack
```

Stack (after push rax):
```
Address         Value
-----------------------
0x00007FFF0FF0  0x0001
0x00007FFF0FE8  0x0002
0x00007FFF0FE0  0x0003   ← RSP (pushed value from RAX)
0x00007FFF0FD8  undefined
```

### Why did RSP move?
Because:

- x86-64 uses 8-byte stack slots
- the stack grows downward
- pushing means make space first, then write

So pushing always moves RSP first, then stores the value.

**push X** = "move RSP down, then write X there"

## POP Instruction

**pop** removes a value from the stack. In 64-bit execution mode, it can pop into a 64-bit register or it can pop into a memory address as given in r/m form.

So, what would it look like if we executed **pop rax**?

### What pop rax does
```asm
pop rax
```

1. Read the value at the top of the stack:
```
RAX = [RSP]
```

2. Move the stack pointer up:
```
RSP = RSP + 8
```

Why **+8**?
- x86-64 is a 64-bit architecture
- Stack entries are 8 bytes
- Popping removes one 8-byte value from the stack

### Concrete POP Example
Assume the stack currently looks like this:

Registers before:
```
RAX = 0xAAAAAAAAAAAAAAAA
RSP = 0x00007FFF0FE0   ; top of the stack
```

Stack before **pop rax**:
```
Address         Value
-----------------------
0x00007FFF0FF0  0x0001
0x00007FFF0FE8  0x0002
0x00007FFF0FE0  0x0003   ← RSP (top of stack)
0x00007FFF0FD8  undefined
```

Execute **pop rax**:
```asm
pop rax
```

### After Execution

Registers:
```
RAX = 0x0000000000000003
RSP = 0x00007FFF0FE8   ; RSP moved up by 8
```

Stack after:
```
Address         Value
-----------------------
0x00007FFF0FF0  0x0001
0x00007FFF0FE8  0x0002   ← RSP (new top of stack)
0x00007FFF0FE0  0x0003   ; still in memory, but now undefined
0x00007FFF0FD8  undefined
```

The value **0x0003** still exists in memory, but it is no longer part of the stack. RSP moved up to 0x00007FFF0FE8, so anything at higher addresses (lower in the diagram) is no longer valid stack data.

### Note
Memory at **higher addresses** than the current stack pointer is considered undefined.

- The hardware does not erase it
- The compiler does not clean it up
- Correct programs must never rely on it
- Only memory at or below RSP is valid stack data

### POP into memory (r/m form)
Just like push, pop also supports memory operands:
```asm
pop [rbx]
```

This means:
- Take the value at the top of the stack
- Store it into the memory address pointed to by RBX
- Increment RSP by 8

Conceptually:
```
[RBX] = [RSP]
RSP  =  RSP + 8
```

## Critical Rule: PUSH and POP Must Be Balanced
Every **push** must have a matching **pop**. 

If not:
- The stack becomes misaligned
- Return addresses break
- Programs crash

**push** saves a value for later, **pop** gives it back.

## 32-bit and 16-bit Modes
In **32-bit mode**:
- push/pop work with 32-bit (4-byte) values
- RSP changes by **4** instead of 8
- Example: `push eax` decrements ESP by 4

In **16-bit mode**:
- push/pop work with 16-bit (2-byte) values  
- SP changes by **2** instead of 8
- Example: `push ax` decrements SP by 2

The principle remains the same—only the operand size and stack pointer adjustment change.

# Finding Offset from RSP/RBP to Reach an Address
```
      HIGH ADDRESSES
            ↑
┌──────────────────────────┐
│ 0xb01dface   ← RBP       │
├──────────────────────────┤
│ 0xaffab1e    ← TARGET    │
├──────────────────────────┤
│ 0x50f7ba11               │
├──────────────────────────┤
│ 0x0000b100d              │
├──────────────────────────┤
│ 0xb100d1e55  ← RSP       │
└──────────────────────────┘
            ↓
      LOW ADDRESSES
```

**Question: What is the offset to 0xaffab1e?**
## The Formula

```
Offset = Target Address - Starting Point
```

Then apply the sign:
```
If offset is positive → use +
If offset is negative → use -
```

## Step-by-Step Solution
### Step 1: Count the slots
How many 8-byte slots between your starting point and target?

**From RBP to 0xaffab1e:** 1 slot  
**From RSP to 0xaffab1e:** 3 slots

### Step 2: Multiply by 8
```
Number of slots × 8 = offset in bytes
```

**From RBP:** 1 × 8 = 8 bytes  
**From RSP:** 3 × 8 = 24 bytes

### Step 3: Determine the sign
- Going **UP** (to higher addresses) = **+** (positive)
- Going **DOWN** (to lower addresses) = **-** (negative)

**From RBP:** Going DOWN → use **-**  
**From RSP:** Going UP → use **+**

> **Remember:** The stack grows downward. RBP is at a HIGHER address than 0xaffab1e, so to reach it from RBP, we subtract.

### Step 4: Convert to hexadecimal
```
Decimal → Hexadecimal
```

8 → 0x08  
24 → 0x18

### Step 5: Write the answer
```
{register}{sign}0x{hex_value}
```

**From RBP:** `rbp-0x08`  
**From RSP:** `rsp+0x18`

So **rbp-0x08** (0xaffab1e is 8 bytes below RBP) and **rsp+0x18** (0xaffab1e is 24 bytes above RSP)


## Quick Reference: Common Conversions
```
Slots  Bytes  Hexadecimal
─────────────────────────
  1      8      0x08
  2     16      0x10
  3     24      0x18
  4     32      0x20
  5     40      0x28
  6     48      0x30
  7     56      0x38
  8     64      0x40
```

## Practice Example
```
      HIGH ADDRESSES
            ↑
┌──────────────────────────┐
│ 0xdeadbeef   ← RBP       │
├──────────────────────────┤
│ 0xcafebabe               │
├──────────────────────────┤
│ 0x1badb002   ← TARGET    │
├──────────────────────────┤
│ 0xfeedface               │
├──────────────────────────┤
│ 0x8badf00d   ← RSP       │
└──────────────────────────┘
            ↓
      LOW ADDRESSES
```

**Question: What is the offset to 0x1badb002?**

**From RBP:**
- Slots: 2 (down from RBP)
- Bytes: 2 × 8 = 16
- Direction: DOWN → negative
- Hex: 16 → 0x10
- **Answer: rbp-0x10**

**From RSP:**
- Slots: 2 (up from RSP)
- Bytes: 2 × 8 = 16
- Direction: UP → positive
- Hex: 16 → 0x10
- **Answer: rsp+0x10**
