> # ⚠️ SPOILER — FULL SOLUTION
>
> **Everything below gives away the answer.** Do not open this file until you
> have finished attempting the challenge, or you will spoil it for yourself.
>
> Want a nudge instead? Use [`../hints/HINTS.md`](../hints/HINTS.md).

---

# Vault — solution

**Flag:** `CTF{vm_r3v3rs1ng_1s_fun}`

Everything below is recoverable statically. The debugger trap is described in
section 7; it is a red herring that only punishes dynamic analysis.

---

## 1. Triage

`vault.exe` is an unpacked 64-bit Windows PE (MinGW, static runtime). Running it:

```text
+-------------------------------------------+
|  V A U L T   access control  v2.7         |
+-------------------------------------------+
Enter the access key: AAAAAAAAAAAAAAAAAAAAAAAA
ACCESS DENIED.
```

`strings` gives you the banner, the prompt and both result messages, but no flag
and no obvious key material. The only useful observation is from the length
check: exactly **24 bytes** are accepted. So we are looking for a 24-byte string
that satisfies some embedded check.

Import table of interest: `IsDebuggerPresent`, `GetThreadContext`,
`GetCurrentThread`, `clock`, `fgets`, `printf`. The first three hint at an
anti-analysis layer; `clock` suggests a timing check.

---

## 2. Locating the encrypted data

Open the file in a hex editor and search for `DE AD BE EF`. There is one hit at
file offset **0x3580** (virtual address `0x140005180`, symbol `g_blob`). The blob
is 132 bytes and has a second sentinel at offset `0x6C`:

```text
offset  0x00  DE AD BE EF                 sentinel (code)
offset  0x04  <100 bytes of encrypted VM bytecode>
offset  0x6C  FE ED FA CE                 sentinel (data)
offset  0x70  <24 bytes of encrypted expected ciphertext>
```

The compiler kept the blob contiguous because it is a single `static const`
array, so both arrays are easy to carve out by sentinel.

Decrypted (later in this writeup, section 5): the 100-byte blob is exactly 25
instructions of 4 bytes each.

---

## 3. Recovering the XOR keys from `main`

`main` starts at `0x140003440`. The first loop decrypts the bytecode. Intel
syntax:

```asm
14000345d  lea  r9,[rip+0x1d0c]      # 140005170 <CODE_KEY>
140003464  lea  r8,[rip+0x1d19]      # 140005184 <g_blob+0x4>
140003480  mov  rdx,rax              ; i
140003483  add  rcx,0x1
140003487  and  edx,0x7              ; i % 8
14000348a  movzx edx,BYTE PTR [r9+rdx]   ; CODE_KEY[i%8]
14000348f  xor  dl,BYTE PTR [r8+rax]     ; ^ enc_prog[i]
140003493  add  rax,0x1
140003497  mov  BYTE PTR [rcx-0x1],dl    ; prog[i]
14000349a  cmp  rax,0x64              ; 100 bytes
14000349e  jne  140003480
```

Dump `0x140005170` for 8 bytes to get the repeating key. The second loop is the
same shape with `DATA_KEY` at `0x140005168` and `g_blob+0x6c` for 24 bytes. GCC
folded the first expected byte into a constant (`mov BYTE PTR [rsp+0x30],0xdc`),
which is harmless.

Recovered keys:

```text
CODE_KEY = 5A C3 27 91 EE 14 7B 3D
DATA_KEY = A7 13 6B FE 42 D9 08 65
```

Expected ciphertext after decryption:

```text
DC BC 7C 25 FA 68 87 E5 1F D5 AB B0 11 18 C1 92 E0 2E 54 59 6B F7 45 59
```

---

## 4. Reversing the VM dispatcher

The decrypted program is executed by a switch compiled into a jump table. At
`0x140003704`:

```asm
140003704  movzx eax,BYTE PTR [rax]          ; opcode
140003707  movsxd rax,DWORD PTR [r9+rax*4]   ; jump table
14000370b  add  rax,r9
14000370e  jmp  rax
```

Immediately before it there is a bounds check against `0x19`, so there are 26
opcodes (0x00–0x19). Walking the 26 handlers gives this table; each instruction
is 4 bytes `[opcode][a][b][unused]`:

