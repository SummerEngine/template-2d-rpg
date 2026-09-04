# OpenRPG (Summer Engine template `2d-rpg`)

GDQuest's open-source OpenRPG demo adapted for Summer Engine (Godot 4.6 feature set; the
Summer binary reports a 4.7-based GDScript analyzer). Code MIT, art CC-BY 4.0 GDQuest, plus
the CC0 packs in `CREDITS.md`. Keep the attribution when shipping.

## Layout

- `src/main.tscn` is the entry scene (town map). Other maps: `overworld/maps/forest`, `overworld/maps/house`.
- `src/field/` grid-based overworld (Gameboard + GamepieceRegistry autoloads, Path2D-based `Gamepiece`, controllers, cutscenes, inventory UI).
- `src/combat/` + `combat/` sequential turn-based combat (Battler, BattlerAction, battlers bear/bugcat/squirrel/wolf, arenas).
- `addons/dialogic/` dialogue; timelines are `.dtl` files next to each map; the custom layout is `dialogue_layout.tscn`.
- Autoloads (project.godot): Camera, CombatEvents, FieldEvents, Gameboard, GamepieceRegistry, Music, Player, Transition, Dialogic.

## Dialogic version and local patch

Vendored: **Dialogic 2.0-Alpha-16** (`addons/dialogic/plugin.cfg`). It is patched locally with
three one-line `return` statements so it parses under the stricter analyzer ("Not all code paths
return a value" is a parse error for `_get`/`_set` overrides there):

- `addons/dialogic/Modules/Text/node_name_label.gd` `_set` -> trailing `return false`
- `addons/dialogic/Modules/Variable/subsystem_variables.gd` `_get` -> trailing `return null`
- `addons/dialogic/Modules/Variable/subsystem_variables.gd` `VariableFolder._get` -> trailing `return null`

This is byte-for-byte what upstream did in dialogic-godot/dialogic PR #2769 (shipped in
2.0-alpha-20). Do not overwrite the addon with a pristine alpha-16. Upgrading to alpha-20 or
later is the proper fix but is breaking (state/save system, custom subsystems, renamed slide
animations, unified audio event, Godot >= 4.5 only); it needs an editor session to run
"Resave all Timelines" and re-verify `dialogue_layout.tscn` and `Dialogic.VAR` usage in
`overworld/maps/**/*.gd`.

## Verify

Projects created with `summer create 2d-rpg <dir>` get `tests/autopilot/run.sh`; a pristine
project must exit 0 with `errors_seen: []`. The `.dtl` ext_resources in `src/main.tscn` and
`overworld/maps/house/wand_pedestal_interaction.tscn` resolve through `.dtl.uid` sidecars once
the Dialogic loaders compile; if "invalid UID ... using text path" warnings persist for them,
the fallback is dropping the `uid=` attribute on those lines.
