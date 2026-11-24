# Responsive layout

The kit default for desktop screens (TL-xx) and phone screens (TM-xx). It replaces the "fixed 1920 by
1080 canvas" (decision T1): with *Scale to fit* on, the app only shrinks and clips text; with it off,
every literal `X`/`Width`/`Height` stays put and the content leaves gaps or gets clipped. Source:
GestaoObras, ADR-002 and Wave 7A (2026-11-23/24), Dashboard and Detail screens pasted in HML.

## 1. App settings

- Studio › **Settings › Display**: *Scale to fit*, *Lock aspect ratio* and *Lock orientation*
  **off**.
- Check `App.MinScreenHeight` (an App property, not in the YAML): the height math assumes ≤ 720.
  Above that, the screen gets a scrollbar.
- Range to prove: **1280×720 to 2560×1440**. Below 1280 wide the screen clips instead of breaking.

## 2. Screen

Destination: pasted YAML.

```yaml
Properties:
  Height: =Max(App.Height, App.MinScreenHeight)
  Width: =Max(App.Width, fxLarguraMinDesktop)
```

`fxLarguraMinDesktop = 1280`. A phone screen uses `App.Width` and `App.Height` directly
(`App.MinScreenHeight` can exceed the real height and clip the footer).

## 3. Width

- **No literal `Width`/`X`.** Use `Parent.Width`, a fraction of `Parent.Width` or an `fx*` token.
  A literal that must stay (icon width, button column) gets `# fixo: <reason>` on the line above.
- Table columns as fractions of the gallery, adding up to 1.00; header and cell use the **same** fraction.
- Position chained to the neighbor: `X: ='xx-lbl-a'.X + 'xx-lbl-a'.Width + fxLayoutGutter`.
- `fxIsCompact = App.Width < 1600` is the density breakpoint (margin, font, filter height).

## 4. Height

- The main content (gallery, panel, column, form) **takes the remaining height**:
  `Height: =Parent.Height - Self.Y - fxFooterHeight - fxLayoutGutter`.
- **No child with a fixed `Y + Height` larger than the parent's height at 1280×720.** Studio pushes
  the control up to fit and it covers its neighbor (gallery over the filters and the table header).
- Footer, counter ("Showing x of y") and action bar anchored to the bottom:
  `Y: =Parent.Height - Self.Height - fxLayoutGutter`.
- `Y` chained to the control above: `Y: ='xx-con-kpis'.Y + 'xx-con-kpis'.Height + fxLayoutGutter`.
- Scrolling lives in the gallery (each column has its own), not in a container with `LayoutOverflowY`.
- A computed height that can go negative: `Max(0, …)`.

Tokens (in [`assets/app-formulas-tokens.md`](../assets/app-formulas-tokens.md)):

| Token | Value |
|---|---|
| `fxHeaderHeight` | 64 |
| `fxFooterHeight` | 32 |
| `fxLayoutMargin` | `If(fxIsCompact, 24, 32)` |
| `fxContentTop` | `fxHeaderHeight + fxLayoutGutter` |
| `fxContentHeight` | `Max(App.Height, App.MinScreenHeight) - fxContentTop - fxFooterHeight - fxLayoutGutter` |

Screen header on a single `fxHeaderHeight` row: title, main action and user centered vertically
(`Y: =(Parent.Height - Self.Height) / 2`).

## 5. Filter grid

- Width **per row**: the row without buttons splits the whole width; the row with Filter/Clear/Export
  subtracts the buttons before splitting. Splitting both rows with the same math leaves a gap in one
  and narrow fields in the other.
- A `dd/mm/yyyy` date field needs ~130 px; combo and text ~110 px. Use a single row only when every
  field clears that: 11 fields need `App.Width >= 2200` (at 1600 they would be 60–79 px). Use a token
  (`fxFiltrosUmaLinha`), not `fxIsCompact`.
- Short label ("Number", not "Request number"): the filter label is a single 20 px line.
- `Classic/DatePicker` **does not accept `Size`** (PA2108); width is the only knob.

## 6. Text that can grow

- One label, one piece of data. Two values in a one-line label (amount + nature) wrap and get
  clipped; move the second to the caption above or to another control.
- Long text in a cell, card or timeline: **2 lines with an ellipsis** and the full text in the
  `Tooltip`. Formula in a `|-` block (`{t: …}` on a plain line breaks the YAML):

Destination: pasted YAML.

```yaml
Text: |-
  =With({t: ThisItem.Texto, n: RoundDown((Parent.TemplateWidth - fxCardPaddingS * 2) / 6.5, 0) * 2 - 2}, If(Len(t) > n, Left(t, n) & "…", t))
Tooltip: =ThisItem.Texto
```

  6.5 px per character holds for font size 11; label height ≥ 38 for 2 lines. A flexible gallery with
  `AutoHeight` is the alternative `[not verified]`.
- `dd/mm/yyyy` date column at font size 11: ≥ 70 px (16% of a ~500 px table).
- Currency in the Brazilian format: `"R$ " & Text(x, "[$-en-US]#,##0.00", "pt-BR")`. With only
  `[$-pt-BR]` in the format, an English browser shows `12,750.00`.

## 7. Modal, menu and table header

- Modal centered on `Parent` with a cap: `Height: =Min(640, Parent.Height - fxLayoutMargin * 2)`.
- The collapsed menu shows an **icon** (`Classic/Icon`) + the button's `Tooltip`, never `Left(name, 1)`.
- Table header rounded only at the ends: see
  [table-gallery](../assets/components/table-gallery.md).

## 8. Phone (TM-xx)

Its own screen, one column, `Parent.Width`, `fxMobile*` tokens, touch target ≥ 44 px. A desktop
screen does not become a phone screen through responsiveness; an action that does not exist on the
phone shows "Available on the computer".

## 9. Before pasting

1. A table per screen with the `Y + Height` of the last child at **1280×720 and 2560×1440**, computed
   from the tokens, ≤ usable height.
2. `validar-telas.py` → `0 erro(s)`.
3. Paste the **whole** screen (a control that changed container is duplicated by a partial block).
4. Do not drag or resize in Studio: it replaces the `X`, `Y`, `Width` and `Height` formulas with
   constants ([Create responsive layout](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/create-responsive-layout)).
