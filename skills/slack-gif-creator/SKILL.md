---
name: Slack GIF 創建器
description: 創建為 Slack 優化的動畫 GIF 的知識和工具。提供限制、驗證工具和動畫概念。當使用者請求為 Slack 製作動畫 GIF 時使用，例如「幫我製作 X 做 Y 的 GIF 給 Slack」。
license: 完整條款請見 LICENSE.txt
---

# Slack GIF 創建器

一個提供工具和知識的工具包，用於創建為 Slack 優化的動畫 GIF。

## Slack 要求

**尺寸：**
- 表情符號 GIF：128x128（建議）
- 訊息 GIF：480x480

**參數：**
- FPS：10-30（較低 = 較小檔案大小）
- 色彩：48-128（較少 = 較小檔案大小）
- 時長：表情符號 GIF 保持在 3 秒以下

## 核心工作流程

```python
from core.gif_builder import GIFBuilder
from PIL import Image, ImageDraw

# 1. 創建建構器
builder = GIFBuilder(width=128, height=128, fps=10)

# 2. 生成幀
for i in range(12):
    frame = Image.new('RGB', (128, 128), (240, 248, 255))
    draw = ImageDraw.Draw(frame)

    # 使用 PIL 基本圖形繪製動畫
    #（圓形、多邊形、線條等）

    builder.add_frame(frame)

# 3. 儲存並優化
builder.save('output.gif', num_colors=48, optimize_for_emoji=True)
```

## 繪製圖形

### 處理使用者上傳的圖片
如果使用者上傳圖片，考慮他們想要：
- **直接使用**（例如「讓這個動起來」、「把這個分成幾幀」）
- **作為靈感**（例如「做類似這樣的東西」）

使用 PIL 載入和處理圖片：
```python
from PIL import Image

uploaded = Image.open('file.png')
# 直接使用，或只是作為色彩/風格的參考
```

### 從頭繪製
從頭繪製圖形時，使用 PIL ImageDraw 基本圖形：

```python
from PIL import ImageDraw

draw = ImageDraw.Draw(frame)

# 圓形/橢圓形
draw.ellipse([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)

# 星形、三角形、任意多邊形
points = [(x1, y1), (x2, y2), (x3, y3), ...]
draw.polygon(points, fill=(r, g, b), outline=(r, g, b), width=3)

# 線條
draw.line([(x1, y1), (x2, y2)], fill=(r, g, b), width=5)

# 矩形
draw.rectangle([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)
```

**不要使用：** 表情符號字體（跨平台不可靠）或假設此技能中存在預打包的圖形。

### 讓圖形看起來更好

圖形應該看起來精緻且有創意，而非基本。方法如下：

**使用較粗的線條** - 輪廓和線條始終設定 `width=2` 或更高。細線（width=1）看起來參差不齊且業餘。

**添加視覺深度**：
- 背景使用漸層（`create_gradient_background`）
- 分層多個形狀增加複雜度（例如大星星裡面有小星星）

**讓形狀更有趣**：
- 不要只畫一個普通圓形——添加高光、光環或圖案
- 星形可以有光暈（在後面畫更大、半透明的版本）
- 組合多個形狀（星星 + 閃光、圓形 + 光環）

**注意色彩**：
- 使用鮮豔、互補的色彩
- 添加對比（淺色形狀用深色輪廓，深色形狀用淺色輪廓）
- 考慮整體構圖

**對於複雜形狀**（心形、雪花等）：
- 使用多邊形和橢圓的組合
- 仔細計算點以確保對稱
- 添加細節（心形可以有高光曲線，雪花有複雜的分支）

發揮創意，注重細節！好的 Slack GIF 應該看起來精緻，而非佔位圖形。

## 可用工具

### GIFBuilder (`core.gif_builder`)
組裝幀並為 Slack 優化：
```python
builder = GIFBuilder(width=128, height=128, fps=10)
builder.add_frame(frame)  # 添加 PIL Image
builder.add_frames(frames)  # 添加幀列表
builder.save('out.gif', num_colors=48, optimize_for_emoji=True, remove_duplicates=True)
```

### 驗證器 (`core.validators`)
檢查 GIF 是否符合 Slack 要求：
```python
from core.validators import validate_gif, is_slack_ready

# 詳細驗證
passes, info = validate_gif('my.gif', is_emoji=True, verbose=True)

# 快速檢查
if is_slack_ready('my.gif'):
    print("準備好了！")
```

