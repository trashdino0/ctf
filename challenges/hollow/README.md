# Hollow

**Category:** Crypto  
**Difficulty:** Hard  
**Points:** 500

You recovered one RSA ciphertext and a service that answers a single yes/no
question about any ciphertext you send it:

> "Does this decrypt to a correctly formatted PKCS#1 v1.5 encryption block?"

Recover the flag from [`dist/ciphertext.hex`](dist/ciphertext.hex).

## Folder contents

| Path | What it is |
| --- | --- |
| [`dist/`](dist/) | Player-facing files: public key, ciphertext, instructions, checksums |
| [`hints/HINTS.md`](hints/HINTS.md) | Optional nudges, hidden in collapsible sections |
| [`SOLUTION/SOLUTION.md`](SOLUTION/SOLUTION.md) | ⚠️ **FULL SPOILER** — the complete writeup |

Start with `dist/README.md` for the public parameters and oracle protocol. The
oracle is hosted by the challenge; players do not get the private key or the
local launcher. Only open the solution when you are done.
