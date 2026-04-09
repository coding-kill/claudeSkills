---
name: my-review-code-skill
description: Use when needing to perform comprehensive code review of a Java game backend project, requiring multi-perspective analysis from architecture, testing, design, player experience, customer service, and operations viewpoints
---

# Multi-Perspective Code Review

## Overview

**多视角代码审核方法论**：以高级架构师、测试工程师、游戏策划、玩家用户、客服客诉、运维、业务逻辑、配置表八大视角进行全面审视。

**核心理念：**
- 架构师看"能不能改"（扩展性、复杂度）
- 测试工程师看"会不会坏"（边界、异常）
- 策划看"是否符合设计"（数值、玩法）
- 玩家看"稳不稳定"（崩溃、卡顿、公平）
- 客服看"能不能找到问题"（日志、追溯）
- 运维看"能不能扛住"（监控、告警）
- 业务逻辑看"流程对不对"（事务、一致性、幂等）
- 配置表看"配置全不全"（非空、类型、热更新）

**风险最大化原则：优先审核高风险区域，以最小成本发现最多问题。**

## When to Use

**触发场景：**
- 陌生项目/模块的全面审核
- 提交前的最终质量把关
- 重构前的基线审核
- 核心链路变更后的专项审核
- 版本发布前的整体评估

**不适用的场景：**
- 单一文件的语法/风格问题（直接修复即可）
- 紧急热修复（走简化审核流程）

---

## Core Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  PHASE 1: Architecture Exploration                          │
│  模块划分 → 依赖关系 → 核心链路 → 入口点识别                 │
└────────────────────────┬────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 2: Multi-Perspective Risk Assessment                 │
│  六视角风险矩阵 → 高风险模块排序 → 审核优先级                 │
└────────────────────────┬────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 3: Targeted Review by Perspective                    │
│  各视角定向审核 → 问题归类 → 输出综合报告                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Architecture Exploration

### 探索维度

**1. 模块划分（Module Boundaries）**
- 顶级包/模块结构及命名
- 模块职责单一性
- 模块粒度合理性

**2. 依赖关系（Dependencies）**
- 循环依赖检测
- 核心模块依赖度分析
- 外部依赖管理

**3. 核心链路（Critical Paths）**
- API入口、消息处理、定时任务
- 跨模块调用路径
- 关键业务链：交易链、战斗链、物品链

### 探索命令

```bash
# 项目结构
find . -type d -name "src" | head -20
ls -la */

# 模块依赖
grep -r "import " --include="*.java" | sed 's/.*import \([^*]*\)\..*/\1/' | sort | uniq -c | sort -rn

# 入口点
grep -r "@RequestMapping\|@GetMapping\|@PostMapping" --include="*.java"
grep -r "extends.*Application\|SpringBootApplication" --include="*.java"

# 游戏特有入口
grep -r "MessageHandler\|CommandHandler\|Controller" --include="*.java"
```

---

## Phase 2: Multi-Perspective Risk Assessment

### 六视角风险矩阵

| 视角 | 高风险指标 | 审核优先级 |
|------|-----------|-----------|
| **架构师** | 循环依赖、职责混乱、缺乏抽象 | P0 |
| **测试工程师** | 边界条件、异常场景、数值溢出 | P0 |
| **策划** | 数值公式错误、配置不一致、概率操控 | P1 |
| **玩家用户** | 崩溃风险、线程死锁、内存泄漏 | P0 |
| **客服客诉** | 日志缺失、操作无追溯、常见bug模式 | P1 |
| **运维** | 监控缺失、告警遗漏、无自愈能力 | P1 |
| **业务逻辑** | 事务边界、状态机流转、幂等性校验 | P0 |
| **配置表** | 非空校验缺失、热更新影响、类型校验 | P0 |

### 高风险模块识别

**游戏后端典型高风险模块：**

```
P0-Critical:
  - battle/        战斗结算、数值计算
  - trade/auction  交易系统、货币原子性
  - item/          物品增删改、库存一致性
  - role/          玩家数据、状态管理

P1-Important:
  - quest/         任务完成判定
  - skill/         技能效果、CD冷却
  - chat/          消息过滤、内容安全

P2-Low:
  - config/        配置加载
  - util/          工具类
  - data/          数据结构
```

---

## Phase 3: Targeted Review by Perspective

### A. 架构师视角（Architect）

**审核清单：**