### 緩動函數 (`core.easing`)
平滑運動而非線性：
```python
from core.easing import interpolate

# 進度從 0.0 到 1.0
t = i / (num_frames - 1)

# 應用緩動
y = interpolate(start=0, end=400, t=t, easing='ease_out')

# 可用：linear、ease_in、ease_out、ease_in_out、
#       bounce_out、elastic_out、back_out
```

### 幀輔助函數 (`core.frame_composer`)
常見需求的便利函數：
```python
from core.frame_composer import (
    create_blank_frame,         # 純色背景
    create_gradient_background,  # 垂直漸層
    draw_circle,                # 圓形輔助
    draw_text,                  # 簡單文字渲染
    draw_star                   # 五角星
)
```

## 動畫概念

### 抖動/震動
用振盪偏移物件位置：
- 使用 `math.sin()` 或 `math.cos()` 搭配幀索引
- 添加小的隨機變化以獲得自然感覺
- 應用於 x 和/或 y 位置

### 脈動/心跳
有節奏地縮放物件大小：
- 使用 `math.sin(t * frequency * 2 * math.pi)` 實現平滑脈動
- 對於心跳：兩次快速脈動然後暫停（調整正弦波）
- 在基本大小的 0.8 到 1.2 之間縮放

### 彈跳
物件落下並彈跳：
- 使用 `interpolate()` 搭配 `easing='bounce_out'` 實現落地
- 使用 `easing='ease_in'` 實現下落（加速）
- 透過每幀增加 y 速度來應用重力

### 旋轉/轉動
圍繞中心旋轉物件：
- PIL：`image.rotate(angle, resample=Image.BICUBIC)`
- 對於擺動：使用正弦波而非線性改變角度

### 淡入/淡出
逐漸出現或消失：
- 創建 RGBA 圖片，調整 alpha 通道
- 或使用 `Image.blend(image1, image2, alpha)`
- 淡入：alpha 從 0 到 1
- 淡出：alpha 從 1 到 0

### 滑動
將物件從螢幕外移動到位置：
- 起始位置：在幀邊界外
- 結束位置：目標位置
- 使用 `interpolate()` 搭配 `easing='ease_out'` 實現平滑停止
- 對於超越效果：使用 `easing='back_out'`

### 縮放
縮放和定位以實現縮放效果：
- 放大：從 0.1 縮放到 2.0，裁剪中心
- 縮小：從 2.0 縮放到 1.0
- 可添加運動模糊增加戲劇效果（PIL 濾鏡）

### 爆炸/粒子爆發
創建向外輻射的粒子：
- 生成具有隨機角度和速度的粒子
- 更新每個粒子：`x += vx`、`y += vy`
- 添加重力：`vy += gravity_constant`
- 隨時間淡出粒子（降低 alpha）

## 優化策略

只有在被要求縮小檔案大小時，實施以下幾種方法：

1. **更少幀** - 較低 FPS（10 而非 20）或較短時長
2. **更少色彩** - `num_colors=48` 而非 128
3. **更小尺寸** - 128x128 而非 480x480
4. **移除重複** - save() 中 `remove_duplicates=True`
5. **表情符號模式** - `optimize_for_emoji=True` 自動優化

```python
# 表情符號的最大優化
builder.save(
    'emoji.gif',
    num_colors=48,
    optimize_for_emoji=True,
    remove_duplicates=True
)
```

## 理念

此技能提供：
- **知識**：Slack 的要求和動畫概念
- **工具**：GIFBuilder、驗證器、緩動函數
- **靈活性**：使用 PIL 基本圖形創建動畫邏輯

它**不**提供：
- 僵化的動畫模板或預製函數
- 表情符號字體渲染（跨平台不可靠）
- 內建於技能的預打包圖形庫

**關於使用者上傳的說明**：此技能不包含預建圖形，但如果使用者上傳圖片，使用 PIL 載入和處理它——根據他們的請求解釋他們想要直接使用它還是只是作為靈感。

發揮創意！組合概念（彈跳 + 旋轉、脈動 + 滑動等）並使用 PIL 的全部功能。

## 依賴項

```bash
pip install pillow imageio numpy
```
