---
name: fifa2026-daily-update
description: 每日更新FIFA 2026世界杯观赛指南数据，包括比赛比分、射手榜、助攻榜、红黑榜、夺冠赔率等。当用户要求更新世界杯数据、更新红黑榜、更新比分、更新射手榜、更新赔率时使用此skill。
---

# FIFA 2026 每日数据更新指南

本skill用于更新 `index.html` 中的FIFA 2026世界杯观赛指南数据。

## 数据源

| 数据类型 | 数据源 | 说明 |
|---------|--------|------|
| 比分/射手榜/助攻榜 | [ESPN FIFA World Cup Stats](https://www.espn.com/soccer/stats/_/league/fifa.world/season/2026/view/scoring) | 官方统计数据，每日更新 |
| 红黑榜 | 懂球帝文章 | 用户提供的文章链接 |
| 夺冠赔率 | DraftKings Sportsbook | 需手动更新 |
| 比赛结果详情 | 懂球帝比赛报告 | 用于获取进球球员和助攻球员 |

## 更新内容清单

### 1. 比分更新 (groupSchedule)

**位置**: `index.html` 约第3708-3801行

**操作**: 为已完成的比赛添加 `score1` 和 `score2` 字段，同时**删除 `odds` 字段**。

```javascript
// 更新前（有赔率，无比分）：
{ date: '6月24日', time: '01:00', group: 'K', home: '葡萄牙', homeFlag: '🇵🇹', away: '乌兹别克斯坦', awayFlag: '🇺🇿', venue: '休斯顿', odds: '葡萄牙胜@1.20 | ...' }

// 更新后（有比分，无赔率）：
{ date: '6月24日', time: '01:00', group: 'K', home: '葡萄牙', homeFlag: '🇵🇹', away: '乌兹别克斯坦', awayFlag: '🇺🇿', venue: '休斯顿', score1: 5, score2: 0 }
```

**规则**: 有比赛结果的比赛**不显示赔率**，只显示比分。

### 2. 射手榜更新 (getGoalsLeaderboard)

**位置**: `index.html` 约第4991-5070行

**数据源**: ESPN Scoring Stats 页面的 "Top Scorers" 表格

**操作步骤**:
1. 访问 `https://www.espn.com/soccer/stats/_/league/fifa.world/season/2026/view/scoring`
2. 提取所有射手数据（Name, Team, Goals）
3. 将英文球员名和球队名翻译成中文（参考现有数据）
4. 替换 `getGoalsLeaderboard()` 函数中的 `scorers` 数组
5. 添加当日比赛的进球球员（ESPN可能未及时更新）

**注意**: 
- ESPN数据"updated nightly"，当日比赛数据可能需要手动添加
- 使用 `grep -n "name: '球员名'" index.html` 检查重复条目
- 去重后再添加新球员
- 球员名和球队名需翻译成中文，保持与现有数据一致

### 3. 助攻榜更新 (getAssistsLeaderboard)

**位置**: `index.html` 约第5076-5095行

**数据源**: ESPN Stats 页面的 "Top Assists" 表格

**操作步骤**:
1. 访问 `https://www.espn.com/soccer/stats/_/league/fifa.world/season/2026/view/scoring` 页面的 "Top Assists" 表格
2. 提取所有助攻数据（Name, Team, Assists）
3. 将英文球员名和球队名翻译成中文（参考现有数据）
4. 替换 `getAssistsLeaderboard()` 函数中的 `assists` 数组
5. 添加当日比赛的助攻球员（ESPN可能未及时更新）

**注意**: 
- 同射手榜，需翻译球员名和球队名
- 检查重复条目

### 4. 红黑榜更新 (REDBLACK_DATA)

**位置**: `index.html` 约第5773-5840行

**数据源**: 用户提供的懂球帝文章链接

**操作步骤**:
1. 访问用户提供的懂球帝文章URL
2. 从页面标题提取日期和标题
3. 从 `<meta name="description">` 或 `<img>` 标签提取图片URL
4. 在 `REDBLACK_DATA` 数组**开头**添加新条目

**数据结构**:
```javascript
{
  date: '6月24日',
  title: '懂球帝美加墨世界杯6月24日红黑榜',
  articleUrl: 'https://m.dongqiudi.com/article/5962471.html?frm=copy',
  imageUrl: 'https://bdimg7.qunliao.info/fastdfs8/M00/4C/05/xxx.jpg'
}
```

**图片URL提取技巧**:
- 优先使用 `orig-src` 属性（无水印版本）
- 如果没有，使用 `data-src` 属性
- 图片URL格式: `https://bdimg7.qunliao.info/fastdfs8/...`

### 5. 夺冠赔率更新 (renderOdds)

**位置**: `index.html` 约第5140-5158行

**数据源**: DraftKings Sportsbook 或其他博彩网站

**操作**:
1. 访问 DraftKings Sportsbook 或其他博彩网站获取最新赔率
2. 更新各队的赔率和战绩描述
3. 更新日期 `更新于YYYY年M月D日`
4. 根据最新比赛结果调整排名

**赔率格式**: `+400` 表示4赔1，`+800` 表示8赔1

**注意**: 
- 如果无法访问博彩网站，可根据最新比赛结果调整描述
- 更新日期为当前日期

### 6. 小组排名更新

**位置**: `index.html` 动态计算，无需手动更新

**说明**: 小组排名根据已结束的比赛自动计算，无需手动更新。确保比赛结果（比分）已更新即可。

**排名规则**:
1. 积分（胜3分，平1分，负0分）
2. 相互比赛积分（积分相同的球队之间）
3. 净胜球
4. 进球数

**验证**: 更新比分后，检查小组排名是否正确显示。可对照ESPN排名验证。

## 更新流程

### 每日更新步骤

1. **获取当日比赛结果**
   - 访问 ESPN FIFA World Cup Scoreboard
   - 或访问懂球帝比赛页面 `https://m.dongqiudi.com/match/125`

2. **更新比分**
   - 在 `groupSchedule` 中找到对应比赛
   - 添加 `score1`, `score2`
   - 删除 `odds` 字段

3. **验证小组排名**
   - 更新比分后，小组排名会自动计算
   - 检查小组排名是否正确显示

4. **更新射手榜/助攻榜**
   - 访问 ESPN Stats 页面
   - 提取最新数据
   - 翻译球员名和球队名
   - 添加当日进球/助攻球员（从比赛报告获取）

5. **更新红黑榜**（如有新文章）
   - 从用户获取懂球帝文章链接
   - 提取标题、图片URL
   - 添加到 `REDBLACK_DATA` 数组开头

6. **更新夺冠赔率**
   - 访问 DraftKings Sportsbook 或其他博彩网站获取最新赔率
   - 更新各队的赔率和战绩描述
   - 更新日期为当前日期

## 注意事项

- **去重**: 更新射手榜/助攻榜前，用 `grep` 检查是否有重复条目
- **赔率规则**: 有比分的比赛不显示赔率
- **数据来源**: ESPN数据是权威来源，懂球帝用于补充当日未更新的数据
- **红黑榜顺序**: 新条目添加到数组**开头**（最新在前）
- **时间转换**: ESPN显示美东时间(ET)，需+12小时转北京时间

## 常用命令

```bash
# 检查重复条目
grep -n "name: '球员名'" index.html

# 查看射手榜位置
grep -n "getGoalsLeaderboard" index.html

# 查看红黑榜位置
grep -n "REDBLACK_DATA" index.html

# 查看赔率位置
grep -n "renderOdds" index.html
```
