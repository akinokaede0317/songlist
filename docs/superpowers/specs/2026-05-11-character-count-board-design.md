# 字數分類看板模式

## 概覽

在現有歌單 app 新增「字數分類」tab，點擊後切換為水平捲動看板（kanban）版型，將所有歌曲依歌名字數分欄顯示。設計參考：hamigar.notion.site 的字數分類看板頁面。

## 架構

### Tab 設定

在 `CONFIG.CATEGORY_TABS` 陣列加入 `"字數分類"`。點擊該 tab 時，隱藏現有的搜尋篩選工具列（`.tools`）、grid 版型、雙窗格版型，改顯示看板版型。

### Alpine 新增 computed / 狀態

```js
// 判斷是否為看板 tab
get isBoardTab() {
  return this.activeTab === '字數分類';
}

// 將 this.rows 按歌名字數分到 7 個 bucket
get boardColumns() {
  const buckets = [
    { label: '1字', songs: [] },
    { label: '2字', songs: [] },
    { label: '3字', songs: [] },
    { label: '4字', songs: [] },
    { label: '5字', songs: [] },
    { label: '6字', songs: [] },
    { label: '7字以上', songs: [] },
  ];
  for (const song of this.rows) {
    const n = [...(song.title || '')].length;
    const idx = Math.min(n - 1, 6); // 7字以上都到 idx 6
    if (idx >= 0) buckets[idx].songs.push(song);
  }
  return buckets;
}

// 被點擊的歌曲（用於 Modal）
selectedSong: null,
```

`isGridTab` 的定義同時排除 `'字數分類'`，確保舊版型不會誤顯示。

## 版型：水平捲動看板

### 容器

```css
.board {
  display: flex;
  gap: 16px;
  overflow-x: auto;
  padding-bottom: 16px;
  align-items: flex-start;
}
```

### 欄位

- 固定寬度：`200px`（`min-width: 200px; width: 200px`）
- 最大高度：`75vh`，超出後欄內垂直捲動（`overflow-y: auto`）
- 背景：`--card`（深暗紅），邊框：`#3d1515`，圓角 `14px`

### 欄頭

顯示字數文字（如 `2字`）＋歌曲數量 badge。數量 badge 沿用現有 `.badge` 樣式，顏色使用 `--acc`（紅色）。

### 看板歌曲卡

比現有卡片更簡單：
- 只顯示歌名
- 背景略深於欄位底色，圓角 `10px`，padding `10px 12px`
- hover 時底部出現「複製歌名」橫條（與現有 `.copy-bar` 行為一致）
- 點擊觸發 Modal（設定 `selectedSong`）

## Modal：歌曲詳細資料

### 觸發與關閉

- 點擊看板歌曲卡 → `selectedSong = song`
- 關閉方式：點擊背景遮罩 / 點擊右上角 ✕ 按鈕 / 按 ESC 鍵
- 關閉時 `selectedSong = null`

### Modal 內容

| 欄位 | 說明 |
|------|------|
| 歌名 | 大標，`font-size: 24px` |
| 歌手 | 次行，muted 色 |
| 分類 | `.tag` pill |
| 標籤 | `.badge`（推薦 ⭐、倍數、自訂標籤） |
| 播放連結 | 若 `song.url` 存在則顯示按鈕，開新分頁 |
| 複製歌名 | 按鈕，複製後顯示 ✓ 確認 |
| 更新時間 | muted 色小字 |

### Modal 樣式

- 遮罩：`position:fixed; inset:0; background:rgba(0,0,0,.6); z-index:100`
- 視窗：`position:fixed; top:50%; left:50%; transform:translate(-50%,-50%)`，`background:--card`，圓角 `16px`，`max-width:400px; width:90%`
- 使用現有暗紅色系，不引入外部元件庫

## 不在範圍內

- 看板模式內的搜尋／篩選（純瀏覽用途）
- 語言篩選（顯示全部歌曲）
- 手機響應式優化（水平捲動即可）
