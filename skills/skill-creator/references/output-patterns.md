# 輸出模式

當技能需要產生一致、高品質的輸出時，請使用這些模式。

## 範本模式

為輸出格式提供範本。根據您的需求匹配嚴格程度。

**對於嚴格要求（如 API 回應或資料格式）：**

```markdown
## 報告結構

始終使用這個確切的範本結構：

# [分析標題]

## 執行摘要
[關鍵發現的單段概述]

## 關鍵發現
- 發現 1 及支持資料
- 發現 2 及支持資料
- 發現 3 及支持資料

## 建議
1. 具體可行的建議
2. 具體可行的建議
```

**對於靈活指導（當調整有用時）：**

```markdown
## 報告結構

這是一個合理的預設格式，但請使用您的最佳判斷：

# [分析標題]

## 執行摘要
[概述]

## 關鍵發現
[根據您的發現調整章節]

## 建議
[根據具體情境調整]

根據具體分析類型按需調整章節。
```

## 範例模式

對於輸出品質取決於看到範例的技能，提供輸入/輸出配對：

```markdown
## 提交訊息格式

依照這些範例產生提交訊息：

**範例 1：**
輸入：Added user authentication with JWT tokens
輸出：
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**範例 2：**
輸入：Fixed bug where dates displayed incorrectly in reports
輸出：
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

遵循此風格：type(scope): brief description，然後是詳細說明。
```

範例比單純描述更能幫助 Claude 理解所需的風格和詳細程度。
