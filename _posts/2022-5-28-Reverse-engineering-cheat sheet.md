---
title: Reverse Engineering Cheat sheet
layout: default
tags: [Reverse engineering]
blog_post: true
---


# Introduction
I’ll post some Assembly to understand this article, you must know what asm is :slight_smile: here a cheat sheet if you got stuck, Assembly language, often referred to as asm, is a low-level programming language that closely corresponds to the machine code instructions of a specific computer architecture. It provides a more human-readable representation of machine instructions and allows direct manipulation of hardware resources.

# Stuff in the Code
Let's get to know with a few important parts of the code before we dive into the assembly language.

-   **Instructions**: Assembly code consists of a series of instructions that represent individual operations the processor can perform. These instructions typically correspond to specific machine-level operations, such as moving data, performing arithmetic operations, or manipulating memory.
    
-   **Registers**: Registers are small storage locations within the processor that can hold data. In assembly code, we often see instructions involving registers, such as moving values between registers or performing operations on register contents. Registers play a critical role in efficient data processing.
    
-   **Memory Access**: Assembly instructions can interact with memory to read from or write to specific locations. In a example of  "Hello, World!\n" program, the string is stored in memory and the assembly code accesses it to pass it as an argument to the `printf` function.

If you're new to Assembly or need a quick reference, Here's one:

Each instruction in assembly language represents a unique operation that the processor is capable of carrying out. These instructions could also involve control flow, memory access, and math calculations. Understanding how code is executed at the machine level requires an understanding of typical assembly instructions.

## Data Movement:
```assembly
-   MOV: Copy
-   XCHG: Exchange
-   LEA: Load effective address
```

### Memory Access

Memory access instructions are crucial for reading from and writing to specific memory locations. These instructions allow us to interact with variables, arrays, and other data structures. For instance:

```assembly
movl    $0, -4(%rbp)
```

This instruction moves the value 0 into the memory location `-4(%rbp)`. Analyzing memory access instructions enables us to understand how variables are stored, retrieved, and modified.

## Arithmetic and Logic:
Assembly instructions often involve arithmetic and logical operations performed on registers and memory locations.

```assembly
movl    $10, -4(%rbp) 
movl    $20, -8(%rbp) 
addl    -8(%rbp), -4(%rbp)
```

In this example, the values 10 and 20 are moved into memory locations `-4(%rbp)` and `-8(%rbp)`, respectively. The `addl` instruction adds the value stored at `-8(%rbp)` to the value at `-4(%rbp)` and stores the result back into `-4(%rbp)`.

a quick reference: 

```assembly
-   ADD: Add
-   SUB: Subtract
-   DIV: Divide
-   IDIV: Signed integer divide
-   MUL: Multiply
-   IMUL: Signed integer multiply
-   INC: Increment
-   DEC: Decrement
-   SAL: Shift left
-   SAR: Shift right
-   ROL: Rotate left
-   ROR: Rotate right
-   NOT: Invert each bit
-   AND: Logical AND
-   OR: Logical OR
-   XOR: Logical exclusive OR
-   SHL: Shift logical left
-   SHR: Shift logical right
```

## Control Flow:
Control flow instructions determine the program's execution path by allowing conditional branching, unconditional branching, and function calls. By analyzing these instructions, we can reconstruct the program's flow and understand function invocations, loops, and branching conditions. Example:

```assembly
movl    $0, -4(%rbp)
jmp     .L2
.L3:
    movl    -4(%rbp), %eax
    movl    %eax, %esi
    movl    $.L.str, %edi
    movl    $0, %eax
    callq   printf
    addl    $1, -4(%rbp)
.L2:
    cmpl    $4, -4(%rbp)
    jle     .L3

```
Here, a loop is created using `jmp`, `cmpl`, and `jle` instructions. The value of `i` is stored in `-4(%rbp)`, and the `cmpl` instruction compares it with the value 4. If the comparison result indicates that `i` is less than or equal to 4, the loop continues by jumping to `.L3`. Inside the loop, the value of `i` is printed using `printf`, and then the `addl` instruction increments `i` by 1.

