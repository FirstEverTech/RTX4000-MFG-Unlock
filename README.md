# NVIDIA GeForce RTX 4000 Series — DLSS 4 Multi Frame Generation 3x/4x Research

Lately I've been diving into various projects — some of you might know my work on:

- **[Universal Intel Chipset Device Updater](https://github.com/FirstEverTech/Universal-Intel-Chipset-Updater)**
- **[Universal Intel Wi-Fi and Bluetooth Drivers Updater](https://github.com/FirstEverTech/Universal-Intel-WiFi-BT-Updater)**

This time, I got interested in something that has been bothering me for a while:

## **DLSS 4 Multi Frame Generation 3x/4x on RTX 4000 series cards**

<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/d3197e51-cd9c-49b9-a389-af9c446c2f82" />

---

## Why this project exists

It is hard not to notice that NVIDIA increasingly uses software features as product differentiators between generations.

At the same time:

- GPU prices have increased
- native generational performance gains are often modest
- software-locked features have become more important in product segmentation

For RTX 5000, one of the most visible examples is **higher-order Frame Generation (3x/4x)**.

That raises a very natural technical question:

> **Is Multi Frame Generation 3x/4x on RTX 4000 truly blocked by hardware, or is at least part of the limitation enforced in software?**

---

## What this repository is about

This repository documents my research into forced **DLSS 4 Multi Frame Generation 3x/4x** behavior on **GeForce RTX 4000 series** GPUs.

The project currently focuses on three things:

1. documenting observed runtime behavior
2. validating frame-presentation behavior using **NVIDIA Nsight Systems 2026.2.1**
3. investigating the driver-side black-image enforcement path

This is **not** a ready-made unlock package.

It is a public research log.

---

## Current status in one paragraph

At this point, my working conclusion is:

> **Forced MFG 3x/4x on the tested RTX 4070 Ti appears to generate and present additional frames at the pipeline level, but output is obscured by a black-image enforcement path that still has not been fully neutralized.**

That conclusion is based on:
- repeatable benchmark behavior,
- public 3DMark runs,
- local Cyberpunk 2077 testing,
- and direct Nsight Systems analysis of frame presentation.

---

## Basic activation path

Activating FG 3x/4x on RTX 4000 is not especially difficult from the front-end side.

**[NVIDIA Profile Inspector](https://github.com/Orbmu2k/nvidiaProfileInspector)** can be used to set the Frame Generation multiplier globally, bypassing the lock in NVIDIA App.

<img width="1000" height="693" alt="image" src="https://github.com/user-attachments/assets/fadfc3cb-9865-4500-85d5-a278afbc6c5a" />

The setting then appears in NVIDIA App afterwards (read-only).

<img width="1298" height="942" alt="image" src="https://github.com/user-attachments/assets/85129b4f-39b9-4258-b5bd-fb13da1f48d3" />

---

## Important update — my initial `ShowEvalModeOverlay` theory was wrong

I performed **static analysis of `nvlddmkm.sys`** using [**Ghidra**](https://github.com/NationalSecurityAgency/ghidra), a tool developed by the National Security Agency (the NSA).

<img width="1623" height="924" alt="Ghidra" src="https://github.com/user-attachments/assets/0681a4d6-add6-4d17-bbc5-b71fabcbffa0" />

An earlier version of this repository suggested that the `ShowEvalModeOverlay` string inside `nvlddmkm.sys` might be directly responsible for the full-screen black output shown after enabling **MFG 3x/4x** on **RTX 4000 series** GPUs.

After further research, I can state clearly:

> **That assumption was incorrect.**

The tested byte-level control-flow modification near that code path did **not** disable the black image / black screen behavior.

What it actually disabled was the **NVIDIA App overlay menu** functionality — specifically the in-game overlay used for:

- recording,
- screenshots,
- performance statistics,
- and related overlay UI features.

So `ShowEvalModeOverlay` turned out to be a **false lead** with respect to the black output path.

However, this was still useful research, because it revealed a few important things:

- a precise low-level modification can alter driver behavior **without breaking the driver itself**
- a carefully targeted invasive patch does **not necessarily corrupt or destabilize** the modified NVIDIA kernel driver
- during this research, I also confirmed that it is possible to install **newly modified kernel drivers on Windows even with Secure Boot enabled**  
  *(details are intentionally not disclosed here)*

So while that branch was **not** the black-screen switch, the work still uncovered valuable information about driver robustness and patch behavior.

---

## The main issue — black image / black output enforcement

After enabling **Frame Generation 3x/4x on RTX 4000 series**, an unexpected behavior consistently appears:

> **A full-screen black image is rendered on top of the output, despite Multi Frame Generation appearing to function underneath.**

### Observed behavior

- NVIDIA App reports expected FPS scaling
- the game remains interactive
- frame pacing remains consistent
- audio continues normally
- benchmarks correctly detect generated frames

This strongly suggests that the render path is **not simply crashing**.

Instead, the output appears to be **deliberately obscured**.

---

## External reference that helped motivate this research

The main outside reference that pushed me to keep digging is this video from Chinese media:

- **Bilibili:** https://www.bilibili.com/video/BV1WZfZYxEtQ/?spm_id_from=333.1387.homepage.video_card.click

This repository intentionally uses that video only as **external context / motivation**, not as the core proof.
My own observations suggest that what is shown there is real and directionally consistent with my local results.
Still, the strongest evidence collected here comes from local testing and Nsight analysis.

---

## Benchmark evidence

### 3DMark NVIDIA DLSS Feature Test

| DLSS FG Mode | DLSS Off | DLSS On | Multiplier |
|---------------------|----------|---------|------------|
| Frame Generation 2x | 32.27 FPS | 125.16 FPS | 3.9x |
| Frame Generation 3x | 32.16 FPS | 224.02 FPS | 7.0x |
| Frame Generation 4x | 32.20 FPS | 291.56 FPS | 9.1x |

| [DLSS 4 Frame Generation 2x](https://www.3dmark.com/nd/580395) | [DLSS 4 Frame Generation 3x](https://www.3dmark.com/nd/580396) | [DLSS 4 Frame Generation 4x](https://www.3dmark.com/nd/580397) |
|:---------------:|:--------------:|:-------------------:|
| ![DLSS 4 Frame Generation 2x](https://github.com/user-attachments/assets/2ac22f03-4869-4302-8d53-a58d850e5250) | ![DLSS 4 Frame Generation 3x](https://github.com/user-attachments/assets/637b755e-240b-4ff9-8c1d-7e1120cef947) | ![DLSS 4 Frame Generation 4x](https://github.com/user-attachments/assets/9ab7252c-940a-4849-8d4c-7aa2eba55b09) |


**Official 3DMark results (verified, public):**  
https://www.3dmark.com/compare/nd/580395/nd/580396/nd/580397

<img width="1480" height="783" alt="3DMark_2x_3x_4x" src="https://github.com/user-attachments/assets/a93c3096-6e3b-4fb6-8293-72fb88704616" />

### Cyberpunk 2077 Built-in Benchmark — Ray Tracing: Overdrive, 4K

| Mode | Avg FPS | Min FPS | Max FPS |
|------|--------:|--------:|--------:|
| FG 2x | 75.83 | 69.78 | 82.18 |
| FG 3x | 124.49 | 113.24 | 136.96 |
| FG 4x | 164.68 | 149.75 | 180.54 |

| DLSS 4 Frame Generation 2x | DLSS 4 Frame Generation 3x | DLSS 4 Frame Generation 4x |
|:---:|:---:|:---:|
| ![DLSS 4 Frame Generation 2x](https://github.com/user-attachments/assets/d3b99a0a-1060-4d7a-aef3-bf7c1c2be6ee) | ![DLSS 4 Frame Generation 3x](https://github.com/user-attachments/assets/ff6b3315-f1cb-46a3-84eb-6af3f3535d0d) | ![DLSS 4 Frame Generation 4x](https://github.com/user-attachments/assets/3aae4343-93ab-497c-a0dc-0156b21190ea) |

These results alone do **not** prove perfect frame quality.

But they do show that:
- the performance uplift is repeatable,
- the scaling is internally consistent,
- and the frame-generation path is active enough to be measured by external tooling.

<img width="1386" height="1267" alt="image" src="https://github.com/user-attachments/assets/c8de6107-6e8a-4826-bdcc-b1903c47432a" />

The full 3DMark result files (`.3dmark-result`) are available in the [**Releases**](https://github.com/FirstEverTech/RTX4000-MFG-Unlock/releases) section.  
Download them and open locally in 3DMark to inspect the frame‑time graphs and verify the results yourself.

---

## Why I stepped back and used Nsight Systems

Once it became clear that the `ShowEvalModeOverlay` theory was not the answer, the most sensible move was to stop guessing what one branch in the driver might mean and instead inspect the frame pipeline directly.

So I captured three Nsight Systems traces under matching conditions:

- `CP2077-fg2x.nsys-rep`
- `CP2077-fg3x.nsys-rep`
- `CP2077-fg4x.nsys-rep`

### Capture setup

- **Application:** Cyberpunk 2077
- **Mode:** built-in benchmark
- **Preset:** Ray Tracing: Overdrive
- **Resolution:** 4K
- **Tool:** NVIDIA Nsight Systems 2026.2.1
- **Capture length:** 10 seconds
- **Goal:** compare actual present behavior for FG 2x vs 3x vs 4x in the same repeatable scene path

<img width="1368" height="800" alt="image" src="https://github.com/user-attachments/assets/d7316b73-f950-4dc1-a207-63aea72fcf92" />

---

## What the Nsight analysis showed

A full standalone write-up of the Nsight analysis is intended to be shipped as a **Release asset**:

- `RTX4000_MFG_Nsight_Technical_Note.md`

That file is meant to accompany the raw capture files:

- `CP2077-fg2x.nsys-rep`
- `CP2077-fg3x.nsys-rep`
- `CP2077-fg4x.nsys-rep`

### Short version of the findings

The main pattern found in Nsight was highly regular:

- **FG 2x** grouped into **2 presents per base cycle**
- **FG 3x** grouped into **3 presents per base cycle**
- **FG 4x** grouped into **4 presents per base cycle**

In the stable analysis window, the estimated base render cadence stayed in a similar range while the number of presents scaled with the selected mode:

| Mode | Present FPS | Estimated base render cycles/s | Dominant packet size |
|------|------------:|-------------------------------:|---------------------:|
| FG 2x | 72.25 | 36.20 | 2 |
| FG 3x | 111.60 | 37.20 | 3 |
| FG 4x | 153.80 | 38.45 | 4 |

This is important because it does **not** look like a simple fake FPS counter uplift.

It looks like a pipeline that is actually outputting additional presents in structured packets consistent with the selected multiplier.

<img width="1980" height="938" alt="figure_1_present_excerpt" src="https://github.com/user-attachments/assets/6bcb3fe3-5e37-481a-85b2-1cfbeed282f0" />

<img width="1620" height="977" alt="figure_3_rates" src="https://github.com/user-attachments/assets/ea5b1851-db10-42f7-ae0b-daa1c701ddf0" />

<img width="1660" height="977" alt="figure_4_packet_sizes" src="https://github.com/user-attachments/assets/6c055dcd-ee58-492b-9e2f-6072a7b7416b" />

<img width="1980" height="1017" alt="figure_2_delta_histogram" src="https://github.com/user-attachments/assets/1e9a0425-6e79-4b59-b49a-988e7dd93d70" />


---

## What I currently believe the data supports

Based on the benchmark data and the Nsight Systems captures, my current conclusion is:

> **Forced MFG 3x/4x on the tested RTX 4070 Ti appears to generate and present additional frames at the pipeline level.**

That is a much stronger statement than simply saying:

- “the FPS counter goes up”
- “3DMark reports a bigger number”
- or “it only replays the exact same finished frame”

At the same time, I want to be precise and careful:

> **The current evidence supports real additional present activity, but it does not yet prove full image-quality equivalence with native RTX 5000-series MFG.**

---

## What this does NOT prove yet

The current data does **not** fully prove:

- that image quality is identical to native RTX 5000-series MFG
- that motion quality is identical in all edge cases
- that latency behavior is identical
- that every generated frame is equally good in perceptual terms
- that all driver-side enforcement logic has been understood

So the honest conclusion is **not**:

> “RTX 4000 now has perfect RTX 5000-equivalent MFG.”

The honest conclusion is:

> **The feature appears to be genuinely active at the frame-presentation level, while the black-image enforcement path and full quality equivalence remain open questions.**

---

## Release structure

The intended **Release** package for this stage of the project is:

- raw Nsight capture files:
  - `CP2077-fg2x.nsys-rep`
  - `CP2077-fg3x.nsys-rep`
  - `CP2077-fg4x.nsys-rep`
- standalone supplementary analysis:
  - [RTX4000_MFG_Nsight_Technical_Note.md](https://github.com/FirstEverTech/RTX4000-MFG-Unlock/releases/download/v1.0-nsight-notes/RTX4000_MFG_Nsight_Technical_Note.md)
  - [RTX4000_MFG_Nsight_Technical_Note.pdf](https://github.com/FirstEverTech/RTX4000-MFG-Unlock/releases/download/v1.0-nsight-notes/RTX4000_MFG_Nsight_Technical_Note.pdf)
- optional figures exported from the analysis
- optional benchmark artifacts such as `.3dmark-result` files

That way people can:

1. read the short version in the README
2. inspect the deeper technical note
3. and verify the raw capture material independently

**Releases:** https://github.com/FirstEverTech/RTX4000-MFG-Unlock/releases

---

## Current work

The main research target right now is:

> **finding the driver mechanism that forces the black image and preventing it from obscuring the actual generated output**

This work is slow and time-consuming.
Every false lead can cost many hours.
But the Nsight findings are exactly why the project is still worth pursuing: they suggest that something meaningful is happening underneath the black output.

---

## If you want to help

Useful contributions include:

1. peer review of the analysis
2. additional testing on other RTX 4000 cards
3. Nsight captures from other games / scenes
4. image-quality comparisons between forced RTX 4000 MFG and native RTX 5000 MFG
5. independent technical discussion of driver behavior

If you reproduce similar results, feel free to open an issue or discussion with:
- GPU model
- driver version
- game / scene
- test method
- capture files
- observed black-output behavior

---

## Support this project

This project is developed in my free time, and the current stage of work is **extremely time-consuming**. If you find this research useful and want to support further investigation into the black-image enforcement path, donations are very welcome.

[![PayPal](https://img.shields.io/badge/PayPal-Support_Development-00457C?style=for-the-badge&logo=paypal)](https://www.paypal.com/donate/?hosted_button_id=48VGDSCNJAPTJ)[![Buy Me a Coffee](https://img.shields.io/badge/Buy_Me_a_Coffee-Support_Work-FFDD00?style=for-the-badge&logo=buymeacoffee)](https://buymeacoffee.com/firstevertech)[![GitHub Sponsors](https://img.shields.io/badge/GitHub-Sponsor-EA4AAA?style=for-the-badge&logo=githubsponsors)](https://github.com/sponsors/FirstEverTech)

Every contribution helps cover more time spent on:
- reverse engineering
- controlled testing
- Nsight analysis
- result documentation
- publishing raw materials for independent review

---

## Disclaimer / legal

This repository is intended strictly for:

- research
- education
- documentation
- technical discussion

It does **not** distribute modified NVIDIA drivers or binaries.
It does **not** provide a ready-to-use bypass package.
It does **not** disclose the method used to install modified kernel drivers.
It does **not** encourage users to weaken platform security or violate software licenses.
All trademarks, product names, and technologies belong to their respective owners.
Use the information responsibly and in accordance with applicable laws, licenses, and platform policies.

---

## Author & contact

**Marcin Grygiel** aka **FirstEver**

- 🌐 Website: https://www.firstever.tech
- 💼 LinkedIn: https://www.linkedin.com/in/marcin-grygiel/
- 🔧 GitHub: https://github.com/FirstEverTech
- 📧 Contact: https://www.firstever.tech/contact
