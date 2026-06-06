# Part1 投影片微調 — 移除截圖框、改詳細說明

> ⚠️ **歷史筆記（已執行完畢）**：這是早期針對 Part1 投影片的調整計畫，當時 Part1 還是獨立 10 頁。
> 最終成果已合併為單一 15 頁簡報（前段 §1~§5 即此處的 Part1）；以最終簡報內容為準，本檔僅供歷程參考。

> 對象檔：`build_part1_pptx.py` → 重跑後產生新的 `第一部份_預習_v1.pptx`
> 共同改動：以下 4 頁原本下半部是兩個並排的「📷 自行貼上截圖」框,改成把空間還給文字,寫成詳細說明。
> 影響頁:1-1、1-2、2-2、3-1(共 4 頁)。其餘頁面維持原本「要點 + 截圖框」版型。

---

## 1-1 ｜ 介紹 RAM 鑑識(1/2)— 什麼是記憶體鑑識

**保留上方要點(3 條)**

新增下半部詳細說明,擬分三塊:

### 三步驟工作流
- **取得(Acquire)**:在嫌疑機器上(或 VM 快照)把整顆 RAM 內容讀出來,寫成一個檔案。
- **解析(Analyze)**:用 Volatility 之類的工具,把這顆 raw bytes 翻譯成「進程清單、網路連線、命令列」等可讀表格。
- **還原(Reconstruct)**:把這些表格交叉比對,拼出「當時這台電腦正在做什麼」。

### 跟硬碟鑑識的差別(一句話)
> 硬碟鑑識看的是「**留下了什麼檔案**」;記憶體鑑識看的是「**當時正在做什麼**」。
> 兩者互補,不是取代關係 — 真正的事件調查兩個都會做。

### 為什麼非做不可:IR 第一步
- 資安事件爆發時,IR(事件應變)的標準動作是「**先 dump 記憶體,再關機**」。
- 一旦關機,RAM 內容就沒了 — 進程、解密金鑰、未落地的 payload 全部消失。
- 所以「會不會看 RAM dump」基本上是 IR / Blue Team 的入場券。

---

## 1-2 ｜ 介紹 RAM 鑑識(2/2)— 為什麼要查 RAM(硬碟 vs RAM)

**保留上方要點(3 條)**

新增下半部詳細說明,擬分兩塊:

### 四維度對照(以小表格呈現)

| 維度 | 硬碟 | RAM |
|---|---|---|
| 持久性 | 關機仍在 | 關機消失 |
| 攻擊者能不能清理 | 容易(wipe / 覆寫 / 加密) | 困難,通常保留現場 |
| 加密資料 | 多半是密文 | 多半是明文(執行時必須解密) |
| 反映的時間點 | 全歷史(含已刪除) | 拍快照那一刻 |

### 一個具體例子(讓「程式要執行就得進 RAM」具象化)
- 勒索軟體把整顆硬碟加密了 — 硬碟上撈不到任何明文證據。
- 但**只要勒索軟體還在跑**,加密用的金鑰、原始檔名表、C2 連線都仍在 RAM。
- Fileless malware 不在硬碟落地,**但要執行就一定會在 RAM 出現**;這是物理事實,躲不掉。

---

## 2-2 ｜ RAM 裡面有什麼能查(2/2)— 這些資料在記憶體裡怎麼被存

**保留上方要點(4 條)**

新增下半部詳細說明,擬分兩塊:

### 三個關鍵結構各自存什麼欄位

- **`EPROCESS`(每個進程一份)**
  `ImageFileName` 程式名 ・ `UniqueProcessId` PID ・ `InheritedFromUniqueProcessId` PPID ・ `CreateTime` 建立時間
  → `psscan` 就是靠這些欄位還原進程清單與父子關係。

- **`_TCP_ENDPOINT` / `_UDP_ENDPOINT`(每條連線一份)**
  `Owner`(哪個進程) ・ `LocalAddr` / `LocalPort` ・ `RemoteAddr` / `RemotePort` ・ `State`
  → `netscan` 就是把這些 endpoint 結構掃出來,所以能知道「誰連去哪個 IP」。

- **`PEB.ProcessParameters`(每個進程一份)**
  `ImagePathName` 完整路徑 ・ `CommandLine` 啟動參數 ・ `CurrentDirectory` 工作目錄
  → 想看程式被怎麼啟動的就靠它。

### Vol3 怎麼把這些結構讀出來
> 結構在記憶體裡就是一段 raw bytes;Vol3 透過 **symbol table(PDB)**得知每個欄位在結構裡的**偏移量**(例如 `ImageFileName` 是第幾個 byte 起、佔幾個 byte),才能把 raw bytes 翻譯成上面那些可讀欄位。
> 沒有對應 OS / build 的 symbol → Vol3 認不出結構 → 整套解析失效。所以 Vol3 安裝後第一件事就是把 symbol pack 準備好。

---

## 3-1 ｜ RAM dump 怎麼取得(1/2)— 三種來源(最大改動,要詳細)

**移除原本三條要點與兩個截圖框,改為:三種來源各自開展為一段卡片/區塊。**

### ① Live 系統上用工具 dump
- **工具**:DumpIt(單檔可攜、雙擊即用)、winpmem(開源、跨 Windows 版本)、FTK Imager(GUI、IR 老牌工具)。
- **場景**:IR 現場、不能關機的伺服器、嫌疑人電腦剛抓到還在開機。
- **代價**:需要 admin 權限;dump 工具本身載入會佔一點 RAM,造成微量擾動(實務可接受)。

