# 坦克大战游戏规格说明书

## 项目概述

基于 Canvas 的经典坦克大战网页游戏，单 HTML 文件架构，支持多种游戏机制和自定义设置。

---

## 1. 地图系统

### 1.1 地图大小选项
| 选项 | 除数 | 内容比例 | 说明 |
|------|------|----------|------|
| 超小地图 | 6 | 最大 | 网格最大，内容最大 |
| 小地图 | 9 | 较大 | 网格较大，内容较大 |
| 中等 | 14 | 默认 | 标准大小 |
| 大地图 | 20 | 较小 | 网格较小，内容较小 |
| 超大地图 | 28 | 最小 | 网格最小，内容最小 |

### 1.2 内容缩放规则
- 坦克大小：`GRID_STEP × 0.86`
- 障碍物大小：`GRID_STEP × 0.86`
- 炮弹大小：`GRID_STEP × 0.095`（普通）、`GRID_STEP × 0.14`（Boss）
- 道具大小：`GRID_STEP × 0.57`
- 画布大小不变，仅内容缩放

### 1.3 地形效果系统
| 地形 | 格子颜色 | 效果 |
|------|----------|------|
| 沙地 | #c2a84d | 坦克移动速度降低50% |
| 森林 | #1a4a1a | 隐蔽效果：敌人追踪概率降为35%，射击概率降为25% |
| 水面 | #1a5aaa | 阻挡所有坦克和炮弹通过 |

#### 地形生成规则
- 根据地形类型配置生成概率：
  - 平原: 沙地10%, 森林8%, 水面4%
  - 沙漠: 沙地30%, 森林3%, 水面6%
  - 城市: 沙地8%, 森林4%, 水面3%
  - 森林: 沙地5%, 森林28%, 水面4%
- 地形格子不会生成在玩家出生点、Boss出生点和障碍物上

#### 实现机制
```javascript
const TILE = { EMPTY: 0, SAND: 1, FOREST: 2, WATER: 3 };
// 移动时检查当前格子地形
const currentTile = getTerrainAt(this.x + this.width/2, this.y + this.height/2);
if (currentTile === TILE.WATER) return; // 水面阻挡
if (currentTile === TILE.SAND) effectiveSpeed *= 0.5; // 沙地减速
// 森林隐蔽：敌人AI中检测玩家所在格子
const playerTile = getTerrainAt(player.x + player.width/2, player.y + player.height/2);
const playerHidden = playerTile === TILE.FOREST;
const shootMult = playerHidden ? 0.25 : 1;
const detectMult = playerHidden ? 0.35 : 1;
```

---

## 2. 敌人类型系统

### 2.1 敌人属性表
| 类型 | 颜色 | 速度倍率 | 血量倍率 | 体型倍率 | 射击倍率 | 子弹速度 | 伤害 | AI行为 |
|------|------|----------|----------|----------|----------|----------|------|--------|
| 普通坦克 | #00bbff 天蓝 | 1x | 1x | 1x | 1x | 1x | 1 | 随机+追踪 |
| 侦察车 | #ffdd00 金黄 | 2x | 0.5x | 0.7x | 0.8x | 1.2x | 1 | 积极进攻 |
| 重型坦克 | #dd2244 深红 | 0.5x | 3x | 1.3x | 0.6x | 0.8x | 5 | 稳定追踪 |
| 狙击手 | #44ff44 亮绿 | 0.8x | 0.8x | 1x | 1x | 3x | 6 | 保持距离 |

### 2.2 AI行为逻辑
```
普通坦克: 随机转向(1.5%) + 追踪玩家(0.8%) + 射击(2%)
侦察车:   追踪玩家(5%) + 射击(4%)
重型坦克: 追踪玩家(2%) + 射击(1.5%)
狙击手:   距离<150时后退，否则前进 + 射击(2.5%)

森林隐蔽效果：
  玩家在森林格子上时，敌人追踪概率降为35%，射击概率降为25%
```

### 2.3 生成规则
- 敌人池：`['normal', 'normal', 'scout', 'heavy', 'sniper']`
- 随机分配类型
- 避开障碍物和已生成敌人

