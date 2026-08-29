# gbs-setPaletteColorPlugin

**Version 4.3.0. Requires GB Studio 4.3.0 or newer.**

Changes individual palette colours while the game runs, from script variables.

That opens up effects GB Studio's palette events cannot reach: a lava glow that pulses, a sunset
that shifts colour over a minute, a hit flash that turns one enemy white for three frames, a sky
that darkens as a storm builds. Reading colours back means you can fade towards a target colour a
step at a time.

It also adds versions of the stock background and sprite palette events with a **Commit** tickbox,
and an event that copies another scene's background palettes into the current one.

![image](https://github.com/user-attachments/assets/83791fcc-9e21-405f-a8fa-e40c9acb1203)

![image](https://github.com/user-attachments/assets/c9c642bf-7375-45bc-8fb7-da3728da5ef6)

---

## Table of Contents

1. [Concepts](#concepts)
2. [Project Setup](#project-setup)
3. [Size Limits and Restrictions](#size-limits-and-restrictions)
4. [Events Reference](#events-reference)
5. [FAQ](#faq)
6. [Memory Footprint](#memory-footprint)
7. [Bank 0 (HOME) Usage](#bank-0-home-usage)
8. [Changelog](#changelog)

---

## Concepts

### How a colour is written on Game Boy Color

Each colour is one number holding red, green and blue, each from 0 to 31:

$$\text{colour} = R + (G \times 32) + (B \times 1024)$$

| Channel | Range |
|---|---|
| Red | 0 to 31 |
| Green | 0 to 31 |
| Blue | 0 to 31 |

Pure red is 31, pure green is 992, pure blue is 31744, and white is 32767. This is the format the
events use with **Is gameboy color palette** ticked.

### How a colour is written on original Game Boy

Each palette entry is a shade number rather than a colour:

| Value | Shade |
|---|---|
| 0 | White |
| 1 | Light |
| 2 | Dark |
| 3 | Black |

The background palette has 4 shades. A sprite palette has 3 usable ones, because the first is
always transparent.

### Commit

Every palette-writing event here has a **Commit** tickbox.

- **Ticked**, the default, sends the colours to the screen straight away and you see them this
  frame.
- **Unticked**, the colours are stored but not sent. They appear the next time something else
  sends palettes, such as a later committed call or the end of a fade in.

Untick it when you are changing colours during a fade in, so your change and the fade do not fight,
or when you want to set several palettes and show them all at once.

![image](https://github.com/user-attachments/assets/f7b89f5a-2762-43d1-b5c0-dc93d9413abf)

---

## Project Setup

1. Copy the plugin folder into your project's `plugins` folder.
2. There is nothing to configure. All five events are available immediately.

![image](https://github.com/user-attachments/assets/7e715edd-74ae-4643-a3f4-7641b267d8e8)

![image](https://github.com/user-attachments/assets/73c9529b-b413-49ca-a3dd-12371873f2e3)

---

## Size Limits and Restrictions

### Full colours need Game Boy Color

Red, green and blue values apply to Game Boy Color palettes. On a monochrome build, or with **Is
gameboy color palette** unticked, the events read and write shade numbers instead.

### Sprite colour 0 is transparent

On Game Boy Color, the first colour of a sprite palette is always transparent and cannot be set.
**Set colors of a palette** shows only three colour fields for a sprite palette.

### Palette numbers

| Mode | Palette number |
|---|---|
| Color background | 0 to 7 |
| Color sprite | 0 to 7 |
| Monochrome background | 0, the only one |
| Monochrome sprite | 0 or 1 |

### Copy scene palette colors covers background only

It reads the background palettes of the scene you name. Sprite palettes are untouched, and the
source scene is chosen when the project is built.

### The EX events pick whole palettes

**Set Background Palette EX** and **Set Sprite Palette EX** choose palettes from your project's
palette list. They cannot take a colour from a variable. For that, use **Set colors of a palette**.

### No engine files are replaced

The plugin adds a new engine file and changes none of the existing ones, so it has no conflicts
with other engine plugins.

---

## Events Reference

### Set colors of a palette

Group: **Color**.

Sets the colours of one palette while the game runs.

| Field | Default | Description |
|---|---|---|
| Is gameboy color palette | on | Ticked, the values are Game Boy Color colours. Unticked, they are shade numbers from 0 to 3. |
| Is sprite palette | off | Ticked, targets a sprite palette instead of a background one. |
| Palette | 0 | Which palette to write. 0 to 7 in colour, 0 for a monochrome background, 0 or 1 for monochrome sprites. |
| Color 1 | 0 | First colour. On a sprite palette this is the first visible one, since the transparent slot cannot be set. |
| Color 2 | 0 | Second colour. |
| Color 3 | 0 | Third colour. |
| Color 4 | 0 | Fourth colour. Background palettes only, hidden for sprites. |
| Commit | on | Sends the change to the screen straight away. Untick it during fade ins. |

### Get colors of a palette

Group: **Color**.

Reads a palette's current colours into variables, in the same format as the event above.

| Field | Default | Description |
|---|---|---|
| Is gameboy color palette | on | Ticked, reads Game Boy Color colours. Unticked, shade numbers. |
| Is sprite palette | off | Ticked, reads a sprite palette. |
| Palette | 0 | Which palette to read. |
| Color 1 | none | Variable that receives the first colour. |
| Color 2 | none | Variable that receives the second colour. |
| Color 3 | none | Variable that receives the third colour. |
| Color 4 | none | Variable that receives the fourth colour. Background palettes only. |

### Copy scene palette colors

Group: **Screen**.

Loads all 8 background palettes from another scene into the current ones. Handy when you are
showing tiles from another scene, for instance with the SubmappingEx plugin, and want its colours
too without changing scene.

| Field | Default | Description |
|---|---|---|
| Scene | Last scene | The scene whose background palettes are copied. Chosen when the project is built. |
| Commit | on | Sends all 8 palettes to the screen straight away. |

Sprite palettes are left alone. On Super Game Boy the corresponding palette transfer also happens.

### Set Background Palette EX

Group: **Color**.

The stock **Set Background Palette** event with a **Commit** tickbox, so the change can be held
back, for instance during a fade in.

| Field | Default | Description |
|---|---|---|
| Palettes 0 to 7 | keep | Which palette to put in each of the 8 background slots. **keep** leaves a slot as it is, **restore** puts back the scene's own value. |
| Commit | on | Sends the change to the screen straight away. Untick it during fade ins. |

### Set Sprite Palette EX

Group: **Color**.

The same for the 8 sprite palette slots.

| Field | Default | Description |
|---|---|---|
| Palettes 0 to 7 | keep | Which palette to put in each of the 8 sprite slots. **keep** leaves a slot as it is, **restore** puts back the scene's own value. |
| Commit | on | Sends the change to the screen straight away. Untick it during fade ins. |

---

## FAQ

**How do I make an enemy flash white when it takes a hit?**
Read its sprite palette with **Get colors of a palette** and store the values, set all three colours
to 32767 with **Set colors of a palette**, wait three frames, then write the stored values back.

**How do I fade the sky from blue to orange over time?**
Read the current colour, work out a value one step nearer the target, and write it back once a
frame from an update script. Because the colour is a single number, you can step the red, green and
blue parts separately with a little arithmetic.

**How do I compute the number for a colour I want?**
Multiply green by 32 and blue by 1024, then add red. Each part runs from 0 to 31, so mid grey is
15 + 15×32 + 15×1024, which is 15855.

**Can I do any of this on original Game Boy?**
Yes, with shades rather than colours. Untick **Is gameboy color palette** and use 0 to 3. Swapping
shades gives you a screen flash or an inverted look.

**My colour change is wiped out at the start of a scene.**
The fade in sends palettes when it finishes, overwriting anything committed during it. Untick
**Commit** while the fade is running, or apply the change after it has completed.

**Why can I only set three colours on a sprite palette?**
The first slot of every sprite palette is transparent on the hardware and cannot be given a colour.

**How do I change several palettes at once without seeing them arrive one at a time?**
Untick **Commit** on all but the last call. Nothing is sent until the last one, and everything
appears together.

**Can I use another scene's colours without going to that scene?**
Yes. **Copy scene palette colors** loads its 8 background palettes into the current ones.

**Does it clash with other plugins?**
No. It adds a new engine file and replaces none of the stock ones.

---

## Memory Footprint

Measured against the stock GB Studio **4.3.0-e1** engine at default engine settings, report of
2026-08-13. Figures are the difference against a stock project. Each event you use also compiles a
few bytes of script into your project, on top of the fixed cost below.

| Budget | Cost |
|---|---|
| Bank 0 (HOME) | 0 bytes |
| WRAM | 0 bytes |
| Banked ROM | +1,341 bytes |

- **Bank 0:** nothing. Everything the plugin adds is compiled into a switchable ROM bank.
- **WRAM:** no change. The plugin works on the palette data GB Studio already keeps.
- **Banked ROM:** 1,341 bytes for the colour code.
- **Engine WRAM headroom:** a stock GB Studio 4.3.0 project leaves about **854 bytes** of WRAM
  free (the engine has 7,776 bytes to work with and uses 6,922 of them). With this plugin
  installed roughly **854 bytes** remain. Adding more global variables to your project does not
  change that figure, because script memory is a fixed 3,584 byte block at stock engine settings.
- **SRAM:** not used.

---

<!-- BANK0:BEGIN -->
## Bank 0 (HOME) Usage

Bank 0 is the 16 KB fixed ROM bank shared by the GB Studio engine core, the
interrupt handlers and the GBDK runtime. Extra banked ROM is cheap to add,
bank 0 is not, so bank 0 is usually the first thing a project runs out of.

| | Bytes |
|---|---|
| Bank 0 used by this plugin | **0** |

**This plugin costs nothing in bank 0.** Everything it adds is compiled into a
switchable ROM bank.
<!-- BANK0:END -->

## Changelog

Grouped by the date each change was merged into the official
[gb-studio-plugins](https://github.com/gb-studio-dev/gb-studio-plugins) repository.

Only bug fixes, new features and feature changes are listed. Engine version bumps, patch
regeneration, packaging fixes and documentation edits are omitted.

### 2026-06-14

- Added custom script parameter and stack support to the events.

### 2025-10-29

- Fixed an unused monochrome palette setting and an inverted commit flag.

### 2025-06-02

- Fixed the commit value being pushed the wrong way.

### 2025-04-23

- The palette number is now ignored for monochrome background palettes.

### 2025-04-02

- Added the commit option to palette colour changes, and fixed an inverted field.
- Fixed sprite colour changes.
- Added the event that copies another scene's palette colours.
- Added the event that reads a palette's colours.

### 2025-02-24

- Initial release.
