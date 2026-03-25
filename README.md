# NVIDIA GeForce RTX 4000 Series - DLSS 4 Frame Generation 3x/4x Unlock

Lately I've been diving into various projects — some of you might know my work on:

— **[Universal Intel Chipset Device Updater](https://github.com/FirstEverTech/Universal-Intel-Chipset-Updater)**  
— **[Universal Intel Wi-Fi and Bluetooth Drivers Updater](https://github.com/FirstEverTech/Universal-Intel-WiFi-BT-Updater)**

This time, I got interested in something that has been bothering me for a while:
## **DLSS 4 Frame Generation 3x/4x on RTX 4000 series cards**

<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/d3197e51-cd9c-49b9-a389-af9c446c2f82" />

---

## My take on NVIDIA's current situation

It's hard not to notice that NVIDIA's focus has been shifting more and more towards AI and data center products (H100, B200, etc.), where margins are significantly higher than in the consumer GPU market.

At the same time:
- GPU prices have increased  
- Native generational performance gains remain relatively modest  
- New features often become key differentiators between generations  

> For RTX 5000, one of the most visible differentiators appears to be higher-order Frame Generation (3x/4x)

This raises a natural question:

> **To what extent are these limitations hardware-driven, and to what extent are they software-defined?**

---

## What I've done so far

Activating FG 3x/4x on RTX 4000 is actually not that difficult — **[NVIDIA Profile Inspector](https://github.com/Orbmu2k/nvidiaProfileInspector)** allows you to set it globally, bypassing the lock in NVIDIA App.

<img width="1000" height="693" alt="image" src="https://github.com/user-attachments/assets/fadfc3cb-9865-4500-85d5-a278afbc6c5a" />

The setting even shows up in NVIDIA App afterwards (read-only of course).

<img width="1298" height="942" alt="image" src="https://github.com/user-attachments/assets/85129b4f-39b9-4258-b5bd-fb13da1f48d3" />

---

## The problem #1 — driver-level validation & black overlay

After enabling **Frame Generation 3x/4x on RTX 4000 series**, an unexpected behavior consistently appears:

> A full-screen black overlay is rendered on top of the output — despite Frame Generation functioning correctly underneath.

### Observed behavior

- NVIDIA App reports correct FPS scaling  
- Games remain fully interactive  
- Frame pacing remains consistent  
- Audio continues normally  
- Benchmarks correctly detect generated frames  

> **Conclusion:** rendering is not broken — it is being deliberately obscured.

---

## Reverse engineering findings (nvlddmkm.sys)

To investigate further, I performed **static analysis of `nvlddmkm.sys`** using [**Ghidra**](https://github.com/NationalSecurityAgency/ghidra), a tool developed by the National Security Agency.

<img width="1623" height="924" alt="Ghidra" src="https://github.com/user-attachments/assets/0681a4d6-add6-4d17-bbc5-b71fabcbffa0" />

### Key observations

- Identified string reference:
  ```cpp
  ShowEvalModeOverlay
  ```
- Associated logic suggests a conditional execution path tied to runtime validation (likely feature gating / hardware checks)

### Potential control point

Driver version:
```
32.0.15.9579
```

Offset:
```
0x14C8A58
```

Instruction:
```
JNZ (0x75)
```

Test modification:
```
0x75 → 0xEB  (JNZ → JMP)
```

This change appears to alter execution flow in a way consistent with **preventing the overlay from being triggered**.

> ⚠️ This is not yet definitive proof of the exact enforcement mechanism, but observed behavior strongly aligns with this hypothesis.

---

## The problem #2 — kernel-mode validation & platform restrictions

Working with kernel-mode drivers introduces strict platform-level constraints.

### Windows 11 security model

- `nvlddmkm.sys` operates in **kernel mode**
- Enforced by **Kernel Mode Code Integrity (KMCI)**
- Requires valid Microsoft-trusted signing (e.g., WHQL)

### What does NOT work

- Test signing mode:
  ```batch
  bcdedit /set testsigning on
  ```
- Disabling integrity checks  
- Self-signed certificates  

On modern systems with **Secure Boot enabled**, modified drivers result in:

```
Code 52 — Windows cannot verify the digital signature for this driver
```

### Implication

> Even with a valid technical modification, the driver cannot be loaded under standard Windows 11 security conditions.

This is currently the main blocker for:
- end-to-end validation  
- reproducible testing  
- confirming exact execution paths  

---

## Benchmark results — proof that FG 3x/4x works on RTX 4070 Ti

Here's what my **RTX 4070 Ti** achieves with FG 3x/4x active:

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

---

### Cyberpunk 2077 Built-in Benchmark (Ray Tracing: Overdrive 4K)

| Mode | Avg FPS | Min FPS | Max FPS |
|------|---------|---------|---------|
| FG 2x | 75.83 | 69.78 | 82.18 |
| FG 3x | 124.49 | 113.24 | 136.96 |
| FG 4x | 164.68 | 149.75 | 180.54 |

**Important observations:**

- Scaling is mathematically consistent  
- Benchmarks correctly detect generated frames  
- Frame-time graphs show identical patterns (scaled proportionally)  
- Drops occur at identical moments across all modes  

| DLSS 4 Frame Generation 2x | DLSS 4 Frame Generation 3x | DLSS 4 Frame Generation 4x |
|:---------------:|:--------------:|:-------------------:|
| ![DLSS 4 Frame Generation 2x](https://github.com/user-attachments/assets/d3b99a0a-1060-4d7a-aef3-bf7c1c2be6ee) | ![DLSS 4 Frame Generation 3x](https://github.com/user-attachments/assets/ff6b3315-f1cb-46a3-84eb-6af3f3535d0d) | ![DLSS 4 Frame Generation 4x](https://github.com/user-attachments/assets/3aae4343-93ab-497c-a0dc-0156b21190ea) |

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

## DISCLAIMER / LEGAL

This repository is intended **strictly for research, educational, and documentation purposes**.

- No modified drivers or binaries are distributed  
- No instructions are provided for bypassing platform security mechanisms  
- All findings are based on **static analysis and observable behavior**  
- Any mentioned modifications are **hypothetical and unverified in production environments**  

This project does **not encourage or support**:
- Circumventing software protections  
- Violating license agreements  
- Disabling operating system security features  

All trademarks, product names, and technologies mentioned belong to their respective owners, including NVIDIA.

> Use this information responsibly and in accordance with applicable laws and platform policies.

---

## Author & Contact

**Marcin Grygiel** aka FirstEver

- 🌐 **Website**: https://www.firstever.tech  
- 💼 **LinkedIn**: https://www.linkedin.com/in/marcin-grygiel/  
- 🔧 **GitHub**: https://github.com/FirstEverTech  
- 📧 **Contact**: https://www.firstever.tech/contact  

---

## Support This Project

This project is maintained in my free time.

[![PayPal](https://img.shields.io/badge/PayPal-Support_Development-00457C?style=for-the-badge&logo=paypal)](https://www.paypal.com/donate/?hosted_button_id=48VGDSCNJAPTJ)
[![Buy Me a Coffee](https://img.shields.io/badge/Buy_Me_a_Coffee-Support_Work-FFDD00?style=for-the-badge&logo=buymeacoffee)](https://buymeacoffee.com/firstevertech)
[![GitHub Sponsors](https://img.shields.io/badge/GitHub-Sponsor-EA4AAA?style=for-the-badge&logo=githubsponsors)](https://github.com/sponsors/FirstEverTech)

**Your support means everything!**

- ⭐ Star the project  
- 🔁 Share it  
- 💬 Join the [**Discussion**](https://github.com/FirstEverTech/RTX4000-MFG-Unlock/discussions/1)
