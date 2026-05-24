# 問題反饋


- adding basket panel 非常卡頓 不管是打字還是做啥
- emoji icon 沒有選擇panel, 需要注意一下, 可以提供選擇 如果要套件就又更好
- benchmark應該要和symbol一樣 能夠有標的提示
  - 還有symbol目前仍不支持',' 的分隔, 這很影響體驗
    - 準確來說 我希望的是可以直接貼上 'Nvda, aapl, tsla, msft' 並且要regex 不管大小寫, 不管空格和逗號, 以及toast if not found , 但是跳過 其他的也會進入


![alt text](image-1.png)n
- spolight 現在挺不錯的 
- 但還有以下功能
  - 搜尋功能, 主要只要fit title, symbol 就有 比方說我打spy
    - 所有的benchmark/symbol title都會有
    - 下面有tags可以點 就是快速語法 比方說只想搜symbol
      - 就是symbol:ionq 類似這樣


# bug
- 所有emoji都是箱子
- sliderwindow bar設計邏輯整個有問題（小Emoji選擇）
  - 現在選心加入的會歪掉
  - 現在沒有動畫
  - emoji沒有顯示

> 補充 pplx對於該pill的解釋和推薦
--

從截圖  來看，你的 UI 是一個 **Segmented Control / Pill Tab Bar**，裡面放著字母圖示（U, U, C, G, T, S…），目前 active 的是 "C"（圓形 highlight）。這不是 emoji picker，而是一個**可以切換的 icon 選擇器**，搭配類似刻度/dot indicator 的視覺。 [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/71030829/a834394d-2938-41ee-bec4-761d02134265/image.jpg?AWSAccessKeyId=ASIA2F3EMEYESEB5HBXX&Signature=BnencOt7jRkwYZi2mgnqJn5K270%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEI3%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIAZZ4w3Du%2FQW2nYZIJK2fyOeV34pW06QGL5c1dRHq%2FlsAiEAs8oBS0R22E78vmEc6a%2FaIjdT2Jq00cufKizFsLgtj%2BUq8wQIVRABGgw2OTk3NTMzMDk3MDUiDGi1inaXcAxdDyZSZCrQBIG4EYQUkRLZ%2FuPMvZKV1TizqIrZJfctAPOrub6SM4rihfOH1mVR5e%2FQj6lM0ZVRuckGJ9uKHHkWa6vhuPvBcYMoT7rkWXpdlUxRYKIrsHo5fMg3SodHDySrFCmI%2BymZfX%2BASIH3v2cWihTo4bYIfk2kXroBt%2FsWs%2BbXSpknBybMIlZP%2F%2FBtVtvyqOsakN3cSabayBRFxiFSdASi0Sxo1lEHe7m6ZfGof3SBC8qfsjQ6C6T%2BEt28BPT7gnzHJHwhDFvM%2BpxfQeBax05w9w73G8zojvUw%2BQUO3hFydhNvv3umKK3qTj9lBvxm3Y76vbjEwG8d%2FiUutjmoiQhviLOISiFYhReSqhg9pXi%2BiVh3pA%2B1Jj7WwAIBvwLGi9uBP9b8v6Ujm6xXoGRcNL%2BrTOveVsoaZ5FLJ4mK2JXt779Sd4oUREvJOj1M6QtmZEEQDgcvYIeA6i3Mo4d%2Fnrq4FYgMykXS9EmTWZl4L0H0pM%2BZvO2mp0Vo%2BusoEmYHx5bAf3oe0FXal0w6f0fBuygh4mmcwG%2FwvcH%2FUwTYaClhPGv%2BAKZPZ1eSlTE89ATb%2FBekdl%2BL%2FmFqFJ%2F74DuvJOzHj6LH3atLO8EGf%2FCpWBXXKzf%2FcU2zhzWOBgllAkuc62dJ5Al52C7QWmzVvz2PytfcwYGzM9LdjVntB2JAcIMehycGkAaR1JpbaWwhz2Hx3B4WHH7TlT2pG97anI6iyEKED2L6V97WhbeQGaxCoKOGQgq8eXY9c0g%2FOvgey1yTOX498PdpzjpZa6ARk7bd4B1kBjRXOp4w%2B%2FjczwY6mAG0zsHQwzJTnTBKcMu8zL4iax%2B%2FA8NgroIJxloqbd2zImVWchJk%2FOZK9x2kAl1YHW1rU8NUEwYMgdVVUhQxFF8k0wOup1c%2B1tqe2ap66LIX6PowYOJy%2BLbI8Vir1U7D89CrLSMgblSKyRmjtJUSVDDF1reWnkgKskFN4mypaBh4SYRmHeP867Bh5aN%2FbVaTACZ3YXxFFrn0DA%3D%3D&Expires=1777811987)

