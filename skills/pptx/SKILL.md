---
name: pptx
description: "簡報創建、編輯和分析。當 Claude 需要處理簡報（.pptx 檔案）時使用：(1) 創建新簡報、(2) 修改或編輯內容、(3) 處理版面、(4) 添加註解或講者備註，或任何其他簡報任務"
license: 專有。LICENSE.txt 包含完整條款
---

# PPTX 創建、編輯和分析

## 概述

使用者可能會要求你創建、編輯或分析 .pptx 檔案的內容。.pptx 檔案本質上是一個包含 XML 檔案和其他資源的 ZIP 壓縮檔，你可以讀取或編輯。對於不同的任務，你有不同的工具和工作流程可用。

## 讀取和分析內容

### 文字擷取
如果你只需要讀取簡報的文字內容，應該將文件轉換為 markdown：

```bash
# 將文件轉換為 markdown
python -m markitdown path-to-file.pptx
```

### 原始 XML 存取
你需要原始 XML 存取來處理：註解、講者備註、投影片版面、動畫、設計元素和複雜格式。對於這些功能，你需要解壓簡報並讀取其原始 XML 內容。

#### 解壓檔案
`python ooxml/scripts/unpack.py <office_file> <output_dir>`

**注意**：unpack.py 腳本位於專案根目錄的相對路徑 `skills/pptx/ooxml/scripts/unpack.py`。如果腳本不在此路徑，使用 `find . -name "unpack.py"` 來定位它。

#### 關鍵檔案結構
* `ppt/presentation.xml` - 主要簡報中繼資料和投影片引用
* `ppt/slides/slide{N}.xml` - 個別投影片內容（slide1.xml、slide2.xml 等）
* `ppt/notesSlides/notesSlide{N}.xml` - 每張投影片的講者備註
* `ppt/comments/modernComment_*.xml` - 特定投影片的註解
* `ppt/slideLayouts/` - 投影片的版面模板
* `ppt/slideMasters/` - 母版投影片模板
* `ppt/theme/` - 主題和樣式資訊
* `ppt/media/` - 圖片和其他媒體檔案

#### 排版和色彩擷取
**當給予要模仿的範例設計時**：始終先使用以下方法分析簡報的排版和色彩：
1. **讀取主題檔案**：檢查 `ppt/theme/theme1.xml` 以獲取色彩（`<a:clrScheme>`）和字體（`<a:fontScheme>`）
2. **取樣投影片內容**：檢查 `ppt/slides/slide1.xml` 以獲取實際字體使用（`<a:rPr>`）和色彩
3. **搜尋模式**：使用 grep 在所有 XML 檔案中尋找色彩（`<a:solidFill>`、`<a:srgbClr>`）和字體引用

## **不使用模板**創建新 PowerPoint 簡報

從頭創建新 PowerPoint 簡報時，使用 **html2pptx** 工作流程將 HTML 投影片轉換為具有精確定位的 PowerPoint。

### 設計原則

**關鍵**：在創建任何簡報之前，分析內容並選擇適當的設計元素：
1. **考慮主題內容**：這個簡報是關於什麼的？它暗示什麼樣的調性、行業或情緒？
2. **檢查品牌**：如果使用者提到公司/組織，考慮他們的品牌色彩和識別
3. **將調色盤與內容匹配**：選擇反映主題的色彩
4. **陳述你的方法**：在撰寫程式碼前解釋你的設計選擇

**要求**：
- 在撰寫程式碼前陳述你的內容導向設計方法
- 僅使用網頁安全字體：Arial、Helvetica、Times New Roman、Georgia、Courier New、Verdana、Tahoma、Trebuchet MS、Impact
- 透過大小、粗細和色彩創建清晰的視覺層次
- 確保可讀性：強對比、適當大小的文字、乾淨的對齊
- 保持一致：在投影片間重複模式、間距和視覺語言

#### 色彩調色盤選擇

**創意選擇色彩**：
- **跳脫預設思維**：什麼色彩真正匹配這個特定主題？避免自動化的選擇。
- **考慮多個角度**：主題、行業、情緒、能量等級、目標受眾、品牌識別（如有提及）
- **勇於冒險**：嘗試意想不到的組合——醫療保健簡報不必是綠色，金融不必是海軍藍
- **建構你的調色盤**：選擇 3-5 個協調的色彩（主色 + 輔助色調 + 強調色）
- **確保對比**：文字在背景上必須清晰可讀

**範例色彩調色盤**（使用這些激發創意——選擇一個、調整它，或創建你自己的）：

