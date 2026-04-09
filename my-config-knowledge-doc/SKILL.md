---
name: my-config-knowledge-doc
description: 游戏项目配置表知识文档生成。当用户要求分析游戏配置表、生成配置表文档、说明表字段含义和关联关系、输出联动Checklist时触发。适用于Excel配置表、@Excel注解映射的Java实体类项目。
---

# 配置表知识文档生成

将游戏项目的配置表（Excel + @Excel/@ExcelColumn 注解）分析并生成可学习的知识文档。

## 适用场景

- 新项目需要建立配置表知识库
- 策划频繁配置错误，需要文档指导
- 项目交接时需要完整配置说明
- 后续其他分支复用此流程

## 输出产物

`docs/配置表知识文档_完整版.md`，包含：
- 系统架构（注解机制、解析流程）
- 分隔符约定（`;` `:` `|` `,` 用途）
- 物品/属性格式规范（含示例）
- 所有配置表冷热分类索引
- 核心表详细字段说明
- 表间依赖树与关联图
- 联动配置 Checklist（新增道具/怪物/技能/地图）
- 常见配置错误排查

## 执行流程

### Step 1: 确定项目结构

查找以下关键文件/目录：

```
# 注解定义
y1000-common/.../annotion/Excel.java
y1000-common/.../annotion/ExcelColumn.java

# 配置实体类目录
y1000-server/src/main/java/com/jorsun/game/server/config/excel/

# 配置管理
y1000-server/src/main/java/com/jorsun/game/server/config/ConfigService.java

# 解析工具（如有）
y1000-server/src/main/java/com/jorsun/game/server/config/qiannian/QiannianUtil.java
```

### Step 2: 并行探索配置表

启动 **14个 Explore agent** 并行扫描各类配置表：

| Agent | 探索范围 | 说明 |
|-------|----------|------|
| 1 | core | ItemConfig, CurrencyConfig 等核心经济 |
| 2 | battle | SkillConfig, BuffConfig, NpcConfig 等战斗 |
| 3 | drop | DropConfig, BoxConfig 等掉落 |
| 4 | map | MapConfig, GateConfig, ChannelSpecialMapConfig |
| 5 | shop | ShopConfig, ExchangeConfig, RechargeConfig |
| 6 | task | TaskConfig, ActiveConfig, GuideConfig |
| 7 | social | MarriageConfig, SectConfig, ChatConfig |
| 8 | appearance | DressUpNewConfig, HeadConfig, ArtifactBaptizeConfig |
| 9 | partner | PartnerConfig, PartnerSkillConfig, PartnerUpgradeConfig |
| 10 | resource | BuildConfig, GatherConfig, TitleConfig |
| 11 | arena | ArenaConfig, RankMatchConfig, CrossArenaConfig |
| 12 | function | FunctionOpenConfig, MonitorConfig |
| 13 | skill | SkillSpecialConfig, PassiveSkillConfig, SkillLevelConfig |
| 14 | collection/effect | AchievementConfig,成就, 特效相关 |

每个 Agent 需报告：
- Excel 文件名与 @Excel name 值
- 所有字段 @ExcelColumn（value, type, desc）
- initRow() 中的 addExcelRelationshipId() 调用
- 特殊分隔符或格式说明

### Step 3: 汇总所有配置表清单

收集所有配置表，按以下维度分类：

**热表（高频使用）**：ItemConfig, MapConfig, SkillConfig, NpcConfig, DropConfig, GateConfig, BuffConfig, TaskConfig, ShopConfig, CurrencyConfig

**冷表（低频但不可少）**：TitleConfig, AchievementConfig, MarriageConfig, ChatConfig, AuctionConfig 等

### Step 4: 生成文档

按以下结构组织 Markdown：

```markdown
# 游戏配置表知识文档

> 项目: {项目名}
> 生成时间: {日期}
> 配置表总数: {N}个

---

## 一、配置表架构
### 1.1 系统架构
### 1.2 核心注解

## 二、分隔符约定
| 分隔符 | 用途 |
|--------|------|

## 三、物品相关格式
### 3.1 物品列表格式 (items)
### 3.2 物品概率格式 (itemRates)
### 3.3 属性格式 (prop)
### 3.4 列表格式 (list)

## 四、配置表总览
### 4.1 热表（核心高频）
### 4.2 冷表（辅助功能）

## 五、核心表详解
（按需详细展开每个热表）

## 六、表间依赖树
## 七、关联关系图
## 八、联动配置Checklist
### 新增道具
### 新增怪物
### 新增技能
### 新增地图

## 九、配置表索引（按Excel名称）
## 十、常见错误排查
```

### Step 5: 输出到 docs/

文档路径：`{项目根目录}/docs/配置表知识文档_完整版.md`

## 关键文件路径参考

项目不同时，需要调整以下路径模式：
- `{common模块}/.../annotion/Excel.java` - 注解定义
- `{server模块}/.../config/excel/*.java` - 配置实体类（关键）
- `{server模块}/.../config/ConfigService.java` - 配置管理与关系映射
- `{server模块}/.../config/qiannian/QiannianUtil.java` - 字符串解析工具

## 分隔符规范（通用）

| 分隔符 | 值 | 用途 | 示例 |
|--------|----|------|------|
| `;` | semicolon | 多元素分隔 | `10001:1;10002:5` |
| `:` | colon | 键值对 | `10001:1` = ID:数量 |
| `\|` | pipe | 组分隔 | 多组物品 |
| `,` | comma | 坐标分隔 | `x,y` 坐标 |

## 物品格式规范（通用）

**items**: `物品ID:数量;物品ID:数量`
**itemRates**: `物品ID:数量:概率;...` （概率通常为‰）
**prop**: `属性枚举:值;属性枚举:值`
**list**: `ID;ID;ID` （纯ID列表）

## 联动Checklist模板

当添加新配置时，必须同步检查以下表：

**新增道具**：
- [ ] item 表（基础属性）
- [ ] shop 表（如果有售卖）
- [ ] drop 表（如果会掉落）
- [ ] task/active 表（如果任务奖励）
- [ ] buff 表（如果有使用效果）

**新增怪物**：
- [ ] npc 表（基础属性）
- [ ] skill 表（攻击技能）
- [ ] drop/gate 表（掉落/刷新点）
- [ ] buff 表（如果有特殊能力）

**新增技能**：
- [ ] skill 表（基础属性）
- [ ] skill_level 表（等级数据）
- [ ] npc 表（如果怪物使用）
- [ ] buff 表（如果有附加状态）

**新增地图**：
- [ ] map 表（基础属性）
- [ ] gate 表（关卡配置）
- [ ] npc 表（如果刷新怪物）
- [ ] channel_special_map 表（特殊通道）
