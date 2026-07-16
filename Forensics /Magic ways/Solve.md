# Description

I rubbed this lamp and out came challenge.png. But, I can't open it for some reason. Help me out and maybe I'll grant you a wish (flag).

## Flag : bronco{wh4t_ar3_mag1c_byt3s} 

In this challenge, I was given a corrupted PNG image named
`challenge.png`. The goal was to repair the image manually and recover
the hidden flag.

At first, I tried opening the image normally, but it did not open in any
image viewer. This indicated that the PNG file was damaged.

Since PNG files have a fixed structure, I decided to inspect the file
using a hex editor.

------------------------------------------------------------------------

# Step 1 -- Opening the File in a Hex Editor

I opened `challenge.png` using a hex editor (HxD on Windows or `xxd` on
Linux).

The very first bytes of every PNG image should always be:

``` text
89 50 4E 47 0D 0A 1A 0A
```

These bytes are called the **PNG Magic Bytes** or **PNG Signature**.
They tell the operating system that the file is a PNG image.

Instead, I saw:

``` text
DE AD BE EF 00 00 00 00
```

This clearly showed that someone had intentionally corrupted the PNG
header.

------------------------------------------------------------------------

# Step 2 -- Repairing the PNG Signature

I replaced the first 8 bytes with the correct PNG signature.

### Incorrect

``` text
DE AD BE EF 00 00 00 00
```

### Correct

``` text
89 50 4E 47 0D 0A 1A 0A
```

After saving the file, I tried opening it again.

Unfortunately, it still did not work, which meant there were more
damaged parts inside the PNG.

------------------------------------------------------------------------

# Step 3 -- Understanding the PNG Structure

A PNG image is made up of different chunks.

  Chunk   Purpose
  ------- --------------------------------------------------
  IHDR    Stores image width, height and color information
  IDAT    Stores compressed image data
  IEND    Marks the end of the PNG

The challenge description mentioned that three things were corrupted:

-   PNG magic bytes
-   Image height
-   IHDR CRC

So next I focused on the IHDR chunk.

------------------------------------------------------------------------

# Step 4 -- Recovering the Image Height

The height field inside the IHDR chunk had been replaced with:

``` text
00 00 00 00
```

The actual image data inside the IDAT chunks was still present, so I
calculated the original height.

The image width was:

``` text
500 pixels
```

Since the image uses RGB colors, each pixel takes 3 bytes.

Each row therefore contains:

``` text
1 + (500 × 3) = 1501 bytes
```

I decompressed the IDAT data using Python:

``` python
import zlib

raw = zlib.decompress(all_idat_bytes)
```

Then calculated the height:

``` python
row_size = 1 + 500 * 3
height = len(raw) // row_size
```

The recovered height was:

``` text
200
```

------------------------------------------------------------------------

# Step 5 -- Restoring the Height

``` python
import struct

data[20:24] = struct.pack(">I", 200)
```

------------------------------------------------------------------------

# Step 6 -- Recalculating the CRC

The CRC had also been erased.

I recalculated it using:

``` python
import zlib

crc = zlib.crc32(b"IHDR" + data[16:29]) & 0xffffffff
data[29:33] = struct.pack(">I", crc)
```

------------------------------------------------------------------------

# Step 7 -- Saving the Image

After fixing:

-   PNG Signature
-   Height
-   CRC

I saved the repaired image as:

``` text
fixed_flag.png
```

The image opened successfully and displayed the flag.

------------------------------------------------------------------------

# Flag

``` text
bronco{wh4t_ar3_mag1c_byt3s}
```

------------------------------------------------------------------------

## Complete Solve Script which i used : 

``` python
import struct
import zlib

with open("challenge.png", "rb") as f:
    data = bytearray(f.read())

data[0:8] = bytes.fromhex("89 50 4E 47 0D 0A 1A 0A")

width = struct.unpack(">I", data[16:20])[0]

offset = 8
idat = b""

while offset < len(data):
    length = struct.unpack(">I", data[offset:offset+4])[0]
    chunk_type = data[offset+4:offset+8]

    if chunk_type == b"IDAT":
        idat += data[offset+8:offset+8+length]

    offset += length + 12

raw = zlib.decompress(idat)

row_size = 1 + width * 3
height = len(raw) // row_size

data[20:24] = struct.pack(">I", height)

crc = zlib.crc32(b"IHDR" + data[16:29]) & 0xffffffff
data[29:33] = struct.pack(">I", crc)

with open("fixed_flag.png", "wb") as f:
    f.write(data)

print("Recovered Height:", height)
print("Image repaired successfully!")
```

------------------------------------------------------------------------