***

## 你看到的其實是兩個獨立組件

| 元素 | 類型 | 說明 |
|---|---|---|
| `U U C G T S` 那排 | **Segmented Control / Pill Tabs** | 互斥切換，active 有 filled circle |
| `· · ·` 那些點 | **Dot indicator / overflow indicator** | 代表還有更多 hidden items |
| `+` 按鈕 | **Add action** | 新增新 basket/tab |
| `v` chevron | **Dropdown / expand** | 展開更多選項 |

***

## 現成套件推薦

### Segmented Control / Pill Tabs
最輕量直接用 **Radix UI Tabs** 或 **Headless UI**，styled 成你要的 pill 樣式：

```bash
npm install @radix-ui/react-tabs
```

完全 headless，你控制 CSS，active state 用 `data-[state=active]` selector 控制 。 [konstaui](https://konstaui.com/react/segmented)

### Emoji / Icon Picker（如果你想讓 basket 的 icon 可換）
**emoji-mart** 是目前最主流、最成熟的選擇：

```bash
npm install emoji-mart @emoji-mart/react @emoji-mart/data
```

特點：支援 dark theme、可 custom emoji、bundle size 合理（~14kB gzip）。 [github](https://github.com/missive/emoji-mart)

如果只要純 web component 不綁 React framework：
```html
<script type="module" src="https://cdn.jsdelivr.net/npm/emoji-picker-element@^1/index.js"></script>
<emoji-picker></emoji-picker>
```
這是 `emoji-picker-element`，41kB minified，用 IndexedDB cache，非常輕量 。 [skypack](https://www.skypack.dev/view/emoji-picker-element)

***

## Claude Code 邏輯對不上的根本原因

你說「死客的邏輯很有問題」——這個組件的問題在於它的**狀態語意很模糊**：

- `U U C G T S` 到底是 symbol？是 basket shortname？還是 display type（Unit/Candle/Grid/Tick/Signal）？
- Active 狀態是 **globally selected basket** 還是 **currently focused item in the list**？
- `+` 新增的是 basket 還是 tab item？

**給 Claude Code 的 prompt 建議**——先寫清楚 data model，例如：

```ts
interface BasketTab {
  id: string
  label: string        // 顯示在 segmented control 的字母
  emoji: string        // 🏦 這種 basket icon
  isCustom: boolean
  tickers: string[]
}

// State
activeTabId: string    // 哪個 tab 是 selected
visibleTabs: BasketTab[] // 顯示在 pill row 的
hiddenTabs: BasketTab[]  // overflow 進 dropdown 的
```

把這段 type 定義直接貼進 prompt，Claude Code 就不會猜了，邏輯會乾淨很多。

--




## 以後應該考慮
- 應該以BDD為主 瀑布開發 (BDD-Waterfall Hybrid)



![alt text](image-2.png)

- title消失了
- 左右箭頭現在只是placeholder刪掉吧
- spolight的入口有好幾種
  - title
  - 雙擊pill
  - 點下拉箭頭
  - 快速鍵 沒有按任何情況或者focus rrg的時候按下空格鍵
- 右上角的custom 本質上跟'+'一樣 需要統一行為或刪除
- 現在的 pill basket licon太小了
| 情境               | 建議尺寸    | Touch Target               |
| ---------------- | ------- | -------------------------- |
| 工具列/Toolbar icon | 18–20px | 需加 padding 至 40-44px 可點擊區域 |
| 側邊欄 nav icon     | 20–24px | 44px touch target          |
| 行內文字 icon（與文字對齊） | 14–16px | —                          |
| 卡片/Modal 裝飾 icon | 24–32px | —                          |
| Empty state 大圖示  | 48–64px | —                          |

應該要大點 並且遵從ux體驗的邏輯 以及手機上一根手指不會誤觸的程度
