---
name: 網頁 Artifacts 建構器
description: 使用現代前端網頁技術（React、Tailwind CSS、shadcn/ui）創建精緻、多元件的 claude.ai HTML artifacts 的工具套件。用於需要狀態管理、路由或 shadcn/ui 元件的複雜 artifacts——不適用於簡單的單檔案 HTML/JSX artifacts。
license: 完整條款請見 LICENSE.txt
---

# 網頁 Artifacts 建構器

要建構強大的前端 claude.ai artifacts，遵循以下步驟：
1. 使用 `scripts/init-artifact.sh` 初始化前端儲存庫
2. 透過編輯生成的程式碼開發你的 artifact
3. 使用 `scripts/bundle-artifact.sh` 將所有程式碼捆綁成單一 HTML 檔案
4. 向使用者展示 artifact
5.（可選）測試 artifact

**技術堆疊**：React 18 + TypeScript + Vite + Parcel（捆綁）+ Tailwind CSS + shadcn/ui

## 設計與樣式指南

非常重要：為避免通常所謂的「AI 廢話」，避免使用過多的置中版面、紫色漸層、統一的圓角和 Inter 字體。

## 快速開始

### 步驟 1：初始化專案

執行初始化腳本創建新的 React 專案：
```bash
bash scripts/init-artifact.sh <project-name>
cd <project-name>
```

這會創建一個完全配置的專案，包含：
- React + TypeScript（透過 Vite）
- Tailwind CSS 3.4.1 帶 shadcn/ui 主題系統
- 路徑別名（`@/`）已配置
- 40+ shadcn/ui 元件預安裝
- 包含所有 Radix UI 依賴項
- Parcel 已配置用於捆綁（透過 .parcelrc）
- Node 18+ 相容性（自動偵測並固定 Vite 版本）

### 步驟 2：開發你的 Artifact

要建構 artifact，編輯生成的檔案。參見下方**常見開發任務**以獲取指導。

### 步驟 3：捆綁為單一 HTML 檔案

要將 React 應用捆綁成單一 HTML artifact：
```bash
bash scripts/bundle-artifact.sh
```

這會創建 `bundle.html`——一個自包含的 artifact，所有 JavaScript、CSS 和依賴項都已內聯。此檔案可以直接在 Claude 對話中作為 artifact 分享。

**要求**：你的專案必須在根目錄有一個 `index.html`。

**腳本做什麼**：
- 安裝捆綁依賴項（parcel、@parcel/config-default、parcel-resolver-tspaths、html-inline）
- 創建帶有路徑別名支援的 `.parcelrc` 配置
- 使用 Parcel 建構（無 source maps）
- 使用 html-inline 將所有資產內聯到單一 HTML

### 步驟 4：與使用者分享 Artifact

最後，在對話中與使用者分享捆綁的 HTML 檔案，讓他們可以作為 artifact 查看。

### 步驟 5：測試/視覺化 Artifact（可選）

注意：這是完全可選的步驟。只有在必要或被請求時才執行。

要測試/視覺化 artifact，使用可用工具（包括其他技能或內建工具如 Playwright 或 Puppeteer）。一般來說，避免提前測試 artifact，因為這會在請求和看到完成的 artifact 之間增加延遲。如果被請求或出現問題，在展示 artifact 之後再測試。

## 參考資料

- **shadcn/ui 元件**：https://ui.shadcn.com/docs/components
