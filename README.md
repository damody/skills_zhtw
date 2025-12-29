> **注意：** 此儲存庫包含 Anthropic 為 Claude 實作的技能。有關 Agent Skills 標準的資訊，請參閱 [agentskills.io](http://agentskills.io)。

# 技能 (Skills)
技能是由指令、腳本和資源組成的資料夾，Claude 會動態載入這些內容以提升在專業任務上的表現。技能教導 Claude 如何以可重複的方式完成特定任務，無論是使用貴公司的品牌指南建立文件、使用貴組織的特定工作流程分析資料，還是自動化個人任務。

更多資訊，請參閱：
- [什麼是技能？](https://support.claude.com/en/articles/12512176-what-are-skills)
- [在 Claude 中使用技能](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [如何建立自訂技能](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [透過 Agent Skills 為真實世界裝備代理](https://anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

# 關於此儲存庫

此儲存庫包含展示 Claude 技能系統可能性的技能。這些技能範圍從創意應用（藝術、音樂、設計）到技術任務（測試網頁應用程式、MCP 伺服器生成）再到企業工作流程（通訊、品牌等）。

每個技能都獨立包含在自己的資料夾中，並有一個 `SKILL.md` 檔案，其中包含 Claude 使用的指令和中繼資料。瀏覽這些技能可以為您自己的技能獲取靈感，或了解不同的模式和方法。

此儲存庫中的許多技能都是開源的（Apache 2.0）。我們還包含了驅動 [Claude 文件功能](https://www.anthropic.com/news/create-files) 的文件建立和編輯技能，位於 [`skills/docx`](./skills/docx)、[`skills/pdf`](./skills/pdf)、[`skills/pptx`](./skills/pptx) 和 [`skills/xlsx`](./skills/xlsx) 子資料夾中。這些是開放原始碼，但非開源軟體。我們希望與開發者分享這些作為在生產 AI 應用程式中實際使用的更複雜技能的參考。

## 免責聲明

**這些技能僅供示範和教育目的。** 雖然 Claude 可能提供其中一些功能，但您從 Claude 獲得的實作和行為可能與這些技能中顯示的不同。這些技能旨在說明模式和可能性。在將技能用於關鍵任務之前，請務必在您自己的環境中徹底測試。

# 技能集
- [./skills](./skills)：創意與設計、開發與技術、企業與通訊以及文件技能的範例
- [./spec](./spec)：Agent Skills 規範
- [./template](./template)：技能模板

# 在 Claude Code、Claude.ai 和 API 中試用

## Claude Code
您可以透過在 Claude Code 中執行以下命令，將此儲存庫註冊為 Claude Code 外掛程式市場：
```
/plugin marketplace add anthropics/skills
```

然後，要安裝特定的技能集：
1. 選擇 `Browse and install plugins`
2. 選擇 `anthropic-agent-skills`
3. 選擇 `document-skills` 或 `example-skills`
4. 選擇 `Install now`

或者，直接透過以下方式安裝任一外掛程式：
```
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
```

安裝外掛程式後，您只需提及它即可使用該技能。例如，如果您從市場安裝了 `document-skills` 外掛程式，您可以要求 Claude Code 執行類似：「使用 PDF 技能從 `path/to/some-file.pdf` 提取表單欄位」

## Claude.ai

這些範例技能都已在 Claude.ai 的付費方案中提供。

要使用此儲存庫中的任何技能或上傳自訂技能，請按照[在 Claude 中使用技能](https://support.claude.com/en/articles/12512180-using-skills-in-claude#h_a4222fa77b)中的說明操作。

## Claude API

您可以透過 Claude API 使用 Anthropic 的預建技能和上傳自訂技能。詳情請參閱 [Skills API 快速入門](https://docs.claude.com/en/api/skills-guide#creating-a-skill)。

# 建立基本技能

技能很容易建立 - 只需一個包含 `SKILL.md` 檔案的資料夾，其中包含 YAML 前置內容和指令。您可以使用此儲存庫中的 **template-skill** 作為起點：

```markdown
---
name: my-skill-name
description: 清楚描述此技能的功能以及何時使用它
---

# 我的技能名稱

[在此添加 Claude 啟用此技能時將遵循的指令]

## 範例
- 使用範例 1
- 使用範例 2

## 指南
- 指南 1
- 指南 2
```

前置內容只需要兩個欄位：
- `name` - 技能的唯一識別碼（小寫，使用連字符代替空格）
- `description` - 完整描述技能的功能以及何時使用它

下方的 markdown 內容包含 Claude 將遵循的指令、範例和指南。更多詳情，請參閱[如何建立自訂技能](https://support.claude.com/en/articles/12512198-creating-custom-skills)。

# 合作夥伴技能

技能是教導 Claude 如何更好地使用特定軟體的絕佳方式。當我們看到來自合作夥伴的優秀範例技能時，我們可能會在此突出展示其中一些：

- **Notion** - [Notion Skills for Claude](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0)
