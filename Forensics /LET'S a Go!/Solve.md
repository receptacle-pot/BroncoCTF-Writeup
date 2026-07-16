# Description
## Go step by step, brick by brick.

## Flag : bronco{3ve4yth1ng_1s_aw3s0me}

## Intended Solution:

The flag is split character-by-character across **50 hidden files** named `.part_00`
through `.part_49`, scattered non-sequentially through a directory tree that is 6+
levels deep and contains a mix of realistic-looking folders (`.git/objects`,
`.config/systemd/user`, `tmp/system_logs/kernel`, etc.).

An additional **200 decoy files** named `.part_50` through `.part_249` are seeded
throughout the same tree with random characters to make naive concatenation produce
garbage.

### Step 1 — discover the hidden files

Standard `ls` hides dot-files. Use a recursive find that shows hidden entries:

```bash
# Linux / macOS
find lego_bricks_challenge -name '.part_*'

# Windows PowerShell
Get-ChildItem -Recurse -Hidden -Filter '.part_*' lego_bricks_challenge
```

### Step 2 — filter out the decoys

The real parts are indices **00–49**; anything ≥ 50 is a decoy.
A regex on the filename is the cleanest filter:

```bash
find lego_bricks_challenge -name '.part_*' | grep -P '\.part_([0-3][0-9]|4[0-9])$'
```

### Step 3 — sort numerically and concatenate

Zero-padded names sort correctly with a numeric key on the suffix:

```bash
find lego_bricks_challenge -name '.part_*' \
  | grep -P '\.part_([0-3][0-9]|4[0-9])$' \
  | sort -t_ -k2 -n \
  | xargs cat \
  | tr -d '\n'
```

Output:

```
bronco{3ve4yth1ng_1s_aw3s0me}
```

### Automated solver (Python)

```bash
python solver.py lego_bricks_challenge
```

`solver.py` walks the tree with `os.walk` (which descends into hidden directories
automatically), collects indices 0–49, sorts numerically, and concatenates.
