# NVIDIA GeForce RTX 4000 Series - DLSS 4 Frame Generation 3x/4x Unlock


Hey everyone,

Lately I've been diving into various projects — some of you might know my work on the **[Universal Intel Chipset Device Updater](https://github.com/FirstEverTech/Universal-Intel-Chipset-Updater)** and **[Universal Intel Wi-Fi and Bluetooth Drivers Updater](https://github.com/FirstEverTech/Universal-Intel-WiFi-BT-Updater)** projects on GitHub. This time I got interested in something that has been bothering me for a while: **DLSS 4 Frame Generation 3x/4x on RTX 4000 series cards.**

<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/d3197e51-cd9c-49b9-a389-af9c446c2f82" />

---

## My take on NVIDIA's current situation

It's hard not to notice that NVIDIA's focus has been shifting more and more towards AI and data center products (H100, B200, etc.), where margins are significantly higher than in the consumer GPU market.

At the same time:
- GPU prices have increased  
- Native generational performance gains remain relatively modest  
- New features often become key differentiators between generations  

> For RTX 5000, one of the most visible differentiators appears to be higher-order Frame Generation (3x/4x).

This raises a natural question:

> **To what extent are these limitations hardware-driven, and to what extent are they software-defined?**

---

## What I've done so far

Activating FG 3x/4x on RTX 4000 is actually not that difficult — **[NVIDIA Profile Inspector](https://github.com/Orbmu2k/nvidiaProfileInspector)** allows you to set it globally, bypassing the lock in NVIDIA App.

<img width="1000" height="693" alt="image" src="https://github.com/user-attachments/assets/45f94b2f-ed48-4a52-bc8c-6b75be757930" />

The setting even shows up in NVIDIA App afterwards (read-only of course).

<img width="1298" height="942" alt="image" src="https://github.com/user-attachments/assets/6823ffa5-5d0e-4ba2-851a-ec96ba38c463" />

## The problem #1 — NVIDIA behavior

> In certain configurations, a full-screen black overlay is triggered despite frame generation functioning correctly — suggesting the presence of an additional validation or enforcement layer.

The frames are being generated correctly, the FPS counter shows the expected values, but the image is effectively blocked by a black rectangle.

### So I went deeper.

I used **[Ghidra](https://github.com/NationalSecurityAgency/ghidra)** (NSA's reverse engineering tool) to analyze `nvlddmkm.sys` — NVIDIA's kernel mode display driver.

During the analysis, I was able to identify what appears to be the mechanism responsible for this overlay behavior.

> At this stage, this repository focuses on **analysis and understanding of the driver behavior**, rather than distributing or modifying any proprietary components.

<img width="1623" height="924" alt="image" src="https://github.com/user-attachments/assets/03efe52f-170d-4e95-a4c2-7326ba83546e" />  

---

## The problem #2 — driver validation & platform restrictions

Working with kernel mode components comes with strict platform requirements.

Modern versions of Windows enforce strong kernel-mode code integrity policies, which makes validating low-level behavior in real-world scenarios significantly more complex.

> Because of this, testing certain hypotheses outside of controlled environments is non-trivial and requires careful consideration of platform security mechanisms.

At this stage, I am focusing on:
- better understanding how these restrictions interact with the observed behavior  
- identifying **legitimate and compliant ways** to validate the findings  
- gathering additional data from observable behavior (benchmarks, frame analysis)  

---

## Benchmark results — proof that FG 3x/4x works on RTX 4070 Ti

Here's what my **RTX 4070 Ti** achieves with FG 3x/4x active:

### 3DMark NVIDIA DLSS Feature Test (4K)
| Mode | DLSS Off | DLSS On | Multiplier |
|------|----------|---------|------------|
| FG 2x | 32.27 FPS | 125.16 FPS | 3.9x |
| FG 3x | 32.16 FPS | 224.02 FPS | 7.0x |
| FG 4x | 32.20 FPS | 291.56 FPS | 9.1x |

| DLSS 4 - FG 2x | DLSS 4 - FG 3x | DLSS 4 - FG 4x |
|:---------------:|:--------------:|:-------------------:|
| ![Hardware Detection](https://github.com/user-attachments/assets/2ac22f03-4869-4302-8d53-a58d850e5250) | ![Security Check](https://github.com/user-attachments/assets/637b755e-240b-4ff9-8c1d-7e1120cef947) | ![Update Process](https://github.com/user-attachments/assets/9ab7252c-940a-4849-8d4c-7aa2eba55b09) |

**Official 3DMark results (verified, public):**
https://www.3dmark.com/compare/nd/580395/nd/580396/nd/580397

<img width="1480" height="783" alt="3DMark_2x_3x_4x" src="https://github.com/user-attachments/assets/a93c3096-6e3b-4fb6-8293-72fb88704616" />

### Cyberpunk 2077 Built-in Benchmark (Ray Tracing Overdrive, 4K)
| Mode | Avg FPS | Min FPS | Max FPS |
|------|---------|---------|---------|
| FG 2x | 75.83 | 69.78 | 82.18 |
| FG 3x | 124.49 | 113.24 | 136.96 |
| FG 4x | 164.68 | 149.75 | 180.54 |

**Important observations:**
- The scaling between modes is mathematically consistent — this is not placebo or measurement error  
- Both 3DMark and Cyberpunk's built-in benchmark correctly detect and report the generated frames  
- The FPS curves in frame-by-frame graphs show identical patterns across all modes — just scaled up proportionally  
- Demanding scenes cause FPS drops in all modes at exactly the same moments — proving consistency  

| DLSS 4 - FG 2x | DLSS 4 - FG 3x | DLSS 4 - FG 4x |
|:---------------:|:--------------:|:-------------------:|
| ![Hardware Detection](https://github.com/user-attachments/assets/d3b99a0a-1060-4d7a-aef3-bf7c1c2be6ee) | ![Security Check](https://github.com/user-attachments/assets/ff6b3315-f1cb-46a3-84eb-6af3f3535d0d) | ![Update Process](https://github.com/user-attachments/assets/3aae4343-93ab-497c-a0dc-0156b21190ea) |

> The frames are real. The games see them. The benchmarks confirm them.

---

## A message to NVIDIA

If anyone from NVIDIA reads this — there is an interesting opportunity here.

RTX 4000 has a massive installed base. Expanding access to certain features could generate significant goodwill within the community.

> The data presented here suggests that current limitations may be worth revisiting — or at least openly discussing.

---

## What I'm looking for

1. Technical discussion and peer review  
2. Additional testing across RTX 4000 GPUs  
3. Insights into Frame Generation scaling behavior  
4. Any related research or documentation  

---

## Author & Contact

**Marcin Grygiel** aka FirstEver
- 🌐 **Website**: [www.firstever.tech](https://www.firstever.tech)
- 💼 **LinkedIn**: [Marcin Grygiel](https://www.linkedin.com/in/marcin-grygiel/)
- 🔧 **GitHub**: [FirstEverTech](https://github.com/FirstEverTech)
- 📧 **Contact**: [Contact Form](https://www.firstever.tech/contact)

## Support This Project
This project is maintained in my free time.

[![PayPal](https://img.shields.io/badge/PayPal-Support_Development-00457C?style=for-the-badge&logo=paypal)](https://www.paypal.com/donate/?hosted_button_id=48VGDSCNJAPTJ)[![Buy Me a Coffee](https://img.shields.io/badge/Buy_Me_a_Coffee-Support_Work-FFDD00?style=for-the-badge&logo=buymeacoffee)](https://buymeacoffee.com/firstevertech)[![GitHub Sponsors](https://img.shields.io/badge/GitHub-Sponsor-EA4AAA?style=for-the-badge&logo=githubsponsors)](https://github.com/sponsors/FirstEverTech)

**Your support means everything!** Please consider:
- Giving it a ⭐ "Star" on GitHub
- Sharing with friends and colleagues
- Supporting development financially