### ② VM hypervisor 直接做記憶體快照
- **工具/方法**:VMware 把 VM suspend → 產生 `.vmem`;VirtualBox `VBoxManage debugvm dumpvmcore`;**QEMU/libvirt `virsh dump --memory-only`** 或 `dump-guest-memory`。
- **場景**:受害機本身是 VM 時最理想 — 從外面拍快照,受害系統完全無感、零擾動。
- **本報告樣本**:OtterCTF 提供的 `OtterCTF.vmem` 就是 VMware 路線的產物(`.vmem` 副檔名是直接證據)。

### ③ Crash dump(當機檔)
- **來源**:Windows 當機時自動產生 `C:\Windows\MEMORY.DMP`;也可以用 NotMyFault 之類的工具刻意觸發藍白當機來抓。
- **場景**:不太用於主動取證,但事件之後若機器上剛好有舊的 crash dump,仍可以拿來分析。
- **注意**:預設可能是 kernel dump 或 mini dump,**不一定是 full memory**;能撈到的證據受限於 dump 類型。

### 對本報告的影響(收尾一句)
> 本次用的是 ② 號方法的成品。看報告的人**不需要**自己取得 dump — 樣本可直接從 OtterCTF 2018 公開下載。

---

## 待用戶確認再動腳本

以上是內容草案。如果方向沒問題,我就改 `build_part1_pptx.py`:
- 1-1 / 1-2 / 2-2 三頁改用「上方原要點 + 下方一塊大內容區」的新版型函式。
- 3-1 改成「三張並排卡片」版型(沿用 v1 卡片視覺,類似 Part2 新聞案例第一頁的硬碟 vs RAM 卡片)。
- 1-2 的四維度對照表用 grid_table 渲染(類似 Part2 指令路線圖那種)。
- 全部沿用 v1 的霧紫黑 + 淡紫 accent 配色,不破壞既有視覺語言。

---

# ⚠️ 待改:安裝改成 pip / `vol` 版(實作已用 `vol`,文件還停在 clone + `python vol.py`)

> 決策:實際安裝用 `pip install volatility3`,執行用 `vol`,**不 clone GitHub**。
> 下面所有地方目前寫的是「clone 原始碼 + `python vol.py`」舊路線,要全部統一成 pip / `vol`。
> 核心對應:`git clone ... + python vol.py` → `pip install volatility3 + vol`;指令前綴 `python vol.py` → `vol`。

## A. SOP.md — 安裝章節(§2.1 / §2.2)

- **L60~64**:把 clone + venv + `pip install -e .` 區塊改成:
  ```
  pipx install volatility3      # 或：python -m venv .venv && source .venv/bin/activate && pip install volatility3
  ```
- **L67、L71**:`pip install -e .` / `pip install -e ".[full]"` 的說明 → 改成 `pip install volatility3` / `pip install "volatility3[full]"`。
- **L77**:`python vol.py -h | head -20` → `vol -h | head -20`。
- **L176、L421**:`source .venv/bin/activate`(若改用 pipx 全域安裝可整段刪掉;若仍用 venv 則保留)。

## B. SOP.md — 所有跑指令的前綴 `python vol.py` → `vol`(共 9 處)

L77 / L177 / L222 / L285 / L362 / L402 / L425 / L428 / L431
- 一般指令:`python vol.py -f ... windows.xxx` → `vol -f ... windows.xxx`
- 指定 symbol 目錄那兩處(L362 / L402)用了 `-s ~/memforensics/volatility3/volatility3/symbols`:pip 安裝後沒有那個 repo 路徑,**改成讓 `vol` 自動抓 symbol(把 `-s ...` 整段拿掉)**,或改指到 pip 套件的 symbols 目錄。建議直接拿掉 `-s`,靠自動下載。

## C. SOP.md — §1.2 系統套件 & §2.3 symbol

- §2.3「預載 symbol 到 `volatility3/symbols/`」:pip 版沒有該路徑;改成「`vol` 首次跑會自動下載;要離線再手動放到套件的 symbols 目錄」。
- 確認 §1.2 安裝系統套件那段沒有叫人裝 git 只為了 clone(git 仍可留,但安裝 Vol3 不再需要)。

## D. build_part1_pptx.py — §5 安裝兩頁(投影片)

- **L245~250(§5 1/2「下載與安裝步驟」)** 改成 pip 版要點:
  - `pip install volatility3`(或 pipx install volatility3)
  - (可選)用 venv 隔離:`python -m venv .venv → source .venv/bin/activate`
  - 不需 clone GitHub、無 `pyproject.toml` / `pip install -e .` 那段
  - 驗證:`vol -h` 看到「Volatility 3 Framework」
  - 截圖框 label:`vol.py -h 成功` → `vol -h 成功`
- **L252~256(§5 2/2「symbol 準備與首次執行」)**:
  - 「預載 windows.zip 到 `volatility3/symbols/`」→ 改「`vol` 首次跑自動下載 symbol」
  - 截圖框 label 同步
- **L242(§4 2/2 截圖框)** label「plugin 清單 vol.py -h」→「plugin 清單 vol -h」。
- 改完重跑 `python build_part1_pptx.py` 產生新投影片。

## E. 一致性檢查(改完後)

- 全 repo 搜 `vol.py`、`git clone`、`pip install -e`、`.venv` 確認沒有殘留舊寫法。
- README.md / 計畫書.md / 報告規劃.md 若有提到安裝步驟也要同步(目前看來主要集中在 SOP 與 Part1 §5)。