```assembly
-   NOP: No operation
-   INT: Interrupt
-   CALL: Call subroutine
-   JMP: Jump
-   JE, JZ: Jump if equal or zero
-   JCXZ: Jump if CX zero
-   JNE, JNZ: Jump if not equal or not zero
-   JECXZ: Jump if ECX zero
-   RET: Return from subroutine
``` 

## Conditional Jumps:
```assembly
-   JA: Jump if above
-   JAE: Jump if above or equal
-   JB: Jump if below
-   JBE: Jump if below or equal
-   JNA: Jump if not above
-   JNAE: Jump if not above or equal
-   JNB: Jump if not below
-   JNBE: Jump if not below or equal
-   JC: Jump if carry
-   JNC: Jump if not carry
-   JG: Jump if greater
-   JGE: Jump if greater or equal
-   JL: Jump if less
-   JLE: Jump if less or equal
-   JNG: Jump if not greater
-   JNGE: Jump if not greater or equal
-   JNL: Jump if not less
-   JNLE: Jump if not less or equal
-   JO: Jump if overflow
-   JNO: Jump if not overflow
-   JS: Jump if sign
-   JNS: Jump if not sign
```

## String Operations:
```assembly
-   REP: Repeat instruction block
-   CMPS: Compare strings
-   MOVS: Move strings
-   LODS: Load string
```

# Code Patterns

When analyzing assembly code, we often observe specific patterns or sequences of instructions that commonly appear in programs. These code patterns reflect typical programming constructs and idioms.

For example, the sequence of instructions at the end of our example code:

```assembly

xorl    %ecx, %ecx
movl    %eax, -8(%rbp)
movl    %ecx, %eax
addq    $16, %rsp
popq    %rbp
retq

```

represents the epilogue of the `main` function. It involves cleaning up the stack, restoring the base pointer, and returning from the function.

Finally,  here's a bash script that allows you to look up the description of an Assembly instruction: 

```sh
#!/bin/bash

# Instruction list
instructions=(
"MOV: Copy"
"XCHG: Exchange"
"PUSH: Push onto stack"
"POP: Pop from stack"
# Add More instructions and their descriptions here
)

# Function to get the description of an instruction
get_description() {
    local instruction=$1
    for item in "${instructions[@]}"; do
        if [[ $item == "$instruction"* ]]; then
            echo "${item#*: }"
            return
        fi
    done
    echo "Instruction not found."
}

# Read user input
read -p "Enter an Assembly instruction: " input

# Convert input to uppercase
input=$(echo "$input" | tr '[:lower:]' '[:upper:]')

# Get instruction description
description=$(get_description "$input")

# Output the description
echo "Description: $description"
```

Instead of manually searching for each instruction one by one, you can simplify the process by running the script and entering an Assembly instruction (e.g., `MOV`). The script will then provide the corresponding description (e.g., `Copy`) for that instruction.

Here's a python version of the code 

```py
import sys

# Instruction dictionary
instructions = {
    "MOV": "Copy",
    "XCHG": "Exchange",
    "PUSH": "Push onto stack",
    "POP": "Pop from stack",
    # Add more instructions and their descriptions here
}

# Function to get the description of an instruction
def get_description(instruction):
    if instruction in instructions:
        return instructions[instruction]
    else:
        return None

# Read user input
try:
    while True:
        input_instruction = input("Enter an Assembly instruction (or 'q' to quit): ")

        # Check if user wants to quit
        if input_instruction.lower() == "q":
            print("Goodbye!")
            break

        # Convert input to uppercase
        input_instruction = input_instruction.upper()

        # Get instruction description
        description = get_description(input_instruction)

        # Output the description or error message
        if description:
            print("Description:", description)
        else:
            print("Instruction not found. Please try again.")

except KeyboardInterrupt:
        sys.exit(0)
```


# References 
[x86 Assembly Guide 148](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html)

[The Art of Assembly Language 118](https://www.plantation-productions.com/Webster/www.artofasm.com/Linux/HTML/AoATOC.html)