| 检查项 | 具体内容 | 风险 |
|--------|---------|------|
| **SOLID原则** | SRP: 职责是否单一；OCP: 是否开放扩展 | Critical |
| **循环依赖** | 模块间 A→B→C→A | Critical |
| **依赖倒置** | 高层模块依赖低层实现而非抽象 | Important |
| **接口抽象** | 关键逻辑是否有接口隔离 | Important |
| **技术债务** | 临时方案、重复代码、未来隐患标记 | Important |
| **架构模式** | 是否使用合适的模式（MQ、事件驱动等） | Minor |

**架构坏味道识别：**
```
🚨 神类（God Class）：单个类超过500行，承担过多职责
🚨 霰弹修改（Shotgun Surgery）：一个改动需要修改多个类
🚨 平行继承（Parallel Inheritance）：新功能需要新增相似子类
🚨 过度耦合：过深调用链（>5层）
```

---

### B. 测试工程师视角（Test Engineer）

**审核清单：**

| 检查项 | 具体内容 | 风险 |
|--------|---------|------|
| **可测试性** | 依赖注入、Mock难度、测试桩是否容易构造 | Critical |
| **边界值** | 数值溢出（Long.MAX_VALUE）、精度丢失 | Critical |
| **异常场景** | 空输入、非法状态、外部服务超时 | Critical |
| **并发测试** | 竞态条件、死锁、线程安全 | Critical |
| **数值精度** | 浮点数计算、货币单位（分/元） | Critical |
| **测试覆盖盲区** | 关键路径是否覆盖、异常分支是否测试 | Important |

**边界值问题模式：**
```java
// 🚨 整数溢出风险
int gold = 1000000 * 1000;  // 超过int范围

// 🚨 浮点数精度
if (rate == 0.1)  // 浮点数比较

// 🚨 边界条件
if (level >= 1 && level <= 100)  // 边界值漏掉0或101

// 🚨 货币精度（游戏常见）
player.addGold(0.1 * price);  // 浮点金币
player.addGold(price / 10);    // 整数除法丢失精度
```

---

### C. 策划视角（Game Designer）

**审核清单：**

| 检查项 | 具体内容 | 风险 |
|--------|---------|------|
| **数值公式** | 公式是否符合策划文档（增长率、衰减曲线） | Critical |
| **配置表一致性** | 代码逻辑与配置表定义是否一致 | Critical |
| **概率实现** | 随机算法是否与设计一致（伪随机/真随机） | Critical |
| **玩法规则边界** | 边界情况是否按策划设计处理 | Important |
| **数值上限** | 最大等级、最大道具数量等硬性限制 | Important |
| **公式可读性** | 数值公式是否有注释说明 | Minor |

**策划视角典型问题：**
```java
// 🚨 数值与配置不一致
// 配置表定义：rate = level * 1.5
// 代码实现：rate = level * 1.6  // 与配置不符

// 🚨 概率实现错误
// 策划需求：10%概率暴击
// 代码实现：if (random() < 0.1f)  // 均匀分布，伪随机
// 实际需求：伪随机保证体验一致性

// 🚨 数值公式硬编码
// 应该是：公式 = 配置表中读取
// 实际：hardcode = level * 100 + 50

// 🚨 边界值未处理
// 策划：0级应该是无效的
// 代码：未检查0值，直接参与计算
```

---

### D. 玩家用户视角（Player/User）

**审核清单：**

| 检查项 | 具体内容 | 风险 |
|--------|---------|------|
| **崩溃风险** | NPE、数组越界、类型转换异常 | Critical |
| **线程安全** | 静态变量、多线程竞争 | Critical |
| **内存泄漏** | 缓存无限增长、集合未清理 | Critical |
| **反作弊** | 关键逻辑是否放客户端、数值是否服务端校验 | Critical |
| **数据一致性** | 玩家财产（货币、物品）是否会丢失 | Critical |
| **体验影响** | 同步调用超时、N+1查询导致卡顿 | Important |

**玩家视角致命问题：**
```java
// 🚨 玩家财产损失风险
synchronized (this) {
    player.addGold(-100);  // 扣钱
    // 如果这里抛异常，金钱已扣但物品未发
    itemService.giveItem(player, item);
}

// 🚨 反作弊：客户端校验
// 关键数值（伤害、暴击）应该在服务端计算
// 客户端只做表现层

// 🚨 内存泄漏
private static Map<String, List<Player>> cache = new HashMap<>();
// 缓存只增不减，导致OOM

// 🚨 竞态条件导致玩家数据错乱
if (player.getGold() >= cost) {
    player.setGold(player.getGold() - cost);
    // 另一个线程同时扣钱，导致余额错误
}
```

