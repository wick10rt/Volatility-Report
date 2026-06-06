# Volatility-Report

資訊安全期末報告：**用 Volatility 3 還原 RAM Dump 內容**。

- **主題**：記憶體鑑識（Memory Forensics）— 從一份公開的 RAM dump 還原系統狀態、進程、網路與命令列，重建一場攻擊。
- **樣本**：OtterCTF 2018（`OtterCTF.vmem`，Windows 7 SP1 x64）。受害者 Rick 下載盜版「Rick and Morty」種子後，機器被植入偽裝成 `vmware-tray.exe` 的勒索軟體（HollowCrypt）。
- **工具**：純 Volatility 3，三個核心指令 — `windows.info`（系統資訊）/ `windows.psscan`（掃描還原進程與父子關係）/ `windows.netscan`（網路連線）。

## 最終成果

**單一合併簡報（共 15 頁）**，前段課前預習（§1~§5）＋後段上課報告（§6~§8）：

- **§1~§5 課前預習**：RAM 鑑識是什麼、能查什麼、dump 怎麼來、實作環境、安裝 Vol3。
- **§6~§8 上課報告（約 35~40 分）**：新聞案例（CyCraft Operation Skeleton Key）→ ▶ 實作影片（10 分，`https://youtu.be/MObLNDzExd4`）→ 影片後拆解（串攻擊鏈，命脈，結論收在 §8 末頁）。

逐頁規劃見 `報告規劃.md`。

> 📌 簡報檔（`.pptx`）、PDF 與產生腳本（`.py`）為成品/工具，**不進版控**；本 repo 只收文字文件，報告內容以下列 md 描述為準。

## 檔案

| 檔案 | 內容 |
|---|---|
| `計畫書.md` | 報告計畫書（主題、攻防設計、規劃） |
| `報告規劃.md` | 最終簡報逐頁、時間分配、影片腳本、Q&A 準備 |
| `SOP.md` | 實作會用到的完整指令 |
| `todo.md` | 當前待辦 |

## 狀態

✅ **已用實跑 Vol3（2.28.0）的真實輸出回填**。確認值：dump 時間 `2018-08-04 19:34:22`；`Rick And Morty`=PID 3820（父 explorer 2728）；`vmware-tray.exe`=PID 3720（父 Rick And Morty 3820，晚 7 秒建立）；`LunarMS`=708→`77.102.199.102:7575`（遊戲伺服器）；`vmware-tray` 無對外連線；md5 `ad51f4ada4151eab76f2dce8dea69868`。

⚠️ 惡意程式磁碟落地路徑（`...\Temp\RarSFX0\vmware-tray.exe`）與「勒索軟體 HollowCrypt」定性取自公開 Vol2 writeup，非本次三指令輸出，文中已標明。
