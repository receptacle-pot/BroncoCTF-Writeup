## Custom Cipher

### Challenge Description


> *I've finally made the perfect public-key encryption algorithm! Its security is rivaled only by its message space efficiency. In fact, I'm so confident that I'll send you the flag along with my public key, and there's nothing you can do to read it.*


### Flag : bronco{f4ct0r1ng_i5_fr3e???}

We are given two files:

* `pscheme.py` – Python source code of the encryption algorithm.
* `enc.txt` – Contains the public key and the encrypted flag.

Our goal is to recover the flag.

---

# Step 1: Reading the Source Code

The first thing I did was open `pscheme.py` and understand how the encryption works.

There are three important functions:

* `keygen()`
* `encrypt()`
* `encrypt_string()`

---

# Step 2: Understanding Key Generation

Inside `keygen()` we see:

```python
private = sorted([random.randint(1, 2**8 - 1) for _ in range(64)])

public = Poly([1])

for root in private:
    root_poly = Poly([-root, 1])
    public = public * root_poly
```

This means that for every private key value `r`, the program creates a polynomial:

[
(x-r)
]

and multiplies all of them together.

For example, if the private roots were:

```
3
5
8
```

then the public polynomial would be

[
(x-3)(x-5)(x-8)
]

After multiplying everything together, only the final polynomial is published.

At first, this looked secure because we only see a huge polynomial.

---

# Step 3: Understanding Encryption

Next I looked at the `encrypt()` function.

```python
for root in message:
    root_poly = Poly([-root,1])
    public = public * root_poly
```

This code does **not** encrypt the message using any mathematical trapdoor.

Instead, it simply multiplies four more factors into the public polynomial.

If the message bytes are

```
65
66
67
68
```

then encryption becomes

[
Public(x)
\times
(x-65)
\times
(x-66)
\times
(x-67)
\times
(x-68)
]

So the encrypted polynomial is simply the public polynomial with four additional roots.

This was the biggest clue.

---

# Step 4: Small Example

Suppose the private roots are

```
3
5
8
```

The public key is

[
(x-3)(x-5)(x-8)
]

Now suppose the plaintext is

```
A
B
```

ASCII values

```
65
66
```

The ciphertext becomes

[
(x-3)(x-5)(x-8)(x-65)(x-66)
]

If we factor this polynomial we obtain

```
3
5
8
65
66
```

The first three roots belong to the public key.

The remaining roots are

```
65
66
```

which are exactly the plaintext bytes.

So the encryption is leaking the message directly.

---

# Step 5: Looking at enc.txt

The file contains two sections.

```
====PUBLIC KEY====
...

====MESSAGE====
...
```

The first section is one very large polynomial.

The second section contains several encrypted blocks separated by `/`.

Each block contains

* Polynomial coefficients
* One integer called `order`

---

# Step 6: Recovering the Public Roots

The public polynomial has degree 64.

That means it has exactly 64 roots.

By factoring the polynomial we recover every public root.

These are exactly the private values used during key generation.

---

# Step 7: Recovering the Message

Each encrypted polynomial contains

* 64 public roots
* 4 message roots

After factoring an encrypted block, I simply removed every root that already appeared in the public key.

The four remaining roots were the ASCII values of the plaintext.

For example

```
Public roots

12
18
25
31
...
```

Encrypted roots

```
12
18
25
31
...
98
111
114
95
```

Removing the public roots leaves

```
98
111
114
95
```

ASCII conversion gives

```
b
o
r
_
```

---

# Step 8: Understanding the Order Value

One problem still remained.

The program sorts the message before multiplying.

```python
order = sorted(message)
```

This means the recovered roots are always sorted.

For example

Original plaintext

```
o
b
r
_
```

ASCII

```
111
98
114
95
```

Sorted

```
95
98
111
114
```

Without extra information we would recover

```
_bor
```

instead of

```
obr_
```

To solve this, the program stores the original order inside an integer called `order`.

```python
order = sum([x << (2 * i) for i, x in enumerate(order)])
```

Each character position is stored using two bits.

By unpacking these bits we can reconstruct the original order of the four characters.

---

# Step 9: Recovering Every Block

The ciphertext contains several encrypted blocks separated by `/`.

For each block I repeated the same process:

1. Factor the encrypted polynomial.
2. Remove all public roots.
3. Keep the four remaining roots.
4. Decode the `order` value.
5. Restore the original order.
6. Convert ASCII values into characters.

Finally, joining every block together revealed the complete flag.

---

# Flag

```text
bronco{f4ct0r1ng_i5_fr3e???}
```

---

# Why the Scheme is Broken

The security of the scheme depends on the public polynomial hiding its roots.

However, encryption simply appends four new roots corresponding to the plaintext characters.

So the ciphertext is

[
Public(x)
\times
(x-m_1)
\times
(x-m_2)
\times
(x-m_3)
\times
(x-m_4)
]

Anyone can:

* Factor the public polynomial.
* Factor the ciphertext polynomial.
* Remove the common roots.
* Read the remaining roots as ASCII values.

Therefore, the encryption provides no secrecy at all.

---

