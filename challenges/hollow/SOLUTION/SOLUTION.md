> # ⚠️ SPOILER — FULL SOLUTION
>
> **Everything below gives away the answer.** Do not open this file until you
> have finished attempting the challenge, or you will spoil it for yourself.
>
> Want a nudge instead? Use [`../hints/HINTS.md`](../hints/HINTS.md).

---

# Hollow — solution

## 1. What the oracle actually leaks

RSA with PKCS#1 v1.5 encryption encodes a message `M` into a `k`-byte block:

```text
EM = 0x00 0x02 || PS || 0x00 || M
      └─ 2 B ─┘   ≥8 B      └ separator
```

`PS` is at least eight nonzero random bytes. Numeric constraints follow:

```text
2B <= int(EM) < 3B        where B = 2^(8(k-2))
```

The oracle decrypts a submitted ciphertext and answers whether the result has
that shape. It never reveals the bytes — only the one-bit fact that the padded
integer landed in `[2B, 3B)` and has a valid separator.

That is enough. RSA is multiplicatively homomorphic:

```text
dec(c * s^e mod n) = dec(c) * s mod n
```

So by choosing `s`, we can slide the unknown plaintext around the modulus and
learn which intervals it can occupy. This is Bleichenbacher's 1998 chosen-
ciphertext attack.

## 2. Attack outline

Let `m0` be the original padded plaintext. Blinding step finds `s0` such that

```text
m = m0 * s0^-1 mod n   has valid padding, so m ∈ [2B, 3B)
```

Here the challenge ciphertext itself is valid, so `s0 = 1`.

We then keep a set of candidate intervals `M` for `m` (the blinded plaintext).
Each step searches a multiplier `s` such that some `m ∈ M` satisfies

```text
2B + r·n <= m·s <= 3B - 1 + r·n
```

for some integer `r`. For each interval `[a, b]` and each feasible `r`, that
gives a refined interval:

```text
new_lo = max(a, ceil((2B + r·n) / s))
new_hi = min(b, floor((3B - 1 + r·n) / s))
```

Intersect and repeat until one integer remains. Finally:

```text
m0 = m * s0^-1 mod n
```

then strip `0x00 0x02 PS 0x00` to get the flag.

### Multiplier search

- **Step 2a** — one initial interval: start at `s = ceil(n / 3B)` and walk up.
- **Step 2b** — several intervals: increment `s` until the oracle accepts.
- **Step 2c** — one interval `[a, b]`: iterate `r` from
  `ceil(2(b·s - 2B)/n)` and test every `s` in

  ```text
  ceil((2B + r·n)/b) <= s <= floor((3B + r·n)/a)
  ```

`tools/solve.py` implements exactly this. It talks to the oracle over a
subprocess pipe, never reading `author/secret.json`.

## 3. Running it

```powershell
python tests\verify.py
python tools\solve.py
```

The solver prints progress (`queries`, interval count, total interval width)
to stderr. The shipped key takes about **42,000 oracle calls** and finishes in
~20 seconds. `e = 3` is deliberate: it makes the `pow(s, e, n)` in each query
cheap, so the attack finishes in seconds instead of minutes.

## 4. Why this was avoidable

1. **Use RSA-OAEP**, not PKCS#1 v1.5, for encryption.
2. If v1.5 must be used, never let decryption errors distinguish bad padding
   from bad plaintext — return one identical error, in constant time.
3. Prefer authenticated encryption; RSA is a key-encapsulation primitive, not
   a general message-encryption scheme.

The bug is not weak math. It is the oracle: a one-bit side channel turned into
a full plaintext recovery by multiplicative blinding.
