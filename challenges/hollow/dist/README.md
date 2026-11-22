# Hollow — a PKCS#1 v1.5 padding oracle

You recovered one RSA ciphertext and a service that will answer exactly one
question about any ciphertext you send it:

> "Does this decrypt to a correctly formatted PKCS#1 v1.5 encryption block?"

It will not give you the plaintext. It will not give you the private key. It only
answers `1` (padding looks valid) or `0` (padding is invalid).

## Files

| File | Purpose |
| --- | --- |
| `public.txt` | Modulus and public exponent |
| `ciphertext.hex` | The encrypted flag |
| `SHA256SUMS.txt` | Hashes of the player-facing files |

## Talking to the oracle

The oracle is a service run by the challenge host. Submit one hexadecimal
ciphertext and it answers `1` (padding looks valid) or `0` (padding invalid):

```text
> 4f3a...c1
1
> 0000...00
0
```

The reference implementation and an offline launcher live in the private author
kit and are **not** part of this public repo — players only get the endpoint.
Each oracle session allows 500,000 queries.

## Goal

Recover the plaintext of `ciphertext.hex` and submit the flag inside it.
