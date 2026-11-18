---
name: openjutsu
description: Set up, run and troubleshoot Openjutsu, a local open-source character and object swap for video built on a Next.js app over ComfyUI and MiniMax H3, and choose the right mode and settings for a swap. Use when the user mentions Openjutsu, wants to replace a person in a video with a character from a photo, swap several people, change or remove an object, hair, clothes or background in a video, make a video from a picture with relighting, fix small faces or harsh colours in an H3 video, or install its ComfyUI custom nodes and models.
---

# Openjutsu

Openjutsu replaces people or objects in a video while keeping the camera, cuts, motion and soundtrack. It's a Next.js app (`127.0.0.1:3000`) that sends ComfyUI workflows through `comfy-api-proxy` (`127.0.0.1:8189`) to ComfyUI (`127.0.0.1:8188`). Everything runs locally on the user's NVIDIA GPU.

## Before anything else

- Needs: an NVIDIA GPU (tested on an RTX 4060 Ti 16 GB with 64 GB RAM), ComfyUI 0.41.2 or newer, Node.js 22.6 or newer, and Python for the proxy.
- The required custom nodes and model files are listed in [references/requirements.md](references/requirements.md). Read it when installing or when the app reports missing nodes or models.
- Downloading models and installing custom nodes changes the user's ComfyUI install and can take many GB. Confirm with the user before doing it, and never overwrite an existing ComfyUI.
- Generation is slow: a 6-second clip at 480p takes about 6–13 minutes on a 4060 Ti. Set expectations before starting a job.

## Start it

**Windows:** with ComfyUI running, double-click `start_openjutsu.bat`. The first run installs `comfy-api-proxy` 0.1.6 into `.venv-proxy`, patches it and installs the node packages. Each run builds the app and opens the proxy and app windows plus the browser. Close those two windows to stop.

**Manual:**

1. Start ComfyUI on `127.0.0.1:8188`. On Windows set `AIOHTTP_NOSENDFILE=1` first (`$env:AIOHTTP_NOSENDFILE = "1"` in PowerShell). Without it, seeking in the video player can stall ComfyUI's file downloads until it restarts.
2. Start the proxy with the same Python as the patch:
   ```sh
   pip install comfy-api-proxy==0.1.6
   python openjutsu/patch_proxy.py
   comfy-api-proxy run --max-upload-mb 500
   ```
3. Start the app:
   ```sh
   cp .env.example .env.local
   npm ci
   npm run build
   npm start          # or npm run dev while changing code
   ```
4. Open http://127.0.0.1:3000.

`.env.local`: `COMFY_BASE_URL=http://127.0.0.1:8189` (the proxy); `COMFY_API_KEY` empty for local use; `COMFYUI_URL` points at ComfyUI itself, for the model and LoRA pickers; `OPENJUTSU_PASSWORD` is required before serving on any non-loopback address, at least 12 characters, with login name `openjutsu`. Never expose the app through a tunnel or reverse proxy without a password and HTTPS.

On the first visit, pick the **visual describer**: Qwen3-VL 8B INT8 by default, or 4B FP8 or Gemma 4 E2B INT8 for less VRAM.

## Pick the mode

| The user wants | Mode | Notes |
|---|---|---|
| One person replaced by a character photo | **Simple motion transfer** | Replaces the main (largest) person |
| 1–4 people, each from their own photo | **Advanced motion transfer** | Give roles like `the drummer - man in a cap`; only the part before ` - ` reaches H3. **One pass per person** (2 characters) is higher quality and about twice as slow |
| People taking turns within one shot (whip pans, handovers) | Advanced → **Montage** | **Find people** (about 3 min for 18 s) lets the user review the matches first |
| Change hair, clothes or a thing | **Object swap → Change** | Name it in a word or two (`hair, top`); add a picture to copy a look |
| Remove a thing | **Object swap → Remove** | Say what shows instead only when it's not obvious |
| New background, same person | **Object swap → Background** | |
| Add an object | **Object swap → Add** | Describe it and where it goes, then draw a box in the placement editor |
| A character posed from a video frame, then a video | **From picture** | Review the image, then **Use this image**. **Relight** offers 17 looks or a custom light |
| Small faces drawn badly | **Post-processing → Fix faces** | **Match the photo** for likeness, **Follow the mouth** to keep expressions |
| Harsh H3 look | **Post-processing → Fix colours** | Detail contrast plus the **H3 soft** tone-mapper preset |

## Settings that matter

- **Check mask** (Simple only) shows in about a minute per 3 s what will be regenerated: cyan is regenerated, orange is the tracked person. If the character is cut off or too much background changes, change **Mask margin** and check again.
- **Part to use** swaps only part of the video; shorter is faster.
- **Who do they replace?**: name the person by role (`the singer`), never by clothes, because clothing words leak into the result.
- **What to replace** (Simple): `person` by default, or `cat`, or a short description when the biggest match is the wrong one.
- **No lip-sync**: for songs the people don't sing along to.
- **Hat fix**: also replaces a hat, about 45 s more per clip.
- **Resolution**: 736p by default. 1080p is about 2.5× slower and needs much more memory. Custom sizes go from 256 to 2048 px.
- **Seed**: change it for a new take. The same inputs again resume from the last finished clip.
- **Steps, base model, LoRAs**: defaults are the H3 pruned INT8 model with the DMAD speed LoRA at 6 steps. Without a speed LoRA, use 20 or more steps.

## Known limits

- **Mirrors and reflections don't work yet**: H3 copies the original person's look back from the reflection. Suggest videos without one.
- People on screen for under 12 frames, or under 2% of the frame, are left unchanged in Montage.
- Uploads are limited to 512 MB per request.

## Troubleshoot

- **Missing nodes or models:** the app checks the workflow before uploading and lists missing packs (with links) and model files (with folders). Install them, restart ComfyUI, then try again. See [references/requirements.md](references/requirements.md).
- **Progress stops after a reconnect:** install `comfyui-websocket-reconnect-fix` and rerun `patch_proxy.py` with the proxy's Python.
- **Downloads or seeking hang on Windows:** ComfyUI was started without `AIOHTTP_NOSENDFILE=1`. Restart it with that set.
- **Model pickers are plain text fields:** `COMFYUI_URL` doesn't reach ComfyUI.
- **Out of memory:** lower the resolution, the motion reference size (512 px) or the clip length, or use a smaller describer.
- **Tests:** `npm test` checks the workflow pruning against the exported workflows.

Never claim a video finished without seeing the job's final output. Model licences are separate from Openjutsu's MIT licence: Fix faces uses InsightFace `buffalo_l`, which is for non-commercial research only, and AGPL-3.0 YOLO weights.
