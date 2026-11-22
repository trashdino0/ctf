> # ⚠️ SPOILER — FULL SOLUTION
>
> **Everything below gives away the answer.** Do not open this file until you
> have finished attempting the challenge, or you will spoil it for yourself.
>
> Want a nudge instead? Use [`../hints/HINTS.md`](../hints/HINTS.md).

---

# Casket — solution

**License key:** `S3ALED-C4SK3T-720`  
**Flag:** `CTF{cl4ssl0ad3r_c4sk3t_1nt3gr4l}`

## 1. The outer JAR

`casket.jar` is a normal ZIP with one unusual feature: its end-of-central-
directory (EOCD) record declares a nonzero ZIP comment length, and those
"comment" bytes are actually AES-CBC ciphertext. A plain unzip does not show
them as a file, so this is the first place to look.

The visible `Main.class` is intentionally small. It:

- reads custom manifest attributes,
- gathers four key shards,
- decrypts the ZIP-comment blob,
- defines the decrypted class from memory.

All failure paths print the same generic `Invalid license.` message.

## 2. The four shards

The first AES key is `SHA-256(A ‖ B ‖ C ‖ D)`.

| Shard | Source | Value |
| --- | --- | --- |
| `A` | XOR-obfuscated constants in `Main` | `11 22 33 44` |
| `B` | Hex in resource name `res/k_abcd.bin` | `ab cd` |
| `C` | Bytes appended after the PNG `IEND` chunk | `5a 5a 5a 5a` |
| `D` | Gauss-law total charge, big-endian `uint16` | `02 d0` |

So:

```text
combined = 11223344abcd5a5a5a5a02d0
key1     = SHA256(combined)
```

The manifest supplies `Casket-IV` for this first decryption.

### The physics question

The image describes a spherically symmetric charge density:

```text
ρ(r) = (90/π) e^-r
```

Total charge comes from integrating density over all space. In spherical
coordinates, `dV = 4πr² dr`:

```text
Q = ∫ ρ dV
  = 4π ∫₀^∞ r² ρ(r) dr
  = 4π (90/π) ∫₀^∞ r² e^-r dr
  = 360 · Γ(3)
  = 360 · 2!
  = 720
```

The π cancels, and the remaining radial integral is the Gamma-function
factorial `Γ(3) = 2! = 2`. `720 = 0x02d0`, which is shard `D`.

## 3. Stage 1 — the memory loader

Decrypting the ZIP-comment blob with `key1` and `Casket-IV` yields a small
JAR containing `L1.class`. It is loaded with a custom in-memory `ClassLoader`,
so it never appears as a normal resource.

`L1` then:

1. reads three `res/shard_*.bin` files,
2. concatenates them in the order from `Casket-Order` — here `2,0,1`,
3. derives `key2 = SHA256(combined ‖ "2,0,1")`,
4. decrypts the reassembled blob with `key2` and `Casket-IV2`,
5. loads `L2.class` from the resulting nested JAR.

The shard order matters: using `0,1,2` gives a padding error and therefore the
same generic failure message.

## 4. Stage 2 — the license check

`L2` contains two byte arrays XORed with `0x5A`:

```text
WANT      -> S3ALED-C4SK3T-720
DECOY_WANT -> S3ALED-C4SK3T-721
```

The comparison is deliberately flattened into a small state machine so it does
not read like a simple `String.equals` in a decompiler. The `720` suffix is a
deliberate callback to the integral; `721` is the decoy. Submitting the decoy
prints:

```text
CTF{n0t_th3_r34l_fl4g_y0u_f00l}
```

## 5. Unsealing the flag

With the correct key, `L2` computes:

```text
key  = SHA256("S3ALED-C4SK3T-720")
iv   = 10 11 12 13 14 15 16 17 18 19 1a 1b 1c 1d 1e 1f
```

It uses those to AES-CBC-decrypt `flag.enc` from the nested JAR. The plaintext
is the real flag:

```text
CTF{cl4ssl0ad3r_c4sk3t_1nt3gr4l}
```

## 6. Why it is built this way

The challenge is deliberately distinct from a simple "guess the flag" binary:

- the accepted license key is **not** the flag,
- the flag bytes do not appear anywhere in the public JAR,
- neither does the license key nor the generic error string,
- the first key is assembled from four unrelated-looking places,
- each stage only reveals the location and shape of the next one.

That makes static reconstruction of the pipeline the intended path.
