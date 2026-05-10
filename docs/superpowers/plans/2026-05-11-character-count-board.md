# 字數分類看板模式 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在歌單 app 新增「字數分類」tab，點擊後切換為水平捲動看板，依歌名字數分成 7 欄，點擊歌曲卡片彈出 Modal 顯示詳細資料。

**Architecture:** 所有變更集中在 `index.html` 一個檔案。Alpine.js data 新增 `selectedSong`、`isBoardTab`、`boardColumns` 三個 computed/state；HTML 新增看板區塊和 Modal 區塊；CSS 新增對應樣式。

**Tech Stack:** Alpine.js 3.x（已引入），純 HTML/CSS，無額外依賴。

---

### Task 1: 更新 Alpine 資料模型

**Files:**
- Modify: `index.html:270` (CONFIG.CATEGORY_TABS)
- Modify: `index.html:349` (Alpine data 初始狀態)
- Modify: `index.html:352-354` (isGridTab getter)
- Modify: `index.html:372-381` (init 方法)
- Modify: `index.html:580-597` (新增 closeModal 方法)

- [ ] **Step 1: 在 CONFIG.CATEGORY_TABS 加入「字數分類」**

  找到 `index.html` 第 270 行：
  ```js
  CATEGORY_TABS: ["所有歌曲","男歌手","女歌手","合唱","其他...","加倍歌單"]
  ```
  改為：
  ```js
  CATEGORY_TABS: ["所有歌曲","男歌手","女歌手","合唱","其他...","加倍歌單","字數分類"]
  ```

- [ ] **Step 2: 在 Alpine data 加入 selectedSong**

  找到 `index.html` 第 349 行：
  ```js
  tagMap: {},
  ```
  改為：
  ```js
  tagMap: {},
  selectedSong: null,
  ```

- [ ] **Step 3: 更新 isGridTab，排除字數分類**

  找到 `index.html` 第 352-354 行：
  ```js
  get isGridTab() {
    return ['所有歌曲', '加倍歌單', '其他...'].includes(this.activeTab);
  },
  ```
  改為：
  ```js
  get isGridTab() {
    return ['所有歌曲', '加倍歌單', '其他...'].includes(this.activeTab);
  },

  get isBoardTab() {
    return this.activeTab === '字數分類';
  },

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
      const idx = Math.min(n - 1, 6);
      if (idx >= 0) buckets[idx].songs.push(song);
    }
    return buckets;
  },
  ```

- [ ] **Step 4: 在 init() 加入 ESC 鍵監聽**

  找到 `init()` 方法內，`this.fetchTags();` 那行之前插入：
  ```js
  document.addEventListener('keydown', e => {
    if (e.key === 'Escape') this.closeModal();
  });
  ```

- [ ] **Step 5: 加入 closeModal 方法**

  找到 `luckyPick()` 方法結尾的 `},`，在它後面加入：
  ```js
  closeModal() {
    this.selectedSong = null;
  },
  ```

- [ ] **Step 6: 驗證 JS 語法正確**

  在瀏覽器開啟 `index.html`，開 DevTools Console，確認沒有紅色 JS 錯誤。
  在 nav 看到「字數分類」tab 出現即為成功。

