# Text

Script-aware text processing, built on `unicode`. Pure Dolet, so every rule in
it is testable without a font, a window, or a display server.

## Arabic shaping

Arabic letters change shape according to what they connect to. The same letter
has up to four forms, and text that ignores this renders as disconnected
stumps a reader has to decipher rather than read.

```dolet
count: i32 = utf8_decode_all("مرحبا", source, 32)
shaped: i32 = arabic_shape(source, count, out, 32)
```

or straight between strings:

```dolet
written: i64 = arabic_shape_str("مرحبا", buffer, 256)
```

The rule, once joining types are known, is small: a letter joins backwards
when the letter before it can join forwards, and forwards when the letter
after it can join backwards. Joining both ways gives the medial form, one way
gives initial or final, neither gives isolated.

Two cases make it more than that rule:

- **Marks are transparent.** A fatha between two letters sits on the first
  without breaking its connection to the second, so the search for a
  neighbour skips it.
- **Lam followed by an alef is one ligature**, not two letters side by side,
  and Unicode gives it its own codepoints. This is why the output can be
  shorter than the input.

Output is in the Arabic Presentation Forms blocks, which a font's character
map can be asked for directly without a font shaping engine.

## Not yet here

Bidirectional ordering (UAX #9). Shaping decides what each letter looks like;
bidi decides what order the letters are drawn in. Arabic mixed with Latin or
with numbers needs both.
