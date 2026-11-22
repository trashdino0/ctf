# Hollow — hints

These escalate from vague to specific. Each block is collapsed so you only see
the next hint when you choose to.

<details>
<summary>Hint 1 — what the oracle actually tells you</summary>

It returns exactly one bit: whether the decrypted block has well-formed
PKCS#1 v1.5 padding. It does not return plaintext bytes, and it does not return
the private key. One bit is still enough if you can choose which ciphertext to
submit.

</details>

<details>
<summary>Hint 2 — turn padding into an interval</summary>

A correctly padded PKCS#1 v1.5 encryption block starts with `00 02`. As an
integer, any such block lies in:

```text
2B <= block < 3B        where B = 2^(8(k-2)) and k = modulus byte length
```

So a `1` answer means "the padded plaintext landed in this range".

</details>

<details>
<summary>Hint 3 — choose the ciphertext</summary>

RSA is multiplicatively homomorphic:

```text
dec(c * s^e mod n) = dec(c) * s mod n
```

By multiplying the challenge ciphertext by `s^e`, you get to multiply the
unknown plaintext by `s` without knowing it.

</details>

<details>
<summary>Hint 4 — narrow it down</summary>

Search for values of `s` such that `dec(c * s^e)` still has valid padding. Each
accepted `s` restricts the possible original plaintext to a smaller set of
intervals. Intersect those intervals until one integer remains.

</details>

<details>
<summary>Hint 5 — two cases</summary>

When you have several intervals, walk `s` upward. When you have one interval,
search over an integer `r` and test a bounded range of `s` values. The solver
needs both cases.

</details>

<details>
<summary>Hint 6 — give up?</summary>

If you are truly done, the full writeup is in
[`../SOLUTION/SOLUTION.md`](../SOLUTION/SOLUTION.md). It contains the answer.

</details>
