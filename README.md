# ztrx

**ztrx** runs the Xbox Series build of the game natively on Windows PCs with
NVIDIA graphics cards, from game files you supply yourself.

**This repository contains no game files or assets.** You need your own copy
of the game. Not affiliated with or endorsed by its developer or publisher.

## Status (2026-11-25)

Tested on Windows 11 with an **NVIDIA GeForce RTX 3050** (8 GB):

| | |
|---|---|
| Boot, first-boot screens, intro, opening credits | works |
| Prologue | playable from the cell to the boat chase: cell, cafeteria, robbery, police chase (wanted level, minimap), boat, binoculars |
| Stability | 1 h 30 min+ in one session without a crash |
| Frame rate | 1-3 fps in the world (the first run is slower while shaders translate; they are cached after that) |
| Known problems | anything in shadow is black and bright scenes overexpose (the game's ray-traced lighting does not run yet); some blocky artifacts; boats sit low in the water |

## Requirements

- Windows 10/11 x64, a CPU with AVX2
- A Direct3D 12 GPU with shader model 6.6 and mesh shaders (NVIDIA RTX 20 series or newer)
- Rust (the version in `rust-toolchain.toml`)
- Several GB of free Windows commit memory (raise the paging file if `crashes.log` reports error 1455)

## Build

```powershell
cargo build --release -p victory
```

## Play

```powershell
powershell -ExecutionPolicy Bypass -File scripts\play.ps1 -Game "<your game folder>\<the game's exe>"
```

`scripts\play.ps1` keeps the shader cache and saves in folders you choose
(`-Cache`, `-Data`) and presses no buttons for you. Controller: XInput pads
(a DualShock/DualSense through DS4Windows works). With no pad connected the
keyboard acts as one while the game window has focus.

## Performance branch (`perf-aggressive`)

Experimental, aimed at frame rate over picture. Every switch below can be
turned back off by setting it to the value shown.

| Change | Turn off |
|---|---|
| CPU and GPU overlap: submissions sent ahead, copy-back on its own thread (`VICTORY_RELAY=2` is now the default) | `VICTORY_RELAY=0` (or `=1` for the first stage only) |
| Snapshots/compares, refreshes and merges spread over several threads | `VICTORY_WIDE_BACK=0`, `VICTORY_WIDE_REFRESH=0`, `VICTORY_WIDE_MERGE=0` |
| Texture refreshes do not wait for the runtime's remapping lock | `VICTORY_TEXTURE_NO_WAIT=0` |
| Adjacent title buffer copies joined into one | `VICTORY_JOIN_COPIES=0` |
| GPU re-hash passes joined into runs | `VICTORY_CARD_REHASH_RUNS=0` |
| GPU compare/hash shader: one 256-thread group per 64 KiB piece instead of one thread | (code change) |
| No EcoQoS throttling (hybrid CPUs keep the game on performance cores) and above-normal process priority | `VICTORY_PRIORITY=0` (priority only) |
| Kept worker threads for snapshots/compares/merges: half the logical cores, up to 10 (was a quarter, up to 6) | (code change) |
| Release build: fat LTO, one codegen unit, `x86-64-v3` (AVX2) | edit `Cargo.toml` / `.cargo/config.toml` |

Check the compare shader on Windows with
`cargo test --release -p victory-gpu warp_card_compare`.

## Debugging switches

| Switch | Effect |
|---|---|
| `VICTORY_CAPTURE=<dir>` + `VICTORY_CAPTURE_FRAMES=<list>` | save those frames as PNG (diagnostics build) |
| `VICTORY_KEYS=30:Enter,35:X` | press keys at those seconds (diagnostics build) |
| `VICTORY_D3D12_DRED=1` | on a GPU failure, report the commands in flight |
| `VICTORY_PIPELINE_WAIT_MS=24` | shorter per-frame wait for pipelines still being built |

Diagnostics build: `cargo build --profile diag -p victory --features diagnostics`

## License

GPL-3.0-or-later (see `LICENSE`). Based on VIctory (Marvisak) and victory-fork (fxceless).
