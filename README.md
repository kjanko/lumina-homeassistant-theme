<p align="center"><img src="docs/banner.png" alt="Lumina" /></p>

<p align="center">
  <a href="https://my.home-assistant.io/redirect/hacs_repository/?owner=kjanko&repository=lumina-homeassistant-theme&category=theme"><img src="https://img.shields.io/badge/HACS-Custom-41BDF5?logo=home-assistant&logoColor=white" alt="HACS Custom" /></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/kjanko/lumina-homeassistant-theme" alt="MIT license" /></a>
  <a href="https://github.com/kjanko/lumina-homeassistant-theme/commits/main"><img src="https://img.shields.io/github/last-commit/kjanko/lumina-homeassistant-theme?label=last%20commit" alt="Last commit" /></a>
</p>

Lumina turns a [Bubble Card](https://github.com/Clooos/Bubble-Card) dashboard into frosted glass on a soft, glowing background. Lights glow in their own colour while they're on, switches in green or red, and buttons that would do nothing right now fade out of the way. Graphs, thermostats and the rest of Home Assistant match.

<p align="center">
  <a href="https://my.home-assistant.io/redirect/hacs_repository/?owner=kjanko&repository=lumina-homeassistant-theme&category=theme"><img src="https://my.home-assistant.io/badges/hacs_repository.svg" alt="Open this repository in HACS" /></a>
</p>

![The same dashboard with Lumina and with the default theme](docs/compare-living-room.png)

## What it does

- **Glass everywhere**: every Bubble Card type (button, select, climate, cover, media player, calendar), plus graphs, thermostats, gauges and entity cards. No setup per card.
- **Rooms in glass panels**: A room is one glass panel, and the devices in it sit in smaller panels inside it.
- **Glow**: A card can glow in any colour: a light in its own colour, a switch green or red, a robot by what it's doing. A soft bevel along the top edge follows the card's colour.
- **One button style**: Every button is tinted glass with a light glass edge and an icon in its colour. A card only picks the colour; buttons that are off are flat grey.
- **Greyed out when it would do nothing**: The TV's inputs and volume while it's off, "turn off" while nothing is on.
- **One palette**: Eight colours shared by glows, graph lines and thermostat modes.
- **Quiet titles, a centred divider**: Small uppercase section titles with a fading line, and a divider that's brightest in the middle.

<p align="center"><img src="docs/dashboard.png" alt="A full Lumina dashboard" /></p>

<table>
  <tr>
    <td align="center"><img src="docs/phone.png" width="300" alt="Lumina on a phone" /><br /><sub>On a phone</sub></td>
    <td align="center"><img src="docs/popup-sheet.png" width="300" alt="A Bubble Card pop-up in Lumina" /><br /><sub>Pop-ups and dropdowns</sub></td>
  </tr>
</table>

## Requirements

| | Tested with |
|---|---|
| Home Assistant | 2026.10 |
| [Bubble Card](https://github.com/Clooos/Bubble-Card) | 3.4.2 |
| [card-mod](https://github.com/thomasloven/lovelace-card-mod) | 4.2.1 |
| [stack-in-card](https://github.com/custom-cards/stack-in-card) (for glass panels) | 0.2.0 |

All three cards are available in HACS.

## Installation

### HACS (recommended)

1. Open HACS → ⋮ → **Custom repositories**.
2. Add `https://github.com/kjanko/lumina-homeassistant-theme` with category **Theme**.
3. Search for "Lumina" and download it.

### Manual

Copy `themes/lumina.yaml` into the `themes` folder in your Home Assistant config directory.

### Turn it on

1. Make sure Home Assistant loads themes from that folder. In `configuration.yaml`:
   ```yaml
   frontend:
     themes: !include_dir_merge_named themes
   ```
2. Restart Home Assistant, or call the `frontend.reload_themes` action.
3. Open your profile, set **Theme** to **Lumina** and **Mode** to **Dark**. Lumina is a dark design.

## Building blocks

The theme styles every card on its own. The pieces below need a line or two on the card. [`examples/dashboard.yaml`](examples/dashboard.yaml) uses all of them.

### Glass panels

Put a room's cards in a `stack-in-card` with the `glass` class. For a device inside a room, nest another `stack-in-card` with `glass-inner`.

```yaml
type: custom:stack-in-card
card_mod:
  class: glass
cards:
  - type: custom:bubble-card
    card_type: separator
    name: Living Room
    icon: mdi:sofa-outline
  - type: custom:stack-in-card
    card_mod:
      class: glass-inner
    cards:
      - ...
```

### Divider

A Bubble separator with no name, with the `divider` class:

```yaml
type: custom:bubble-card
card_type: separator
card_mod:
  class: divider
```

### Glow

A card opts in from its Bubble `styles` by adding the `lumina-glow` class and setting two variables:

- `--glow`: the colour, as `r,g,b` (e.g. `255,180,50`) or a palette variable (e.g. `var(--lumina-green)`).
- `--glow-on`: the strength. `0` is off and looks like a plain card, `1` is the standard glow, and above `1` is stronger.

Optional: `--glow-dim` sets a second colour for the gradient, and `--glow-icon` sets an icon colour for glows too dark to use as one.

A light that glows in its own colour while it's on:

```yaml
styles: |
  ${(() => {
    card.classList.add('lumina-glow');
    const [r, g, b] = hass.states[entity]?.attributes?.rgb_color || [255, 180, 50];
    const on = state === 'on';
    return `.bubble-button-card-container {
      --glow: ${on ? `${r},${g},${b}` : '255,255,255'};
      --glow-on: ${on ? 1 : 0};
    }`;
  })()}
```

A switch that is green when on and red when off:

```yaml
styles: |
  ${(() => { card.classList.add('lumina-glow'); return ''; })()}
  .bubble-button-card-container {
    --glow: ${state === 'on' ? 'var(--lumina-green)' : 'var(--lumina-red)'};
    --glow-on: ${state === 'on' ? 0.65 : 0.4};
  }
```

### Buttons

Every Bubble sub-button with a background gets the same look: a tint of its colour, a 1px glass edge that's lighter at the top, and an icon in the colour. A card only sets the colour, per button, with `--btn` (an `r,g,b` triplet or a palette variable). Without it a button is cyan.

A button that is off looks like a plain grey Lumina button: set `--btn: var(--lumina-neutral); --btn-off: 1`.

```yaml
styles: |
  .bubble-sub-button-1 { --btn: var(--lumina-red); }  /* stop */
  ${(() => {
    const open = state === 'open';
    return `.bubble-sub-button-2 { --btn: ${open ? 'var(--lumina-teal)' : 'var(--lumina-neutral); --btn-off: 1'}; }`;
  })()}
```

Sub-buttons with `show_background: false` stay plain text or icons.

### Greyed out

Add `lumina-disabled` to a card, or to one of its buttons, while its action would do nothing. It is dimmed, grey and can't be tapped.

```yaml
styles: |
  ${(() => {
    const off = ['off', 'unavailable', 'unknown'].includes(hass.states['media_player.tv']?.state);
    card.classList.toggle('lumina-disabled', off); // the whole card
    card.querySelector('.bubble-sub-button-1')?.classList.toggle('lumina-disabled', off); // or one button
    return '';
  })()}
```

### Palette

Each colour is an `r,g,b` triplet, so it works as a `--glow` value and in `rgba(var(--lumina-green), 0.3)`.

| Variable | Colour |
|---|---|
| `--lumina-amber` | 255,180,50 |
| `--lumina-green` | 80,200,140 |
| `--lumina-teal` | 45,212,191 |
| `--lumina-cyan` | 56,189,248 |
| `--lumina-blue` | 100,180,255 |
| `--lumina-indigo` | 99,102,241 |
| `--lumina-red` | 220,80,80 |
| `--lumina-magenta` | 226,0,116 |

## Customising

All sizes and colours are variables at the top of `themes/lumina.yaml`: radii, card and icon backgrounds, border, shadows, text colours, blur, the glass panel look, the separator line, the divider and the background. Change them there, or make a copy of the theme under a new name.

The background is `lovelace-background`. A dashboard view with its own `background` replaces it.

## Good to know

- **Other cards.** Standard cards (graphs, thermostats, gauges, entities, weather) get the glass background, rounded corners and palette colours from Home Assistant's own theme variables. Their layout stays Home Assistant's own; only Bubble Card gets the glow and greyed-out styles.
- **Cards with their own border colour.** The bevel line under a Bubble card's top edge uses the glow colour. A card that draws its own coloured border without `lumina-glow` sets `--glow` and `--glow-on` on `.bubble-button-card-container` so the line matches it, and `.bubble-button-card-container::after { display: none !important; }` removes the line from a card that has no edge at all.
- **Card styles and the theme.** The glow, greyed-out and glass rules use `!important` so they win over Bubble's inline colours. A card's own rule that should beat them needs a more specific selector, e.g. `ha-card .bubble-separator .bubble-line` instead of `.bubble-line`.
- **Cards that appear later.** Cards created after the page loads, such as inside a `conditional` card, may not pick up theme styles until the page is reloaded. To show and hide rows, keep the cards on the page and hide them from their own styles instead:
  ```yaml
  styles: |
    ${hass.states['input_boolean.show_more']?.state === 'on' ? '' : ':host { display: none !important; }'}
  ```

## License

[MIT](LICENSE)
