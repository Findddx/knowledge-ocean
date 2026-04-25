# Image Attribution and Source Candidates

本文档用于记录 OS Hub 与 Hardware Hub 后续插图的来源、许可、适用章节和使用限制。当前不下载图片，只建立候选清单和归因模板。

## 1. Attribution Template

复制以下模板为每张实际插入的图片建立记录。

```markdown
## Image ID

- Local file:
- Original title:
- Original URL:
- Source type: official press/media kit | official product page | Wikimedia Commons | standards body | other
- Creator / copyright holder:
- License / terms:
- Required attribution text:
- Download date:
- Last license check:
- Intended hub / chapter:
- Intended use: cover | inline hardware photo | explanatory figure | diagram reference
- Cropping / edits:
- Trademark / endorsement notes:
- Redistribution risk: low | medium | high
- Approval status: candidate | approved | inserted | retired
- Notes:
```

## 2. Source Selection Rules

- Prefer official press/media kit pages for current commercial hardware when editorial or educational use is allowed.
- Prefer Wikimedia Commons for freely licensed photos where the exact file page lists author and license.
- Treat vendor product pages and PDFs as **candidate only** unless their terms explicitly allow reuse outside normal web viewing.
- Do not use third-party review photos unless the site provides a clear reuse license or permission.
- Do not use search-engine image URLs as sources; always record the original page URL.
- For Creative Commons images, preserve license version and required attribution. Watch for ShareAlike requirements.
- For official press assets, credit the company as copyright holder and avoid implying endorsement.

## 3. Inserted Images

## wikimedia-pci-express-slot

- Local file: `hubs/assets/images/hardware/12_pcie_cxl_interconnect/wikimedia-pci-express-slot.png`
- Original title: `File:Pci Express Slot.png`
- Original URL: <https://commons.wikimedia.org/wiki/File:Pci_Express_Slot.png>
- Source type: Wikimedia Commons
- Creator / copyright holder: Lucianolll
- License / terms: CC0 1.0 Universal Public Domain Dedication
- Required attribution text: Attribution not required by CC0, but source retained for traceability.
- Download date: 2026-04-25
- Last license check: 2026-04-25
- Intended hub / chapter: Hardware Hub / `12_pcie_cxl_interconnect`
- Intended use: inline hardware photo
- Cropping / edits: none
- Trademark / endorsement notes: none
- Redistribution risk: low
- Approval status: inserted
- Notes: Shows PCIe x16/x1 and PCI slot comparison.

## wikimedia-ssd-m2

- Local file: `hubs/assets/images/hardware/18_storage_hardware/wikimedia-ssd-m2.svg`
- Original title: `File:SSD M.2.svg`
- Original URL: <https://commons.wikimedia.org/wiki/File:SSD_M.2.svg>
- Source type: Wikimedia Commons
- Creator / copyright holder: Tom Cowap
- License / terms: Creative Commons Attribution-ShareAlike 4.0 International
- Required attribution text: SSD M.2.svg by Tom Cowap, licensed under CC BY-SA 4.0, via Wikimedia Commons.
- Download date: 2026-04-25
- Last license check: 2026-04-25
- Intended hub / chapter: Hardware Hub / `18_storage_hardware`
- Intended use: inline explanatory figure
- Cropping / edits: none
- Trademark / endorsement notes: none
- Redistribution risk: low, subject to attribution and ShareAlike
- Approval status: inserted
- Notes: SVG line illustration of an M.2 SSD. If modified, derivative should remain under CC BY-SA compatible terms.

## 4. First-Batch Hardware Image Source Candidates