| Op | Name | Effect |
| --- | --- | --- |
| 00 | NOP | — |
| 01 | PUSH *imm* | push `a` |
| 02 | POP *reg* | `R[a] = pop()` |
| 03 | LDI *reg, imm* | `R[a] = b` |
| 04 | LDIN *reg, idxreg* | `R[a] = input[R[b]]` |
| 05 | XORI *reg, imm* | `R[a] ^= b` |
| 06 | ADDI *reg, imm* | `R[a] = (R[a]+b) & 0xff` |
| 07 | SUBI *reg, imm* | `R[a] = (R[a]-b) & 0xff` |
| 08 | MULI *reg, imm* | `R[a] = (R[a]*b) & 0xff` |
| 09 | ROL *reg, n* | 8-bit rotate left by `b` |
| 0A | ROR *reg, n* | 8-bit rotate right by `b` |
| 0B | XORR *dst, src* | `R[a] ^= R[b]` |
| 0C | MOVR *dst, src* | `R[a] = R[b]` |
| 0D | ADDR *dst, src* | `R[a] = (R[a]+R[b]) & 0xff` |
| 0E | INC *reg* | `R[a]++` |
| 0F | STOUT *idxreg, valreg* | `output[R[a]] = R[b]` |
| 10 | CMPR *a, b* | `ZF = (R[a]==R[b])` |
| 11 | CMPI *reg, imm* | `ZF = (R[a]==b)` |
| 12 | CMPEXP *valreg, idxreg* | `ZF = (R[a]==expected[R[b]])` |
| 13 | JMP *addr* | `pc = a` |
| 14 | JZ *addr* | jump if ZF |
| 15 | JNZ *addr* | jump if !ZF |
| 16 | ACCUM | `mismatches += !ZF` |
| 17 | HALT | stop |
| 18 | SWAP *a, b* | swap registers |
| 19 | NOT *reg* | bitwise complement |

`R0..R7` are byte-wide for the purposes of the check (all arithmetic is masked
to `0xff`). `ACCUM` is the only way the VM reports failure: final `mismatch == 0`
means every byte matched.

---

## 5. The decrypted program

Decrypting the 100 bytes with `CODE_KEY` and decoding 4-byte instructions yields:

```text
  0:  LDI R2, 0x5c             ; 03 02 5C 00
  1:  LDI R3, 0x00             ; 03 03 00 00
  2:  LDI R4, 0x18             ; 03 04 18 00
  3:  NOP                      ; 00 00 00 00
  4:  LDIN R0, R3              ; 04 00 03 00
  5:  MOVR R1, R3              ; 0C 01 03 00
  6:  MULI R1, 0x25            ; 08 01 25 00
  7:  ADDI R1, 0x0b            ; 06 01 0B 00
  8:  XORI R1, 0x6d            ; 05 01 6D 00
  9:  XORR R0, R1              ; 0B 00 01 00
 10:  ROL R0, 0x03             ; 09 00 03 00
 11:  MOVR R1, R3              ; 0C 01 03 00
 12:  MULI R1, 0x35            ; 08 01 35 00
 13:  ADDI R1, 0x1d            ; 06 01 1D 00
 14:  XORI R1, 0x4a            ; 05 01 4A 00
 15:  ADDR R0, R1              ; 0D 00 01 00
 16:  XORR R0, R2              ; 0B 00 02 00
 17:  STOUT R3, R0              ; 0F 03 00 00
 18:  MOVR R2, R0              ; 0C 02 00 00
 19:  CMPEXP R0, R3            ; 12 00 03 00
 20:  ACCUM                    ; 16 00 00 00
 21:  INC R3                   ; 0E 03 00 00
 22:  CMPR R3, R4              ; 10 03 04 00
 23:  JNZ 0x04                 ; 15 04 00 00
 24:  HALT                     ; 17 00 00 00
```

Register roles:

- `R0` — working byte
- `R1` — key temporary
- `R2` — chaining value, initialised to the IV `0x5C`
- `R3` — loop index `i` (0…23)
- `R4` — length 24

Instructions 5–8 compute the first per-position key, 11–14 the second, and
instructions 19–20 compare the result with the embedded expected byte and count
mismatches.

---

## 6. The algorithm

For input byte `p[i]` and output byte `c[i]`:

```text
k1[i] = ((37*i + 11) & 0xff) ^ 0x6D
k2[i] = ((53*i + 29) & 0xff) ^ 0x4A

t      = rol8(p[i] ^ k1[i], 3)
c[i]   = (t + k2[i]) & 0xff
c[i]  ^= prev

prev   = 0x5C            for i = 0
prev   = c[i-1]          for i > 0
```

