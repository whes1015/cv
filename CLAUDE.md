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

**下一個新版是 `v3/`。**

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

- 學歷缺**預計畢業年月**。v1 的 `\role{...}` 與 v2 的 `\education{...}` 都留了 TODO 註解；
  要補上就開 `v3/`（不可直接改 v1/v2）。
