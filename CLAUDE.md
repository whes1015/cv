# CV 儲存庫規則

## 1. 版本目錄：已存在的版本一律唯讀

CV 以資料夾做版本管理：`v1/`、`v2/`、`v3/`……

**已經存在的 `vN/` 是不可變的（immutable）。**

- 絕對不要修改、覆寫、重新編譯或刪除任何已存在 `vN/` 裡的檔案 —— `.tex`、`.sty`、`.pdf` 都算。
- 任何改動都要開新版本，**即使只是修一個錯字**。
- 新版做法：把最新版整個複製一份，只在新目錄裡編輯。

  ```bash
  cp -R v1 v2        # 之後只動 v2/，v1/ 不再碰
  ```

- 每個版本必須**自我完備**：自己的 `cvstyle.sty`、自己的 `.tex`、自己編出的 `.pdf`。
  不要用 symlink，不要把樣式檔抽到根目錄共用 —— 共用檔案一旦被改，舊版重新編譯就會產生不同的 PDF，
  不可變性就破了。重複幾份 `.sty` 是刻意的成本，不是要修掉的問題。
- 版本號只遞增。不要重新編號，不要插入 `v1.5`，不要把 `v2` 退回改成 `v1`。
- `.pdf` 要跟 `.tex` 一起留在版本目錄裡並一起 commit。版本目錄是**成品快照**，不是只有原始碼。

### 目前狀態

| 版本 | 日期 | 版面 | 內容 |
|------|------|------|------|
| `v1/` | 2026-09-04 | 自製 `cvstyle.sty` | 首版；中英雙版本，各 2 頁 A4 |
| `v2/` | 2026-09-04 | `resume.cls`（liweitianux） | 改用參考樣式重製；中英雙版本，各 2 頁 A4 |
| `v3/` | 2026-09-14 | `resume.cls`（同 v2） | 由 v2 複製；工作經歷新增三段受僱經驗；中英雙版本，各 2 頁 A4 |
| `v4/` | 2026-09-14 | `resume.cls`（同 v2） | 由 v3 複製；日期改單行區間；內容重新配重、不再一面倒寫地震；中英雙版本，各 2 頁 A4 |

**下一個新版是 `v5/`。**

各版本的版面與工具鏈可以不同 —— 建置指令與字型需求請看該版本目錄自己的說明（下一節）。

## 2. 建置

在版本目錄裡執行：

```bash
cd vN
tectonic -X compile cv-en.tex
tectonic -X compile cv-zh.tex
```

- 引擎：XeLaTeX（透過 `tectonic`，用 `brew install tectonic` 安裝）。

### v1 的字型
- 拉丁：TeX Gyre Pagella / Heros，**以檔名載入**（`\setmainfont{texgyrepagella}[Extension=.otf, ...]`）。
  不要改成用字型家族名稱載入 —— tectonic 的 bundle 查不到家族名，會編譯失敗。
- 中文：`Heiti TC`（macOS 內建）。
- `xeCJK` 會把破折號與引號當成 CJK 字元、排成全形。`\xeCJKDeclareCharClass{Default}{...}` 那行是用來擋這件事的，不要刪。

### v2 的字型與版面
```bash
brew install --cask font-ibm-plex-serif font-ibm-plex-mono \
                    font-noto-serif-cjk-tc font-noto-serif-cjk-sc
```
- 版面來自 `resume.cls`，作者 Weitian LI，授權 **LPPL 1.3c**，取自
  <https://github.com/liweitianux/resume>（其本身衍生自 Christophe Roger 的 YACC 與 Plasmati Graduate CV）。
- **`resume.cls` 是上游原檔，一個字都沒改，也不要改。** 需要調整就在 `.tex` 裡覆寫
  （字型縮放、`\geometry`、`\linespread`、`\titlespacing` 都是這樣做的）。保持原檔才能隨時換上游新版。
- 實際使用的中文字型是 **Noto Serif CJK TC**；但 `resume.cls` 在 `[zh]` 選項下於載入時就寫死了
  `Noto Serif CJK SC`，所以 **SC 也必須安裝**，否則編譯失敗。這是保留原檔的代價。
- `cv-en.tex` 沒有用 `[zh]` 選項，因此自行 `\usepackage{xeCJK}` 來排中文人名。
- 兩份都在 `\project`／`\sectionTitle` 前用 `\newpage` 手動分頁，避免章節標題落單在頁尾。
  改動內容後要重新確認頁面平衡。

