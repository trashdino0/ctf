# Casket

**Category:** Reverse / Java  
**Difficulty:** Hard  
**Points:** 500

> A sealed Java archive was recovered from an old build server. It prints a
> generic error for every wrong license key. Somewhere inside it is the real
> checker. Find the license key and make the archive reveal the flag.

## Usage

```powershell
java -jar casket.jar <license-key>
```

A correct key prints the flag. Everything else prints `Invalid license.`.

## Files

| File | Purpose |
| --- | --- |
| `casket.jar` | The challenge |
| `SHA256SUMS.txt` | Hashes of the player-facing files |

The JAR has no external dependencies. Java 17+ is recommended.
