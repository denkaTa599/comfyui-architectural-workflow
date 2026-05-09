# Advanced Arch-Viz Flux Workflow for ComfyUI

This repository contains `Advanced_Flux_Arch_Workflow.json`, a highly creative and optimized ComfyUI workflow tailored specifically for architectural visualization.

## Goal
The purpose of this workflow is to transform a basic, low-resolution 3D viewport render into a high-quality, photorealistic architectural masterpiece. It uses advanced multi-conditioning techniques to dictate materials, mood, and precise geometry.

## System Requirements
Optimized for:
- **GPU:** NVIDIA RTX 5080 (or equivalent with 16GB VRAM)
- **RAM:** 64GB DDR5
- **CPU:** AMD Ryzen 7 7700 8-core or similar.

## Advanced Workflow Architecture

### 1. The Multi-LoRA Stack
To achieve true photorealism, relying on a single LoRA isn't enough. We use a **3-LoRA stack**:
1.  **ArchViz Base (`strength: 0.65`):** Dictates the structural, architectural photography aesthetic (V-Ray styling).
2.  **Cinematic Lighting (`strength: 0.45`):** Modifies the global illumination to create dramatic, volumetric light (e.g., golden hour or moody overcast).
3.  **Hyper Materials (`strength: 0.50`):** Adds micro-details to surfaces, ensuring concrete looks porous and glass looks appropriately reflective.

### 2. The Multi-IP-Adapter Pipeline
Materials are everything in architecture. We use **three chained IP-Adapters**, each with a specific purpose:
1.  **Primary Material (Concrete/Wood):** A reference image specifically for the main building texture. Applied early in the generation (`start: 0.0, end: 0.60`).
2.  **Mood/Lighting Reference:** An image depicting the desired atmosphere. Applied heavily throughout the generation (`start: 0.1, end: 0.85`).
3.  **Secondary Material (Glass/Metal):** A reference for facade reflections. Applied later to dictate surface finish (`start: 0.3, end: 0.90`).

### 3. Dual ControlNet Geometry Preservation
A single ControlNet cannot understand both rigid lines and volumetric space perfectly. We use a **Dual ControlNet setup**:
1.  **Canny Edge ControlNet (`strength: 0.75`):** Extracts precise, hard lines from your 3D viewport. This ensures the walls, window frames, and rooflines remain exactly as modeled.
2.  **Depth Anything V2 ControlNet (`strength: 0.60`):** Understands the spatial relationships (foreground vs background, overhangs). This helps Flux understand where shadows should drop and how light bounces between surfaces.

### 4. 3-Stage Generation & Upscaling
To squeeze massive detail out of 16GB VRAM, the generation happens in three stages:
1.  **Img2Img Base Pass:** Uses the dual ControlNets and triple IP-Adapters to imagine the building at a base resolution.
2.  **Latent Refinement:** Upscales the latent image by 1.5x and runs a low-denoise (`0.35`) pass to bake in micro-details (like brick mortar or glass imperfections) before pixel decoding.
3.  **Ultimate SD Upscale:** Uses `4x-UltraSharp` upscaler in `1024x1024` tiles, upscaling by a further 2.0x. Tiling ensures we never hit an Out-Of-Memory (OOM) error while producing 8k level imagery.

### 5. Final Color Grade
An `ImageColorMatch` node takes the color palette of your Mood Reference Image and mathematically applies it to the final high-res output, ensuring the mood is perfectly captured.

## Usage
1. Load `Advanced_Flux_Arch_Workflow.json` into ComfyUI.
2. Provide your base 3D image.
3. Provide three specific reference images (Main Material, Mood/Lighting, Secondary Material).
4. Queue Prompt.
