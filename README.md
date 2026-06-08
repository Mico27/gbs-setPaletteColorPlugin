# gbs-setPaletteColorPlugin

**Version 4.3.0 — Requires GB Studio ≥ 4.3.0**

A GB Studio engine plugin that provides fine-grained runtime control over palette colors. It lets scripts read and write individual color slots for both background and sprite palettes on CGB hardware, and the equivalent shade indices on DMG hardware, all at runtime via script variables. It also adds extended versions of the standard Set Background/Sprite Palette events with a commit toggle, and a utility event that copies all background palette data from any other scene into the current runtime palette buffers.

![image](https://github.com/user-attachments/assets/83791fcc-9e21-405f-a8fa-e40c9acb1203)

![image](https://github.com/user-attachments/assets/c9c642bf-7375-45bc-8fb7-da3728da5ef6)

![image](https://github.com/user-attachments/assets/f7b89f5a-2762-43d1-b5c0-dc93d9413abf)

![image](https://github.com/user-attachments/assets/7e715edd-74ae-4643-a3f4-7641b267d8e8)

![image](https://github.com/user-attachments/assets/73c9529b-b413-49ca-a3dd-12371873f2e3)

---

## Table of Contents

1. [Concepts](#concepts)
2. [Project Setup](#project-setup)
3. [Technicalities and Restrictions](#technicalities-and-restrictions)
4. [Events Reference](#events-reference)
5. [Inner Workings](#inner-workings)

---

## Concepts

### CGB 15-bit RGB Color Format

On Game Boy Color, each color is stored as a 15-bit value packed into two bytes. The three channels each occupy 5 bits, giving a range of 0–31:

$$\text{color} = R + (G \times 32) + (B \times 1024)$$

| Channel | Bits | Range |
|---|---|---|
| Red   | 0–4  | 0–31 |
| Green | 5–9  | 0–31 |
| Blue  | 10–14 | 0–31 |

This is the format expected by the **Set colors of a palette** and returned by **Get colors of a palette** when **Is gameboy color palette** is checked.

### DMG Shade Indices

On original Game Boy hardware, each palette entry is a 2-bit shade index rather than an RGB value:

| Value | Shade |
|---|---|
| 0 | White |
| 1 | Light grey / light green |
| 2 | Dark grey / dark green |
| 3 | Black |

The background palette has 4 shade slots (indices 0–3). Each sprite palette (OBP0, OBP1) has 3 usable shade slots (indices 0–2, corresponding to hardware color positions 1–3; color 0 is always transparent).

### Commit vs. Deferred

Every palette-writing event in this plugin has a **Commit** checkbox:

- **Checked (default):** the new color values are immediately sent to the hardware palette registers (`set_bkg_palette`, `set_sprite_palette`, `BGP_REG`, `OBP0_REG`, `OBP1_REG`). The change is visible on-screen within the same frame.
- **Unchecked:** the in-memory palette buffers (`BkgPalette`, `SprPalette`, `DMG_palette`) are updated, but nothing is written to hardware. The change will take effect the next time hardware is updated — either by a later committed call or by the fade manager writing the palette buffers to hardware at the end of a fade-in.

Uncheck **Commit** when changing palette colors during a fade-in to avoid conflicting with the fade manager, or to batch several changes before pushing them to hardware in one operation.

---

## Project Setup

1. Copy the plugin folder into your GB Studio project's `plugins/` directory.
2. No additional configuration or engine fields are required — all five events become available immediately.

---

## Technicalities and Restrictions

### CGB Only for Color Writing

The `set_palette_colors` and `get_palette_colors` native functions read and write the `BkgPalette` / `SprPalette` arrays, which are CGB hardware structures. On a DMG build, or when **Is gameboy color palette** is unchecked, only the `DMG_palette` array is read or written. Committing on a non-CGB build writes the appropriate `BGP_REG` / `OBP0_REG` / `OBP1_REG` register.

### Sprite Color 0 Is Transparent and Not Writable

For CGB sprite palettes, color index 0 is the hardware transparency color and cannot be set. The **Set colors of a palette** event therefore exposes only 3 color fields (Color 1–3) for sprite palettes, corresponding to `SprPalette[idx].c1`, `.c2`, `.c3`. Color 4 is hidden for sprite targets.

### Palette Index Range

| Mode | Max palette index |
|---|---|
| CGB background | 0–7 |
| CGB sprite | 0–7 |
| DMG background | 0 (single palette) |
| DMG sprite | 0–1 (OBP0, OBP1) |

### Copy Scene Palette Colors Copies Background Only

**Copy scene palette colors** reads background palette data from the target scene's ROM definition. It does not copy sprite palettes. The scene symbol reference is resolved at compile time; the target scene must exist in the project.

### Set Background/Sprite Palette EX — Compile-Time Colors Only

**Set Background Palette EX** and **Set Sprite Palette EX** select palette data from the project's design-time palette list. They cannot accept runtime variable values for individual color channels. For runtime dynamic color control use **Set colors of a palette** instead.

### No New Engine Files Modified

This plugin does not patch any existing GB Studio engine files. It only adds a new engine source file (`manage_palette_colors.c`) that compiles alongside the engine.

---

## Events Reference

### Set Colors of a Palette

**Event ID:** `EVENT_SET_PALETTE_COLORS`  
**Group:** Color

Sets the four (or three, for sprite palettes) color values of a single palette slot at runtime. Colors are either 15-bit RGB values (CGB) or 2-bit shade indices (DMG).

| Field | Type | Default | Description |
|---|---|---|---|
| Is gameboy color palette | Checkbox | ✓ | When checked, colors are 15-bit CGB RGB values. When unchecked, colors are 2-bit DMG shade indices (0–3). |
| Is sprite palette | Checkbox | ✗ | When checked, targets a sprite palette slot. When unchecked, targets a background palette slot. |
| Palette | Value expression | 0 | Index of the palette slot to write. Range 0–7 for CGB, 0 for DMG background, 0–1 for DMG sprites. |
| Color 1 | Value expression | 0 | First color. For CGB background: `c0`. For CGB sprite: `c1` (color 0 is always transparent). For DMG: shade index for position 0 (background) or position 1 (sprite). |
| Color 2 | Value expression | 0 | Second color. |
| Color 3 | Value expression | 0 | Third color. |
| Color 4 | Value expression | 0 | Fourth color. Background palettes only — hidden for sprite targets. |
| Commit | Checkbox | ✓ | When checked, immediately pushes changes to hardware registers. Uncheck during fade-ins. |

---

### Get Colors of a Palette

**Event ID:** `EVENT_GET_PALETTE_COLORS`  
**Group:** Color

Reads the current in-memory color values of a palette slot into script variables. The values match whatever was last committed or loaded from the scene, in the same format as **Set colors of a palette**.

| Field | Type | Default | Description |
|---|---|---|---|
| Is gameboy color palette | Checkbox | ✓ | When checked, reads 15-bit CGB values. When unchecked, reads 2-bit DMG shade indices. |
| Is sprite palette | Checkbox | ✗ | When checked, reads from a sprite palette slot. |
| Palette | Value expression | 0 | Index of the palette slot to read. |
| Color 1 | Variable | — | Variable to store the first color value. |
| Color 2 | Variable | — | Variable to store the second color value. |
| Color 3 | Variable | — | Variable to store the third color value. |
| Color 4 | Variable | — | Variable to store the fourth color value. Background palettes only — not present for sprite targets. |

---

### Copy Scene Palette Colors

**Event ID:** `EVENT_COPY_BKG_COLORS_TO_BKG`  
**Group:** Screen

Reads all 8 background palette entries directly from another scene's ROM data and loads them into the current runtime `BkgPalette` buffers, optionally committing them to hardware. Useful for applying the background palette of a different scene without triggering a full scene change — for example, when using the SubmappingExPlugin to display tiles from another scene.

| Field | Type | Default | Description |
|---|---|---|---|
| Scene | Scene picker | Last scene | The source scene whose background palette data will be copied. Resolved at compile time. |
| Commit | Checkbox | ✓ | When checked, immediately writes all 8 copied palette entries to the CGB hardware registers. |

**Notes:**
- Only background palettes are copied. Sprite palettes are not affected.
- On SGB hardware, SGB palette transfers are also triggered.
- The target scene's palette data is read directly from ROM using `MemcpyBanked` — no scene change occurs.

---

### Set Background Palette EX

**Event ID:** `EVENT_PALETTE_SET_BACKGROUND_EX`  
**Group:** Color

Extended version of the standard GB Studio "Set Background Palette" event. Identical in functionality but adds a **Commit** checkbox, allowing the palette write to be deferred (e.g. during a fade-in transition). Supports "keep" (leave the slot unchanged) and "restore" (reset to the scene's defined palette) per slot.

| Field | Type | Default | Description |
|---|---|---|---|
| Palettes 0–7 | Palette picker (×8) | keep | The palette to apply to each of the 8 background palette slots. "keep" leaves the slot unchanged; "restore" resets it to the current scene's defined value. |
| Commit | Checkbox | ✓ | When checked, the palette is pushed to hardware immediately. Uncheck during fade-ins. |

---

### Set Sprite Palette EX

**Event ID:** `EVENT_PALETTE_SET_SPRITE_EX`  
**Group:** Color

Extended version of the standard GB Studio "Set Sprite Palette" event. Identical to **Set Background Palette EX** but targets the 8 sprite palette slots. "restore" resets the slot to the scene's defined sprite palette value.

| Field | Type | Default | Description |
|---|---|---|---|
| Palettes 0–7 | Palette picker (×8) | keep | The palette to apply to each of the 8 sprite palette slots. "keep" leaves the slot unchanged; "restore" resets to the scene's defined value. |
| Commit | Checkbox | ✓ | When checked, the palette is pushed to hardware immediately. Uncheck during fade-ins. |

---

## Inner Workings

### New Engine File: `manage_palette_colors.c`

All runtime logic lives in a single new engine source file. It provides three native VM functions and an internal palette-loading helper used by **Copy scene palette colors**.

### Palette Index Packing

Both `set_palette_colors` and `get_palette_colors` receive a single packed `palettes` integer that encodes three fields:

```
Bits 0-2 : palette slot index (0–7)
Bit  3   : is_sprite  (1 = sprite palette, 0 = background palette)
Bit  4   : is_dmg     (1 = DMG mode, 0 = CGB mode)
```

This packing is performed on the JS side using an RPN expression:

```js
_rpn()
  .ref(tmp_palette_idx)          // base index
  .int16(is_sprite ? 8 : 0)      // OR bit 3 if sprite
  .operator(".B_OR")
  .int16(!is_gbc ? 16 : 0)       // OR bit 4 if DMG
  .operator(".B_OR")
  .refSet(tmp_palette_idx)
  .stop();
```

### `set_palette_colors`

```c
void set_palette_colors(SCRIPT_CTX * THIS) OLDCALL BANKED {
    int16_t palettes  = *(int16_t*)VM_REF_TO_PTR(FN_ARG0);  // packed index
    int16_t color0    = *(int16_t*)VM_REF_TO_PTR(FN_ARG1);
    int16_t color1    = *(int16_t*)VM_REF_TO_PTR(FN_ARG2);
    int16_t color2    = *(int16_t*)VM_REF_TO_PTR(FN_ARG3);
    int16_t color3    = *(int16_t*)VM_REF_TO_PTR(FN_ARG4);
    int16_t is_commit = *(int16_t*)VM_REF_TO_PTR(FN_ARG5);

    UBYTE palette_from_idx = palettes & 7;
    UBYTE is_sprite        = (palettes >> 3) & 1;
    UBYTE is_dmg           = (palettes >> 4) & 1;
    ...
}
```

**DMG path:** Packs the shade indices into a `DMG_PALETTE()` macro byte and writes `DMG_palette[0]` (background), `DMG_palette[1]` (OBP0), or `DMG_palette[2]` (OBP1). If committed, the corresponding hardware register (`BGP_REG`, `OBP0_REG`, `OBP1_REG`) is written directly.

**CGB path:** Writes directly into the appropriate `BkgPalette[idx]` or `SprPalette[idx]` struct fields. For background: all four `.c0`–`.c3` slots are written. For sprites: only `.c1`–`.c3` are written (`.c0` is transparency and is left untouched). If `is_commit` is set and the build is CGB, `set_bkg_palette` or `set_sprite_palette` is called to flush one palette entry to hardware.

### `get_palette_colors`

The read counterpart. It decodes the same packed `palettes` argument to select the source array, then writes the color values into `script_memory` at the variable indices passed as arguments:

```c
script_memory[color0Var] = BkgPalette[palette_from_idx].c0;
script_memory[color1Var] = BkgPalette[palette_from_idx].c1;
...
```

For DMG, the shade indices are extracted by shifting and masking the packed `DMG_palette` byte. There is no commit argument — reading never writes to hardware.

Note that the variable indices are passed as literal constant stack values (`_stackPushConst(color0Alias)`) rather than pushed variable references, because the native function needs the index of the variable to write to, not the variable's current value.

### `copy_scene_palette_colors` and the Internal `load_bkg_palette`

The file re-implements `load_bkg_palette` as a private `inline` function (matching the engine's own implementation) so it can be called without depending on it being exported:

```c
inline void load_bkg_palette(const palette_t * palette, UBYTE bank) {
    palette_entry_t * dest = BkgPalette;
    UBYTE mask = ReadBankedUBYTE(&palette->mask, bank);
    palette_entry_t * sour = palette->cgb_palette;
    for (UBYTE i = mask; (i); i >>= 1, dest++) {
        if ((i & 1) == 0) continue;
        MemcpyBanked(dest, sour, sizeof(palette_entry_t), bank);
        sour++;
    }
    DMG_palette[0] = ReadBankedUBYTE(palette->palette, bank);
    // SGB transfer if applicable ...
}
```

`copy_scene_palette_colors` reads the `scene_t` struct from the target scene's ROM bank, extracts the `palette` far pointer, passes it to `load_bkg_palette`, then optionally flushes all 8 entries to hardware with `set_bkg_palette(0, 8, BkgPalette)`.

Note that the **Commit** field is inverted in the JS compile function: `_stackPushConst(input.commit ? 0 : 1)` — a `0` is passed to the native function when commit is **checked**. The C code interprets this as `is_commit = 0 means do commit`, which is the opposite convention of the other events. The end result is the same: committing when the box is checked.

### Set Background/Sprite Palette EX — Compile-time Only

These two events do not call any native function from this plugin. They use the standard GB Studio `_paletteLoad` / `_paletteColor` compile helpers to generate the same bytecode as the built-in palette events. The only difference in behavior is the `commit` field being passed into `_paletteLoad`, allowing the hardware write step to be deferred.
