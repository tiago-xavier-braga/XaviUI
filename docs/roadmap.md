# Roadmap

Implementation plan for building XaviUI from scratch: a portable UI kit for
Godot — themed components, ready-made menu templates, and automatic
keyboard/gamepad/touch switching — designed to drop into new projects (web,
mobile, Steam) so UI stops being rebuilt from zero every time.

## Core Architecture

- A `Theme` resource generated from a token resource pair (`XaviPalette` +
  `XaviTypography`), writing directly into Godot's native Theme Type
  Variations (`PrimaryButton`, `SecondaryButton`, ...) — colors/fonts/sizes
  live as data on a resource, not as `StyleBox`/override edits repeated by
  hand across every variation.
- An autoload singleton `XaviInput` listens to every `_input(event)` and
  classifies it by type (`InputEventKey`/`InputEventMouseButton`,
  `InputEventJoypadButton`/`Motion`, `InputEventScreenTouch`/`Drag`),
  tracking which device produced the *last* input and emitting
  `device_changed(device: InputDevice)` only on an actual change.
- Most components need no script at all — just a Theme Type Variation
  applied to a native `Button`/`CheckBox`/`Slider`/`Panel`, which already
  handles hover/pressed/disabled/focus on its own. A single focus overlay
  (not a script per component) listens to `XaviInput.device_changed` +
  `get_viewport().gui_get_focus_owner()` to draw the right prompt over
  whatever currently has focus.
- A helper autoload `XaviBreakpoints` that only exposes which named
  breakpoint is active (`mobile_portrait`/`mobile_landscape`/`desktop`)
  from the viewport size — the actual restructuring is left to Godot's
  native containers (`FlowContainer`, `BoxContainer.vertical`) reacting to
  that value, instead of a custom `Container` reimplementing what they
  already do.
- Full-screen templates (main menu, pause, settings) are plain scenes
  composed from the component kit — no separate runtime, so they inherit
  theming, input switching, and responsive layout for free.

## Value-Add Features

Four features go beyond what Godot's native `Control`/`Theme` system gives
you out of the box. Each is paired with the manual workaround it replaces,
built in the same phase as the feature itself, to make the gap concrete
rather than asserted:

- **Design-token theming** (Phase 1) — Godot already has Theme Type
  Variations (named styles over a base type, like `PrimaryButton`), but
  none of them reference a shared color — changing the game's "accent"
  means editing the same hex in every `StyleBoxFlat`, variation by
  variation. `XaviPalette`/`XaviTypography` hold the tokens once; a
  builder writes into the existing Type Variations from them, so changing
  the game's look becomes a handful of field edits in one resource.
- **Automatic input-device detection & switching** (Phase 2) — Godot has
  no built-in "what device is the player using right now" concept; the
  usual workaround (`Input.get_connected_joypads().size() > 0`) is wrong
  the moment a gamepad is connected but unused. `XaviInput` tracks the
  actual last-used device from real input events, so prompts/icons swap
  the instant the player actually switches, in either direction.
- **Named breakpoints across export targets** (Phase 4) — the native
  stretch modes (`canvas_items`/`viewport`) scale a fixed layout but don't
  restructure it; a row of buttons that fits a Steam window overflows a
  mobile portrait screen. Godot already has containers that restructure
  (`FlowContainer` wraps, `BoxContainer.vertical` can be toggled at
  runtime), but no concept of "what screen class is this right now" —
  `XaviBreakpoints` covers just that gap, without duplicating the
  containers themselves.
- **Ready-made, fully-navigable menu templates** (Phase 5) — Godot already
  computes `focus_neighbor_*` automatically from the nearest Control
  geometrically, so keyboard/gamepad navigation usually already works just
  by sitting inside a `Container`; the real work is reviewing the cases
  where that auto-computation gets it wrong, guaranteeing a minimum touch
  target size, and shipping the three screens already assembled — instead
  of every project rebuilding that composition from zero.

## Prerequisites

- [ ] Godot's `Control` node system: anchors/margins, containers,
      `focus_mode` + `focus_neighbor_*` navigation (and when the native
      auto-computation is enough vs. when it needs a manual override)
- [ ] Native containers with built-in restructuring: `FlowContainer`
      (`HFlowContainer`/`VFlowContainer`, automatic wrap) and
      `BoxContainer.vertical` (toggleable at runtime) — the base
      `XaviBreakpoints` composes with, not replaces