---

## 3. 主动技能系统

### 3.1 技能定义
| 技能 | 按键 | 基础冷却 | 基础效果 | 强化效果 |
|------|------|----------|----------|----------|
| 护盾 | Q | 15秒 | 无敌3秒 | 每级+1秒 |
| 电磁脉冲 | E | 20秒 | 减速50%持续3秒 | 每级+10%减速 |
| 空袭 | R | 30秒 | 范围伤害(3+关卡×0.3) | 每级+范围+伤害 |

### 3.2 技能实现细节

#### 护盾
```javascript
player.invincible = true;
player.invincibleTimer = now + 3000 + shieldUpgrade * 1000;
```

#### 电磁脉冲
```javascript
const slowPercent = 0.5 + empUpgrade * 0.1; // 最高90%
e.speed = e.speed * (1 - slowPercent);
e.slowed = true;
e.slowEnd = empEndTime;
```
- 视觉效果：紫色边框脉冲 + 敌人变色 + 血条紫色标记 + 屏幕紫色滤镜

#### 空袭
```javascript
const radius = GRID_STEP * 3 + airstrikeUpgrade * GRID_STEP * 0.8;
const dmg = 3 + Math.floor(level * 0.3) + airstrikeUpgrade;
// 范围判定：距离² < 半径²
```
- 视觉效果：虚线圆圈扩大 → 爆炸闪烁 → 淡出

---

## 4. 技能强化道具

### 4.1 道具定义
| 道具ID | 名称 | 颜色 | 符号 | 效果 | 上限 |
|--------|------|------|------|------|------|
| shieldUp | 护盾强化 | #00ffaa | 🛡️ | 护盾持续时间+1秒 | 5级 |
| airstrikeUp | 空袭强化 | #ff4444 | 💚 | 空袭范围和伤害增加 | 5级 |
| empUp | 脉冲强化 | #aa00ff | 🌀 | 减速效果+10% | 5级(最高90%) |

---

## 5. Boss系统

### 5.1 Boss属性
- 体型：与玩家坦克一致（`TANK_SIZE`）
- 外观：紫色脉冲光圈标识（半径 `width × 0.75`），光圈接触造成伤害
- 速度：`enemySpeed × 2`（与侦察车一致），随关卡增加
- 血量：`enemyHP × 6 × 1.6^(关卡-1)`（重型坦克血量的2倍，随关卡指数增长）
- 伤害：`6 × 1.3^(关卡-1)`（与狙击手一致，随关卡增长）
- 射击冷却：`enemyCooldown × 0.4`
- 子弹速度：`enemyBulletSpeed × 3`（与狙击手一致）

### 5.2 Boss生成
- 位置：随机生成（避开障碍物、水面和玩家，最小距离4格）
- 清除生成点周围半径 `bs + GRID_STEP` 内的障碍物
- 智能寻路：卡住时尝试其他方向
- 通过 `bossSpawning` 标志防止多次重复生成
- **Boss关卡同时刷新与当前关卡相同数量和类型的敌人坦克**

---

## 6. 玩家系统

### 6.1 玩家属性
- 初始HP：3（可自定义1-10）
- 被敌人炮弹或Boss光圈击中时扣除伤害值对应HP，HP归零时游戏结束
- 被击中后**不会**传送回出生点，也**不会**获得无敌时间
- 被击中后清空所有增益效果（移速/射速/伤害/散弹/护盾强化/空袭强化/脉冲强化）
- 仅游戏开始时获得2秒无敌时间

### 6.2 安全区规则
```javascript
// 生成障碍物时跳过安全区
const safeRadius = GRID_STEP * 3;
if (wallCenterY < topSafeY && Math.abs(wallCenterX - centerX) < safeRadius) continue;
if (wallCenterY > bottomSafeY && Math.abs(wallCenterX - centerX) < safeRadius) continue;

// 生成后清除玩家周围障碍物
const clearRadius = TANK_SIZE + GRID_STEP * 2;
// 删除与clearRadius范围重叠的障碍物
```

---

## 7. 音效系统

