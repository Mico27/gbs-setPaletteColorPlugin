# gbs-setPaletteColorPlugin

**Version 4.3.0 — Requires GB Studio ≥ 4.3.0**

A GB Studio engine plugin that provides fine-grained runtime control over palette colours. Scripts can read and write individual colour slots for both background and sprite palettes on Game Boy Color, and the equivalent shade indices on original Game Boy hardware, all from script variables.

It also adds extended versions of the standard Set Background/Sprite Palette events with a commit toggle, and a utility event that copies all background palette data from any other scene into the current palettes.

![image](https://github.com/user-attachments/assets/83791fcc-9e21-405f-a8fa-e40c9acb1203)

![image](https://github.com/user-attachments/assets/c9c642bf-7375-45bc-8fb7-da3728da5ef6)

---

## Table of Contents

1. [Concepts](#concepts)
2. [Project Setup](#project-setup)
3. [Size Limits and Restrictions](#size-limits-and-restrictions)
4. [Events Reference](#events-reference)
5. [Memory Footprint](#memory-footprint)

---

## Concepts

### Game Boy Color 15-bit RGB

On Game Boy Color, each colour is a 15-bit value: three channels of five bits each, in the range 0–31.

$$\text{colour} = R + (G \times 32) + (B \times 1024)$$

| Channel | Range |
|---|---|
| Red | 0–31 |
| Green | 0–31 |
| Blue | 0–31 |

This is the format the colour events use when **Is gameboy color palette** is checked.

### DMG shade indices

On original Game Boy hardware, each palette entry is a 2-bit shade index instead of an RGB value:

| Value | Shade |
|---|---|
| 0 | White |
| 1 | Light grey / light green |
| 2 | Dark grey / dark green |
| 3 | Black |

The background palette has 4 shade slots. Each sprite palette has 3 usable shade slots, because colour 0 is always transparent.

### Commit vs. deferred

Every palette-writing event in this plugin has a **Commit** checkbox:

- **Checked (default)** — the new colours are sent to the hardware immediately and are visible within the same frame.
- **Unchecked** — the palettes are updated in memory but nothing is written to hardware. The change takes effect the next time hardware is updated, either by a later committed call or by the fade manager at the end of a fade-in.

Uncheck **Commit** when changing palette colours during a fade-in, to avoid fighting the fade, or to batch several changes and push them all at once.

![image](https://github.com/user-attachments/assets/f7b89f5a-2762-43d1-b5c0-dc93d9413abf)

---

## Project Setup

1. Copy the plugin folder into your GB Studio project's `plugins/` directory.
2. No additional configuration or engine fields are required — all five events become available immediately.

![image](https://github.com/user-attachments/assets/7e715edd-74ae-4643-a3f4-7641b267d8e8)

![image](https://github.com/user-attachments/assets/73c9529b-b413-49ca-a3dd-12371873f2e3)

---

## Size Limits and Restrictions

### Colour writing needs Game Boy Color

Reading and writing full RGB colours only applies to Game Boy Color palettes. On a DMG build, or with **Is gameboy color palette** unchecked, the events read and write DMG shade indices instead.

### Sprite colour 0 is transparent and not writable

For Game Boy Color sprite palettes, colour index 0 is the hardware transparency colour and cannot be set. **Set colors of a palette** therefore exposes only three colour fields for sprite palettes; the fourth is hidden.

### Palette index range

| Mode | Palette index |
|---|---|
| Color background | 0–7 |
| Color sprite | 0–7 |
| DMG background | 0 (single palette) |
| DMG sprite | 0–1 |

### Copy scene palette colors copies background only

**Copy scene palette colors** reads background palette data from the target scene. It does not copy sprite palettes, and the source scene is chosen when the project is built.

### The EX events use design-time palettes only

**Set Background Palette EX** and **Set Sprite Palette EX** select palettes from the project's palette list; they cannot take runtime variables for individual colour channels. For dynamic colour control use **Set colors of a palette**.

### No engine files modified

The plugin only adds a new engine source file, so it has no compatibility conflicts with other engine plugins.

---

## Events Reference

---

### Set colors of a palette

**`EVENT_SET_PALETTE_COLORS`** — group: **Color**

Sets the colour values of a single palette slot at runtime. Colours are 15-bit RGB values on Game Boy Color, or 2-bit shade indices on DMG.

| Field | Default | Description |
|---|---|---|
| Is gameboy color palette | ✓ | When checked, colours are 15-bit Game Boy Color RGB values. When unchecked, they are DMG shade indices (0–3). |
| Is sprite palette | ✗ | When checked, targets a sprite palette slot instead of a background one. |
| Palette | 0 | Index of the palette slot to write. 0–7 for Color, 0 for DMG background, 0–1 for DMG sprites. |
| Color 1 | 0 | First colour. For sprite palettes this is the first *visible* colour, since colour 0 is transparent. |
| Color 2 | 0 | Second colour. |
| Color 3 | 0 | Third colour. |
| Color 4 | 0 | Fourth colour. Background palettes only — hidden for sprite targets. |
| Commit | ✓ | When checked, pushes the change to hardware immediately. Uncheck during fade-ins. |

---

### Get colors of a palette

**`EVENT_GET_PALETTE_COLORS`** — group: **Color**

Reads the current colour values of a palette slot into script variables, in the same format as **Set colors of a palette**.

| Field | Default | Description |
|---|---|---|
| Is gameboy color palette | ✓ | When checked, reads 15-bit Game Boy Color values; otherwise DMG shade indices. |
| Is sprite palette | ✗ | When checked, reads from a sprite palette slot. |
| Palette | 0 | Index of the palette slot to read. |
| Color 1 | — | Variable that receives the first colour. |
| Color 2 | — | Variable that receives the second colour. |
| Color 3 | — | Variable that receives the third colour. |
| Color 4 | — | Variable that receives the fourth colour. Background palettes only. |

---

### Copy scene palette colors

**`EVENT_COPY_BKG_COLORS_TO_BKG`** — group: **Screen**

Reads all 8 background palette entries from another scene and loads them into the current palettes, optionally committing them to hardware. Useful for applying another scene's background palette without a scene change — for example when using the SubmappingExPlugin to display tiles from another scene.

| Field | Default | Description |
|---|---|---|
| Scene | Last scene | The source scene whose background palette data is copied. Chosen when the project is built. |
| Commit | ✓ | When checked, writes all 8 copied palette entries to hardware immediately. |

Only background palettes are copied; sprite palettes are unaffected. On Super Game Boy hardware, SGB palette transfers are also triggered.

---

### Set Background Palette EX

**`EVENT_PALETTE_SET_BACKGROUND_EX`** — group: **Color**

The standard *Set Background Palette* event plus a **Commit** checkbox, so the palette write can be deferred — during a fade-in, for example.

| Field | Default | Description |
|---|---|---|
| Palettes 0–7 | keep | The palette to apply to each of the 8 background palette slots. *keep* leaves the slot unchanged; *restore* resets it to the current scene's defined value. |
| Commit | ✓ | When checked, pushes the palette to hardware immediately. Uncheck during fade-ins. |

---

### Set Sprite Palette EX

**`EVENT_PALETTE_SET_SPRITE_EX`** — group: **Color**

The same as **Set Background Palette EX**, but targeting the 8 sprite palette slots.

| Field | Default | Description |
|---|---|---|
| Palettes 0–7 | keep | The palette to apply to each of the 8 sprite palette slots. *keep* leaves the slot unchanged; *restore* resets it to the scene's defined value. |
| Commit | ✓ | When checked, pushes the palette to hardware immediately. Uncheck during fade-ins. |

---

## Memory Footprint

Measured against the stock GB Studio **4.3.0-e1** engine (per-file SDCC compile with GB Studio's build flags, default engine settings). Values are the plugin's *delta* versus the stock engine; DMG build, with CGB noted where it differs. ROM cost lands in banked ROM (GB Studio's autobanker spreads it across switchable banks); using the plugin's events additionally compiles a few bytes of GBVM script per call into your project's script banks.

| | Cost |
|---|---|
| WRAM | +0 bytes |
| ROM | +1,341 bytes (DMG) / +1,451 bytes (CGB) |

- **WRAM:** no change — the plugin works directly on the engine's existing palette buffers.
- **Engine WRAM headroom:** the stock GB Studio 4.3.0 engine leaves about **854 bytes** of WRAM free (usable engine WRAM is 7,776 bytes at 0xC0A0–0xDF00; the stock engine uses 6,922 bytes). With this plugin installed roughly **854 bytes** remain. This figure does not depend on how many global variables your project defines: the script memory array has a fixed size of VM_HEAP_SIZE + (VM_MAX_CONTEXTS × VM_CONTEXT_STACK_SIZE) words — 768 + 16 × 64 = 1,792 words (3,584 bytes) with stock engine settings.
- **SRAM:** not used.

---

<!-- BANK0:BEGIN -->
## Bank 0 (HOME) Usage

Bank 0 is the 16 KB non-switchable ROM bank that the GB Studio engine core,
the interrupt handlers and the GBDK runtime all share. Banked ROM is cheap
(add another bank), bank 0 is not, so it is usually the first thing a project
runs out of.

| | Bytes |
|---|---|
| Bank 0 used by this plugin | **0** |
| Bank 0 free with this plugin installed | **1,451** of 16,384 (91% used) |

**This plugin costs nothing in bank 0.** All of its code lives in a switchable
ROM bank; nothing it adds is resident in bank 0.

<details><summary>How this was measured</summary>

GB Studio 4.3.2, DMG target, default engine settings. Each module's bank 0
contribution is the `A _HOME size` record that SDCC writes into its `.rel`
object, summed over the engine sources this plugin provides. Stock sizes come
from building projects whose only plugin ships no engine C, so every module in
them is the untouched engine; two such builds were compared and agreed on all
73 shared modules.

The "free" figure is a stock project with this plugin and nothing else. Your
own number will differ: other plugins, and any engine settings that change what
the core compiles, move it independently of this plugin.

</details>
<!-- BANK0:END -->

## Changelog

Grouped by the date each change was merged into the official
[gb-studio-plugins](https://github.com/gb-studio-dev/gb-studio-plugins) repository.

Only bug fixes, new features and feature changes are listed. Engine version
bumps, patch regeneration, packaging fixes and documentation edits are omitted.

### 2026-06-14

- Added custom script parameter / stack support to the events.

### 2025-10-29

- Fixed an unreferenced `DMG_PALETTE` and an inverted commit flag.

### 2025-06-02

- Fixed the palette update commit using `stackpush` instead of `stackpushconst`.

### 2025-04-23

- The palette index is now ignored for DMG background palettes.

### 2025-04-02

- Added a commit option to palette colour changes, and fixed an inverted parameter.
- Fixed sprite colour changes.
- Added an event to copy another scene's palette colour data.
- Added a "get palette colour" event.

### 2025-02-24

- Initial release.
