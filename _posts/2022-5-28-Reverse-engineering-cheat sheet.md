---
title: Reverse Engineering Cheat sheet
layout: default
tags: [Reverse engineering]
blog_post: true
---


# Introduction
I’ll post some Assembly to understand this article, you must know what asm is :slight_smile: here a cheat sheet if you got stuck, Assembly language, often referred to as asm, is a low-level programming language that closely corresponds to the machine code instructions of a specific computer architecture. It provides a more human-readable representation of machine instructions and allows direct manipulation of hardware resources.

If you're new to Assembly or need a quick reference, Here's one: 

## Data Movement:
```assembly
-   MOV: Copy
-   XCHG: Exchange
-   LEA: Load effective address
```

## Arithmetic and Logic:
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
