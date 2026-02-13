# 切吧水果遊戲改進計劃

## 當前狀態分析
- **遊戲類型**: Canvas-based 切水果遊戲
- **現有功能**: 技能選擇系統、連擊系統、炸彈系統、計時模式
- **程式碼規模**: 1524行 (單一HTML文件)

## PRD要求 vs 當前實現對照表

| PRD要求 | 當前狀態 | 狀態 |
|---------|----------|------|
| 漸進難度系統 | 部分實現 (水果頻率增加) | ⚠️ 需加強 |
| 三次機會系統 | 未實現 (負分立即結束) | ❌ 缺失 |
| 連擊系統 | 已實現 (最高10倍) | ✅ 完成 |
| 特殊道具系統 | 部分實現 (開始前技能選擇) | ⚠️ 需擴展 |
| 道具生成機制 | 未實現 | ❌ 缺失 |
| 成就系統 | 未實現 | ❌ 缺失 |
| 視覺回饋優化 | 部分實現 | ⚠️ 需加強 |

## 優先級1: 核心遊戲玩法改進

### 1. 三次機會系統 (生命值系統)
**問題**: 當前遊戲在分數變負時立即結束，不符合PRD要求。

**解決方案**:
```javascript
// 新增生命值變數
game.lives = 3;

// 修改遊戲結束條件
function loseLife() {
    game.lives--;
    updateLivesDisplay();
    
    if (game.lives <= 0) {
        endGame(false);
    } else {
        // 顯示生命值減少特效
        showLifeLostEffect();
    }
}

// 修改水果錯過和炸彈擊中的處理
// 原本: game.score -= 5; 改為: loseLife();
// 原本: game.score -= 1000; 改為: loseLife();
```

**UI改動**:
- 在遊戲界面添加生命值顯示 (心形圖標 ×3)
- 生命值減少時添加視覺特效
- 遊戲結束畫面顯示剩餘生命值

### 2. 特殊道具系統 (遊戲中動態生成)
**問題**: 當前只有開始前的技能選擇，缺少遊戲中的動態道具。

**解決方案**: 新增5種特殊道具水果

| 道具名稱 | 圖標 | 效果 | 持續時間 |
|----------|------|------|----------|
| 冰凍道具 | ❄️ | 凍結時間3秒 | 3秒 |
| 雙倍分數 | ✨ | 分數翻倍 | 30秒 |
| 時間延長 | ⏱️ | 增加15秒 | 立即生效 |
| 全屏爆炸 | 💥 | 清除所有水果 | 立即生效 |
| 護盾道具 | 🛡️ | 抵擋一次炸彈傷害 | 直到使用 |

**實現步驟**:
1. 擴展 `fruitTypes` 陣列，添加道具水果類型
2. 創建 `PowerUp` 類別，繼承自 `Fruit`
3. 添加道具生成邏輯 (低概率出現)
4. 實現道具效果觸發機制
5. 添加道具激活視覺特效

### 3. 漸進難度系統增強
**問題**: 當前只有水果生成頻率增加，缺少炸彈概率增加。

**解決方案**:
```javascript
// 計算動態炸彈生成概率
function getBombSpawnProbability() {
    const baseProbability = 0.01; // 1%基礎概率
    const timeFactor = (60 - game.timeLeft) / 60; // 時間因素 (0到1)
    const scoreFactor = Math.min(game.score / 1000, 1); // 分數因素
    
    return baseProbability + (timeFactor * 0.05) + (scoreFactor * 0.03);
}

// 修改炸彈生成邏輯
if (Math.random() < getBombSpawnProbability()) {
    game.bombs.push(new Bomb());
}
```

## 優先級2: 視覺與音效改進

### 1. 視覺回饋增強
- **連擊特效**: 連擊達到3x、5x、10x時顯示特殊動畫
- **切割特效**: 添加更豐富的果汁飛濺粒子效果
- **道具特效**: 每個道具有獨特的視覺效果
- **屏幕震動**: 切到炸彈或達成高連擊時輕微屏幕震動

### 2. 音效系統
- **背景音樂**: 輕鬆愉快的遊戲音樂
- **音效**:
  - 切割水果音效
  - 炸彈爆炸音效
  - 連擊增加音效
  - 道具獲取音效
  - 遊戲結束音效

**實現**:
```html
<audio id="sliceSound" src="sounds/slice.mp3" preload="auto"></audio>
<audio id="comboSound" src="sounds/combo.mp3" preload="auto"></audio>
```

## 優先級3: 成就與進度系統

### 1. 成就類別
- **連擊成就**: 達成5x、10x連擊
- **分數成就**: 達到500、1000、5000分
- **生存成就**: 無傷通關、時間內無錯過水果
- **收集成就**: 收集所有類型道具

### 2. 實現方案
```javascript
const achievements = {
    firstCombo5: { name: "連擊新手", desc: "達成5倍連擊", unlocked: false },
    score1000: { name: "千分大師", desc: "獲得1000分", unlocked: false },
    noBombHit: { name: "炸彈剋星", desc: "一局遊戲中未切到炸彈", unlocked: false }
};

// 使用localStorage保存成就狀態
function unlockAchievement(id) {
    achievements[id].unlocked = true;
    localStorage.setItem('fruitAchievements', JSON.stringify(achievements));
    showAchievementPopup(achievements[id]);
}
```

## 技術架構改進

### 1. 代碼重構計劃
**當前問題**: 所有代碼在單一HTML文件中，難以維護。

**重構結構**:
```
fruit-game/
├── index.html          # 主HTML文件
├── css/
│   ├── game.css       # 遊戲樣式
│   └── ui.css         # UI組件樣式
├── js/
│   ├── game.js        # 遊戲核心邏輯
│   ├── fruits.js      # 水果和道具類
│   ├── ui.js          # 用戶界面控制
│   ├── audio.js       # 音效管理
│   └── achievements.js # 成就系統
└── assets/
    ├── sounds/        # 音效文件
    └── images/        # 圖像資源
```

### 2. 性能優化
- **對象池**: 重用水果和特效對象，減少垃圾回收
- **離屏Canvas**: 預渲染靜態元素
- **請求動畫幀優化**: 非活動時降低更新頻率

## 實施路線圖

### 階段1: 核心玩法改進 (1-2天)
1. 實現三次機會系統
2. 添加基礎道具系統 (先實現2-3種道具)
3. 增強漸進難度系統

### 階段2: 視覺與音效 (1天)
1. 添加視覺特效
2. 實現音效系統
3. 優化UI反饋

### 階段3: 成就與進度 (1天)
1. 實現成就系統
2. 添加本地存儲
3. 創建統計頁面

### 階段4: 代碼重構 (1-2天)
1. 分離HTML/CSS/JavaScript
2. 模塊化重構
3. 性能優化

## 預期效果
- **遊戲深度增加**: 道具系統和成就系統提供更多重玩價值
- **玩家體驗改善**: 更好的視覺反饋和更公平的難度曲線
- **代碼可維護性**: 模塊化結構便於未來擴展
- **PRD合規**: 完全實現PRD中要求的所有功能

## 下一步行動
1. 確認優先級順序
2. 開始實現三次機會系統
3. 設計道具視覺效果
4. 收集/創建音效資源