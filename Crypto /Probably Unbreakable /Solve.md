## Description
I set up a place to for everyone to enjoy their favorite thing: random numbers! Random numbers are great because they let you do really great, really easy encryption. I even left the flag in there, in case that's what you're here for. There's definitely a way to find it. Just kidding! Probably.

The challenge gives us three options:

```text
How many list-scrambles do you want?
How many random-letter-pickings do you want?
How many flag encryptions do you want?
```


### Flag

```text
bronco{4t_l3a5t_1mpr0b4b1e_th0ugh}
```

---


At first, the challenge looks like it is related to randomness and PRNGs. However, after looking carefully, I noticed that the server also allows us to encrypt the flag multiple times.

Instead of attacking the random number generator, I decided to collect many encrypted versions of the same flag and compare them.

---

# Step 1 - Connect to the Server

First, I connected to the remote challenge using the `pwntools` library.

```python
from pwn import remote

HOST = "0.cloud.chals.io"
PORT = 16474

r = remote(HOST, PORT)
```

This establishes a connection with the challenge server.

---

# Step 2 - Skip the Random Features

The server first asks:

```text
How many list-scrambles do you want?
How many random-letter-pickings do you want?
```

These options are not needed for solving the challenge.

So I answered both with `0`.

```python
r.sendline(b"0")
r.sendline(b"0")
```

Now the server asks:

```text
How many flag encryptions do you want?
```

---

# Step 3 - Request Multiple Encryptions

Instead of requesting one encrypted flag, I requested 100 encryptions.

```python
r.sendline(b"100")
```

The server returned 100 different ciphertexts.

I stored each ciphertext inside a list.

```python
ciphertexts = []

for _ in range(100):
    line = r.recvline().strip().decode()
    ciphertexts.append(bytes.fromhex(line))
```

Now I have many encrypted versions of the same flag.

---

# Step 4 - Understand the Encryption

The plaintext (flag) is always the same.

Only the encryption key changes each time.

The important observation is that every key character belongs to the following character set:

```text
abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_-
```

Since the key can only contain these characters, we can use this information to recover the original flag.

---

# Step 5 - Recover the Flag

The script recovers the flag one character at a time.

For every byte position:

- Try every possible byte value (0–255).
- XOR the guessed byte with the ciphertext byte.
- Check if the result is a valid key character.

```python
for f in range(256):
```

The following condition checks whether the guessed byte works for **all** ciphertexts.

```python
all((enc[i] ^ f) in keystring for enc in ciphertexts)
```

If the condition is true, then the guessed byte is the correct flag character.

The script repeats this process until every character of the flag is recovered.

---

# Step 6 - Print the Flag

Finally, the script prints the recovered flag.

```python
print(flag)
```

Output:

```text
bronco{4t_l3a5t_1mpr0b4b1e_th0ugh}
```

---

# Complete Solution Script

```python
from pwn import remote

HOST = "0.cloud.chals.io"
PORT = 16474

r = remote(HOST, PORT)

r.recvuntil(b"How many list-scrambles do you want?")
r.sendline(b"0")

r.recvuntil(b"How many random-letter-pickings do you want?")
r.sendline(b"0")

r.recvuntil(b"How many flag encryptions do you want?")
r.sendline(b"100")

ciphertexts = []

for _ in range(100):
    line = r.recvline().strip().decode()
    if line:
        ciphertexts.append(bytes.fromhex(line))

r.close()

keystring = b"abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_-"

flag = ""

flag_length = len(ciphertexts[0])

for i in range(flag_length):
    for f in range(256):
        if all((enc[i] ^ f) in keystring for enc in ciphertexts):
            flag += chr(f)
            break

print(f"[+] Recovered Flag: {flag}")
```

---

# Why This Works

The challenge encrypts the same flag multiple times.

Although each encryption uses a different key, every key character always comes from the same limited set of characters.

By collecting many ciphertexts, we can test every possible flag byte.

Only the correct flag character will produce valid key characters for every ciphertext.

Using this observation, the entire flag can be recovered without needing to know the random key.



This was a fun beginner-friendly crypto challenge that demonstrated how repeated encryptions and a restricted key space can make an encryption scheme vulnerable.