---

### E. 客服客诉视角（Customer Service）

**审核清单：**

| 检查项 | 具体内容 | 风险 |
|--------|---------|------|
| **日志完整性** | 关键操作是否记录日志（增删改、异常） | Critical |
| **日志可读性** | 日志是否包含关键上下文、是否可查询 | Important |
| **操作可追溯** | 玩家关键操作是否有审计日志 | Critical |
| **问题可定位** | 错误日志是否包含堆栈、参数、上下文 | Important |
| **客诉高频bug** | 历史客诉中常见的bug模式是否重复出现 | Important |
| **补偿记录** | GM补偿操作是否完整记录 | Important |

**日志问题模式：**
```java
// 🚨 关键操作无日志
player.addGold(100);  // 金币变动无日志
itemService.removeItem(item);  // 物品删除无日志

// 🚨 日志缺失关键上下文
logger.info("扣钱成功");  // 缺失：玩家ID、金额、时间、原因

// 🚨 异常被吞掉
} catch (Exception e) {
    // 什么都不记录，客诉无法定位
}

// ✅ 正确示范
logger.info("[ITEM_LOG] playerId={}, action=REMOVE, itemId={}, reason={}, time={}",
    playerId, itemId, reason, System.currentTimeMillis());
```

**客诉快速定位指南：**
```
发现问题 → 查询 itemLog/playerLog 表 → 定位玩家和时间 → 查对应日志
```

---

### F. 运维视角（Operations）

**审核清单：**

| 检查项 | 具体内容 | 风险 |
|--------|---------|------|
| **监控埋点** | 核心方法是否有监控指标 | Critical |
| **告警阈值** | 是否有合理阈值、是否覆盖关键指标 | Important |
| **故障自愈** | 是否有熔断、降级、重试机制 | Important |
| **配置外部化** | 环境相关配置是否外部管理 | Important |
| **健康检查** | 是否有 /health 接口 | Minor |
| **日志级别** | 生产环境是否关闭debug日志 | Minor |

**运维视角问题模式：**
```java
// 🚨 监控埋点缺失
// 关键方法无metrics
public void processTrade() {
    // 无耗时监控、无QPS统计
}

// 🚨 无熔断降级
// 外部服务挂了导致主流程失败
callExternalService();  // 无超时、无降级

// 🚨 硬编码配置
private static final String URL = "http://prod-server";  // 应该在配置中心

// ✅ 正确示范
@Value("${trade.service.url}")
private String tradeServiceUrl;

// 监控埋点
@Autowired
private MeterRegistry meterRegistry;

meterRegistry.counter("trade.process.success").increment();
meterRegistry.timer("trade.process.duration").record(duration);
```

---

### G. 业务逻辑视角（Business Logic）

**审核清单：**

| 检查项 | 具体内容 | 风险 |
|--------|---------|------|
| **流程完整性** | 关键业务流程是否覆盖所有路径（正向/逆向/异常） | Critical |
| **状态机校验** | 状态流转是否合法、边界是否校验 | Critical |
| **事务边界** | 跨模块操作的原子性是否保证（一件事要么全做要么全不做） | Critical |
| **幂等性** | 重复操作是否安全（防重放、防重复扣款） | Critical |
| **补偿机制** | 失败后是否有补偿/回滚操作 | Important |
| **业务规则一致性** | 同一规则在不同模块是否一致实现 | Important |
| **异步可靠性** | MQ消息是否可靠投递、消费是否幂等 | Important |

**业务逻辑问题模式：**
```java
// 🚨 事务边界不清
// 扣钱和发物品不在同一个事务中
playerService.deductGold(player, cost);  // 成功
itemService.giveItem(player, item);       // 失败 → 玩家钱扣了但没收到物品

// 🚨 状态机流转未校验
// 订单已取消后不能再完成支付
if (order.getStatus() == OrderStatus.CANCELLED) {
    // 应该有明确校验，而不是静默处理
    return;
}

// 🚨 幂等性缺失
// 同一个请求重复发送导致重复扣款
@PostMapping("/pay")
public void pay(PayRequest request) {
    playerService.deductGold(request.getAmount());  // 无幂等校验
}

// ✅ 正确示范：状态机校验
if (order.getStatus() != OrderStatus.PAID) {
    throw new BusinessException("订单状态不允许支付");
}

// ✅ 正确示范：幂等性
private Set<String> processedRequestIds = new ConcurrentHashSet<>();
if (!processedRequestIds.add(request.getRequestId())) {
    return;  // 重复请求，直接返回
}

// ✅ 正确示范：事务边界
@Transactional
public void purchase(Player player, Item item, int count) {
    playerService.deductGold(player, cost);  // 同一事务
    itemService.giveItem(player, item, count); // 同一事务
    // 失败时自动回滚
}
```

