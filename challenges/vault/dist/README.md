# Vault

**Category:** Reverse
**Difficulty:** Advanced
**Points:** 400
**File:** `vault.exe` (Windows x64, no packer, no network access required)

> The off-site backup vault has been sitting in the server room since the
> migration. Nobody remembers the access key, but the firmware is still
> there. The lock screen claims it checks a 24-byte key. Someone in the
> ticket queue swears it "works fine on their machine."

Find the access key the vault accepts.

## Usage

```text
vault.exe
Enter the access key: <your input>
```

A correct key prints `ACCESS GRANTED.` followed by the flag. Anything else
prints `ACCESS DENIED.`

## Notes

- The flag is the access key itself, in the standard `CTF{...}` format.
- The binary is not packed and does not phone home. Everything you need is
  inside the executable.
- Static analysis is recommended. If you attach a debugger, be aware that
  the program may not behave the same way as when it runs on its own.