1. **經典藍**：深海軍藍（#1C2833）、石板灰（#2E4053）、銀色（#AAB7B8）、米白（#F4F6F6）
2. **青綠與珊瑚**：青綠（#5EA8A7）、深青綠（#277884）、珊瑚（#FE4447）、白色（#FFFFFF）
3. **大膽紅**：紅色（#C0392B）、亮紅（#E74C3C）、橙色（#F39C12）、黃色（#F1C40F）、綠色（#2ECC71）
4. **溫暖腮紅**：淡紫（#A49393）、腮紅（#EED6D3）、玫瑰（#E8B4B8）、奶油色（#FAF7F2）
5. **勃艮第奢華**：勃艮第（#5D1D2E）、深紅（#951233）、鏽色（#C15937）、金色（#997929）
6. **深紫與翡翠**：紫色（#B165FB）、深藍（#181B24）、翡翠（#40695B）、白色（#FFFFFF）
7. **奶油與森林綠**：奶油色（#FFE1C7）、森林綠（#40695B）、白色（#FCFCFC）
8. **粉紅與紫色**：粉紅（#F8275B）、珊瑚（#FF574A）、玫瑰（#FF737D）、紫色（#3D2F68）
9. **萊姆與梅子**：萊姆（#C5DE82）、梅子（#7C3A5F）、珊瑚（#FD8C6E）、藍灰（#98ACB5）
10. **黑與金**：金色（#BF9A4A）、黑色（#000000）、奶油色（#F4F6F6）
11. **鼠尾草與赤陶**：鼠尾草（#87A96B）、赤陶（#E07A5F）、奶油色（#F4F1DE）、炭灰（#2C2C2C）
12. **炭灰與紅**：炭灰（#292929）、紅色（#E33737）、淺灰（#CCCBCB）
13. **活力橙**：橙色（#F96D00）、淺灰（#F2F2F2）、炭灰（#222831）
14. **森林綠**：黑色（#191A19）、綠色（#4E9F3D）、深綠（#1E5128）、白色（#FFFFFF）
15. **復古彩虹**：紫色（#722880）、粉紅（#D72D51）、橙色（#EB5C18）、琥珀（#F08800）、金色（#DEB600）
16. **復古大地**：芥末（#E3B448）、鼠尾草（#CBD18F）、森林綠（#3A6B35）、奶油色（#F4F1DE）
17. **海岸玫瑰**：老玫瑰（#AD7670）、海狸（#B49886）、蛋殼（#F3ECDC）、灰綠（#BFD5BE）
18. **橙與青綠**：淺橙（#FC993E）、灰青綠（#667C6F）、白色（#FCFCFC）

#### 視覺細節選項

**幾何圖案**：
- 對角線分隔而非水平
- 不對稱欄寬（30/70、40/60、25/75）
- 90° 或 270° 旋轉的文字標題
- 圖片的圓形/六邊形框架
- 角落的三角形強調形狀
- 重疊形狀增加深度

**邊框與框架處理**：
- 只在一側的粗單色邊框（10-20pt）
- 對比色的雙線邊框
- 角括號而非完整框架
- L 形邊框（上+左或下+右）
- 標題下的底線強調（3-5pt 粗）

**排版處理**：
- 極端尺寸對比（72pt 標題 vs 11pt 正文）
- 寬字距的全大寫標題
- 超大展示字型的編號章節
- 資料/統計/技術內容使用等寬字體（Courier New）
- 密集資訊使用窄字體（Arial Narrow）
- 強調用的空心文字

**圖表與資料樣式**：
- 單色圖表搭配單一強調色用於關鍵資料
- 水平長條圖而非垂直
- 點狀圖而非長條圖
- 極簡網格線或完全沒有
- 資料標籤直接在元素上（不用圖例）
- 關鍵指標使用超大數字

**版面創新**：
- 帶文字覆蓋的全出血圖片
- 用於導覽/上下文的側欄（20-30% 寬度）
- 模組化網格系統（3×3、4×4 區塊）
- Z 模式或 F 模式內容流
- 色塊上的浮動文字框
- 雜誌風格多欄版面

**背景處理**：
- 佔投影片 40-60% 的純色區塊
- 漸層填充（僅垂直或對角）
- 分割背景（兩種色彩，對角或垂直）
- 邊緣到邊緣的色帶
- 作為設計元素的負空間

