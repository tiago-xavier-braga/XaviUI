# Code Style (GDScript)

Based on Godot's official GDScript style guide.

- **Indentation**: tabs, never spaces.
- **Static typing**: always declare types on variables, parameters, and
  return values (`var speed: float = 0.0`, `func ease(t: float) -> float:`).
- **Naming**:
  - `snake_case` for variables, functions, and signals.
  - `PascalCase` for classes (`class_name`) and enums.
  - `CONSTANT_CASE` for constants and enum values.
  - `_` prefix for private members/methods (`_internal_state`,
    `_process_tween`).
  - signals in the past tense, describing what happened (`tween_finished`,
    not `on_tween_finish`).
- **Order within a script**:
  1. `@tool` / `class_name` / `extends`
  2. signals
  3. enums and constants
  4. `@export` vars
  5. public variables
  6. private variables / `@onready`
  7. engine virtual methods (`_init`, `_ready`, `_process`, ...)
  8. public methods
  9. private methods
- **Exports**: use Godot 4's `@export` syntax (not the legacy `export`).
- **No magic numbers**: repeated or non-obvious values become named
  constants.
- **Comments**: only when the "why" isn't obvious from the code; never a
  comment explaining the "what".
- **Line length**: ~100 columns as a guideline, not a hard rule.
