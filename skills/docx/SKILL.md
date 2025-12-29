---
name: docx
description: "全面的文件創建、編輯和分析，支援追蹤修訂、註解、格式保留和文字擷取。當 Claude 需要處理專業文件（.docx 檔案）時使用：(1) 創建新文件、(2) 修改或編輯內容、(3) 處理追蹤修訂、(4) 添加註解，或任何其他文件任務"
license: 專有。LICENSE.txt 包含完整條款
---

# DOCX 創建、編輯和分析

## 概述

使用者可能會要求你創建、編輯或分析 .docx 檔案的內容。.docx 檔案本質上是一個包含 XML 檔案和其他資源的 ZIP 壓縮檔，你可以讀取或編輯。對於不同的任務，你有不同的工具和工作流程可用。

## 工作流程決策樹

### 讀取/分析內容
使用下方的「文字擷取」或「原始 XML 存取」章節

### 創建新文件
使用「創建新 Word 文件」工作流程

### 編輯現有文件
- **你自己的文件 + 簡單更改**
  使用「基本 OOXML 編輯」工作流程

- **他人的文件**
  使用**「紅線標記工作流程」**（建議預設）

- **法律、學術、商業或政府文件**
  使用**「紅線標記工作流程」**（必需）

## 讀取和分析內容

### 文字擷取
如果你只需要讀取文件的文字內容，應該使用 pandoc 將文件轉換為 markdown。Pandoc 提供出色的文件結構保留支援，並可顯示追蹤修訂：

```bash
# 將文件轉換為 markdown 並保留追蹤修訂
pandoc --track-changes=all path-to-file.docx -o output.md
# 選項：--track-changes=accept/reject/all
```

### 原始 XML 存取
你需要原始 XML 存取來處理：註解、複雜格式、文件結構、嵌入媒體和中繼資料。對於這些功能，你需要解壓文件並讀取其原始 XML 內容。

#### 解壓檔案
`python ooxml/scripts/unpack.py <office_file> <output_directory>`

#### 關鍵檔案結構
* `word/document.xml` - 主要文件內容
* `word/comments.xml` - document.xml 中引用的註解
* `word/media/` - 嵌入的圖片和媒體檔案
* 追蹤修訂使用 `<w:ins>`（插入）和 `<w:del>`（刪除）標籤

## 創建新 Word 文件

從頭創建新 Word 文件時，使用 **docx-js**，它允許你使用 JavaScript/TypeScript 創建 Word 文件。

### 工作流程
1. **必須 - 讀取整個檔案**：從頭到尾完整讀取 [`docx-js.md`](docx-js.md)（約 500 行）。**讀取此檔案時絕不設定任何範圍限制。** 讀取完整檔案內容以獲取詳細語法、關鍵格式規則和最佳實踐，然後再進行文件創建。
2. 使用 Document、Paragraph、TextRun 元件創建 JavaScript/TypeScript 檔案（你可以假設所有依賴項都已安裝，如果沒有，請參閱下方的依賴項章節）
3. 使用 Packer.toBuffer() 匯出為 .docx

## 編輯現有 Word 文件

編輯現有 Word 文件時，使用 **Document 程式庫**（用於 OOXML 操作的 Python 程式庫）。該程式庫自動處理基礎設施設置並提供文件操作方法。對於複雜場景，你可以透過程式庫直接存取底層 DOM。

### 工作流程
1. **必須 - 讀取整個檔案**：從頭到尾完整讀取 [`ooxml.md`](ooxml.md)（約 600 行）。**讀取此檔案時絕不設定任何範圍限制。** 讀取完整檔案內容以獲取 Document 程式庫 API 和直接編輯文件檔案的 XML 模式。
2. 解壓文件：`python ooxml/scripts/unpack.py <office_file> <output_directory>`
3. 使用 Document 程式庫創建並執行 Python 腳本（參見 ooxml.md 中的「Document 程式庫」章節）
4. 打包最終文件：`python ooxml/scripts/pack.py <input_directory> <office_file>`

Document 程式庫為常見操作提供高階方法，也為複雜場景提供直接 DOM 存取。

## 文件審查的紅線標記工作流程

此工作流程允許你在實施 OOXML 之前使用 markdown 規劃全面的追蹤修訂。**關鍵**：對於完整的追蹤修訂，你必須系統地實施所有更改。

**批次策略**：將相關更改分組為 3-10 個更改的批次。這使除錯易於管理同時保持效率。在進行下一批次之前測試每個批次。

**原則：最小、精確的編輯**
實施追蹤修訂時，只標記實際更改的文字。重複未更改的文字使編輯更難審查且顯得不專業。將替換分解為：[未更改文字] + [刪除] + [插入] + [未更改文字]。透過從原始文件中擷取 `<w:r>` 元素並重複使用，保留原始 run 的 RSID 用於未更改的文字。

範例 - 在句子中將「30 天」更改為「60 天」：
```python
# 錯誤 - 替換整個句子
'<w:del><w:r><w:delText>The term is 30 days.</w:delText></w:r></w:del><w:ins><w:r><w:t>The term is 60 days.</w:t></w:r></w:ins>'

# 正確 - 只標記更改的部分，為未更改的文字保留原始 <w:r>
'<w:r w:rsidR="00AB12CD"><w:t>The term is </w:t></w:r><w:del><w:r><w:delText>30</w:delText></w:r></w:del><w:ins><w:r><w:t>60</w:t></w:r></w:ins><w:r w:rsidR="00AB12CD"><w:t> days.</w:t></w:r>'
```