### 版面提示
**創建帶圖表或表格的投影片時：**
- **雙欄版面（建議）**：使用橫跨全寬的標題，然後在下方分兩欄——文字/項目符號在一欄，精選內容在另一欄。這提供更好的平衡，使圖表/表格更易讀。使用不等欄寬的 flexbox（例如 40%/60% 分割）為每種內容類型優化空間。
- **全投影片版面**：讓精選內容（圖表/表格）佔據整張投影片以獲得最大影響力和可讀性
- **絕不垂直堆疊**：不要在單欄中將圖表/表格放在文字下方——這會導致可讀性差和版面問題

### 工作流程
1. **必須 - 讀取整個檔案**：從頭到尾完整讀取 [`html2pptx.md`](html2pptx.md)。**讀取此檔案時絕不設定任何範圍限制。** 讀取完整檔案內容以獲取詳細語法、關鍵格式規則和最佳實踐，然後再進行簡報創建。
2. 為每張投影片創建具有適當尺寸的 HTML 檔案（例如 16:9 為 720pt × 405pt）
   - 對所有文字內容使用 `<p>`、`<h1>`-`<h6>`、`<ul>`、`<ol>`
   - 對將添加圖表/表格的區域使用 `class="placeholder"`（以灰色背景渲染以便可見）
   - **關鍵**：首先使用 Sharp 將漸層和圖示柵格化為 PNG 圖片，然後在 HTML 中引用
   - **版面**：對於帶圖表/表格/圖片的投影片，使用全投影片版面或雙欄版面以獲得更好的可讀性
3. 創建並執行 JavaScript 檔案，使用 [`html2pptx.js`](scripts/html2pptx.js) 程式庫將 HTML 投影片轉換為 PowerPoint 並儲存簡報
   - 使用 `html2pptx()` 函數處理每個 HTML 檔案
   - 使用 PptxGenJS API 將圖表和表格添加到佔位區域
   - 使用 `pptx.writeFile()` 儲存簡報
4. **視覺驗證**：生成縮圖並檢查版面問題
   - 創建縮圖網格：`python scripts/thumbnail.py output.pptx workspace/thumbnails --cols 4`
   - 讀取並仔細檢查縮圖以發現：
     - **文字截斷**：文字被標題列、形狀或投影片邊緣截斷
     - **文字重疊**：文字與其他文字或形狀重疊
     - **定位問題**：內容太靠近投影片邊界或其他元素
     - **對比問題**：文字與背景對比不足
   - 如發現問題，調整 HTML 邊距/間距/色彩並重新生成簡報
   - 重複直到所有投影片視覺正確

## 編輯現有 PowerPoint 簡報

編輯現有 PowerPoint 簡報中的投影片時，你需要使用原始 Office Open XML（OOXML）格式。這涉及解壓 .pptx 檔案、編輯 XML 內容，然後重新打包。

### 工作流程
1. **必須 - 讀取整個檔案**：從頭到尾完整讀取 [`ooxml.md`](ooxml.md)（約 500 行）。**讀取此檔案時絕不設定任何範圍限制。** 讀取完整檔案內容以獲取 OOXML 結構和編輯工作流程的詳細指導，然後再進行任何簡報編輯。
2. 解壓簡報：`python ooxml/scripts/unpack.py <office_file> <output_dir>`
3. 編輯 XML 檔案（主要是 `ppt/slides/slide{N}.xml` 和相關檔案）
4. **關鍵**：每次編輯後立即驗證並在繼續前修復任何驗證錯誤：`python ooxml/scripts/validate.py <dir> --original <file>`
5. 打包最終簡報：`python ooxml/scripts/pack.py <input_directory> <office_file>`

## **使用模板**創建新 PowerPoint 簡報

當你需要創建遵循現有模板設計的簡報時，你需要複製和重新排列模板投影片，然後替換佔位內容。

