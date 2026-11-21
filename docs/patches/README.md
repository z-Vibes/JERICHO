# Patches for the mod repositories

These fixes belong in the mod repositories (`JERICHO-Mod-MP`, `JERICHO-Mod-carhacks`,
`JERICHO-D1` for bake), not in this tree. They were made against the commits this
tree currently pins, and each one applies cleanly there. Apply them in each repo,
push, then bump the pins here with `tools/update_mod_submodules.sh`.

| Repo | Patch | What it fixes |
|---|---|---|
| mp (`JERICHO/MODS/mp`) | `mp/0001-...patch` | **The dedicated server's example config refused every client.** Its `core_mods` required `d1cars`, which was renamed `bake`. Also fixes the dedi README, tests, e2e rig, pack_lan packagers and a log line. |
| carhacks (`JERICHO/MODS/carhacks`) | `carhacks/0001-...patch` | `bigrig_extract.py` and `slotmap.py` looked in `MODS/d1cars/tools`, which no longer exists. Also fixes the `DRIVER1.md` doc links. |
| bake (`JERICHO/MODS/bake`) | `bake/0001-...patch` | `tools/car/README.md` set `T=JERICHO/MODS/d1cars/tools`, so every command in it pointed at a missing folder. |

To apply one (example: mp), from the mod repo's own clone:

```sh
git am /path/to/JERICHO/docs/patches/mp/0001-*.patch
```

Checked before export:
- **mp:** `tools/servers/dediserver/test_console.py` passes. The example config now admits a client running the core suite; before, it refused one.
- **carhacks:** both tools start, and the new paths resolve (`pages1.decomp_asm` imports).
- **bake:** README and comment only.
