# Vault — hints

These escalate from vague to specific. Each block is collapsed so you only see
the next hint when you choose to.

<details>
<summary>Hint 1 — where to look</summary>

Everything you need is inside `dist/vault.exe`. It is not packed and does not
phone home. Run it once to see the interface, then inspect the raw bytes rather
than only the disassembly.

</details>

<details>
<summary>Hint 2 — find the hidden data</summary>

Search the binary in a hex editor for these byte sequences:

```text
DE AD BE EF
FE ED FA CE
```

Two encrypted arrays sit at/after those markers.

</details>

<details>
<summary>Hint 3 — decrypt them</summary>

Both arrays are XOR-encrypted with short repeating keys. The decryption loops
in the binary reveal the key bytes. After decrypting, one array is a VM program
made of fixed 4-byte instructions; the other is the expected result.

</details>

<details>
<summary>Hint 4 — understand the check</summary>

Map the interpreter's opcode numbers from its jump table. Focus on the loop that
walks the 24 input bytes. The transform uses:

- two keys that depend on the byte index,
- a rotate,
- chaining with the previous output byte.

The first chaining value is a single immediate in the bytecode.

</details>

<details>
<summary>Hint 5 — anti-debug</summary>

Several debugger checks flip one immediate by `0x1F`, which makes every result
wrong under a debugger. Solve statically from the decrypted bytecode so you get
the real parameters.

</details>

<details>
<summary>Hint 6 — give up?</summary>

If you are truly done, the full writeup is in
[`../SOLUTION/SOLUTION.md`](../SOLUTION/SOLUTION.md). It contains the answer.

</details>
