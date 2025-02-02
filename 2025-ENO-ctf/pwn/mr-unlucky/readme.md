# Mr. Unlucky

Difficulty: Easy (50 pts) \
Category: Web

> I have a love/hate relationship with dota2, I either always win or always lose. there is no inbetween :( \
However, Oracle told me that If I win at his cursed game I'll win every match and gain the aegis of Immortality in real life as well!
CAN YOU HELP ME??? \
Note: The challenge autor is an immortal player already ;)

`nc 52.59.124.14:5021`

Attachments: `Dockerfile`, `mr_unlucky` (binary)

## Idea

The challenge comes from the application needing you to guess 50 times correctly a random Hero name from dota 2. If you guess correctly 50 times in a row, then you get the flag.

Here is a cleaned code output from ghidra:

```c
  // in main
  puts("I have always been unlucky. I can\'t even win a single game of dota2 :(");
  puts("however, I heard that this tool can lift the curse that I have!");
  puts("YET I CAN\'T BEAT IT\'S CHALLENGE. Can you help me guess the names?");
  seed = time((time_t *)0x0);
  srand((uint)seed);
  sleep(3);
  puts(
      "Welcome to dota2 hero guesser! Your task is to guess the right hero each time to win the chal lenge and claim the aegis!"
      );
  for (i = 0; i < 50; i = i + 1) {
    index = rand();
    printf("Guess the Dota 2 hero (case sensitive!!!): ");
    fgets(input,0x1e,stdin);
    len = strcspn(input,"\n");
    input[len] = '\0';
    result = strcmp(input, heroes[index]);
    if (result == 0) {
      printf("%s was right! moving on to the next guess...\n",input);
    }
    else {
      printf("Wrong guess! The correct hero was %s.\n", heroes[index]);
      exit(0);
    }
  }
  puts(
      "Wow you are one lucky person! fine, here is your aegis (roshan will not be happy about this!) "
      );
  print_flag("flag.txt");
```

## Repeating pseudorandom sequence

Since the number generator is seeded once with the current time:

```c
  seed = time((time_t *)0x0);
  srand((uint)seed);
```

And that we have the binary ourselves, **we can run the local binary at the same time as we connect to the remote server**. Since the time is about the same on both programs, their random seed is the same, so they will **generate the same list** of hero names!

Here's what we do:

1. Start local and remote program at the same time (same seed)
2. Obtain output from local
3. Send output to remote
4. ???
5. Profit!

## Getting Output from Local Binary

Since the program exits if we get a guess wrong, but we want to keep the program running to match the remote program, we can do a trick in Ghidra which is to patch and remove the `exit()` instruction.

![patch](image.png)

```c
    printf("Wrong guess! The correct hero was %s.\n", heroes[index]);
    exit(0); // patch with NOP (0x90)
```

This way, **the program prints out the answer for us**, and keeps going to the next guess, which we can submit to the remote server!!!

## Solve Script

```py
from pwn import *

r = remote("52.59.124.14", 5021, level="debug")
r.recvuntil(b"names?\n")
l = process("./mr_unlucky_patched", level='debug')
l.recvuntil(b"names?\n")

sleep(3)

for i in range(50):
    l.sendlineafter(b": ", b"a")
    l.recvuntil(b"hero was ")
    hero = l.recvuntil(b".", drop=True).strip().decode()
    success(hero)
    r.recvuntil(b": ")
    r.sendline(hero.encode())

r.interactive()
```

## Flag

`ENO{0NLY_TH3_W0RTHY_0N35_C4N_CL41M_THE_AEGIS_OF_IMMORTALITY!!!}`