- [ ] Theme Type Variations — how Godot already solves "a named style
      over a base type" (`PrimaryButton` over `Button`) via the inspector,
      with no subclass needed; `StyleBox` (`StyleBoxFlat`) per state
      (normal/hover/pressed/disabled/focus), and how override lookup
      cascades through the scene tree
- [ ] `Resource` + `class_name` for custom data (`XaviPalette`,
      `XaviTypography`) vs. hand-editing `.tres` files
- [ ] The `Input` singleton and event types: `InputEventKey`,
      `InputEventMouseButton`, `InputEventJoypadButton`/`Motion`,
      `InputEventScreenTouch`/`Drag`, plus `Input.joy_connection_changed`
- [ ] `Node.process_mode` (`PROCESS_MODE_ALWAYS`) + `get_tree().paused` —
      Godot's native pause system, which the pause menu only needs to
      use, not reimplement
- [ ] `Viewport`/`DisplayServer` sizing and Godot's stretch modes, to know
      what a breakpoint system needs to read
- [ ] Signals/`Callable` for reactive theme + input-device updates
- [ ] `FontVariation.fallbacks` (native font fallback chain) and
      `DisplayServer.get_display_safe_area()` (native safe-area/notch) —
      cover Phase 6 without custom logic; Steam Input controller glyph
      conventions remain manual work (art, not engine)

## Phased Plan

### Phase 0 — Baseline
- [ ] Sketch the public API you want (`XaviPalette`, `XaviButton`,
      `XaviInput.get_active_device()`, template scene names) before
      writing addon internals.
- [ ] **Deliverable:** written notes on the desired API shape (this file +
      `api_design.md`).

### Phase 1 — Design tokens & Theme generation
- [ ] `XaviPalette` resource: token colors + semantic roles (background,
      surface, primary/secondary text, accent,
      info/warning/error/critical).
- [ ] `XaviTypography` resource: bundled default fonts + a type scale
      (heading, body, button label, caption).
- [ ] A builder (`XaviThemeBuilder` or a static method) that writes into a
      `Theme`'s native Type Variations (`PrimaryButton`, `SecondaryPanel`,
      ...) from a `XaviPalette` + `XaviTypography` pair — no separate
      Theme resources generated per variant.
- [ ] **Deliverable:** `demo/theming/` scene with a stock Godot `Button`
      and `Panel` (no script, just a Type Variation), where swapping the
      `XaviPalette` resource in the inspector re-skins both live.

### Phase 2 — Input-device detection & switching
- [ ] Reproduce the naive workaround by hand first (guessing the device
      from `Input.get_connected_joypads().size()`), to see the gap
      firsthand.
- [ ] `XaviInput` autoload: tracks the last-used device from real
      `_input(event)` events, exposes `get_active_device() -> InputDevice`
      and emits `device_changed(device: InputDevice)`.
- [ ] **Deliverable:** `demo/input_switching/` scene with a live label
      showing the active device, updating the instant you touch a key,
      the mouse, a connected gamepad, or the touchscreen (web/mobile
      export).

### Phase 3 — Core component kit
- [ ] Theme Type Variations for `Button` (primary/secondary/icon),
      `CheckBox`, `HSlider`/`VSlider`, `Panel` — native Godot nodes, no
      subclass; hover/pressed/disabled/focus already come free from the
      per-state `StyleBox` those native controls already support.
- [ ] A single focus overlay (`XaviFocusPrompt`, not a script per
      component): listens to `XaviInput.device_changed` +
      `get_viewport().gui_get_focus_owner()` and draws the right glyph
      over whatever currently has focus — covers button, slider,
      checkbox, etc. in one place.
- [ ] `XaviButton` as the one real component script, only for what needs
      logic beyond styling (e.g. a dynamic icon per variant) —
      everything else stays script-free.
- [ ] **Deliverable:** `demo/components/` scene showcasing the full kit,
      navigable end-to-end by keyboard, gamepad, and touch.

### Phase 4 — Responsive breakpoints
- [ ] Reproduce the problem by hand first (a fixed layout scaled by the
      native stretch modes, overflowing on a mobile portrait screen), to
      see the gap firsthand.
- [ ] `XaviBreakpoints` autoload: reads the viewport size, exposes
      `get_active_breakpoint() -> StringName` and emits
      `breakpoint_changed(breakpoint: StringName)` for defined names
      (e.g. `mobile_portrait`, `mobile_landscape`, `desktop`).
- [ ] Scenes react to the active breakpoint using native containers —
      `FlowContainer` for wrapping, `BoxContainer.vertical` toggled — no
      custom `Container` at all.
