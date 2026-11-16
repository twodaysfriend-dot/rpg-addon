# RPG addon development and delivery

## User preferences

- Respond in Korean. When the user requests an addon update, implement it, run the relevant regressions, build the three-pack `.mcaddon`, and deliver a downloadable GitHub link after pushing the release to this repository. The user explicitly requested this workflow and asked for it to persist across chats.
- Publish release files on `codex/rpg-v0531-download` unless the user requests a different branch. Do not force-push. Preserve other work and inspect the remote branch before pushing.
- Include matching source ZIP, download ZIP and Korean release notes. Update root README links and SHA-256. Report what was tested and distinguish mock API/data checks from actual Minecraft engine/rendering checks.
- Keep pack UUIDs and stable item IDs during upgrades. Increment all three manifest/module and cross-pack dependency versions together.

## Source of truth and workspace

- This is a release/download repository: current source and tests are in the latest `downloads/RPG_v*_source.zip` referenced by README. Do not assume an old extracted directory is current.
- Each cloud task is isolated. Use the existing checkout; do not create a Git worktree unless explicitly requested.
- Extract the selected source ZIP into a separate development directory under `/workspace`, validating paths and preserving existing edits. Internal pack directory suffix `v0638` is intentionally retained for tool/test compatibility, even though manifest/release versions are newer.
- Use Node.js 24 and Python 3.12. No third-party packages or secrets are required. Read extracted README/STATUS and tools/package.py before changing or building a release.

## Validation and release

From the extracted source root, run:

```sh
set -e
python3 checks/validate.py .
node tools/build_monster_balance.mjs --check
python3 tools/build_abyssal_armor.py --check
node checks/largeArmor.mjs .
node --experimental-vm-modules checks/integration.mjs . --gameplay-only-wings
for suite in integration worldUnlock behavior balance world rebalance frost holyOrb vitality dragonSnow statInput wings townMuda townVillagers beastHoly michael; do
  node --experimental-vm-modules "checks/$suite.mjs" .
done
python3 tools/package.py /workspace/rpg-release/<release-version>
```

Update tests when intentionally changing behavior; keep meaningful coverage for walls, allies, range, repeated hit timing, native rejection fallback, buff consumption, existing-player migration and item state preservation. For damage questions, exercise the actual `skillDamageFor` with real equipment/rune data and state the assumed buffs, job and enemy defenses.

Copy the verified new artifacts into `downloads/` without overwriting historical releases. Check nested `.mcpack` integrity and all version/dependency references. Commit only the intended repository changes and push normally. Provide the raw download URL (not a workspace-only link):

`https://github.com/twodaysfriend-dot/rpg-addon/raw/refs/heads/codex/rpg-v0531-download/downloads/<filename>`

If push is rejected or authentication fails, diagnose it, preserve the release locally, and report the exact blocker. Do not claim GitHub delivery succeeded until push and remote commit verification succeed.
