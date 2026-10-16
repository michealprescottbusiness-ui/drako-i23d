# drako-i23d — image-to-3D on the Windows RTX 5070 laptop

Goal: turn our concept images (`inputs/`) into textured 3D models (`.glb`) with **free, local** image-to-3D models, and push them back to this repo. The Mac session rigs and animates them.

Machine: Windows, **RTX 5070 Laptop GPU (8 GB VRAM)**. 5070 = Blackwell (sm_120) → needs **PyTorch built for CUDA 12.8+** (cu128 wheels). Older cu121/cu124 wheels will fail with "no kernel image".

## Rules
- Free / open-source tools only. Don't sign up for anything, don't pay for anything, don't enter credentials.
- Don't change system settings beyond installing Python packages, Git, the CUDA toolkit or VS Build Tools if needed — ask the user first if a system-level install is required.
- Keep everything under one folder, e.g. `C:\drako-i23d\` (this repo) + `C:\drako-i23d\tools\` (clones, venvs). `tools/` is git-ignored.
- Log what you did (versions, errors, timings per model, VRAM peak if you can see it) in `outputs/LOG.md`.

## Inputs
| file | what |
|---|---|
| `inputs/mossling_front.png` | creature, front 3/4 view (main image) |
| `inputs/mossling_side.png` | same creature, side view |
| `inputs/mossling_back.png` | same creature, back 3/4 view |
| `inputs/trainer_front.png` | chibi kid, front view (main image) |
| `inputs/trainer_side.png` | side view |
| `inputs/trainer_back.png` | back view |

Grey backgrounds — let each tool's background remover handle them.

## Jobs, in priority order
1. **Hunyuan3D-2** (github.com/Tencent-Hunyuan/Hunyuan3D-2 — or 2.1 if it installs more easily). Fits 8 GB best.
   - Shape: use the **mini** / low-VRAM model if the full one runs out of memory (`Hunyuan3D-2mini`, `--low_vram_mode`, CPU offload).
   - Texture (paint) step: try it with low-VRAM / offload settings. If it truly can't fit, export the untextured shape and say so in the log.
   - Multi-view: Hunyuan3D-2**mv** takes front/side/back views — use all three inputs if you can get it running; otherwise the front image only.
   - Outputs: `outputs/mossling_hunyuan.glb`, `outputs/trainer_hunyuan.glb` (+ `_mv` variants if multi-view worked).
2. **TRELLIS** (github.com/microsoft/TRELLIS, MIT). Officially wants ~16 GB; try fp16 / low-VRAM community settings. Use its **multi-image** mode with all three views (`run_multi_image` / multiimage `stochastic`).
   - Outputs: `outputs/mossling_trellis_mv.glb`, `outputs/trainer_trellis_mv.glb`.
   - If it can't run in 8 GB after a reasonable try (~30 min), log why and move on.
3. *(Optional, if time allows)* **UniRig** (github.com/VAST-AI-Research/UniRig) auto-rig on `outputs/mossling_*.glb` → `outputs/mossling_unirig.glb`. The Mac already has a working Blender auto-rig, so this is a comparison only.

GLB requirements: texture embedded, roughly 10k–50k triangles is fine (the Mac decimates), Y-up, creature facing the camera that saw the front image.

## Hand-back
`git add outputs && git commit -m "outputs: <what>" && git push` after each finished model (so the Mac can start rigging early). GLBs are typically 2–20 MB — fine for git. If any single file goes over 90 MB, decimate it first.