This is essentially CBC-style chaining wrapped around a position-dependent
substitution. Note that `prev` is the *previous ciphertext byte*, which is part
of the public expected array — that is what makes the transform invertible
without knowing the plaintext in advance.

---

## 7. The anti-debug trap

There are four independent checks in `anti_debug()`:

```asm
1400034e0  call IsDebuggerPresent
1400034e8  mov rax,QWORD PTR gs:0x60          ; PEB
...
140003743  movzx ecx,BYTE PTR [rax+0x2]       ; PEB!BeingDebugged
140003747  mov   eax,DWORD PTR [rax+0xbc]     ; PEB!NtGlobalFlag
14000374d  and   eax,0x700
140003533  call GetThreadContext              ; Dr0-Dr3 / Dr7 check
140003573  call clock                         ; 4,000,000-iteration churn loop
1400035b1  call clock
```

If any check fires, execution reaches:

```asm
140003739  xor BYTE PTR [rsp+0x52],0x1f
```

`prog` lives at `rsp+0x50`, so this flips `prog[2]` — the `b` operand of the very
first instruction, `LDI R2, 0x5C`. The IV becomes `0x43`. The VM still runs
normally, but the very first output byte is wrong and the chaining propagates the
error through all 24 bytes. Under a debugger the program therefore **always**
says `ACCESS DENIED.`, even with the correct key.

Practical bypasses:

- Patch the `xor ...,0x1f` to NOPs, or set the byte back to `0x5C`.
- Patch `IsDebuggerPresent`'s IAT entry to return 0 and skip the PEB branches.
- Easiest of all: do the analysis statically and never trigger the trap.

---

## 8. Inverting the transform

Because `prev` comes from the expected ciphertext, we can walk forward:

```text
for i in 0..23:
    prev   = IV                 if i == 0 else expected[i-1]
    t      = expected[i] ^ prev
    s      = (t - k2[i]) & 0xff
    u      = ror8(s, 3)
    p[i]   = u ^ k1[i]
```

Applying this to the expected bytes from section 3:

```text
CTF{vm_r3v3rs1ng_1s_fun}
```

A subtlety worth calling out: the natural mistake is to invert the chaining
using the *recovered plaintext* of the previous byte. That does not work — the
VM chains on the transformed output, not on the input.

---

## 9. Automated solving

A solver can do the whole job from the raw PE. It finds the blob by sentinel,
decrypts it, disassembles the 25 instructions, and **extracts the parameters
from the bytecode itself** (the `LDI R2, imm` for the IV, the two
`MULI`/`ADDI`/`XORI` triples for the key schedules, and the `ROL` amount) rather
than hardcoding them. Example output:

```text
[+] blob found at file offset 0x3580
[+] VM program (100 bytes) decrypted
[+] expected ciphertext: DCBC7C25FA6887E51FD5ABB01118C192E02E54596BF74559

[+] recovered parameters
    IV      = 0x5c
    length  = 24
    rotate  = 3
    k1[i]   = ((i*37 + 11) & 0xff) ^ 0x6d
    k2[i]   = ((i*53 + 29) & 0xff) ^ 0x4a
    ...
[+] recovered key: CTF{vm_r3v3rs1ng_1s_fun}
[+] forward verification OK
```

---

## 10. Verification

```text
$ echo CTF{vm_r3v3rs1ng_1s_fun} | vault.exe
+-------------------------------------------+
|  V A U L T   access control  v2.7         |
+-------------------------------------------+
Enter the access key: ACCESS GRANTED.
Flag: CTF{vm_r3v3rs1ng_1s_fun}
```

Feeding the recovered key back through the binary is the final sanity check.
A verification harness can also run negative cases and a solver round-trip.

---

## 11. Notes for challenge authors

- To change the answer, update the generator's `FLAG` constant (it must be
  24 bytes), regenerate the encrypted blob, and rebuild the binary. The
  expected ciphertext and blob are regenerated automatically.
- The anti-debug IV flip is intentionally a *wrong answer* trap rather than a
  crash: a crash tells the player immediately that they found something, while
  a silent denial makes them question their analysis.
- The VM has several unused opcodes (`PUSH`, `POP`, `ROR`, `SWAP`, `NOT`,
  `CMPI`, `JZ`, `JMP`) specifically so that reversing the dispatcher is worth
  doing; they are decoys, not required by the check.
