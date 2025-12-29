---
name: xlsx
description: "全面的試算表創建、編輯和分析，支援公式、格式、資料分析和視覺化。當 Claude 需要處理試算表（.xlsx、.xlsm、.csv、.tsv 等）時使用：(1) 創建帶公式和格式的新試算表、(2) 讀取或分析資料、(3) 修改現有試算表同時保留公式、(4) 試算表中的資料分析和視覺化，或 (5) 重新計算公式"
license: 專有。LICENSE.txt 包含完整條款
---

# 輸出要求

## 所有 Excel 檔案

### 零公式錯誤
- 每個 Excel 模型必須以零公式錯誤交付（#REF!、#DIV/0!、#VALUE!、#N/A、#NAME?）

### 保留現有模板（修改模板時）
- 修改檔案時研究並精確匹配現有格式、樣式和慣例
- 永遠不要對已建立模式的檔案強加標準化格式
- 現有模板慣例始終優先於這些指南

## 財務模型

### 色彩編碼標準
除非使用者或現有模板另有說明

#### 行業標準色彩慣例
- **藍色文字（RGB: 0,0,255）**：硬編碼輸入，以及使用者將為情境更改的數字
- **黑色文字（RGB: 0,0,0）**：所有公式和計算
- **綠色文字（RGB: 0,128,0）**：從同一活頁簿內其他工作表拉取的連結
- **紅色文字（RGB: 255,0,0）**：連結到其他檔案的外部連結
- **黃色背景（RGB: 255,255,0）**：需要注意的關鍵假設或需要更新的儲存格

### 數字格式標準

#### 必需的格式規則
- **年份**：格式為文字字串（例如「2024」而非「2,024」）
- **貨幣**：使用 $#,##0 格式；始終在標題中指定單位（「收入（$mm）」）
- **零**：使用數字格式使所有零顯示為「-」，包括百分比（例如「$#,##0;($#,##0);-」）
- **百分比**：預設為 0.0% 格式（一位小數）
- **倍數**：估值倍數格式為 0.0x（EV/EBITDA、P/E）
- **負數**：使用括號 (123) 而非減號 -123

### 公式建構規則

#### 假設放置
- 將所有假設（成長率、利潤率、倍數等）放在單獨的假設儲存格中
- 在公式中使用儲存格引用而非硬編碼值
- 範例：使用 =B5*(1+$B$6) 而非 =B5*1.05

#### 公式錯誤預防
- 驗證所有儲存格引用正確
- 檢查範圍中的差一錯誤
- 確保所有預測期間的公式一致
- 用邊緣情況測試（零值、負數）
- 驗證沒有非預期的循環引用

#### 硬編碼的文件要求
- 在旁邊的儲存格中註解或說明（如果在表格末尾）。格式：「來源：[系統/文件]、[日期]、[具體引用]、[URL（如適用）]」
- 範例：
  - 「來源：公司 10-K、FY2024、第 45 頁、收入附註、[SEC EDGAR URL]」
  - 「來源：公司 10-Q、2025 年第 2 季、附件 99.1、[SEC EDGAR URL]」
  - 「來源：Bloomberg 終端、2025/8/15、AAPL US Equity」
  - 「來源：FactSet、2025/8/20、共識估計畫面」

# XLSX 創建、編輯和分析

## 概述

使用者可能會要求你創建、編輯或分析 .xlsx 檔案的內容。對於不同的任務，你有不同的工具和工作流程可用。

## 重要要求

**公式重新計算需要 LibreOffice**：你可以假設 LibreOffice 已安裝，用於使用 `recalc.py` 腳本重新計算公式值。該腳本在首次執行時自動配置 LibreOffice

## 讀取和分析資料

### 使用 pandas 進行資料分析
對於資料分析、視覺化和基本操作，使用 **pandas**，它提供強大的資料操作功能：