| Area | Candidate source | License / use notes | Suitable chapters |
| --- | --- | --- | --- |
| CPU package / die / wafer | Intel Newsroom Press Kits Archive: <https://newsroom.intel.com/press-kits/> | Official press assets. Intel press hub notes image assets must credit Intel Corporation; Intel terms indicate editorial/educational use may be allowed subject to terms. Verify per asset before insertion. | `09_cpu_architecture`, `14_server_platform` |
| CPU die / wafer manufacturing | Intel Manufacturing Media Kit Photos: <https://www.intel.com/pressroom/archive/photos/manufacturing_photos.htm> | Official Intel archive. Good for wafer/die process visuals; use with Intel credit and terms check. | `09_cpu_architecture`, OS memory/kernel cover concepts |
| CPU package / chiplet architecture | AMD Newsroom / Media Library: <https://www.amd.com/en/newsroom/media-library.html> | AMD Brand Assets Terms govern use; official assets are not the same as open CC content. Record exact terms and asset page. | `09_cpu_architecture`, `13_gpu_accelerator_platform` |
| AMD EPYC product visuals | AMD EPYC product page: <https://www.amd.com/epyc> | Official product imagery, likely restricted by AMD site terms. Candidate for reference or permitted editorial use after terms check. | `09_cpu_architecture`, `14_server_platform` |
| DIMM / RAM modules | Wikimedia Commons Category DIMM: <https://commons.wikimedia.org/wiki/Category:DIMM> | File-level licenses vary. Commons generally hosts free licenses, but each file must be checked for author, CC variant, attribution and ShareAlike. | `10_memory_hierarchy`, `14_server_platform` |
| RAM modules broader pool | Wikimedia Commons Category RAM modules: <https://commons.wikimedia.org/wiki/Category:RAM_modules> | File-level license required. Good fallback for DDR/FB-DIMM/RDIMM physical comparison. | `10_memory_hierarchy` |
| PCIe connector / slot | Wikimedia Commons Category PCI Express connectors: <https://commons.wikimedia.org/wiki/Category:PCI_Express_connectors> | File-level license required. Useful for showing slot/keying and lane-length physical form. | `12_pcie_cxl_interconnect`, `14_server_platform` |
| CXL memory expander | Samsung Semiconductor CXL news: <https://semiconductor.samsung.com/newsroom/news/samsung-unveils-industry-first-memory-module-incorporating-new-cxl-interconnect-standard/> | Official Samsung newsroom image; check Samsung Semiconductor Newsroom terms before reuse. Candidate for CXL hardware appearance. | `12_pcie_cxl_interconnect`, `10_memory_hierarchy` |
| CXL product / module family | Samsung CMM-D product page: <https://semiconductor.samsung.com/us/cxl-memory/cmm-d/> | Official product imagery; likely restricted. Use only after confirming terms or as source reference. | `12_pcie_cxl_interconnect` |
| GPU accelerator | NVIDIA Newsroom Media Assets: <https://nvidianews.nvidia.com/multimedia/> | Official NVIDIA media assets; terms may be asset-specific. Credit NVIDIA and avoid endorsement implication. | `13_gpu_accelerator_platform` |
| GPU accelerator, older freely licensed examples | Wikimedia Commons Category Nvidia Tesla series: <https://commons.wikimedia.org/wiki/Category:Nvidia_Tesla_series> | File-level license required. Useful for physical accelerator card form factor where current official images are too restricted. | `13_gpu_accelerator_platform` |
| AMD Instinct accelerator | AMD Instinct MI300 product page: <https://www.amd.com/en/products/accelerators/instinct/mi300.html> | Official AMD product imagery, governed by AMD terms. Candidate for product appearance, not open reuse by default. | `13_gpu_accelerator_platform` |
| Server motherboard / platform | Wikimedia Commons Category Computer motherboards: <https://commons.wikimedia.org/wiki/Category:Computer_motherboards> | File-level license required. Includes general and server motherboard subcategories. | `14_server_platform`, `11_firmware_boot_bmc` |
| Server hardware / racks | Wikimedia Commons Category Server hardware: <https://commons.wikimedia.org/wiki/Category:Server_hardware> | File-level license required. Good for rack/server exterior context. | `14_server_platform`, `16_power_cooling_physical` |
| Supermicro server / motherboard product visuals | Supermicro Product Resources: <https://www.supermicro.com/en/support/product-resources> | Official vendor resources. Product imagery/manual diagrams are not automatically reusable; verify terms or request permission. | `14_server_platform`, `23_raid_hotspare_rebuild` |
| NIC / RDMA adapter | NVIDIA ConnectX Ethernet adapters: <https://www.nvidia.com/en-us/networking/multi-host/> | Official product imagery; check NVIDIA terms. Strong source for modern RDMA/SuperNIC visuals. | `15_network_rdma` |
| InfiniBand adapters | NVIDIA InfiniBand adapters: <https://www.nvidia.com/en-in/networking/infiniband-adapters/> | Official product imagery; check reuse terms and regional page consistency. | `15_network_rdma`, `13_gpu_accelerator_platform` |
| NVMe / SSD | Wikimedia Commons storage and SSD-related categories via Computer hardware: <https://commons.wikimedia.org/wiki/Category:Computer_hardware> | Use exact file page after selecting SSD/NVMe image; license varies by file. | `18_storage_hardware`, `24_ssd_namespace_zoned`, `26_nvmf_spdk_userspace_storage` |
| HDD | Wikimedia Commons Category Hard disks: <https://commons.wikimedia.org/wiki/Category:Hard_disks> | File-level license required. Good for platter/head/internal structure photos. | `18_storage_hardware`, `22_storage_health_telemetry` |
| RAID / JBOD diagrams | Wikimedia Commons Category RAID: <https://commons.wikimedia.org/wiki/Category:RAID> | File-level license required. Many are diagrams rather than photos; useful as reference, but Mermaid may be cleaner for final diagrams. | `23_raid_hotspare_rebuild`, `28_backup_snapshot_clone_replication` |
| JBOD / backplane product context | RAID Inc. Ability 4U 106-Bay JBOD: <https://www.raidinc.com/products/ebods/4u-106-bay-ability-ebod/> | Vendor product imagery, likely restricted. Candidate for permission-based or reference-only use. | `23_raid_hotspare_rebuild`, `18_storage_hardware` |
| Backplane / JBOD manual diagrams | Supermicro SC846 JBOD manual: <https://www.supermicro.org.cn/manuals/chassis/4U/SC846_JBOD_1.0.pdf> | Manual diagrams are copyrighted unless terms permit reuse. Useful as technical reference; prefer redrawing concepts in Mermaid/original diagrams. | `23_raid_hotspare_rebuild`, `14_server_platform` |

## 5. General License References

- Wikimedia Commons licensing policy: <https://commons.wikimedia.org/wiki/Commons:Licensing>
- Intel Terms of Use: <https://www.intel.com/content/www/us/en/legal/terms-of-use.html>
- Intel Press Hub: <https://newsroom.intel.com/press-hub>
- AMD Brand Assets Terms and Conditions: <https://www.amd.com/en/newsroom/media-library.html>
- Samsung Semiconductor Newsroom Terms of Use: <https://news.samsungsemiconductor.com/global/terms-of-use/>
- NVIDIA Newsroom Media Assets: <https://nvidianews.nvidia.com/multimedia/>

## 6. Practical Approval Checklist

- Exact original page is recorded, not only an image CDN URL.
- License or terms are recorded from the page as checked on the insertion date.
- Attribution text is ready to place near the image or in a page-level credits section.
- The image supports a concrete explanation in the chapter.
- The image is not merely decorative when a diagram would explain the concept better.
- Any crop preserves product identity, connector shape, or technical feature being discussed.
