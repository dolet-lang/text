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

- **Marks are transparent, and shaped.** A fatha between two letters sits on
  the first without breaking its connection to the second, so the search for a
  neighbour skips it — and the mark itself takes its own presentation form
  from FE70..FE7F, because only that form is in a font's character map where
  a renderer can find it.
- **Lam followed by an alef is one ligature**, not two letters side by side,
  and Unicode gives it its own codepoints. This is why the output can be
  shorter than the input.

Output is in the Arabic Presentation Forms blocks, which a font's character
map can be asked for directly without a font shaping engine.

## Bidirectional ordering

Shaping decides what each letter looks like. Bidi decides what order they are
drawn in, which is a separate problem: `"ب 12"` stores the digits after the
Arabic and draws them before it, still reading `12` and not `21`.

```dolet
written: i64 = text_display_str("مرحبا abc", buffer, 256)
```

`text_display_order` is the whole pipeline: shape in logical order, because
joining depends on logical neighbours, then reorder for display. A lam-alef
ligature shortens the text, so each output glyph carries the level of the
character it came from and the ligature follows its lam.

Underneath, `bidi_paragraph_level`, `bidi_resolve` and `bidi_reorder` are
available separately for a caller that wants the levels themselves.

This is UAX #9 for one paragraph treated as a single run: the weak rules
(W1-W7), the neutral rules (N1-N2), the implicit levels (I1-I2) and the
reordering rules (L1-L2).

**Deliberately absent**, and worth knowing before relying on it:

- Explicit embedding and override codes (the X rules). Those codepoints are
  classed BN and drop out as neutrals rather than changing levels.
- Paired-bracket resolution (N0). A bracket around mixed-direction text takes
  the surrounding direction rather than matching its pair.

Both are rare in ordinary text, and neither is guessed at: what is not
implemented is not claimed.

## Not yet here

Line breaking (UAX #14), and grapheme cluster boundaries.