- [ ] **Deliverable:** `demo/responsive/` scene, resized/tested across
      mobile-portrait, mobile-landscape, and desktop/Steam window aspect
      ratios, reacting to `XaviBreakpoints` using only native containers.

### Phase 5 — Full-screen templates
- [ ] `templates/main_menu/`, `templates/pause_menu/`,
      `templates/settings_menu/` scenes assembled from the Phase 3 kit,
      inside `Container`s to take advantage of Godot's native
      `focus_neighbor_*` auto-computation; manual overrides only where
      that auto-computation picks wrong.
- [ ] Touch-sized hit areas via `custom_minimum_size`.
- [ ] `pause_menu`: `process_mode = PROCESS_MODE_ALWAYS` on the scene +
      `get_tree().paused = true` — Godot's native pause system, no
      custom pause logic.
- [ ] Settings menu includes a theme/palette picker as a live example of
      Phase 1's token swap.
- [ ] **Deliverable:** `demo/showcase/` scene chaining all three
      templates, fully navigable by keyboard, gamepad, and touch, with
      the active-device indicator from Phase 2 visible throughout.

### Phase 6 — Cross-platform polish
- [ ] `FontVariation.fallbacks` (native fallback chain) configured on the
      bundled fonts, validated on an HTML5/web export — no custom
      fallback logic.
- [ ] `DisplayServer.get_display_safe_area()` (native) applied as margin
      on the full-screen templates — no custom notch detection.
- [ ] Export and manually verify on at least: Web (HTML5), one mobile
      target (Android/iOS), and Windows/Steam — input switching and
      responsive breakpoints must hold on each.
- [ ] Lightweight test coverage in `tests/` for `XaviInput` device-detection
      edge cases (device disconnect mid-session, multiple gamepads).
- [ ] **Deliverable:** the addon usable standalone (drop `addons/xavi_ui/`
      into another project), validated on all three export targets above.

## Self-Check Questions

1. Why generate a `Theme` from a token resource instead of editing
   `Theme` overrides directly in the inspector — what breaks at
   "re-skin the whole game" scale if you don't?
2. What's wrong with guessing the active input device from
   `Input.get_connected_joypads().size() > 0`, and what does `XaviInput`
   need to track instead to get it right?
3. Why do full-screen templates belong in the addon at all, instead of
   leaving each consuming project to wire its own focus navigation?
4. Why does `XaviBreakpoints` only expose which breakpoint is active,
   without restructuring anything itself — what does that avoid
   duplicating from Godot's native containers (`FlowContainer`,
   `BoxContainer.vertical`)?
5. Why does bundling default fonts inside `addons/xavi_ui/` matter
   specifically for the web export target?
6. If a player has a gamepad connected but is currently pressing
   keyboard keys, what should `XaviInput.get_active_device()` return,
   and why?
7. Why doesn't most of the kit's components need any script at all —
   what does Godot's Theme Type Variation already solve on its own, and
   what's left for `XaviFocusPrompt`/`XaviButton` to cover?
8. Why centralize the focus prompt in a single `XaviFocusPrompt` instead
   of repeating the same `device_changed` logic in every component?

## Done Criteria

- Can explain the token → `Theme` generation pipeline and the
  input-device detection approach from memory, no notes.
- Component kit, responsive layout, input switching, and all three menu
  templates implemented end-to-end.
- A brand-new Godot project can copy `addons/xavi_ui/`, drop in the
  three templates, and ship a themed, fully keyboard/gamepad/
  touch-navigable menu flow without writing new UI code.
- Verified working on Web, one mobile export, and Windows/Steam.

## References

- [Godot: Control node UI tutorials](https://docs.godotengine.org/en/stable/tutorials/ui/index.html)
- [Godot: Containers (Size and anchors)](https://docs.godotengine.org/en/stable/tutorials/ui/size_and_anchors.html)
- [Godot: GUI skinning (Theme)](https://docs.godotengine.org/en/stable/tutorials/ui/gui_skinning.html)
- [Godot: Theme Type Variations](https://docs.godotengine.org/en/stable/tutorials/ui/gui_theme_type_variations.html)
- [Godot: Pausing games (process mode)](https://docs.godotengine.org/en/stable/tutorials/scripting/pausing_games.html)
- [Godot: InputEvent class reference](https://docs.godotengine.org/en/stable/classes/class_inputevent.html)
- [Godot: Multiple resolutions](https://docs.godotengine.org/en/stable/tutorials/rendering/multiple_resolutions.html)
