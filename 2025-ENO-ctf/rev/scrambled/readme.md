# Scrambled

Category: Rev \
Difficulty: Easy

## Source

```py
import random

def encode_flag(flag, key):
    xor_result = [ord(c) ^ key for c in flag]

    chunk_size = 4
    chunks = [xor_result[i:i+chunk_size] for i in range(0, len(xor_result), chunk_size)]
    seed = random.randint(0, 10)
    random.seed(seed)
    random.shuffle(chunks)
    
    scrambled_result = [item for chunk in chunks for item in chunk]
    return scrambled_result, chunks

def main():
    flag = "REDACTED"
    key = REDACTED

    scrambled_result, _ = encode_flag(flag, key)
    print("result:", "".join([format(i, '02x') for i in scrambled_result]))


if __name__ == "__main__":
    main()
```

```txt
result: 1e78197567121966196e757e1f69781e1e1f7e736d6d1f75196e75191b646e196f6465510b0b0b57
```

## Idea

The flag is byte XOR'd by some unknown `key` and then the separated into 4 byte chunks. The chunks are then shuffled by `random.shuffle()`.

We can observe that since the XOR is carried out byte-wise, the `key` can only be 1 byte (256 possibilities). At the same time, the shuffle is seeded by a random integer generated between 0 and 10:

```py
seed = random.randint(0, 10)
random.seed(seed)
```

This leaves us with just 10 x 256 = 2560 possibilities. **Very easy to bruteforce.**

## Script

```py
import random

def decode_flag(scrambled_result, key, seed):
    chunk_size = 4
    num_chunks = len(scrambled_result) // chunk_size
    chunks = [scrambled_result[i*chunk_size:(i+1)*chunk_size] for i in range(num_chunks)]
    
    random.seed(seed)
    idx = [i for i in range(num_chunks)]
    random.shuffle(idx)

    chunks = [chunks[idx.index(i)] for i in range(num_chunks)]
    
    xor_result = [item for chunk in chunks for item in chunk]
    flag = ''.join([chr(c ^ key) for c in xor_result])
    return flag

def hex_string_to_list(hex_string):
    return [int(hex_string[i:i+2], 16) for i in range(0, len(hex_string), 2)]

def main():
    # Example scrambled result and key
    scrambled_result = hex_string_to_list('1e78197567121966196e757e1f69781e1e1f7e736d6d1f75196e75191b646e196f6465510b0b0b57')
    for key in range(256):
        for seed in range(11):
            flag = decode_flag(scrambled_result, key, seed)
            if flag.startswith("ENO{"):
                print("Decoded flag:", flag)

if __name__ == "__main__":
    main()
```
