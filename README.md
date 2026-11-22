# CTFs

A small collection of self-contained CTF challenges. This repo is **public**, so
only player-facing files, hints, and clearly marked solutions are committed.
Author source, private keys, and reference solvers stay in the gitignored
`private/` tree.

## Layout

```text
challenges/
  <name>/
    dist/                 player files only (executable / ciphertext / public key)
    hints/HINTS.md        collapsible, escalating hints
    SOLUTION/SOLUTION.md  ⚠️ FULL SPOILER — open only when you are done
private/                  author-only, gitignored — never commit this
```

## Challenges

| Challenge | Category | Difficulty | Player files | Hints | Solution |
| --- | --- | --- | --- | --- | --- |
| Vault | Reverse | Advanced | [`dist/`](challenges/vault/dist/) | [`HINTS.md`](challenges/vault/hints/HINTS.md) | ⚠️ [`SOLUTION.md`](challenges/vault/SOLUTION/SOLUTION.md) |
| Hollow | Crypto | Hard | [`dist/`](challenges/hollow/dist/) | [`HINTS.md`](challenges/hollow/hints/HINTS.md) | ⚠️ [`SOLUTION.md`](challenges/hollow/SOLUTION/SOLUTION.md) |

## Spoiler policy

- `hints/HINTS.md` uses collapsed `<details>` blocks, escalating from vague to
  specific. Nothing is revealed until you click.
- `SOLUTION/SOLUTION.md` opens with a loud spoiler banner and contains the full
  answer. Do not open it while you are still trying to solve the challenge.

## For challenge authors

Everything under `private/` is excluded by [`.gitignore`](.gitignore). That
includes `vault.c`, `vm_data.h`, the Hollow RSA private key, the reference
solvers, and the verification scripts. Keep it local — committing it to a public
repo would spoil both challenges.
