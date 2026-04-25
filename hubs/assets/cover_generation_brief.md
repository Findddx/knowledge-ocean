# Cover Generation Brief

This brief defines prompts for the first OS Hub and Hardware Hub cover images. It is public and contains no generated files, private links, or operational logs.

## Shared Direction

Use generated raster covers for high-level hub identity and chapter openings. Use Mermaid or hand-authored diagrams for precise technical structure. Covers should feel like refined technical publishing, not generic AI art. The active hub covers are PNG assets under `hubs/assets/covers/`; do not use SVG placeholders for hub covers.

Avoid:

- Neon palettes, electric purple/blue gradients, glowing brains, floating random circuits.
- Fake chip labels, fake product logos, fake UI panels, unreadable decorative text.
- Overcrowded components or photorealistic claims that imply a real product.

Prefer:

- Restrained editorial composition.
- Matte graphite, warm white, muted teal, copper, amber, and soft steel blue.
- One strong focal structure, generous negative space, subtle depth.
- No visible text inside the image; titles should be added by the document platform if needed.

## OS Hub Total Cover

Current asset: `hubs/assets/covers/os-hub-cover.png`

```text
Use case: scientific-educational
Asset type: knowledge hub cover
Primary request: Create an abstract editorial cover for an operating systems learning hub.
Scene/backdrop: A quiet, layered technical landscape showing user space, kernel boundary, scheduling queues, memory pages, file I/O paths, and observability traces as clean architectural forms.
Subject: A central matte kernel core surrounded by subtle rings for syscalls, processes, memory, filesystems, containers, and logs. The forms should be abstract, not literal product hardware.
Composition: Wide cover composition, one clear focal center, strong structure, enough negative space for a page title outside the image.
Palette: graphite black, warm off-white, muted teal, copper, and small amber highlights.
Style: restrained editorial technical illustration, premium research manual, precise layered geometry, soft studio lighting, matte materials.
Constraints: no text, no logos, no glowing brain, no neon, no purple-blue gradient, no cyberpunk, no fake code, no random circuit clutter, no people.
```

## Hardware Hub Total Cover

Current asset: `hubs/assets/covers/hardware-hub-cover.png`

```text
Use case: scientific-educational
Asset type: knowledge hub cover
Primary request: Create an abstract editorial cover for a hardware and infrastructure learning hub.
Scene/backdrop: A composed technical still life inspired by server platforms: CPU package, DIMM slots, PCIe lanes, GPU accelerator geometry, NIC, storage backplane, airflow and power paths, all abstracted into clean non-branded forms.
Subject: A layered server-board topology viewed from a slightly elevated angle, with modules arranged like an elegant engineering map rather than a real motherboard photo.
Composition: Wide cover composition, balanced asymmetry, one strong hardware topology focus, visible depth but no clutter.
Palette: graphite, warm white, brushed steel, muted teal, copper, and small amber diagnostic accents.
Style: refined technical publication cover, matte industrial materials, precise geometry, soft shadows, high design quality, calm and credible.
Constraints: no text, no logos, no fake model numbers, no neon, no cheap AI glow, no blue-purple gradient, no random circuit background, no photorealistic claim to a real product.
```

## Chapter Cover Rule

Only generate a chapter cover when it helps the reader understand or remember the chapter. Good candidates:

- OS: scheduling, virtual memory, VFS/I/O path, container isolation, observability.
- Hardware: CPU architecture, memory hierarchy, PCIe/CXL, GPU platform, network/RDMA, storage hardware, Ceph/failure domains, cyber recovery vault.

For chapters where a deterministic diagram is clearer, create Mermaid or a hand-authored explanatory figure instead of a generated cover.