### 工作流程
1. **擷取模板文字並創建視覺縮圖網格**：
   * 擷取文字：`python -m markitdown template.pptx > template-content.md`
   * 讀取 `template-content.md`：讀取整個檔案以理解模板簡報的內容。**讀取此檔案時絕不設定任何範圍限制。**
   * 創建縮圖網格：`python scripts/thumbnail.py template.pptx`
   * 參見[創建縮圖網格](#創建縮圖網格)章節了解更多細節

2. **分析模板並將清單儲存到檔案**：
   * **視覺分析**：審查縮圖網格以理解投影片版面、設計模式和視覺結構
   * 創建並儲存模板清單檔案到 `template-inventory.md`，包含：
     ```markdown
     # 模板清單分析
     **投影片總數：[數量]**
     **重要：投影片從 0 開始索引（第一張投影片 = 0，最後一張投影片 = 數量-1）**

     ## [類別名稱]
     - 投影片 0：[版面代碼（如有）] - 描述/用途
     - 投影片 1：[版面代碼] - 描述/用途
     - 投影片 2：[版面代碼] - 描述/用途
     [... 每張投影片都必須單獨列出其索引 ...]
     ```
   * **使用縮圖網格**：參考視覺縮圖以識別：
     - 版面模式（標題投影片、內容版面、章節分隔頁）
     - 圖片佔位位置和數量
     - 投影片組間的設計一致性
     - 視覺層次和結構
   * 此清單檔案是下一步選擇適當模板所必需的

3. **根據模板清單創建簡報大綱**：
   * 審查步驟 2 中的可用模板。
   * 為第一張投影片選擇介紹或標題模板。這應該是前幾個模板之一。
   * 為其他投影片選擇安全的、基於文字的版面。
   * **關鍵：將版面結構與實際內容匹配**：
     - 單欄版面：用於統一敘事或單一主題
     - 雙欄版面：僅當你確實有 2 個不同的項目/概念時使用
     - 三欄版面：僅當你確實有 3 個不同的項目/概念時使用
     - 圖片 + 文字版面：僅當你有實際圖片要插入時使用
     - 引言版面：僅用於人物的實際引言（帶出處），絕不用於強調
     - 絕不使用佔位多於你內容的版面
     - 如果你有 2 個項目，不要強制放入 3 欄版面
     - 如果你有 4+ 個項目，考慮分成多張投影片或使用列表格式
   * 在選擇版面前計算你的實際內容數量
   * 驗證所選版面中的每個佔位都將填入有意義的內容
   * 選擇一個選項代表每個內容區段的**最佳**版面。
   * 儲存 `outline.md`，包含內容和利用可用設計的模板對應
   * 模板對應範例：
      ```
      # 要使用的模板投影片（從 0 開始索引）
      # 警告：驗證索引在範圍內！有 73 張投影片的模板索引為 0-72
      # 對應：大綱中的投影片編號 -> 模板投影片索引
      template_mapping = [
          0,   # 使用投影片 0（標題/封面）
          34,  # 使用投影片 34（B1：標題和正文）
          34,  # 再次使用投影片 34（複製第二個 B1）
          50,  # 使用投影片 50（E1：引言）
          54,  # 使用投影片 54（F2：結尾 + 文字）
      ]
      ```

4. **使用 `rearrange.py` 複製、重新排序和刪除投影片**：
   * 使用 `scripts/rearrange.py` 腳本創建按所需順序排列投影片的新簡報：
     ```bash
     python scripts/rearrange.py template.pptx working.pptx 0,34,34,50,52
     ```
   * 腳本自動處理複製重複的投影片、刪除未使用的投影片和重新排序
   * 投影片索引從 0 開始（第一張投影片是 0，第二張是 1 等）
   * 相同的投影片索引可以出現多次以複製該投影片

5. **使用 `inventory.py` 腳本擷取所有文字**：
   * **執行清單擷取**：
     ```bash
     python scripts/inventory.py working.pptx text-inventory.json
     ```
   * **讀取 text-inventory.json**：讀取整個 text-inventory.json 檔案以理解所有形狀及其屬性。**讀取此檔案時絕不設定任何範圍限制。**

   * 清單 JSON 結構：
      ```json
        {
          "slide-0": {
            "shape-0": {
              "placeholder_type": "TITLE",  // 或 null 表示非佔位符
              "left": 1.5,                  // 位置（英寸）
              "top": 2.0,
              "width": 7.5,
              "height": 1.2,
              "paragraphs": [
                {
                  "text": "段落文字",
                  // 可選屬性（僅當非預設時包含）：
                  "bullet": true,           // 偵測到明確的項目符號
                  "level": 0,               // 僅當 bullet 為 true 時包含
                  "alignment": "CENTER",    // CENTER、RIGHT（非 LEFT）
                  "space_before": 10.0,     // 段落前間距（點）
                  "space_after": 6.0,       // 段落後間距（點）
                  "line_spacing": 22.4,     // 行距（點）
                  "font_name": "Arial",     // 來自第一個 run
                  "font_size": 14.0,        // 點
                  "bold": true,
                  "italic": false,
                  "underline": false,
                  "color": "FF0000"         // RGB 色彩
                }
              ]
            }
          }
        }
      ```

   * 關鍵功能：
     - **投影片**：命名為「slide-0」、「slide-1」等
     - **形狀**：按視覺位置排序（上到下、左到右）為「shape-0」、「shape-1」等
     - **佔位類型**：TITLE、CENTER_TITLE、SUBTITLE、BODY、OBJECT 或 null
     - **預設字型大小**：`default_font_size` 以點為單位，從版面佔位擷取（可用時）
     - **投影片編號已過濾**：SLIDE_NUMBER 佔位類型的形狀自動從清單中排除
     - **項目符號**：當 `bullet: true` 時，`level` 始終包含（即使為 0）
     - **間距**：`space_before`、`space_after` 和 `line_spacing` 以點為單位（僅當設定時包含）
     - **色彩**：`color` 用於 RGB（例如「FF0000」），`theme_color` 用於主題色彩（例如「DARK_1」）
     - **屬性**：輸出中僅包含非預設值

6. **生成替換文字並將資料儲存到 JSON 檔案**
   根據前一步的文字清單：
   - **關鍵**：首先驗證清單中存在哪些形狀——只引用實際存在的形狀
   - **驗證**：replace.py 腳本將驗證替換 JSON 中的所有形狀都存在於清單中
     - 如果你引用不存在的形狀，你會收到顯示可用形狀的錯誤
     - 如果你引用不存在的投影片，你會收到指示投影片不存在的錯誤
     - 腳本退出前會一次顯示所有驗證錯誤
   - **重要**：replace.py 腳本內部使用 inventory.py 識別所有文字形狀
   - **自動清除**：除非你為它們提供「paragraphs」，否則清單中的所有文字形狀都將被清除
   - 為需要內容的形狀添加「paragraphs」欄位（不是「replacement_paragraphs」）
   - 替換 JSON 中沒有「paragraphs」的形狀將自動清除其文字
   - 帶項目符號的段落將自動左對齊。當 `"bullet": true` 時不要設定 `alignment` 屬性
   - 為佔位文字生成適當的替換內容
   - 使用形狀大小確定適當的內容長度
   - **關鍵**：包含原始清單中的段落屬性——不要只提供文字
   - **重要**：當 bullet: true 時，不要在文字中包含項目符號符號（、-、*）——它們會自動添加
   - **基本格式規則**：
     - 標題/標題通常應有 `"bold": true`
     - 列表項應有 `"bullet": true, "level": 0`（當 bullet 為 true 時 level 必需）
     - 保留任何對齊屬性（例如，`"alignment": "CENTER"` 用於置中文字）
     - 當與預設不同時包含字型屬性（例如，`"font_size": 14.0`、`"font_name": "Lora"`）
     - 色彩：使用 `"color": "FF0000"` 用於 RGB 或 `"theme_color": "DARK_1"` 用於主題色彩
     - 替換腳本期望**格式正確的段落**，不只是文字字串
     - **重疊形狀**：偏好 default_font_size 較大或 placeholder_type 更適當的形狀
   - 將帶替換的更新清單儲存到 `replacement-text.json`
   - **警告**：不同的模板版面有不同的形狀數量——在創建替換前始終檢查實際清單

   顯示正確格式的範例 paragraphs 欄位：
   ```json
   "paragraphs": [
     {
       "text": "新簡報標題文字",
       "alignment": "CENTER",
       "bold": true
     },
     {
       "text": "章節標題",
       "bold": true
     },
     {
       "text": "不帶項目符號符號的第一個項目符號",
       "bullet": true,
       "level": 0
     },
     {
       "text": "紅色文字",
       "color": "FF0000"
     },
     {
       "text": "主題色彩文字",
       "theme_color": "DARK_1"
     },
     {
       "text": "無特殊格式的普通段落文字"
     }
   ]
   ```

   **替換 JSON 中未列出的形狀將自動清除**：
   ```json
   {
     "slide-0": {
       "shape-0": {
         "paragraphs": [...] // 此形狀獲得新文字
       }
       // 清單中的 shape-1 和 shape-2 將自動清除
     }
   }
   ```

   **簡報的常見格式模式**：
   - 標題投影片：粗體文字，有時置中
   - 投影片內的章節標題：粗體文字
   - 項目符號列表：每個項目需要 `"bullet": true, "level": 0`
   - 正文：通常不需要特殊屬性
   - 引言：可能有特殊對齊或字型屬性

7. **使用 `replace.py` 腳本應用替換**
   ```bash
   python scripts/replace.py working.pptx replacement-text.json output.pptx
   ```

   腳本將：
   - 首先使用 inventory.py 的函數擷取所有文字形狀的清單
   - 驗證替換 JSON 中的所有形狀都存在於清單中
   - 清除清單中識別的所有形狀的文字
   - 只對替換 JSON 中定義了「paragraphs」的形狀應用新文字
   - 透過應用 JSON 中的段落屬性保留格式
   - 自動處理項目符號、對齊、字型屬性和色彩
   - 儲存更新的簡報

   驗證錯誤範例：
   ```
   ERROR: Invalid shapes in replacement JSON:
     - Shape 'shape-99' not found on 'slide-0'. Available shapes: shape-0, shape-1, shape-4
     - Slide 'slide-999' not found in inventory
   ```

   ```
   ERROR: Replacement text made overflow worse in these shapes:
     - slide-0/shape-2: overflow worsened by 1.25" (was 0.00", now 1.25")
   ```

## 創建縮圖網格

為快速分析和參考創建 PowerPoint 投影片的視覺縮圖網格：

```bash
python scripts/thumbnail.py template.pptx [output_prefix]
```

**功能**：
- 創建：`thumbnails.jpg`（或大型投影片組的 `thumbnails-1.jpg`、`thumbnails-2.jpg` 等）
- 預設：5 欄，每網格最多 30 張投影片（5×6）
- 自定義前綴：`python scripts/thumbnail.py template.pptx my-grid`
  - 注意：如果你想在特定目錄輸出，輸出前綴應包含路徑（例如 `workspace/my-grid`）
- 調整欄數：`--cols 4`（範圍：3-6，影響每網格投影片數）
- 網格限制：3 欄 = 12 張投影片/網格，4 欄 = 20，5 欄 = 30，6 欄 = 42
- 投影片從零開始索引（投影片 0、投影片 1 等）

**使用案例**：
- 模板分析：快速理解投影片版面和設計模式
- 內容審查：整個簡報的視覺概覽
- 導覽參考：透過視覺外觀找到特定投影片
- 品質檢查：驗證所有投影片格式正確

**範例**：
```bash
# 基本用法
python scripts/thumbnail.py presentation.pptx

# 組合選項：自定義名稱、欄數
python scripts/thumbnail.py template.pptx analysis --cols 4
```

## 將投影片轉換為圖片

要視覺化分析 PowerPoint 投影片，使用兩步驟過程將它們轉換為圖片：

1. **將 PPTX 轉換為 PDF**：
   ```bash
   soffice --headless --convert-to pdf template.pptx
   ```

2. **將 PDF 頁面轉換為 JPEG 圖片**：
   ```bash
   pdftoppm -jpeg -r 150 template.pdf slide
   ```
   這會創建 `slide-1.jpg`、`slide-2.jpg` 等檔案。

選項：
- `-r 150`：設定解析度為 150 DPI（調整以平衡品質/大小）
- `-jpeg`：輸出 JPEG 格式（如偏好可使用 `-png` 輸出 PNG）
- `-f N`：要轉換的第一頁（例如 `-f 2` 從第 2 頁開始）
- `-l N`：要轉換的最後一頁（例如 `-l 5` 在第 5 頁停止）
- `slide`：輸出檔案的前綴

特定範圍範例：
```bash
pdftoppm -jpeg -r 150 -f 2 -l 5 template.pdf slide  # 只轉換第 2-5 頁
```

## 程式碼風格指南
**重要**：生成 PPTX 操作程式碼時：
- 撰寫簡潔的程式碼
- 避免冗長的變數名稱和多餘的操作
- 避免不必要的 print 陳述式

## 依賴項

必需的依賴項（應已安裝）：

- **markitdown**：`pip install "markitdown[pptx]"`（用於從簡報擷取文字）
- **pptxgenjs**：`npm install -g pptxgenjs`（用於透過 html2pptx 創建簡報）
- **playwright**：`npm install -g playwright`（用於 html2pptx 中的 HTML 渲染）
- **react-icons**：`npm install -g react-icons react react-dom`（用於圖示）
- **sharp**：`npm install -g sharp`（用於 SVG 柵格化和圖片處理）
- **LibreOffice**：`sudo apt-get install libreoffice`（用於 PDF 轉換）
- **Poppler**：`sudo apt-get install poppler-utils`（用於 pdftoppm 將 PDF 轉換為圖片）
- **defusedxml**：`pip install defusedxml`（用於安全的 XML 解析）
