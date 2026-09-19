# File and Folder Naming

Follows the official Godot style guide.

- **Folders**: `snake_case`, lowercase. E.g. `leaf_tween/`, `basic_demo/`.
- **Scenes (`.tscn`)**: `snake_case`, matching the script/root node name.
  E.g. `player.tscn`, `main_menu.tscn`.
- **Scripts (`.gd`)**: `snake_case`, matching the scene name when attached to
  it. E.g. `player.gd` for `player.tscn`.
- **Resources (`.tres`, `.res`)**: `snake_case`, describing the content. E.g.
  `ease_out_cubic.tres`.
- **Classes (`class_name`)**: `PascalCase`, to stay consistent with the
  engine's native classes. E.g. `class_name LeafTween`.
- **Autoloads/Singletons**: file name in `snake_case`, but the name
  registered in the project (used as the global identifier) in `PascalCase`.
- Avoid spaces, accents, and uppercase letters in file/folder names — only
  `a-z`, `0-9`, and `_`.
