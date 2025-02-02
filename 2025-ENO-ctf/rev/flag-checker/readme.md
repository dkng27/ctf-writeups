# Flag Checker

Difficulty: Easy \
Category: Rev

## Binary

```c
len = strlen(input);
if (len == 0x22) {
    obfuscation(input,obfuscated);
    for (i = 0; i < 34; i = i + 1) {
        if (obfuscated[i] != (&obfuscated_flag)[i]) {
            correct = 0;
            goto exit;
        }
    }
    correct = 1;
}
else {
    correct = 0;
}
```

```c
// in obfuscation()
for (i = 0; i < 34; i = i + 1) {
    output[i] = (output[i] ^ 0x5a) + i;
    output[i] = output[i] << 3 | output[i] >> 5;
}
return;
```

Simple flag checker style program.
Stores an obfuscated flag after XOR and some addition operations.

Can be easily reversed by z3 solver.

## Script

```py
import z3

# obfuscated flag stored in program, 34 bytes
known = [
    0xF8, 0xA8, 0xB8, 0x21, 0x60, 0x73, 0x90, 0x83, 0x80, 0xC3, 0x9B, 0x80,
    0xAB, 0x09, 0x59, 0xD3, 0x21, 0xD3, 0xDB, 0xD8, 0xFB, 0x49, 0x99, 0xE0,
    0x79, 0x3C, 0x4C, 0x49, 0x2C, 0x29, 0xCC, 0xD4, 0xDC, 0x42,
]

# Create a solver instance
solver = z3.Solver()

# Create a list of 34 BitVec variables
data = [z3.BitVec(f"data_{i}", 8) for i in range(34)]
# Add constraints for each character in the data with the intermediate step
for i in range(34):
    intermediate = ((data[i] ^ 0x5A) + i)
    processed = (intermediate << 3 | z3.LShR(intermediate, 5))
    solver.add(processed == known[i])

# Check if the constraints are satisfiable
if solver.check() == z3.sat:
    model = solver.model()
    flag = "".join([chr(model[data[i]].as_long()) for i in range(34)])
    print(f"Flag: {flag}")
else:
    print("No solution found")
```

It took me a while to find out `z3.LShR` does things differently to `>>` and gives the correct result.

## Flag

`ENO{R3V3R53_3NG1N33R1NG_M45T3R!!!}`