- [ ] **Step 7: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add 字數分類 tab and Alpine data model"
  ```

---

### Task 2: 加入 CSS 樣式

**Files:**
- Modify: `index.html:7-126` (`<style>` 區塊，在最後一條規則後插入)

- [ ] **Step 1: 在 `</style>` 標籤之前插入以下 CSS**

  找到 `index.html` 的 `</style>` 行（目前在第 126 行），在它之前插入：

  ```css
  /* [x-cloak] 防止 Alpine 初始化前閃爍 */
  [x-cloak]{display:none!important}

  /* ===== 字數看板 ===== */
  .board{display:flex;gap:16px;overflow-x:auto;padding-bottom:16px;align-items:flex-start}
  .board-col{min-width:200px;width:200px;background:var(--card);border:1px solid #3d1515;border-radius:14px;overflow:hidden;display:flex;flex-direction:column;max-height:75vh;flex-shrink:0}
  .board-col-head{padding:10px 14px;display:flex;align-items:center;gap:8px;border-bottom:1px solid #3d1515;flex-shrink:0}
  .board-col-label{font-size:15px;font-weight:bold;color:var(--ink)}
  .board-col-count{font-size:11px;padding:2px 8px;border-radius:999px;background:#331014;border:1px solid var(--acc);color:var(--acc)}
  .board-col-body{overflow-y:auto;padding:8px;display:flex;flex-direction:column;gap:6px}
  .board-card{background:#0e0606;border:1px solid #3d1515;border-radius:10px;padding:10px 12px;font-size:15px;cursor:pointer;position:relative;overflow:hidden;transition:transform .15s ease,box-shadow .15s ease}
  .board-card:hover{transform:translateY(-2px);box-shadow:0 6px 18px rgba(0,0,0,.4)}
  .board-card .copy-bar{border-radius:0 0 10px 10px}

  /* ===== Modal ===== */
  .modal-overlay{position:fixed;inset:0;background:rgba(0,0,0,.65);z-index:100;display:flex;align-items:center;justify-content:center}
  .modal-box{background:var(--card);border:1px solid #3d1515;border-radius:16px;padding:24px;max-width:400px;width:90%;position:relative;max-height:90vh;overflow-y:auto}
  .modal-close{position:absolute;top:12px;right:14px;background:none;border:none;color:var(--muted);font-size:20px;cursor:pointer;line-height:1;padding:0}
  .modal-title{font-size:24px;font-weight:bold;margin:0 0 6px;color:var(--ink)}
  .modal-artist{color:var(--muted);font-size:15px;margin-bottom:14px}
  .modal-badges{display:flex;flex-wrap:wrap;gap:6px;margin-bottom:14px}
  .modal-actions{display:flex;gap:10px;flex-wrap:wrap;margin-top:16px}
  .modal-btn{background:var(--acc);color:#fff;border:none;border-radius:10px;padding:8px 16px;font-size:14px;cursor:pointer;font-weight:bold;text-decoration:none;display:inline-flex;align-items:center}
  .modal-btn-copy{background:#2a0808;border:1px solid #600000;color:var(--ink)}
  .modal-btn-copy.copied{background:#14502d;border-color:#4a9060;color:#8fc96e}
  .modal-footer{color:var(--muted);font-size:12px;margin-top:12px}
  ```

- [ ] **Step 2: 驗證 CSS 載入無誤**

  開啟 `index.html`，DevTools Console 無錯誤，頁面外觀與原本相同（新 CSS 尚未生效，因為 HTML 還沒加上去）。

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add board and modal CSS styles"
  ```

---

### Task 3: 更新 HTML 工具列與雙窗格顯示條件

**Files:**
- Modify: `index.html:147` (`.tools` div 的 x-show)
- Modify: `index.html:169` (`.subtools` div — 確認不受影響)
- Modify: `index.html:217` (`.twoPane` div 的 x-show)

- [ ] **Step 1: 工具列加上隱藏條件**

  找到 `index.html` 第 147 行：
  ```html
  <div class="tools">
  ```
  改為：
  ```html
  <div class="tools" x-show="!isBoardTab">
  ```

- [ ] **Step 2: 雙窗格加上排除字數分類的條件**

  找到 `index.html` 第 217 行：
  ```html
  <div x-show="!loading && !error && !isGridTab" class="twoPane">
  ```
  改為：
  ```html
  <div x-show="!loading && !error && !isGridTab && !isBoardTab" class="twoPane">
  ```

- [ ] **Step 3: 驗證**

  開啟 `index.html`，點擊「字數分類」tab：
  - 搜尋工具列消失 ✓
  - 雙窗格（歌手列表）消失 ✓
  - 頁面空白（看板 HTML 尚未加入）✓

  切換回「所有歌曲」tab，工具列和卡片恢復正常 ✓

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "feat: hide toolbar and twoPane on board tab"
  ```

---

### Task 4: 加入看板 HTML 區塊

**Files:**
- Modify: `index.html:259` (在 `.twoPane` 結束的 `</div>` 之後插入)

- [ ] **Step 1: 在 twoPane 結束標籤後插入看板區塊**

  找到 `index.html` 第 259 行（`</div>` 結束 twoPane），在它後面插入：

  ```html
  <!-- 字數分類看板 -->
  <div x-show="!loading && !error && isBoardTab" class="board">
    <template x-for="col in boardColumns" :key="col.label">
      <div class="board-col">
        <div class="board-col-head">
          <span class="board-col-label" x-text="col.label"></span>
          <span class="board-col-count" x-text="col.songs.length"></span>
        </div>
        <div class="board-col-body">
          <template x-for="song in col.songs" :key="song.title + song.artist">
            <div class="board-card" @click="selectedSong = song">
              <span x-text="song.title"></span>
              <button
                class="copy-bar"
                :class="copiedSong === song ? 'copied' : ''"
                @click.stop="copyTitle(song)"
                x-text="copiedSong === song ? '✓ 已複製！' : '複製歌名'">
              </button>
            </div>
          </template>
        </div>
      </div>
    </template>
  </div>
  ```

- [ ] **Step 2: 驗證看板顯示**

  開啟 `index.html`，點擊「字數分類」tab：
  - 出現水平排列的欄位（1字、2字、3字...7字以上）✓
  - 每欄頭部顯示字數 + 歌曲數量 badge ✓
  - 欄內列出歌名 ✓
  - hover 卡片時底部出現「複製歌名」橫條 ✓
  - 點擊「複製歌名」可複製，複製後顯示「✓ 已複製！」✓
  - 點擊卡片本體（非複製橫條）目前尚無反應（Modal 尚未加入）✓

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add character-count board HTML"
  ```

---

### Task 5: 加入 Modal HTML 區塊

**Files:**
- Modify: `index.html:263` (在 `</main>` 之前插入 Modal)

- [ ] **Step 1: 在 `</main>` 之前插入 Modal**

  找到 `index.html` 第 263 行（`</main>`），在它之前插入：

  ```html
  <!-- 歌曲詳細資料 Modal -->
  <div class="modal-overlay" x-show="selectedSong" x-cloak @click.self="closeModal()">
    <div class="modal-box">
      <button class="modal-close" @click="closeModal()">✕</button>
      <template x-if="selectedSong">
        <div>
          <h2 class="modal-title" x-text="selectedSong.title"></h2>
          <div class="modal-artist" x-text="selectedSong.artist || ''"></div>
          <div class="modal-badges">
            <span class="tag" x-text="selectedSong.category || '未分類'"></span>
            <span class="badge badge-gold" x-show="selectedSong.isTop === 'TRUE'">⭐ 推薦</span>
            <template x-for="tok in (selectedSong.tags || [])" :key="tok">
              <span class="badge" :style="tagStyle(tok)" x-text="tok"></span>
            </template>
            <span class="badge"
              x-show="multBadgeText(selectedSong.multiplier)"
              :class="multBadgeClass(selectedSong.multiplier)"
              x-text="multBadgeText(selectedSong.multiplier)"></span>
          </div>
          <div class="modal-actions">
            <a x-show="selectedSong.url"
               class="modal-btn"
               :href="selectedSong.url"
               target="_blank"
               rel="noopener">▶ 播放／連結</a>
            <button
              class="modal-btn modal-btn-copy"
              :class="copiedSong === selectedSong ? 'copied' : ''"
              @click="copyTitle(selectedSong)"
              x-text="copiedSong === selectedSong ? '✓ 已複製！' : '複製歌名'">
            </button>
          </div>
          <div class="modal-footer" x-text="formatDate(selectedSong.updatedAt)"></div>
        </div>
      </template>
    </div>
  </div>
  ```

- [ ] **Step 2: 驗證 Modal 所有互動**

  開啟 `index.html`，切換到「字數分類」tab：

  **開啟：**
  - 點擊任一歌曲卡片 → Modal 在螢幕中央出現 ✓
  - Modal 顯示歌名（大字）、歌手、分類 tag、各標籤 badge ✓
  - 有 URL 的歌曲顯示「▶ 播放／連結」按鈕 ✓
  - 「複製歌名」按鈕可複製，複製後變綠色「✓ 已複製！」✓
  - 底部顯示更新時間 ✓

  **關閉：**
  - 點擊右上角 ✕ → Modal 關閉 ✓
  - 點擊 Modal 外部暗色遮罩 → Modal 關閉 ✓
  - 按 ESC 鍵 → Modal 關閉 ✓

  **其他 tab 不受影響：**
  - 切換到「所有歌曲」，卡片、搜尋正常 ✓
  - 切換到「男歌手」，雙窗格正常 ✓

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add song detail modal for board mode"
  ```
