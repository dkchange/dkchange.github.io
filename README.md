# My Godot RPG Web Game

A Godot‑based RPG / adventure game project, distributed as a **web version (HTML5)**.  
本專案是一個使用 Godot4.5.1 開發的 RPG / 冒險遊戲，以網頁版（HTML5）形式發佈，就是想讓主頁酷一點。

---

## Project Overview

This repository contains:
- The **web version** of the game (built with Godot's HTML5 export).
- Documentation and metadata (README, credits, licenses).

The **source code** of the project is **not publicly available**, but the distributed web build is provided under the MIT License (see the `LICENSE` file).

本專案存放：
- 遊戲的「網頁版」（以 Godot HTML5 匯出產生）  
- 說明文件與相關資訊（本 README、授權說明）

專案的「原始碼」目前不公開，但發佈的網頁版遊戲仍依照 MIT 授權條款提供使用。

---

## Tools & Workflow

Development was done with:
- **Engine:** Godot Engine (MIT License)  
- **Image generation:** ComfyUI (open‑source workflow system)  
- **Models used in ComfyUI:**
  - SPARK TTS
  - SDXL
  - ACE  
- **Plugins used in development:**
  - `dialogue_manager` (nonlinear dialogue system for Godot, MIT License)
  - `gdUnit4` (testing plugin)
  - `mcp-server-godot-editor` (MCP server plugin for Godot editor)
  - Planned / not yet used: `LPCAnimatedSprite` (open‑source sprite plugin)

Development tools:
- OpenCode IDE + GLM‑4.7 (for AI‑assisted coding)

---

## License & Copyright

This project is licensed under the **MIT License**.  
You can find the full license text in the `LICENSE` file in this repository.

本專案採用 **MIT 授權**，詳細條款請見本倉庫中的 `LICENSE` 檔。

---

## Godot Engine

This game is built with **Godot Engine**, which is distributed under the MIT License.  
The Godot Engine license is included in the exported web build and shown in the in‑game Credits / About page (or via a LICENSE link on the web page).

本遊戲使用 Godot 引擎開發，Godot 本身為 MIT 授權，授權條款會包含在匯出的網頁版中，並在遊戲內的「關於／Credits」頁面或網頁上的 LICENSE 連結中顯示。

---

## Third‑party Plugins

This project uses the following plugins:
- `dialogue_manager` – a nonlinear dialogue system for Godot, developed by Nathan Hoad and distributed under the MIT License.  
  GitHub repository: https://github.com/nathanhoad/godot_dialogue_manager  
  The plugin is integrated into the game runtime and is subject to its MIT license terms.

本專案實際使用 `dialogue_manager` 插件，用於遊戲內的對話系統。  
此插件由 Nathan Hoad 開發，採 MIT 授權，並已整合到遊戲執行檔中。  
詳細授權條款請參閱其 GitHub 原始倉庫：https://github.com/nathanhoad/godot_dialogue_manager

---

## Assets and Generated Content

Art assets in this project were generated using **ComfyUI** with the following models:
- SPARK TTS
- SDXL
- ACE  

These models may have separate licenses (e.g. CreativeML Open RAIL‑M, CC‑BY‑NC, or similar).  
Their licenses apply to the **generated images**, not to this project’s source code or compiled binaries.

If you reuse these assets in other projects (outside this repository), please:
- Check the license of the original model on its official page.
- Respect usage restrictions (e.g. commercial use, credit requirements, prohibited content).

---

## Fonts

This project uses the **NotoSansCJKtc-Regular** typeface (also known as Noto Sans Traditional Chinese), provided by Google and licensed under the **SIL Open Font License 1.1**.  
You can view the license text here:  
https://github.com/googlefonts/noto-cjk/blob/main/LICENSE  

You are free to use, embed, and redistribute this font in your own projects (including commercial use), as long as you comply with the SIL OFL 1.1 terms.

本專案使用字型 **NotoSansCJKtc-Regular**（思源黑體繁體），由 Google 提供，採 **SIL Open Font License 1.1** 授權。  
授權條款可於下列連結查看：  
https://github.com/googlefonts/noto-cjk/blob/main/LICENSE  

你可自由使用、嵌入與再分發此字型（包含商業用途），只要遵守 SIL OFL 1.1 的規定即可。

---

## How to Run / Play

The game is available as a **web version (HTML5)**; you can play it directly in a browser at the GitHub Pages URL of this repository (e.g. https://yourusername.github.io/your-repo-name/).

If you wish to run it locally, you can download the exported HTML5 files and open `index.html` in a browser that supports WebAssembly.

---

## Credits

Special thanks to:
- The **Godot Engine** team for the open‑source engine.
- Nathan Hoad for the **dialogue_manager** plugin.
- The creators of the models used in **ComfyUI** (SPARK TTS, SDXL, ACE).
- The authors of the plugins used in development (`gdUnit4`, `mcp-server-godot-editor`, `LPCAnimatedSprite`).

Any other contributors, assets, or tools you want to thank can also be listed here.
