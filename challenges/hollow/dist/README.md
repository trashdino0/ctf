# Hollow — a PKCS#1 v1.5 padding oracle

You recovered one RSA ciphertext and a black box that will answer exactly one
question about any ciphertext you send it:

> "Does this decrypt to a correctly formatted PKCS#1 v1.5 encryption block?"

It will not give you the plaintext. It will not give you the private key. It only
answers `1` (padding looks valid) or `0` (padding is invalid).

## Files

| File | Purpose |
| --- | --- |
| `public.txt` | Modulus and public exponent |
| `ciphertext.hex` | The encrypted flag |
| `oracle.jar` | The black-box padding oracle |
| `SHA256SUMS.txt` | Hashes of the player-facing files |

## Talking to the oracle

The oracle is a compiled JAR. From this `dist/` folder, start it with:

```powershell
java -jar oracle.jar
```

It reads one hexadecimal ciphertext per line and writes one answer per line:

```text
4f3a...c1
1
0000...00
0
```

You can also submit a single ciphertext without the interactive loop:

```powershell
java -jar oracle.jar 4f3a...c1
```

Each oracle process allows 500,000 queries. The checker is a black box: its
source and the private key are not part of this public repo. Treat it like a
remote service and use only its one-bit answers.

Java 17+ is required to run the oracle.

## Goal

Recover the plaintext of `ciphertext.hex` and submit the flag inside it.
