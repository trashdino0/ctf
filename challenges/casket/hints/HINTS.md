# Casket — hints

These escalate from vague to specific. Each block is collapsed so you only see
the next hint when you choose to.

<details>
<summary>Hint 1 — it is still a ZIP</summary>

A `.jar` is a ZIP file. Before you decompile anything, look at the raw bytes as
well as the entries a normal unzip shows. Java accepts some data that unzip
utilities ignore.

</details>

<details>
<summary>Hint 2 — the manifest matters</summary>

Open `META-INF/MANIFEST.MF`. There are custom attributes besides `Main-Class`.
Some are Base64; one is an ordering hint.

</details>

<details>
<summary>Hint 3 — four shards</summary>

The first AES key is `SHA-256` of four concatenated shards:

1. bytes hidden as XOR-obfuscated constants in the visible loader,
2. bytes encoded in a resource filename,
3. bytes hidden after the PNG's `IEND` chunk,
4. a two-byte answer to an AP Physics C E&M question shown in the image.

</details>

<details>
<summary>Hint 4 — the filename shard</summary>

There is a resource named like `res/k_????.bin`. The hex between `k_` and `.bin`
is the second shard. It is not the contents of that file.

</details>

<details>
<summary>Hint 5 — the physics problem</summary>

The image gives a spherical charge density:

```text
ρ(r) = (90/π) e^-r
```

Total charge is `Q = ∫ρ dV = 4π ∫₀^∞ r² ρ(r) dr`. The π in the density cancels
the `4π` from spherical volume. The remaining radial integral is a Gamma
function: `∫₀^∞ r² e^-r dr = Γ(3) = 2!`. Encode `Q` as a big-endian two-byte
value.

</details>

<details>
<summary>Hint 6 — the first encrypted layer</summary>

The encrypted stage-1 loader is in the ZIP end-of-central-directory comment.
The comment length is stored in the EOCD record; the bytes after it are the AES
ciphertext. The IV comes from the manifest.

</details>

<details>
<summary>Hint 7 — the nested JAR</summary>

Stage 1 reassembles three `shard_*.bin` resources in the order given by the
manifest, decrypts them with a key derived from the same four shards plus that
order string, and loads a nested JAR from memory.

</details>

<details>
<summary>Hint 8 — the license key</summary>

The final class compares your input against a byte array XORed with `0x5A`.
The decoded value ends in the same number the integral produced. One nearby
value is a decoy that prints a fake flag.

</details>

<details>
<summary>Hint 9 — give up?</summary>

If you are truly done, the full writeup is in
[`../SOLUTION/SOLUTION.md`](../SOLUTION/SOLUTION.md). It contains the answer.

</details>
