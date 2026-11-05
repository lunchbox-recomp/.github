# Security policy

## Supported branches

There are no releases, and fixes land on one branch in each repo. Pulling Dragoon brings a fork fix once Dragoon's `gcn-recomp` submodule points at a commit that has it, and `git submodule update` checks that commit out.

| Repo | Branch |
|---|---|
| [Dragoon](https://github.com/lunchbox-recomp/dragoon) | `main` |
| [The gcn-recomp fork](https://github.com/lunchbox-recomp/gcn-recomp) | `dragoon` |

## Reporting a vulnerability

**Never open a public issue for a security report.** Use GitHub's private vulnerability reporting, from the repo's Security tab.

- Dragoon at <https://github.com/lunchbox-recomp/dragoon/security/advisories/new>
- The gcn-recomp fork at <https://github.com/lunchbox-recomp/gcn-recomp/security/advisories/new>

Put these in the report.

- What the problem is and what someone could do with it
- The steps that show it, with the commit you ran (`git rev-parse --short HEAD`)
- In a Dragoon clone, the fork's commit too (`git -C gcn-recomp rev-parse --short HEAD`)
- The machine and OS version you ran it on

Never attach a disc image or anything else that comes from a game. A file you made to show the problem is fine when nothing in it comes from a game.

## What counts

These are the places where code in these repos takes input it didn't make. Each one names the file that does it, with fork paths on its `dragoon` branch.

### Disc images

- The fork's `recomp/extract_dol.py` reads the image through `recomp/dol.py` at configure, and checks its disc ID and the SHA-1 of its `main.dol`. `recomp/rel.py` reads the image's modules.
- At run time the fork's `runtime/hw/di.cpp` reads the image, plain or CISO, for every disc read the game makes.
- Dragoon's `launcher/Sources/DragoonCore/DiscImage.swift` reads the disc a player picks in the launcher. It checks the disc when it's picked and again before each build or launch.

### Mods

- The fork's `recomp/riivolution.py` reads a folder of Riivolution packages. `recomp/riivolution_xml.py` parses each XML and `recomp/riivolution_paths.py` resolves its paths. `recomp/riivolution_patch.py` reads the file a memory patch's `valuefile` names.
- The fork's `recomp/dol_patch.py` reads a DOL patch list, and `recomp/disc_overlay.py` builds a disc overlay from a folder of files.
- At run time the fork's `runtime/hw/disc_overlay.cpp` reads an overlay's manifest and opens every host file it names. `runtime/ram_patches.cpp` reads a RAM patch list and writes it into the game's RAM at boot.
- Dragoon's `tools/mods.py` checks a mods folder, reading each mod's `files/` and its `dol.txt`.

### Saves

- The fork's `runtime/hw/memcard.cpp` reads the memory card, `saves/memcard_a.raw` by default. `runtime/hw/sram.cpp` reads `saves/sram.raw`.
- The fork's `runtime/gx/render_gl.cpp` reads the shader cache, `saves/shaders.bin`. On a Mac and in GL ES builds it can hand the program binaries in that file to the GL driver.
- The fork's `runtime/input_log.cpp` reads the folder `--replay=DIR` names, an input log with the saves its run started from.

### Settings

- The fork's `runtime/settings.cpp` reads `settings.txt` from the folder the game starts in. Its `disc_overlay` and `ram_patches` lines can name an overlay and a patch list.
- The fork's `runtime/input_map.cpp` reads `controls.txt`, and `runtime/port_memory.cpp` reads `saves/ports.txt`.
- When telemetry is on, the fork's `runtime/telemetry.cpp` reads `telemetry.txt`. `runtime/vr_config.cpp` reads `vr.txt` for `--eye` and the VR frontends.
- The fork's `runtime/gamepads.cpp` hands a `gamecontrollerdb.txt` beside the executable to SDL.

### Sockets

The game opens a socket in two cases, and neither one listens.

- `--bot=PATH` or `GCN_BOT` makes the fork's `runtime/bot_link.cpp` connect to the Unix socket at PATH. The bot on the other end reads the game's RAM and sets its pads.
- `--telemetry` or `GCN_TELEMETRY` makes the fork's `runtime/telemetry.cpp` send UDP packets of the game's motion to `127.0.0.1` port `33001`, or to the IPv4 address and port given. It never reads from that socket.

Two tools listen, each on a Unix socket rather than a network address. The fork's `tools/bot_script.py` listens at the path it's given. Dragoon's `tools/bot_link.py` listens at `bot.sock` in its run's folder under `build/probe/`, or at `/tmp/dragoon-bot-<hash>.sock` when that path is too long.

Some tests listen on Unix sockets too. The fork's `tools/test_bot_link.cpp` and `tools/test_bot_script.py` do it in folders of their own. Dragoon's `tools/test_bot_link.py` does it in temporary folders, and once at a `/tmp/dragoon-bot-<hash>.sock` path for a run whose folder is too long.

### Downloads

- Dragoon's configure runs the fork's, which downloads SDL 3.4.18's source unless `-DGCN_SYSTEM_SDL=ON`. It downloads Dear ImGui 1.92.9b too unless `-DGCN_OVERLAY=OFF`, and checks each against a SHA-256 in the fork's `CMakeLists.txt`.
- A Windows build downloads Khronos' OpenXR SDK 1.1.63 unless `-DGCN_OPENXR=OFF`, and a visionOS build downloads it for its headers. An Android build downloads Khronos' OpenXR loader 1.1.63 from Maven Central. Each has a SHA-256 in the fork's `CMakeLists.txt`.
- A visionOS configure runs the fork's `tools/build-angle-visionos.sh`, which fetches WebKit's ANGLE with git at a pinned commit. When the Metal compiler is missing it runs `xcodebuild -downloadComponent MetalToolchain`.
- Dragoon's configure with `-DDRAGOON_TESTS=ON` downloads GoogleTest 1.17.0, checked against a SHA-256 in Dragoon's `CMakeLists.txt`.
- Dragoon's launcher has an Install Tools button for when the Command Line Tools are missing. It runs `xcode-select --install`, which opens Apple's installer (`launcher/Sources/Dragoon/EffectRunner.swift`).
- The fork's `tools/fetch_dtk.sh` isn't part of the build. It downloads the latest decomp-toolkit release with curl and checks no hash.

## What doesn't count

- Something the game does the same way on a console
- A build someone changed, or one someone else made and passed on
- A bug in a downloaded library, which belongs with that library's own project
- A bug that isn't a security problem, which goes in [Dragoon's issues](https://github.com/lunchbox-recomp/dragoon/issues)

## What happens to a report

A report opens a private advisory in the repo's Security tab, where the reporter and the maintainer, Anthony Ledesma, can comment on it. Nobody else sees it unless he adds them to it or publishes it.
