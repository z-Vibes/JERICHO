# JERICHO: notes for the next Claude session


## Where things stand
- **Open PR:** `z-Vibes/JERICHO:c-vibes/fix-everything-0.14` into `Njurse/JERICHO:0.14`, 2 commits on top of `0.14` @ `2f01241`. Read its CI result first.
- **The record:** `docs/FIX_PLAN.md` (findings, status and corrections), `docs/FIX_PATCH_PLAN.md` (the approved plan), `docs/patches/README.md` (mod-repo patches). Read those three files, not the whole tree.
- **Already verified on Linux:** both builds (`release_x64`, `release_dev_x64`) and all host tests pass, and the repo checkers are clean. Windows has never been built in a session; only CI covers it.

## What's left, in priority order
1. **Get the PR green.** Two failures are expected:
   - `tools/test_vehicle_profiles.py` needs `JERICHO/MODS/cainescrossfire/vehicles/bank.json`, which was never committed. The owner must commit it or make the test skip when it's absent. Don't fabricate the data.
   - The Windows build is unverified. Fix only what CI actually reports.
2. **Mod repos.** Apply `docs/patches/{mp,carhacks,bake}/*.patch` in their repos (`JERICHO-Mod-MP`, `JERICHO-Mod-carhacks`, `JERICHO-D1`) with `git am`, then bump the pins with `tools/update_mod_submodules.sh`.
3. **Cleanup, only with the owner's OK:** the committed zips at the repo root, duplicate `libs/` copies, `.cd2_patch*.py`, and old appveyor/flatpak/Docker configs. List what would go before deleting anything.
4. **Roadmap, only if asked:**
   - Safe mode: the dispatcher already knows which module owns each handler.
   - Give separately loaded mods access to engine data through `jer_host_api.h` (`check_mod_api.py --gap` lists the gap).
   - mp protocol versioning.

## Don't spend tokens on
- Re-auditing, or re-reading `README.md`, `CHANGELOG.md` or `HOOKS.md` end to end.
- `libs/` (vendored SDL2/OpenAL/libjpeg), `readme_images/`, or `symbols/`.
- `d1cars` mentions in `CHANGELOG.md`, in `jer_d1cars.c` and its API (a real name, not stale), or in mp's alias code (`mp_lobby.c`, `dedi/core.py` treat it as bake's old id on purpose).
- The 114 compiler warnings, splitting `jer_modstore.c`, or doc polish, unless asked.
- `gaildrv2` (hibernated) and `antfarm` (store-only; CI doesn't fetch it).
- Full clean rebuilds (about 10 min). Rebuild incrementally, and only when C code changed.

## Setup (Linux container)
```sh
# submodules: always the PINNED commit, never --recursive
for p in src_rebuild/PsyCross JERICHO/MODS/mp JERICHO/MODS/carhacks JERICHO/MODS/crumple JERICHO/MODS/bake; do
  git submodule update --init --depth=1 "$p"
  sha=$(git rev-parse "HEAD:$p"); git -C "$p" fetch -q --depth=1 origin "$sha"; git -C "$p" checkout -q --detach "$sha"
done
apt-get install -y --no-install-recommends build-essential libsdl2-dev libopenal-dev libjpeg-turbo8-dev libgl1-mesa-dev
curl -fL https://github.com/premake/premake-core/releases/download/v5.0.0-beta1/premake-5.0.0-beta1-linux.tar.gz -o /tmp/p.tgz
echo '074fc07be17888ef41ca07eea1c527c390a04ea4550bf7962eeb0bea0cc9d2f5  /tmp/p.tgz' | sha256sum -c - && tar xf /tmp/p.tgz -C src_rebuild
(cd src_rebuild && ./premake5 gmake2) && make -C src_rebuild/build config=release_x64 -j"$(nproc)"
```
- **Checks:**
  ```sh
  for t in doccheck check_release_mods check_mod_api check_mod_interface check_host_api_abi; do python3 tools/$t.py; done
  ```
- **Tests** (need a finished build of that configuration):
  ```sh
  src_rebuild/Game/C/JERICHO/test/run_tests.sh Release
  ```

## Gotchas that cost time before
- Run premake after adding or removing C files. Anything under `src_rebuild/Game/` except `Game/C/JERICHO/{src,include,test}` is compiled into the game, so new tests go in `Game/C/JERICHO/test/`.
- After deleting sources, delete `JERICHO/MODS/*/obj` (gitignored). Stale dependency files there cause "No rule to make target".
- `jer_registry.c` is generated and untracked. Premake writes it, and `check_release_mods.py` needs it to exist.
- In a runtime test, optional modules are refused unless the core suite (mp, carhacks, bake, pedhacks) is in the registry. Register stand-ins for them; `test_dispatch.c` shows how.
- The mod repos are separate repos. In this one, fix them only as patch files plus a pin bump.
- **Commit rules:**
  - CI rejects commits crediting Cursor (`tools/ci/no_cursor_attribution.sh`); Claude attribution is fine.
  - Never push to `main` or `0.14` directly; use a new branch and a PR.
