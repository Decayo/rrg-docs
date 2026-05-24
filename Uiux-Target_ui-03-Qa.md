 我找到的疏漏

  1. Orbit Bar 細節未定
  - 每個 basket 的 icon 是什麼？文字首字母？顏色圓圈？自訂 icon？
    - 預設首字母 可以有emoji選擇 （這個...可能要pakage？ 找找npm ）
  - 超過 N 個 basket 時怎麼辦？橫向滾動？收合到 "..." 按鈕？
    - 可以滾動 會有 ... 同時選擇的會有小箭頭指向 以及如果不在視窗 會指向...
    - 簡單說  O O O O O …
    -            *
    - 如果超出 ... O O O O O ...
    -          *
    - 類似這樣？
  - 點擊 vs 長按？有沒有右鍵選單（編輯/刪除）？
    - 有 點擊長暗進入編輯介面 可以編輯標的和重命名等等 以及ai
    - singlebasketviewpanel component還需要討論和設計




  1. Basket 編輯/刪除的 UX
  - 在哪裡改名、刪除、編輯 symbols？Basket Panel 裡？長按 Orbit Bar icon？
  - 先在上面setting旁邊有個籃子的icon 是可管理介面
    - 應該有幾層？
    - 總覽 然後在點擊對應的 可以到單一標的的editpanel
    - so... groupbasketviewpanel
    - singlebasketviewpanel?

  1. Drill-down 可用性提示
  - 用戶怎麼知道哪些 symbol 可以 drill-down？DetailPanel 上的 ⤢ 只對大 ETF 顯示，其他
  symbol 看不到這個按鈕？
  - 需要一個清單定義哪些 ETF 支持 drill-down
  - 這個嗎 其實理論上來說 先xl系列 smh/soxx系列 等比較火的先有就行

  1. Back Nav 邊界
  - Layer 0（最頂層）時 ← 按鈕隱藏還是 disabled？
  - Back 是只回一層，還是有「回到首頁」？
  - 忘了 還要有Next Nav 就效仿 瀏覽器就行
  - 回到首頁這東西就是點左上角的我們的orbit icon 這沒啥 挺直覺得？ 

  1. Basket Panel 觸發方式（未定）
  - dropdown 旁按鈕？右鍵 RRG？Orbit Bar 末尾的 "+" 按鈕？
  - Mobile 上右鍵不存在，需要替代方案
  - hmmm... 長按？ mobile上應該右鍵都是對應長按？

  1. 空狀態
  - 新用戶第一次進來，沒有自訂 basket → 自定義欄顯示什麼？引導？
  - 自定義欄如果是免費 會有個lock icon+模糊 hover會有說 pro解鎖
    - 點了會有helppanel?
    - 但這個之後在做
  - 如果有訂閱 會有一個 預設的和的和 O +, O是 smh和tsm那些成份股x5 還有一些台灣(聯發科)/韓國（海力士）/日本（半導體相關）/BTC 標的
    - 目的是表達我們可以做到這種程度
  1. URL state
  - 當前 basket 要反映在 URL 嗎？（方便分享、瀏覽器 back）
    - 這個嘛... 也許？ 如果這樣方便的畫
    - 但這樣就要影響是不是付費用戶 有些url只有付費用戶能看

  1. AI 建 basket 是否 MVP1？
  - 你說 "mvp1就是截圖然後他就會創立filter 就夠了" 但這需要 vision API +
  後端解析，確認要做？
    - hmmm 這個有幾個階段
    - 首先是有api能打通聊天 （你可以先看一下pal mcp頂著用 因為現在我們就是本地端架設 給群友測試也是 然後可能要有個ratelimit因為可能每個請求要格20秒? 因為我會給五個人左右測試 之後再說）
    - 之後再嘗試function call 看看mcp pplx能不能理解和呼叫


