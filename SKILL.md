---
name: tcms-planner
description: |
  For tech-product marketing teams — turns knowledge-base updates, competitor signals, and content calendars into structured topic briefs, not finished articles.
  Outputs 1-3 prioritized briefs per run and never writes article body text or auto-triggers downstream skills.
  Pairs with tcms-writer (drafting) and tcms-compliance-reviewer (pre-publish review); not for neutral industry research.
read_when:
  - 选题
  - 本周内容
  - content plan
  - 选题建议
  - 内容规划
version: 1.0.0
allowed-tools:
  - read_file
  - write_to_file
  - replace_in_file
  - search_content
  - list_dir
disable: false
---

# TCMS Planner

基于多个信号源生成结构化的选题Brief。不做内容创作，只做选题判断和Brief输出。

## 工作模式：路由

按信号来源分流到不同的选题逻辑：

```
信号输入
   ↓
[路由判断]
   ├── 知识库有新增/更新 → 新内容选题
   ├── 竞品有重要动作   → 差异化响应选题
   ├── 内容日历到期     → 排期驱动选题
   └── 用户主动要求     → 按指定方向选题
```

## 工作流程

### Step 1: [确定性] 确认选题触发源

**A. 用户主动要求** → 用户指定了方向/类型 → 直接进入 Step 3

**B. 定期检查** → 读取以下信号源：
1. 知识库最近更新 — 检查文件修改时间，识别最近更新的产品章节
2. 内容日历 — 读取当前月份排期，找到本周/下周应产出的内容
3. 已发布内容目录 — 检查哪些选题已有初稿、哪些还缺

> 以上路径需根据实际项目目录配置。

### Step 2: [确定性] 读取上下文

- **必读**：内容日历当月排期
- **必读**：产品线/产品中心对照（按实际组织结构配置）
- **按需**：知识库中对应产品的章节（用search_content定位，不读全文）
- **按需**：存量内容盘点（判断是否有可二次推广的存量）

### Step 3: [LLM] 生成选题建议

基于信号源和上下文，生成1-3个选题建议。每个选题输出结构化Brief：

```markdown
## 选题Brief

**选题标题**：[工作标题，非最终发布标题]
**文章类型**：技术博客 / 客户案例 / 产品解读
**目标产品**：[产品名，使用官网正式名称]
**目标读者**：[具体画像]
**核心信息**：[一句话概括要传达的核心信息]
**目的**：[文章要达到什么效果]
**素材指引**：
  - 知识库条目：[指向哪个章节]
  - 内部案例：[📋 如有，标注客户名称+脱敏要求]
  - 已发布文章：[如有可参考的]
**目标渠道**：[首发渠道 + 同步渠道]
**审批级别**：L1 / L2 / L3
**优先级**：高 / 中 / 低
**建议发布时间**：[具体日期或时间窗口]
```

### Step 4: [LLM] 选题合理性检查

- [ ] 产品是否在当前优先级范围内
- [ ] 是否与近2周已发布/已排期的内容重复
- [ ] 素材是否充足（知识库中有对应条目）
- [ ] 渠道是否合理

### Step 5: [确定性] 输出

将Brief保存到 `content-calendar/briefs/YYYY-MM-DD-brief.md`

输出执行摘要：
```
## 执行摘要
- 触发源：[用户要求 / 日历排期 / 知识库更新]
- 读取了：[列出读取的文件]
- 产出选题：[N]个
- 需人工确认后进入content-writer
```

## ⚠️ 人工介入

选题Brief产出后**必须经人工确认**才能进入content-writer。不可自动触发下游Skill。

## 硬性规则

1. **不做内容创作**。只输出选题Brief，不写文章正文
2. **不自动触发下游**。Brief必须经人工确认
3. **素材不足不推荐**。知识库中对应条目太薄则标注"素材不足，建议先补充知识库"
4. **内部案例默认标注脱敏要求**。Brief中涉及内部来源客户必须标📋

## 失败处理

| 失败场景 | 处理方式 |
|----------|---------|
| 内容日历文件不存在 | 提示"未找到内容排期，请先创建" |
| 知识库文件不存在 | 提示"产品知识库未找到，请确认路径" |
| 所有排期内容已有初稿 | 报告"本周排期内容均已有初稿，无需新增选题" |
| 用户要求的产品不在知识库中 | 提示"[产品名]在知识库中无对应条目，建议先补充" |
