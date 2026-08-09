# JP Card Trailer (bytes 0x43/0x44) — checksum algorithm PINNED from the ROM

**Status: SOLVED, conf 1.0.** Reproduces bytes 0x43/0x44 on 6/6 real cards, byte-exact.
The load path recomputes and **rejects** the card on mismatch, so a created card MUST
carry a correct trailer.

ROM: `epr-22284a.ic22` (JP DOC 2000, CRC32 1E8E067C). RAM base 0x0C020000.
SH-4 runs **little-endian** here (docre disassembles LE; literals are LE too).

---

## The algorithm

It is a **CRC-16/CCITT (poly 0x1021)** run inside the **high 16 bits of a 32-bit register**,
with a nonstandard init and a message-feed that pushes each byte in through the low byte.
Not reducible to a plain 16-bit CRC (the low half of the register carries state), so the
faithful 32-bit emulation below is the reference.

| Parameter | Value | Source |
|---|---|---|
| Polynomial | `0x10210000` (= 0x1021 << 16) | literal @0xC0750E8, used by `xor r2,r4` |
| MSB test mask | `0x80000000` (bit 31) | literal @0xC0750E4, used by `and r4,r5` |
| Low-byte-clear mask | `0xFFFFFF00` (mov.w 0xFF00, sign-extended) | literal @0xC0751E8, used by `and r9,r14` |
| Init | `0xDEBDEB00` | literal @0xC0751EC, loaded into r14 |
| Input range | card **0x00..0x42** = **67 bytes** (r5 = 67 = 0x43) | wrapper @0xC075142 / writer @0xC0765AA |
| Bit direction | MSB-first (tests bit 31, shifts left) | inner @0xC0750C0 |
| Per-byte | clear low 8 bits, OR the byte in, run 8 shift/xor steps | main loop @0xC075106 |
| Final flush | **8 extra shift/xor steps** (no byte) — ESSENTIAL (0/6 without it) | loop @0xC075120 |
| Result | register **>> 16** (high 16 bits), `shlr16 r0` | epilogue @0xC075132 |
| Write order | **little-endian**: card[0x43] = cksum & 0xFF (low), card[0x44] = cksum >> 8 (high) | writer @0xC0765B4/BC |

### Disassembly — inner processor 0xC0750C0 (one bit)
```
mov.l 0xc0750e4,r5   ; r5 = 0x80000000
and   r4,r5          ; test bit 31 of r4
tst   r5,r5
bt/s  0xc0750ce
add   r4,r4          ; r4 <<= 1   (delay slot, always)
mov.l 0xc0750e8,r2   ; r2 = 0x10210000
xor   r2,r4          ; r4 ^= poly (only if bit31 was set)
rts
mov   r4,r0          ; return r0 = r4
```

### Disassembly — checksum body 0xC0750EC
```
mov r5,r11           ; r11 = length (67)
mov r4,r10           ; r10 = buffer ptr
mov.w 0xc0751e8,r9   ; r9 = 0xFFFFFF00  (0xFF00 sign-extended)
mov.l 0xc0751ec,r14  ; r14 = 0xDEBDEB00 (init)
mov #8,r12
loop_byte:
  mov.b @r10+,r3     ; next byte
  and  r9,r14        ; clear low 8 bits of CRC
  mov  r12,r13       ; bit counter = 8
  extu.b r3,r3
  or   r3,r14        ; OR byte into low 8 bits
  loop_bit:
    bsr 0xc0750c0    ; one shift/xor step, r4=r14
    mov r14,r4
    dt  r13
    bf/s loop_bit
    mov r0,r14
  add #-1,r11
  tst r11,r11; bf loop_byte
; final 8-bit flush:
  mov r12,r13
  flush:
    bsr 0xc0750c0
    mov r14,r4
    dt r13; bf/s flush; mov r0,r14
mov r14,r0
shlr16 r0            ; result = r14 >> 16
rts
```

---

## Reference implementation (Python)

```python
def jp_card_trailer(card):
    """Compute the 2-byte trailer for a JP DOC 2000 card (derbyo2k).
    card: bytes-like, first 67 bytes (0x00..0x42) are the input.
    Returns (byte_0x43, byte_0x44) to store little-endian."""
    POLY = 0x10210000
    crc  = 0xDEBDEB00           # init
    for byte in card[0x00:0x43]:      # 67 bytes
        crc = (crc & 0xFFFFFF00) | byte
        for _ in range(8):
            hit = crc & 0x80000000
            crc = (crc << 1) & 0xFFFFFFFF
            if hit:
                crc ^= POLY
    for _ in range(8):                # final flush
        hit = crc & 0x80000000
        crc = (crc << 1) & 0xFFFFFFFF
        if hit:
            crc ^= POLY
    cksum = (crc >> 16) & 0xFFFF
    return (cksum & 0xFF, (cksum >> 8) & 0xFF)   # card[0x43], card[0x44]
```

To CREATE/WRITE a card: build bytes 0x00..0x42, then set
`card[0x43], card[0x44] = jp_card_trailer(card)`.

---

## Validation table (6/6 real cards from jp_full_after.json)

| Horse | card[0x43] card[0x44] | computed (LE) | match |
|---|---|---|---|
| ラヘェロポ         | b9 cb | b9 cb | ✓ |
| キキーウイロー     | 3e 9e | 3e 9e | ✓ |
| ウィロー           | 07 25 | 07 25 | ✓ |
| ァロロァ           | f1 a9 | f1 a9 | ✓ |
| アアナナナポ       | dd 28 | dd 28 | ✓ |
| タカネハッハーーー | 91 0d | 91 0d | ✓ |

(The prior "CRC-CCITT over 0x00..0x42, init 0/0xFFFF" claim failed because it (a) used a
plain 16-bit register, (b) used init 0x0000/0xFFFF not 0xDEBDEB00, (c) omitted the final
8-bit flush, and (d) fed the byte into the high byte instead of the low byte. All four
differences matter.)

---

## Does the game validate the trailer on load? YES — it rejects a bad trailer.

Two ROM sites:

**Writer / serialize** — `0xC0765A2` region:
```
mov.l 0xc076798,r2   ; r2 = &checksum (0x0C0750EC)
mov.l 0xc076794,r4   ; r4 = card buffer base
jsr  @r2             ; checksum(buf, 67)
mov #67,r5
...
mov #67,r0; mov.b r4,@(r0,r5)   ; card[0x43] = cksum & 0xFF   (low)
extu.w r4,r4
mov #68,r0; shlr8 r4; mov.b r4,@(r0,r5) ; card[0x44] = cksum>>8 (high)
```

**Loader / validate** — wrapper `0xC075142`, called from `0xC075EB4`:
```
0xC075142: reads stored = card[0x43] | (card[0x44] << 8)   ; little-endian
           bsr 0xc0750ec (recompute over 67 bytes)
           extu.w both; cmp/eq computed,stored
           bf -> return 0 (INVALID) ; else return 1 (VALID)

0xC075EB4: bsr 0xc075142
           tst r0,r0
           bf 0xc075ec0     ; r0 != 0 (VALID) -> continue normal load
           bra 0xc076022    ; r0 == 0 (INVALID) -> reject/error path (stores 0 flag)
```

So the load path recomputes the checksum and branches to the reject path on mismatch. A
created/edited JP card **must** carry the correct trailer or this handler rejects it.

(Note: this resolves §7's earlier "for EDIT recomputing may be unnecessary" hedge — for the
codec path exercised here, recomputation IS required.)
