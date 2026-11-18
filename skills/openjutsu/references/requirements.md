# Openjutsu requirements

## Custom nodes

Install into `ComfyUI/custom_nodes`, then restart ComfyUI. ComfyUI Manager finds the first three by name.

| Pack | Used for |
|---|---|
| [SolarRenderer](https://github.com/Enderfga/SolarRenderer/tree/main/comfyui) | From picture: SolarRenderer LoRA loading, conditioning and four-step sampling |
| [ComfyUI-MiniMaxH3-Context-Loop](https://github.com/ethanfel/ComfyUI-MiniMaxH3-Context-Loop) 0.7.3+ | Clip loop, checkpoints, final assembly |
| [ComfyUI-Fantastic-MiniMaxH3-PromptBuilder](https://github.com/Adudeguyman/ComfyUI-Fantastic-MiniMaxH3-PromptBuilder) | Masked video-edit latent |
| [ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes) | Grow Mask With Blur; From picture VRAM cleanup |
| [comfyui_controlnet_aux](https://github.com/Fannovel16/comfyui_controlnet_aux) | From picture: Metric3D normal maps |
| [ComfyUI-VideoHelperSuite](https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite) | Video loading |
| [comfyui-h3-oneclick-swap](https://github.com/D3voz/comfyui-h3-oneclick-swap) | Swap and object plans, people and role masks, mannequins, track labelling, cut finder, shot list, face-fix chunks |
| [ComfyUI-H3-FaceRefine](https://github.com/Carasibana/ComfyUI-H3-FaceRefine) | Fix faces: face tracking, cropping, SAM face masks, pasting back |
| [ComfyUI-Impact-Pack](https://github.com/ltdrdata/ComfyUI-Impact-Pack) | Fix faces: SAM loader |
| [comfyui-image-detail-contrast](https://github.com/D3voz/comfyui-image-detail-contrast) | Fix colours: Image Detail Contrast |
| [Orion4D_FXMax](https://github.com/orion4d/Orion4D_FXMax) | Fix colours: DCTL Tone Mapper and presets |
| [comfyui-websocket-reconnect-fix](https://github.com/D3voz/comfyui-websocket-reconnect-fix) | Every job: live progress after the proxy reconnects |

Optional: [ComfyUI-Spectrum-MiniMax-H3](https://github.com/xmarre/ComfyUI-Spectrum-MiniMax-H3) for Acceleration → Spectrum, which can't be used with an active speed LoRA.

## Models

| Folder in `ComfyUI/models` | File |
|---|---|
| `diffusion_models` | `minimax_h3_ref2va_pruned_int8_convrot.safetensors` |
| `loras` | `minimax_h3_DMAD_4step_full_lora_avg_rank_39_bf16.safetensors` |
| `text_encoders` | `qwen3vl_32b_minimax_h3_int4_convrot.safetensors` (H3 text encoder) |
| `text_encoders` | `qwen3vl_8b_int8_convrot.safetensors` (visual describer) |
| `vae` | `minimax_h3_video_vae_int8_convrot.safetensors`, `minimax_h3_audio_vae_fp32.safetensors` |
| `checkpoints` | `sam3.1_multiplex_fp16.safetensors` |
| `ultralytics/bbox`, `ultralytics/segm` | `face_yolov8m.pt`, `person_yolov8m-seg.pt` from [Bingsu/adetailer](https://huggingface.co/Bingsu/adetailer), for Fix faces |
| `sams` | `sam_vit_b_01ec64.pth`, for Fix faces |
| `insightface/models` | `buffalo_l`, downloaded by InsightFace on first use, for Fix faces |

**From picture → SolarRenderer** also needs `solarrenderer_4step_lora.safetensors` and `solarrenderer_control_lora.safetensors` from [Enderfga/SolarRenderer](https://huggingface.co/Enderfga/SolarRenderer), or the rank-64 pair from [mdevoz/SolarRendererRank64](https://huggingface.co/mdevoz/SolarRendererRank64), in `ComfyUI/models/loras` or a subfolder. Then reload the app and choose their paths in the render settings. See `workflows/picture/README.md` for the full From picture requirements.

Alternative visual describers must be single-file models with their vision weights, in `models/text_encoders`, loadable by ComfyUI's CLIPLoader and Generate Text nodes. Press **Refresh models** in the app after adding one.