**游戏业务常见流程检查点：**

| 业务场景 | 必须校验的逻辑 |
|---------|--------------|
| **购买物品** | 金钱扣除原子性 → 物品发放 → itemLog记录 |
| **拍卖竞拍** | 出价校验 → 最高价更新 → 成交 → 物品转移 → itemLog |
| **战斗结算** | 伤害计算 → 死亡判定 → 奖励发放 → 记录 |
| **邮件领取** | 附件校验 → 物品发放 → 已领取标记 |
| **交易完成** | 双方确认 → 物品转移 → 货币转移 → 记录 |

---

### H. 配置表视角（Config Table）

**审核清单：**

| 检查项 | 具体内容 | 风险 |
|--------|---------|------|
| **非空校验** | 必填字段是否校验、缺失时是否有默认值 | Critical |
| **类型校验** | 字段类型是否匹配、是否有隐式转换 | Critical |
| **枚举值校验** | 配置值是否在有效枚举范围内 | Important |
| **默认值处理** | 新增字段旧配置是否有合理默认值 | Important |
| **热更新影响** | 在线导表对现有玩家数据的影响范围 | Critical |
| **配置刷新安全** | 运行时刷新配置的线程安全、事务完整性 | Critical |
| **配置版本兼容** | 新增/删除字段、修改字段对旧配置的兼容 | Important |
| **跨表关联校验** | 关联ID是否存在、引用完整性 | Critical |

**配置表问题模式：**
```java
// 🚨 非空字段无校验
// 配置表 id 字段为必填，但代码直接使用
int id = config.getId();  // 配置缺失时为0，导致业务逻辑错误

// 🚨 热更新导致数据不一致
// 玩家已加载旧配置，导表后未刷新玩家数据
Map<Integer, SkillConfig> skillCache = ...;  // 静态缓存
// 导表后只刷新了skillCache，但在线玩家已缓存的技能数据未更新

// 🚨 配置表新增字段无默认值处理
public void initPlayerSkill(Player player, int skillId) {
    SkillConfig config = skillConfigMap.get(skillId);
    // 如果skillId是旧玩家数据中存在的，但新配置表中skillId对应的记录被删除了
    // get返回null导致NPE
}

// ✅ 正确示范
SkillConfig config = skillConfigMap.get(skillId);
if (config == null) {
    log.warn("技能配置不存在 skillId={}", skillId);
    return;  // 或使用默认配置
}

// ✅ 正确示范：热更新安全刷新
@Configuration
public class SkillConfigRefresher {
    @Autowired
    private SkillService skillService;

    public void refresh() {
        // 1. 暂停新玩家加载
        // 2. 等当前操作完成
        // 3. 更新配置
        // 4. 刷新缓存
        // 5. 通知在线玩家数据刷新
    }
}
```

**在线导表风险检查清单：**

| 场景 | 风险 | 检查点 |
|------|------|--------|
| **修改数值** | 在线玩家数据与新配置不一致 | 是否需要刷新玩家数据 |
| **新增字段** | 旧玩家无此字段数据 | 是否有默认值/兼容处理 |
| **删除字段** | 旧玩家数据引用已删除字段 | 是否清理玩家数据 |
| **修改结构** | 玩家数据格式与新配置不匹配 | 是否有数据迁移机制 |
| **新增配置** | 新功能上线、旧玩家无此配置 | 是否有默认创建逻辑 |

**配置表加载检查点：**

| 检查项 | 说明 |
|--------|------|
| **启动时校验** | 配置表加载时是否校验必填字段、类型、枚举值 |
| **运行时校验** | 运行时获取配置是否做null检查 |
| **异常熔断** | 配置加载失败时服务是否能启动、是否有降级方案 |
| **版本记录** | 是否记录当前配置版本、便于排查问题 |

---

## Problem Classification

### Critical（阻塞发布）