### 追蹤修訂工作流程

1. **取得 markdown 表示**：將文件轉換為 markdown 並保留追蹤修訂：
   ```bash
   pandoc --track-changes=all path-to-file.docx -o current.md
   ```

2. **識別和分組更改**：審查文件並識別所需的所有更改，將它們組織成邏輯批次：

   **定位方法**（用於在 XML 中找到更改）：
   - 章節/標題編號（例如「第 3.2 節」、「第四條」）
   - 段落識別碼（如有編號）
   - 使用唯一周圍文字的 Grep 模式
   - 文件結構（例如「第一段」、「簽名區塊」）
   - **不要使用 markdown 行號** - 它們不對應 XML 結構

   **批次組織**（每批次分組 3-10 個相關更改）：
   - 按章節：「批次 1：第 2 節修訂」、「批次 2：第 5 節更新」
   - 按類型：「批次 1：日期更正」、「批次 2：當事人名稱更改」
   - 按複雜度：從簡單文字替換開始，然後處理複雜的結構更改
   - 按順序：「批次 1：第 1-3 頁」、「批次 2：第 4-6 頁」

3. **閱讀文件並解壓**：
   - **必須 - 讀取整個檔案**：從頭到尾完整讀取 [`ooxml.md`](ooxml.md)（約 600 行）。**讀取此檔案時絕不設定任何範圍限制。** 特別注意「Document 程式庫」和「追蹤修訂模式」章節。
   - **解壓文件**：`python ooxml/scripts/unpack.py <file.docx> <dir>`
   - **記錄建議的 RSID**：unpack 腳本會建議一個用於追蹤修訂的 RSID。複製此 RSID 以在步驟 4b 中使用。

4. **分批實施更改**：邏輯分組更改（按章節、類型或接近程度）並在單一腳本中一起實施。此方法：
   - 使除錯更容易（較小批次 = 更容易隔離錯誤）
   - 允許漸進式進展
   - 保持效率（3-10 個更改的批次效果良好）

   **建議的批次分組：**
   - 按文件章節（例如「第 3 節更改」、「定義」、「終止條款」）
   - 按更改類型（例如「日期更改」、「當事人名稱更新」、「法律術語替換」）
   - 按接近程度（例如「第 1-3 頁的更改」、「文件前半部分的更改」）

   對於每批相關更改：

   **a. 將文字對應到 XML**：在 `word/document.xml` 中 grep 文字以驗證文字如何分割在 `<w:r>` 元素中。

   **b. 創建並執行腳本**：使用 `get_node` 找到節點，實施更改，然後 `doc.save()`。參見 ooxml.md 中的**「Document 程式庫」**章節了解模式。

   **注意**：在撰寫腳本前，始終先 grep `word/document.xml` 以獲取當前行號並驗證文字內容。每次腳本執行後行號會改變。

5. **打包文件**：所有批次完成後，將解壓目錄轉換回 .docx：
   ```bash
   python ooxml/scripts/pack.py unpacked reviewed-document.docx
   ```

6. **最終驗證**：對完整文件進行全面檢查：
   - 將最終文件轉換為 markdown：
     ```bash
     pandoc --track-changes=all reviewed-document.docx -o verification.md
     ```
   - 驗證所有更改都已正確應用：
     ```bash
     grep "original phrase" verification.md  # 不應該找到它
     grep "replacement phrase" verification.md  # 應該找到它
     ```
   - 檢查沒有引入非預期的更改


## 將文件轉換為圖片

要視覺化分析 Word 文件，使用兩步驟過程將它們轉換為圖片：

1. **將 DOCX 轉換為 PDF**：
   ```bash
   soffice --headless --convert-to pdf document.docx
   ```

2. **將 PDF 頁面轉換為 JPEG 圖片**：
   ```bash
   pdftoppm -jpeg -r 150 document.pdf page
   ```
   這會創建 `page-1.jpg`、`page-2.jpg` 等檔案。

選項：
- `-r 150`：設定解析度為 150 DPI（調整以平衡品質/大小）
- `-jpeg`：輸出 JPEG 格式（如偏好可使用 `-png` 輸出 PNG）
- `-f N`：要轉換的第一頁（例如 `-f 2` 從第 2 頁開始）
- `-l N`：要轉換的最後一頁（例如 `-l 5` 在第 5 頁停止）
- `page`：輸出檔案的前綴

特定範圍範例：
```bash
pdftoppm -jpeg -r 150 -f 2 -l 5 document.pdf page  # 只轉換第 2-5 頁
```

## 程式碼風格指南
**重要**：生成 DOCX 操作程式碼時：
- 撰寫簡潔的程式碼
- 避免冗長的變數名稱和多餘的操作
- 避免不必要的 print 陳述式

## 依賴項

必需的依賴項（如不可用請安裝）：

- **pandoc**：`sudo apt-get install pandoc`（用於文字擷取）
- **docx**：`npm install -g docx`（用於創建新文件）
- **LibreOffice**：`sudo apt-get install libreoffice`（用於 PDF 轉換）
- **Poppler**：`sudo apt-get install poppler-utils`（用於 pdftoppm 將 PDF 轉換為圖片）
- **defusedxml**：`pip install defusedxml`（用於安全的 XML 解析）