### 7.1 音效类型
| 类型 | 波形 | 频率变化 | 时长 |
|------|------|----------|------|
| shoot | 默认 | 800→200 | 0.1s |
| enemyDead | 默认 | 300→100 | 0.3s |
| bossDead | sine | 500→120 | 0.8s |
| powerup | sine | 600→900→1200 | 0.3s |
| bossWarning | triangle | 80→120→80 | 1.0s |
| victory | square | 523→659→784→1047 | 0.6s |
| defeat | sawtooth | 400→80 | 0.5s |
| hit | triangle | 600→100 | 0.08s |
| shield | sine | 400→800 | 0.3s |
| emp | sawtooth | 100→50 | 0.5s |
| airstrike | square | 200→50 | 0.8s |

---

## 8. 游戏流程

```
开始游戏 → 选择设置 → 生成玩家/敌人/障碍物 → 游戏循环
    ↓
消灭所有敌人 → Boss警告(1.5秒) → Boss生成 + 刷新相同数量敌人 → 消灭Boss和敌人 → 过关
    ↓
下一关 → 技能冷却刷新 → 敌人+2 → 继续游戏
    ↓
玩家死亡 → 显示得分 → 重新开始
```

---

## 9. 代码架构

### 9.1 核心常量
```javascript
TANK_SIZE, WALL_SIZE // 基础尺寸（动态计算）
GRID_STEP // 网格步长（动态计算）
CANVAS_W, CANVAS_H // 画布尺寸（动态计算）
BOSS_BASE_HP = 15
BOSS_SCALE_PER_LEVEL = 1.6
BOSS_DMG_SCALE = 1.3
MAP_SIZE_DIVISORS = { tiny: 6, small: 9, medium: 14, large: 20, huge: 28 }
TILE = { EMPTY: 0, SAND: 1, FOREST: 2, WATER: 3 }
TILE_SPEED_MULT = { 1: 0.5, 2: 1.0, 3: 0 } // 沙地减速50%，水面不可通行
```

### 9.2 状态变量
```javascript
gameRunning, gamePaused, score, level, enemyCount
player, enemies[], walls[], bullets[], powerups[]
terrainGrid[][] // 地形格子二维数组
boss, bossActive, bossSpawning // bossSpawning防止多次生成
shieldUpgrade, airstrikeUpgrade, empUpgrade
skillCooldowns[], empActive, empEndTime, airstrikeEffect
LEADERBOARD_KEY = 'tankBattleLeaderboard'
MAX_LEADERBOARD_ENTRIES = 10
```

### 9.3 主要函数
- `initGame()` - 初始化游戏
- `gameLoop()` - 主循环(60fps)
- `createWalls()` - 生成障碍物
- `createTerrain()` - 生成地形格子
- `drawTerrain()` - 渲染地形效果
- `getTerrainAt(x, y)` - 获取指定坐标的地形类型
- `spawnEnemies()` - 生成敌人
- `enemyAI()` - 敌人AI（含森林隐蔽检测）
- `bossAI()` - Boss AI
- `activateSkill()` - 技能激活
- `updateSkills()` - 技能状态更新
- `drawSkillEffects()` - 技能视觉效果

---

## 10. 排行榜系统

### 10.1 数据结构
```javascript
const LEADERBOARD_KEY = 'tankBattleLeaderboard';
const MAX_LEADERBOARD_ENTRIES = 10;

// 排行榜条目
{
    score: number,      // 游戏得分
    level: number,      // 挑战关卡
    date: string,       // 日期（中文格式）
    timestamp: number   // 时间戳
}
```

### 10.2 排序规则
- 主要排序：按关卡数从高到低 (`b.level - a.level`)
- 次要排序：相同关卡按分数从高到低 (`b.score - a.score`)

### 10.3 存储机制
- 使用 `localStorage` 持久化存储
- 存储键：`tankBattleLeaderboard`
- 最大记录数：10条
- 游戏结束时自动保存

### 10.4 显示位置
- 右侧信息栏最上方
- 显示最高关卡数（绿色高亮）
- 表格显示前10名记录（排名、关卡、分数、日期）
