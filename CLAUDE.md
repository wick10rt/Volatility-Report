# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 這是什麼

這不是程式專案,而是一份**資訊安全期末報告**的文件集:「用 Volatility 3 還原 RAM Dump 內容」。內容全是中文 Markdown,沒有 build / lint / test。樣本是 OtterCTF 2018(`OtterCTF.vmem`,Windows 7 SP1 x64)。

報告本體檔案(改動時務必保持彼此一致):

**版控範圍**:repo 只收 **md 文字文件**。簡報 `*.pptx`、PDF、產生腳本 `*.py` 與 `images/` 都已 `.gitignore`(檔案留在本機,不進版控)。改任何 md 時務必彼此一致,並對齊本機那份最終簡報的內容。

| 檔案 | 角色 |
|---|---|
| `報告_合併_v1.pptx` | **最終簡報(15 頁),唯一真相來源**(本機檔,不進版控)。前段預習 §1~§5 + 後段報告 §6~§8。 |
| `計畫書.md` | 繳交用計畫書:主題、攻防設計、規劃。 |
| `報告規劃.md` | 最終簡報逐頁、時間分配、影片腳本、Q&A 準備。 |
| `SOP.md` | 實作會用到的完整指令(純指令,每條空一行)。 |
| `README.md` | 專案索引。 |
| `todo.md` | 暫存當前待辦(內容隨進度變動)。 |
| `fix.md` | 早期 Part1 微調筆記(已執行,歷史參考)。 |
| `第一部份_預習_v1.pptx` / `第二部份_報告_v1.pptx` / `build_part1_pptx.py` / `build_part2_pptx.py` / `merge_pptx.py` | **中間產物,已被合併版取代,且不進版控**。不要再跑這些腳本重生——會用舊內容蓋掉合併版。要改簡報就直接在本機 `報告_合併_v1.pptx` 上手術式改(`conda run -n sbhead python`)。 |
| `images/` | 截圖(已 `.gitignore`,不進版控);pptx 會把圖嵌進檔案。 |
| `memforensics/` | **忽略**。使用者明確要求不要動、不要參考此資料夾內容。 |

## ⚠️ 最重要的一件事:虛構事實 vs 已查證事實

repo 原始文件裡的攻擊故事**大部分是手寫推測,與真實 OtterCTF 不符**。已經用多篇 writeup 交叉驗證並改正,改任何內容前務必沿用**已查證的真實值**,不要回退成舊的虛構版本:

- 惡意進程 `vmware-tray.exe`(PID 3720),**父進程是 `Rick And Morty`(PID 3820)**,**不是 explorer.exe**。
- 惡意檔路徑 **`C:\Users\Rick\AppData\Local\Temp\RarSFX0\vmware-tray.exe`**,**不是 `AppData\Roaming\`**。
- 是**勒索軟體 HollowCrypt**(要比特幣贖金),**不是 Pony loader / Lazagne**;初始向量是盜版「Rick and Morty」torrent,**沒有 WINWORD / Pony.docx 釣魚**。
- `77.102.199.102:7575` 是 **LunarMS 遊戲伺服器**(CLOSED,owner=LunarMS.exe),**不是 C2**。
- 線上 writeup 幾乎都用 Vol2(`--profile`/`imageinfo`);本報告純 Vol3,輸出不可照抄 Vol2。
- 真值已實跑回填:dump 時間 `2018-08-04 19:34:22`;Rick And Morty=3820、vmware-tray=3720(晚 7 秒,19:33:02);LunarMS=708;**vmware-tray 無對外連線**;md5 `ad51f4ada4151eab76f2dce8dea69868`。惡意路徑/HollowCrypt 定性取自外部 writeup,文件已標明非本次指令輸出。

完整對照見 memory `otterctf-real-facts.md`。

## ⚠️ 不要寫回「pslist 跑空 / 走串列 vs 掃 pool」敘事

本樣本上 `pslist`/`pstree`/`cmdline`(走 EPROCESS 串列)會跑出**空表**(缺 vmss 的 vmem,核心位址翻譯失準),`info` 的 `KeNumberProcessors`/`Major-Minor` 同因為垃圾值。這是真的技術事實(細節留在 memory `otterctf-real-facts.md` 供 repo 內參考),但**使用者已明確決定報告/投影片不講這套**——改用 `psscan`/`netscan` 平鋪直敘,連 info 輸出也只列有效欄位。**不要把 pslist 失效、DKOM、list-walk vs pool-scan 那套寫回任何文件。**

## 報告的鎖定結構(最終合併簡報 15 頁,勿擅自重構)

最終成果是**單一合併簡報 `報告_合併_v1.pptx`(15 頁)**,前段預習 + 後段報告連成一條流(**沒有 PART2 分隔頁**):

- **§1~§5 課前預習**(slide 2~9):§1 介紹 RAM 鑑識(2 頁)/ §2 RAM 裡有什麼能查(2 頁,Vol3 symbol/PDB 原理併在這)/ §3 RAM dump 怎麼取得(2 頁)/ §4 實作環境(單頁)/ §5 安裝 Vol3(單頁,含 symbol)。封面為 slide 1。**開頭不放 hook、不放思考題**。
- **§6~§8 上課報告**(slide 10~15,約 35~40 分):§6 新聞案例(2 頁,CyCraft Operation Skeleton Key / Chimera)/ §7 ▶影片(單頁,嵌 YouTube 縮圖連結)/ §8 影片後拆解·命脈(3 頁:結構表 → 攻擊鏈 → 三 takeaway,**結構原理唯一主場、結論收在末頁**)。
- **沒有獨立的「開場地板 / 指令路線圖 / 結論頁 / Q&A 頁」**(這些是早期規劃,最終簡報未做成投影片;Q&A 口頭進行)。
- 避免同件事講三遍的分工:RAM 能查什麼→§2 講全;Vol3 框架級原理(symbol/PDB)→§2(2/2);各指令結構級原理(EPROCESS / PEB / _TCP_ENDPOINT)→§8 拆解。

完整逐頁見 `報告規劃.md` 與 memory `report-structure.md`。

## 三個核心指令

純 Vol3:`windows.info`(系統資訊)/ `windows.psscan`(掃描還原進程,用 PPID 看父子關係)/ `windows.netscan`(網路連線)。定罪靠 psscan 的 PPID:vmware-tray(3720) 父進程是盜版下載 Rick And Morty(3820)。