### v3 的字型與版面
與 v2 相同（`resume.cls` 亦為同一份上游原檔，`diff` 可驗證），另加兩點：

- **沒有描述的 `\experience` 不能寫成 `\experience[起]{迄}{摘要}`。**
  `resume.cls` 只有在後面接了描述（#4）或標籤（#5）時才會收掉表格列；少了那一段，
  起始日期會流進下一列的左欄，變成「Jan 2025 Jun 2025」黏在一起。
  正確寫法是把整段起訖塞進結束日期欄位、不傳起始日期：

  ```latex
  \experience{present\newline Jan 2025}{\textbf{Software Developer} --- ...}
  ```

  `\newline` 在 `R`/`L` 欄位型別裡被 `\let` 成換行（不是換列），排出來與原生的上下堆疊一模一樣。
- `cv-zh.tex` 的日期欄在 `[zh]` 下預設 `\leftcolwidth` = 6em = 60pt，
  但「2024 年 11 月」量起來是 60.44pt，會被硬拆成兩行。
  用 `\addtolength{\leftcolwidth}{3pt}` + `\addtolength{\rightcolwidth}{-3pt}` 補足，總寬不變。

### v4 的字型與版面

由 v3 複製，`resume.cls` 仍是同一份上游原檔（`diff` 可驗證）。差異如下：

- **日期一律寫成單行區間**（`Jan 2025 -- present`、`2023/11 -- 2024/11`），
  不再用 v3 的上下堆疊寫法 —— 本人反映堆疊「非常奇怪」，會被讀成兩個各自獨立的日期。
  `\experience`／`\project`／`\education` 全部照此處理（`\experience` 仍要走 v3 那個
  「整段塞進結束日期欄位、不傳起始日期」的寫法，否則表格列不會收掉）。
- 區間變長，日期欄要加寬，右欄退還同樣寬度讓總寬不變：
  - `cv-en.tex`：最寬區間量起來 96.4pt，`\leftcolwidth` 81pt → `\addtolength{...}{22pt}`／`\rightcolwidth` `-22pt`。
  - `cv-zh.tex`：最寬區間 86.3pt，`\leftcolwidth` 60pt → `+30pt`／`\rightcolwidth` `-30pt`
    （取代 v3 的 `+3pt`／`-3pt`）。
- 條目之間用 `\separator{0.6em}`（工作經歷）與 `\separator{0.7em}`（專案）拉開間距。
- **`\faShieldAlt` 未定義、`\faShield` 在 bundle 的 fontawesome5 free set 裡沒有圖檔**（兩者都會編譯失敗）。
  可用的有 `\faLock`、`\faUserSecret`、`\faBug`、`\faFlask`、`\faKey`；v4 的資安章節用 `\faLock`。
- 為了塞回兩頁：字型 `Scale` 0.96 → **0.93**（mono 0.90 → **0.87**）、
  `\geometry` 左右邊界 1.15cm → **1.05cm**。
- 能力欄的標籤欄很窄，英文 `Systems \& networking` 會折兩行 —— 改用 `Infrastructure`。

## 3. 內容規則：不造假

這份 CV 的**每一條都必須可查證**。寧可少寫，不要寫到面試被拆穿。

- 不要寫任何無法用一道指令或一個公開連結重現的東西。
- **倉庫存在 ≠ 他的作品。** 寫任何專案前先查真實作者歸屬：

  ```bash
  gh api "repos/ExpTechTW/<repo>/contributors?per_page=100" --jq '.[]|"\(.contributions)\t\(.login)"'
  ```

  fork 若沒有自己的 commit，就不是他的作品。
- README 與官網文案是行銷，不是證據。要主張某個技術能力，去讀真正的程式碼。
- 每個數字都要知道怎麼算出來的，必要時在 CV 上註明口徑。
- 媒體報導要嚴格區分「被專訪」與「文中被具名提及」。
- 有疑慮就降級寫法或直接拿掉。

## 4. 事實基準（2026-09 擷取並經對抗式驗證）

身分：**郭宸毓 / Chen-Yu Kuo**，GitHub `whes1015`（顯示名 `YuYu1015`），
ExpTech（探索科技）創辦人，國立高雄科技大學電機工程系。

已驗證的核心數字（不要再往上灌）：