| 类别 | 问题描述 |
|------|---------|
| **架构** | 循环依赖、架构塌陷 |
| **测试** | 整数溢出、货币精度丢失 |
| **策划** | 数值公式与配置不一致 |
| **玩家** | 玩家财产损失风险、崩溃 |
| **客服** | 关键操作无日志 |
| **运维** | 核心链路无监控 |
| **业务** | 事务边界不清、状态机流转错误、幂等性缺失 |
| **配置** | 非空字段无校验导致NPE、热更新数据不一致 |

### Important（下个迭代修复）

| 类别 | 问题描述 |
|------|---------|
| **架构** | 技术债务、缺乏抽象接口 |
| **测试** | 可测试性差、异常场景未覆盖 |
| **策划** | 公式硬编码、边界值未处理 |
| **玩家** | 体验问题（卡顿） |
| **客服** | 日志上下文缺失 |
| **运维** | 监控埋点不完整 |
| **业务** | 业务规则不一致、补偿机制缺失 |
| **配置** | 枚举值未校验、默认值处理不当 |

### Minor（优化项）

| 类别 | 问题描述 |
|------|---------|
| **架构** | 架构模式不匹配 |
| **测试** | 测试覆盖盲区 |
| **策划** | 公式注释缺失 |
| **玩家** | 体验优化 |
| **客服** | 日志格式不规范 |
| **运维** | 健康检查缺失 |
| **业务** | 流程可读性差、异常处理不友好 |
| **配置** | 配置表注释不规范、字段命名不一致 |

---

## Output Format

```
# 代码审核报告

## 基本信息
| 项目/模块 | 审核时间 | 审核范围 | 审核视角 |
|-----------|---------|---------|---------|
| {MODULE}  | {DATE}  | {SCOPE} | 六视角   |

---

## 一、架构概览

**模块划分及职责:**
| 模块 | 职责 | 风险等级 |
|------|------|---------|
| {name} | {desc} | {risk} |

**依赖关系简述:**
{DEP_SUMMARY}

**核心链路:**
{TRADE_PATH}

---

## 二、八视角风险矩阵

| 视角 | 高风险点 | 问题数 | 阻塞/严重/一般 |
|------|---------|-------|--------------|
| 架构师 | xxx | 3 | 1/1/1 |
| 测试工程师 | xxx | 5 | 2/2/1 |
| 策划 | xxx | 2 | 1/1/0 |
| 玩家用户 | xxx | 4 | 2/1/1 |
| 客服 | xxx | 3 | 1/2/0 |
| 运维 | xxx | 2 | 1/1/0 |
| 业务逻辑 | xxx | 3 | 1/1/1 |
| 配置表 | xxx | 4 | 2/1/1 |

---

## 三、架构师视角问题

### Critical
1. **[循环依赖]** {FILE}:{LINE}
   - 问题：{DESC}
   - 影响：{IMPACT}
   - 建议：{FIX}

### Important/Minor
...

---

## 四、测试工程师视角问题

### Critical
1. **[整数溢出]** {FILE}:{LINE}
   - 问题：{DESC}
   - 影响：{IMPACT}
   - 建议：{FIX}

---

## 五、策划视角问题

### Critical
1. **[数值公式不一致]** {FILE}:{LINE}
   - 策划文档：{DOC_FORMULA}
   - 代码实现：{CODE_FORMULA}
   - 建议：{FIX}

---

## 六、玩家用户视角问题

### Critical
1. **[玩家财产损失风险]** {FILE}:{LINE}
   - 问题：{DESC}
   - 影响：{IMPACT}
   - 建议：{FIX}

---

## 七、客服客诉视角问题

### Critical
1. **[关键操作无日志]** {FILE}:{LINE}
   - 操作：{ACTION}
   - 建议：添加 itemLog

---

## 八、运维视角问题

### Critical
1. **[监控埋点缺失]** {FILE}:{LINE}
   - 建议：添加 metrics

---

## 九、业务逻辑视角问题

### Critical
1. **[事务边界不清]** {FILE}:{LINE}
   - 问题：{DESC}
   - 影响：{IMPACT}
   - 建议：{FIX}

---

## 十、配置表视角问题

### Critical
1. **[非空字段无校验]** {FILE}:{LINE}
   - 配置表字段：{FIELD}
   - 问题：获取配置后直接使用，未校验null
   - 影响：运行时NPE
   - 建议：添加非空校验或默认值

### Important
1. **[热更新影响]** {FILE}:{LINE}
   - 问题：在线导表后玩家数据与配置不一致
   - 建议：评估是否需要刷新玩家缓存数据

---

## 十一、总体评估

| 维度 | 评级 | 说明 |
|------|------|------|
| 架构质量 | {A/B/C} | {DESC} |
| 代码质量 | {A/B/C} | {DESC} |
| 业务正确性 | {A/B/C} | {DESC} |
| 配置表完整性 | {A/B/C} | {DESC} |
| 运维友好度 | {A/B/C} | {DESC} |

**综合风险等级:** 🔴 高 / 🟡 中 / 🟢 低

**是否可发布:** ✅ 是 / ❌ 否（阻塞问题待修复）

**预计修复工时:** {X}h

---

## 十二、修复优先级

| 优先级 | 问题数 | 预计修复 |
|--------|-------|---------|
| P0-Critical | {N} | 立即修复 |
| P1-Important | {N} | 下个迭代 |
| P2-Minor | {N} | 持续优化 |
```

