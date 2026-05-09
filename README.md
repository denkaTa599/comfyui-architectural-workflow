# Advanced Arch-Viz Flux Workflow for ComfyUI

This repository contains `Advanced_Flux_Arch_Workflow.json`, a highly optimized ComfyUI workflow tailored specifically for architectural visualization.

## Goal
The purpose of this workflow is to transform a basic, low-resolution 3D viewport render into a high-quality, photorealistic architectural image. It preserves the geometry of the original 3D render while applying realistic materials, lighting, and environmental context guided by a reference mood board.

## System Requirements
This workflow has been specifically designed and optimized for a system with:
- **GPU:** NVIDIA RTX 5080 (or equivalent with 16GB VRAM)
- **RAM:** 64GB DDR5
- **CPU:** AMD Ryzen 7 7700 8-core or similar.

By utilizing FP8 weights for the Flux UNET and implementing Ultimate SD Upscale with tiling, we keep VRAM usage safely within the 16GB limit while producing massive, highly-detailed outputs.

## Workflow Architecture & Node Choices

### 1. Base Model & Precision
*   **Flux.dev UNET (FP8):** We load the `flux1-dev-fp8.safetensors` model. Using FP8 quantization is crucial for fitting Flux into 16GB VRAM while retaining immense detail.
*   **Dual CLIP (T5 FP8):** We use `t5xxl_fp8_e4m3fn.safetensors` and `clip_l.safetensors`. The T5 text encoder is massive, so loading it in FP8 format alongside the UNET ensures we don't hit out-of-memory (OOM) errors.

### 2. Style & Realism Enhancement
*   **Architecture Realism LoRA:** A specialized architectural LoRA (e.g., `flux_archviz.safetensors`) is injected with a strength of 0.85. This biases the model heavily towards professional architectural photography (V-Ray style, global illumination, ray tracing).
*   **IP-Adapter with SigLIP Vision:** `ip-adapter-flux-dev.safetensors` paired with `siglip_vision_patch14_384.safetensors`. We use a *Mood Reference Image* as input here. This forces the model to borrow materials (concrete, glass, wood) and lighting (e.g., sunset, golden hour) from the reference image, ensuring the generated materials look physically accurate.

### 3. Geometry Control
*   **Canny Edge ControlNet:** `flux-canny-controlnet.safetensors`. For architecture, preserving straight, rigid lines from the original 3D model is paramount. Depth maps can sometimes blur sharp corners, so a Canny Edge preprocessor (`low_threshold: 100`, `high_threshold: 200`) is used to extract precise edges from your low-res 3D input. This ensures the output structural design is identical to your CAD/3D model.

### 4. Generation & Upscaling
*   **Image-to-Image Base Pass (KSampler):** The low-res 3D input is encoded and run through a KSampler with a denoise of `0.65`. This is high enough to let Flux invent beautiful materials and lighting, but low enough (when paired with Canny ControlNet) to maintain structural integrity.
*   **Ultimate SD Upscale:** We upscale the initial output by `2.0x` using the `4x-UltraSharp.pth` upscaler model. Ultimate SD Upscale processes the image in `1024x1024` tiles, preventing VRAM spikes on the 16GB GPU. `4x-UltraSharp` is chosen because it excels at retaining crisp, un-blurred edges (unlike some anime or organic-focused upscalers), which is perfect for architecture.
*   **Color Matching Post-Processing:** An `ImageColorMatch` node applies the exact color grading of your mood reference back onto the final upscaled image.

## Required Models to Download
To run this workflow, ensure you have the following in your ComfyUI `models` folders:

*   **UNET:** `flux1-dev-fp8.safetensors`
*   **CLIP:** `t5xxl_fp8_e4m3fn.safetensors`, `clip_l.safetensors`
*   **VAE:** `ae.safetensors`
*   **LoRA:** `flux_archviz.safetensors` (or your preferred architecture LoRA)
*   **ControlNet:** `flux-canny-controlnet.safetensors`
*   **IP-Adapter:** `ip-adapter-flux-dev.safetensors`
*   **CLIP Vision:** `siglip_vision_patch14_384.safetensors`
*   **Upscaler:** `4x-UltraSharp.pth`

## Usage Instructions
1.  Open ComfyUI and drag-and-drop `Advanced_Flux_Arch_Workflow.json` into the workspace.
2.  On the **"Input: Base 3D Viewport"** node, upload your low-res, untextured (or basic textured) 3D viewport render.
3.  On the **"Input: Material/Mood Reference"** node, upload a high-quality photograph or render that has the materials, colors, and lighting you want to achieve.
4.  Optionally tweak the Positive/Negative prompts to match your specific building style (e.g., "modern concrete villa, sunset" vs "brick townhouse, overcast").
5.  Click **Queue Prompt** and wait for the high-resolution masterpiece.