```python
import pandas as pd

# 讀取 Excel
df = pd.read_excel('file.xlsx')  # 預設：第一個工作表
all_sheets = pd.read_excel('file.xlsx', sheet_name=None)  # 所有工作表作為字典

# 分析
df.head()      # 預覽資料
df.info()      # 欄位資訊
df.describe()  # 統計資料

# 寫入 Excel
df.to_excel('output.xlsx', index=False)
```

## Excel 檔案工作流程

## 關鍵：使用公式，而非硬編碼值

**始終使用 Excel 公式，而非在 Python 中計算值然後硬編碼。** 這確保試算表保持動態和可更新。

### 錯誤 - 硬編碼計算值
```python
# 錯誤：在 Python 中計算並硬編碼結果
total = df['Sales'].sum()
sheet['B10'] = total  # 硬編碼 5000

# 錯誤：在 Python 中計算成長率
growth = (df.iloc[-1]['Revenue'] - df.iloc[0]['Revenue']) / df.iloc[0]['Revenue']
sheet['C5'] = growth  # 硬編碼 0.15

# 錯誤：Python 計算平均值
avg = sum(values) / len(values)
sheet['D20'] = avg  # 硬編碼 42.5
```

### 正確 - 使用 Excel 公式
```python
# 正確：讓 Excel 計算總和
sheet['B10'] = '=SUM(B2:B9)'

# 正確：成長率作為 Excel 公式
sheet['C5'] = '=(C4-C2)/C2'

# 正確：使用 Excel 函數計算平均值
sheet['D20'] = '=AVERAGE(D2:D19)'
```

這適用於所有計算——總計、百分比、比率、差異等。試算表應該能夠在來源資料更改時重新計算。

## 常見工作流程
1. **選擇工具**：pandas 用於資料，openpyxl 用於公式/格式
2. **創建/載入**：創建新活頁簿或載入現有檔案
3. **修改**：添加/編輯資料、公式和格式
4. **儲存**：寫入檔案
5. **重新計算公式（如使用公式則必須）**：使用 recalc.py 腳本
   ```bash
   python recalc.py output.xlsx
   ```
6. **驗證並修復任何錯誤**：
   - 腳本返回帶有錯誤詳情的 JSON
   - 如果 `status` 是 `errors_found`，檢查 `error_summary` 以獲取具體錯誤類型和位置
   - 修復識別的錯誤並再次重新計算
   - 常見要修復的錯誤：
     - `#REF!`：無效的儲存格引用
     - `#DIV/0!`：除以零
     - `#VALUE!`：公式中的錯誤資料類型
     - `#NAME?`：無法識別的公式名稱

### 創建新 Excel 檔案

```python
# 使用 openpyxl 處理公式和格式
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = Workbook()
sheet = wb.active

# 添加資料
sheet['A1'] = 'Hello'
sheet['B1'] = 'World'
sheet.append(['Row', 'of', 'data'])

# 添加公式
sheet['B2'] = '=SUM(A1:A10)'

# 格式
sheet['A1'].font = Font(bold=True, color='FF0000')
sheet['A1'].fill = PatternFill('solid', start_color='FFFF00')
sheet['A1'].alignment = Alignment(horizontal='center')

# 欄寬
sheet.column_dimensions['A'].width = 20

wb.save('output.xlsx')
```

### 編輯現有 Excel 檔案

```python
# 使用 openpyxl 保留公式和格式
from openpyxl import load_workbook

# 載入現有檔案
wb = load_workbook('existing.xlsx')
sheet = wb.active  # 或 wb['SheetName'] 用於特定工作表

# 處理多個工作表
for sheet_name in wb.sheetnames:
    sheet = wb[sheet_name]
    print(f"工作表：{sheet_name}")

# 修改儲存格
sheet['A1'] = '新值'
sheet.insert_rows(2)  # 在位置 2 插入行
sheet.delete_cols(3)  # 刪除第 3 欄

# 添加新工作表
new_sheet = wb.create_sheet('NewSheet')
new_sheet['A1'] = '資料'

wb.save('modified.xlsx')
```