---

## Quick Reference

### 审核优先级速判

```
P0 → 立即修复，阻塞发布
     （可能导致玩家财产损失、崩溃、数据不一致）

P1 → 计划修复，下个迭代
     （影响体验、存在隐患但不阻塞）

P2 → 优化项，不阻塞发布
     （代码质量、架构优化）
```

### 审核范围速判

| 范围 | 适用场景 | 耗时 |
|------|---------|------|
| 单包 | 小改动、单一功能 | 15-30min |
| 核心链路 | 交易、战斗、物品 | 1-2h |
| 全项目 | 整体评估、重构基线 | 4-8h |

### 各视角必查项

| 视角 | 必查项 |
|------|--------|
| 架构师 | 循环依赖、静态可变状态、架构塌陷 |
| 测试工程师 | 数值溢出、边界条件、并发安全 |
| 策划 | 数值公式、配置表一致性、概率实现 |
| 玩家 | 财产原子性、反外挂、崩溃风险 |
| 客服 | itemLog、操作追溯、日志完整性 |
| 运维 | 监控埋点、熔断降级、健康检查 |
| 业务逻辑 | 事务边界、状态机流转、幂等性校验 |
| 配置表 | 非空校验、热更新影响、类型校验 |

---

## Integration with Other Skills

- `systematic-debugging`: 发现问题后的根因分析
- `simplify`: 重复代码和坏味道识别与重构
- `verification-before-completion`: 修复后的验证
- `test-driven-development`: 修复时补充测试用例
- `config-knowledge-doc`: 配置表与代码一致性验证

---

## Report Output & Filing

### 报告落地要求

**重要：所有代码审核完成后，必须将报告落地为md文件。**

**落地步骤：**
1. 使用 Write 工具创建报告文件
2. 文件路径：`{项目根目录}/docs/代码审查/`
3. 文件名格式：`{YYYY}{MM}{DD}{HHmm}_代码审核报告.md`
   - 例如：`202604091430_代码审核报告.md` 表示2026年4月9日14:30生成的报告
4. 写入完整的八视角审核报告内容

### 落地检查清单

- [ ] 报告包含基本信息（项目、审核时间、审核范围）
- [ ] 报告包含八视角风险矩阵
- [ ] 报告包含各视角Critical/High/Medium/Low问题分类
- [ ] 报告包含总体评估（综合风险等级、是否可发布）
- [ ] 报告包含修复优先级清单
- [ ] 文件已成功写入到 `docs/代码审查/` 目录

### Example

```
用户: /my-review-code-skill 帮我审核 auction 模块

执行流程:

1. Architecture Exploration
   - 模块：AuctionMsg、AuctionService、AuctionDao
   - 依赖：trade/, item/, player/
   - 链路：挂单→审核→竞拍→成交→发货→日志

2. Six-Perspective Risk Assessment
   - 架构师：潜在循环依赖风险
   - 测试工程师：竞拍价格边界值未校验
   - 策划：成交公式需确认
   - 玩家：并发竞拍可能导致超卖
   - 客服：成交日志是否完整
   - 运维：竞拍接口无QPS监控

3. Targeted Review
   - 重点审核并发控制
   - 审核数值边界
   - 检查日志完整性

4. Report Filing
   - 使用 Write 工具将报告写入 docs/代码审查/202604091430_代码审核报告.md

输出: 六视角综合审核报告 + 报告已落地
```