- 組織內 commit **13,411**；GitHub 全站 **19,284**；PR **477**。
- 作者佔比：DPIP 838/2,339、TREM-Lite 448/847、API 480/803、trem-plugins 417/4,131、
  Nginx 665/665、kekkai 86/86、ES-Net 42/42、qzss 20/20、ml-p-s-picking 49/49、
  gmpe-calibration 18/18、api-gateway-go 33/33、dpip-server-ts 122/122、trem-monitor 105/150。
- TREM 桌面版安裝檔下載 **328,542**（只計 `.exe/.dmg/.deb/.AppImage/.zip`）。
  **不要寫 520 萬** —— 那個數字裡有 430 萬次是 Electron 自動更新器對 `latest.yml` 的輪詢。
- DPIP：Google Play **10 萬+** 安裝；App Store **1,378** 則評分、平均 **4.21**。
- ExpTechTW：234 個公開倉庫、29 名成員，他是兩位 admin 之一。
- 對外專案已合併 PR **12** 個（AnyShake `spectrogram-js`、`NKUST-AP-Flutter`、`smc.peering.tw` 等）。
- Hugging Face：**27** 個模型，累計下載 **73,340**（近 30 天 4,716），建立日期 2026-04-08 ～ 2026-07-14。
  v1／v2／v3 寫的「約 10,600 次」是舊數字，**不要再沿用**；v4 起寫「約 73,000 次（累計）」。重算指令：

  ```bash
  curl -s "https://huggingface.co/api/models?author=YuYu1015&limit=100&expand[]=downloadsAllTime" \
    | jq '[.[].downloadsAllTime] | add'
  ```

### 受僱經歷（2026-09-14 由本人提供）

| 起訖 | 單位 | 職稱 |
|------|------|------|
| 2023/11 – 2024/11 | 衛波科技股份有限公司 | 後端工程師 |
| 2024/12 – 2025/06 | 中央研究院 資訊科學研究所 | 兼任助理 |
| 2025/01 – 現在 | 國家衛生研究院 | 程式開發 |

- 衛波科技的官方英文名是 **P-Waver Inc.**（統編 90772077，臺北市中山區松江路 82 號 6 樓），
  主業為 AI 地震預警與結構安全監測，見 <https://pwaver.com/en>。
- 中研院的職稱他寫的是「兼任助理」，不是「兼任研究助理」。英文只寫 **Part-time Assistant**，
  不要自己升格成 Research Assistant。
- 2025/01 起 NHRI 與 2024/12–2025/06 中研院兩段期間重疊，這是他給的原始資料，照抄不要「修正」。
- **這三段只有職稱與起訖日期，沒有任何工作內容。** 他沒有提供成就，也查不到公開佐證，
  所以 v3 只寫職稱與日期。要補 bullet 必須由他本人提供，不可從職稱推想。
- `ExpTechTW/nhri`（Python，55 commit 全是 `whes1015`，2025-12 建立）疑似與國衛院工作相關，
  但該倉庫是 **private**，不符合「一個公開連結可重現」的標準，**不要寫進 CV、也不要引用**。
- `ExpTechTW/cancer-registry` 的作者是 `PiscesXD`，不是他，已在下方排除名單之列。

### 已排除，不要再寫回去

- **CVE-2025-66478**：MITRE 狀態為 **REJECTED**，是 CVE-2025-55182（React Server Components，CVSS 10.0）
  的重複；兩者的 credits 都**沒有**他。只能寫成「架設蜜罐捕獲該漏洞的在野攻擊並做樣本分析」。
- `proxygate`（作者是 `0x0w0`，他 0 commit）、`eew_server_go`（153 commit，他 0）、
  `eazy-controller`（3/30）、以及 9 個 AI 倉庫（作者 `PiscesXD`，是另一個人）。
- `ExpTechTW/API` 實際上是文件與靜態資料，**不是**伺服器，不要描述成後端服務。
- 親子天下 009420 的主角是**林睿**；天下雜誌 5139812 的主角是氣象署科長**陳達毅**。
  他在前者是並列的共同開發者、在後者是被具名引述的合作者。不要寫成「被專訪」。
- `school-app-hack` 這類倉庫不要放進 CV。

## 5. 待補

- 學歷缺**預計畢業年月**。v1 的 `\role{...}` 與 v2～v4 的 `\education{...}` 都留了 TODO 註解；
  要補上就開 `v5/`（不可直接改 v1～v4）。
- 三段受僱經歷目前只有職稱與日期。若本人提供可查證的工作內容，開 `v5/` 補 bullet。
