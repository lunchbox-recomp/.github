# Contributing

This guide is for the Lunchbox repos that don't have their own, which is the gcn-recomp fork. Dragoon has [its own guide](https://github.com/lunchbox-recomp/dragoon/blob/main/CONTRIBUTING.md).

## The repos

- **[Dragoon](https://github.com/lunchbox-recomp/dragoon)** is a game project built on the fork. It holds its game's side, like function names and the tables that tell the recompiler about the game's quirks.
- **[The gcn-recomp fork](https://github.com/lunchbox-recomp/gcn-recomp)** is a fork of [upstream gcn-recomp](https://github.com/ethannicholas/gcn-recomp), the recompiler and the runtime that stands in for the GameCube's hardware. Its `main` holds only upstream's commits, and the fork's own work lands on its `dragoon` branch.

## Where a change goes

A change to the fork is a pull request against `dragoon`. Pick it as the base when you open one, since the default branch is `main`. It has to help any game, so code that only one game needs goes to that game's own project.

Nothing from any game's disc goes in any repo, whether that's the disc itself or code generated from it. Saves and memory cards come from a game too, so they stay out.

## Bugs

The fork's issues are off, so a bug in it goes in [Dragoon's issues](https://github.com/lunchbox-recomp/dragoon/issues). Give the fork's commit, which `git -C gcn-recomp rev-parse --short HEAD` prints in a Dragoon clone. A security problem goes through the security policy in the repo's Security tab, never an issue.

## Checks

CI runs on every pull request, on a macOS runner. It runs `tools/check_generic.py`, which fails when any file but the README and CLAUDE.md matches a term in `tools/game_terms.txt`. The list itself is exempt, and so is a binary file.

Then it configures with no game and runs `ninja -C build unit`. The same steps run on your own Mac with CMake and Ninja installed beside the Xcode Command Line Tools, from the repo's top folder.

```sh
python3 tools/ci.py unit
```

## Commits

A commit message is one short line saying what changed, with no prefix like `fix:`.

## AI

Claude, Codex or local models running in [Pi](https://github.com/earendil-works/pi) write the fork's own commits on `dragoon`.

AI help is fine. Name the tools you used in the PR description, and leave AI co-author lines out of commits, since the PR description and this section cover them. Only send what you've run on your own disc.