## 重新計算公式

由 openpyxl 創建或修改的 Excel 檔案包含作為字串的公式但沒有計算值。使用提供的 `recalc.py` 腳本重新計算公式：

```bash
python recalc.py <excel_file> [timeout_seconds]
```

範例：
```bash
python recalc.py output.xlsx 30
```

腳本：
- 首次執行時自動設置 LibreOffice 巨集
- 重新計算所有工作表中的所有公式
- 掃描所有儲存格以查找 Excel 錯誤（#REF!、#DIV/0! 等）
- 返回帶有詳細錯誤位置和計數的 JSON
- 在 Linux 和 macOS 上都可工作

## 公式驗證檢查清單

確保公式正確工作的快速檢查：

### 基本驗證
- [ ] **測試 2-3 個樣本引用**：在建構完整模型前驗證它們拉取正確的值
- [ ] **欄對應**：確認 Excel 欄匹配（例如欄 64 = BL，不是 BK）
- [ ] **行偏移**：記住 Excel 行從 1 開始索引（DataFrame 第 5 行 = Excel 第 6 行）

### 常見陷阱
- [ ] **NaN 處理**：使用 `pd.notna()` 檢查空值
- [ ] **最右欄**：FY 資料通常在欄 50+
- [ ] **多個匹配**：搜尋所有出現，不只是第一個
- [ ] **除以零**：在公式中使用 `/` 前檢查分母（#DIV/0!）
- [ ] **錯誤引用**：驗證所有儲存格引用指向預期的儲存格（#REF!）
- [ ] **跨工作表引用**：使用正確格式（Sheet1!A1）連結工作表

### 公式測試策略
- [ ] **從小開始**：在廣泛應用前先在 2-3 個儲存格上測試公式
- [ ] **驗證依賴項**：檢查公式中引用的所有儲存格都存在
- [ ] **測試邊緣情況**：包括零、負數和非常大的值

### 解讀 recalc.py 輸出
腳本返回帶有錯誤詳情的 JSON：
```json
{
  "status": "success",           // 或 "errors_found"
  "total_errors": 0,              // 錯誤總數
  "total_formulas": 42,           // 檔案中的公式數量
  "error_summary": {              // 僅在發現錯誤時出現
    "#REF!": {
      "count": 2,
      "locations": ["Sheet1!B5", "Sheet1!C10"]
    }
  }
}
```

## 最佳實踐

### 程式庫選擇
- **pandas**：最適合資料分析、批量操作和簡單資料匯出
- **openpyxl**：最適合複雜格式、公式和 Excel 特定功能

### 使用 openpyxl
- 儲存格索引從 1 開始（row=1、column=1 指的是儲存格 A1）
- 使用 `data_only=True` 讀取計算值：`load_workbook('file.xlsx', data_only=True)`
- **警告**：如果用 `data_only=True` 開啟並儲存，公式會被值替換並永久丟失
- 對於大檔案：讀取時使用 `read_only=True`，寫入時使用 `write_only=True`
- 公式被保留但不評估——使用 recalc.py 更新值

### 使用 pandas
- 指定資料類型以避免推斷問題：`pd.read_excel('file.xlsx', dtype={'id': str})`
- 對於大檔案，讀取特定欄：`pd.read_excel('file.xlsx', usecols=['A', 'C', 'E'])`
- 正確處理日期：`pd.read_excel('file.xlsx', parse_dates=['date_column'])`

## 程式碼風格指南
**重要**：生成 Excel 操作的 Python 程式碼時：
- 撰寫簡潔的 Python 程式碼，不要不必要的註解
- 避免冗長的變數名稱和多餘的操作
- 避免不必要的 print 陳述式

**對於 Excel 檔案本身**：
- 為複雜公式或重要假設的儲存格添加註解
- 記錄硬編碼值的資料來源
- 為關鍵計算和模型章節包含說明
