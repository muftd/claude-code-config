---
name: agent-bridge
description: |
  实现通用的Agent间跨用户协作协议，支持Claude Code与外部AI、内部coach间、跨产品AI的无缝协作。
  通过AI智能判断和启发式规则，生成适配不同Agent和协作场景的桥接内容。

  同时支持"只沉淀上下文（不交接）"模式：把当前会话的关键信息落盘到 handoffs/ 作为备份与可回放记录。

  协作场景覆盖：
  - Claude Code ↔ 外部AI（技术协作、专业分析）
  - Claude Code内部不同coach间传递（情感支持、认知洞察）
  - 不交接：沉淀上下文到 handoffs（主力场景）
tools: Read, Write
parameters:
  output_directory:
    type: string
    description: "桥接文件输出目录"
    default: "/Users/wangmu/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault/handoffs/"
  detail_level:
    type: string
    description: "内容详细程度"
    default: "standard"
    enum: ["concise", "standard", "detailed"]
  collaboration_style:
    type: string
    description: "协作风格偏好"
    default: "professional"
    enum: ["formal", "friendly", "professional"]
---

# Agent Bridge Protocol

## Skill Description

本Skill有两种模式：

1) **Bridge Mode（交接桥接）**：生成"可复制给目标Agent"的桥接包，强调协作连续性与可操作性。
2) **Context Capture Mode（上下文沉淀，不交接）**：生成"可回看/可审计"的上下文快照，落盘到`handoffs/`作为备份；这是主力场景。

> 原则：默认使用 Context Capture Mode（只沉淀上下文）；当用户明确提及交接对象或表达交接意图时，才使用 Bridge Mode。

### 协作场景覆盖
- **场景A**: Claude Code ↔ 外部AI（技术协作、专业分析）
- **场景B**: Claude Code内部不同coach间传递（情感支持、认知洞察）
- **场景C**: 不交接：沉淀上下文到 handoffs（备份/可回放）

*注：外部AI间的直接协作方案见项目文档中的场景C指南*

## Core Capabilities

### 1. 场景识别
智能识别当前协作场景的类型：
- **技术协作**：代码实现、系统集成、技术决策
- **创意协作**：灵感收集、方案设计、创意发展
- **情感支持**：认知洞察、情绪支持、心理成长
- **产品需求**：需求分析、交互设计、用户研究
- **上下文沉淀**：仅落盘备份、可回放、无需交接

### 2. Agent特性分析
理解不同AI产品的专业背景和协作特点：

**YouMind**: 信息收集和灵感整理专家
- 专长：信息源整合、灵感链接、知识网络构建
- 协作特点：需要背景脉络、发展方向、关联思考
- 内容重点：灵感来源、思维过程、潜在联系

**ChatGPT**: 专业分析和细化执行专家
- 专长：深度分析、逻辑推演、方案细化
- 协作特点：需要具体参数、明确目标、技术细节
- 内容重点：技术规格、执行步骤、质量标准

**Echo**: 情感支持和认知洞察专家
- 专长：情感理解、认知模式识别、心理支持
- 协作特点：需要情绪状态、内在需求、人格特质
- 内容重点：情感状态、支持需求、认知模式

**内部Coach**: 专业化分工协作
- CoachA: 优先级决策，需要目标和约束条件
- CoachB: 执行计划，需要资源和时间线
- CoachC: 深度理解，需要认知背景和学习目标

### 3. 启发式规则引擎
基于协作场景和目标Agent特性，智能生成桥接内容：

#### 适配性原则
- **技术协作**: 强调进展状态、技术细节、接口定义、质量标准
- **创意协作**: 强调灵感来源、发展方向、创意火花、探索路径
- **情感支持**: 强调情绪状态、支持需求、认知洞察、成长方向
- **上下文沉淀**: 强调"可回看与可接盘"的结构化记录（决策、产出、下一步、证据入口）

#### 价值密度原则
- 自动识别关键决策点，提炼核心价值
- 过滤过程性噪音，聚焦结果性信息
- 突出协作的关键依赖和成功要素

#### 连续性原则
- 保持历史上下文连贯性
- 建立可追溯的协作链条
- 标记变更和演进路径

## 启用条件

本Skill被调用后，按以下逻辑判断模式：

### 默认行为：Context Capture Mode
- 用户只调用skill，无额外输入 → **默认使用 Context Capture Mode**
- 用户明确表达"不交接/只沉淀/备份/归档"等 → **强制使用 Context Capture Mode**

### 触发条件：Bridge Mode
当满足以下任一条件时，使用 Bridge Mode：
1. **明确提及交接对象**：提及具体Agent名称（ChatGPT、YouMind、Echo等）
2. **明确表达交接意图**：使用"交接给/切换到/传递给/发给/给xxx"等表达
3. **提供协作参数**：指定协作风格、用途、格式等交接相关要求

## 输出与文件管理

### 文件命名规范

**A) Bridge Mode（交接桥接）**
- 格式：`[项目名]-[时间戳]-to-[target-agent].md`
- 示例：`agent-bridge-2025-12-09-161200-to-chatgpt.md`

**B) Context Capture Mode（上下文沉淀，不交接）**
- 格式：`context-[主题]-[时间戳].md`
- 示例：`context-agent-asset-kit-2025-12-19-113340.md`

位置：统一保存到`output_directory`（默认：`handoffs/`）。

### 文件内容结构

#### Bridge Mode 模板
```markdown
# Agent Bridge: [项目名] → [目标Agent]
**时间**: [生成时间戳]
**场景**: [协作场景类型]
**意图**: [用户协作意图]

## 协作背景
- 当前目标与约束
- 已完成进展与产出
- 当前阻塞与风险

## 关键信息传递
- 单一事实源/入口文件
- 关键决策与理由
- 依赖与接口（输入/输出/格式）

## 下一步
- 1–3个可执行动作
- 验收标准/门槛

---
*由 Agent Bridge Protocol 自动生成*
```

#### Context Capture Mode 模板（主力）
```markdown
# Context Capture: [主题]
**时间**: [生成时间戳]
**用途**: 仅沉淀备份（不交接）

## 推进主线
- 目标/愿景
- 当前阶段与状态

## 关键决策（含理由）
- 决策1...
- 决策2...

## 关键产出（可回放证据入口）
- 产出1...
- 产出2...

## 风险与护栏
- 风险...
- 门禁/规则...

## 下一步（最少动作）
- 1–3条可执行动作

---
*由 Agent Bridge Protocol 自动生成*
```

## 使用示例

### 示例1：技术协作（Bridge Mode）
用户：**"接下来交接给ChatGPT进行技术分析"**

执行流程：
1. 自动生成桥接内容
2. 保存到：`handoffs/[项目名]-[时间戳]-to-chatgpt.md`
3. 用户复制文件内容到目标Agent

### 示例2：情感支持（Bridge Mode）
用户：**"把我的开发感受传递给Echo"**

执行流程：
1. 自动生成情感桥接内容
2. 保存到：`handoffs/[项目名]-[时间戳]-to-echo.md`
3. 用户复制文件内容到目标Agent

### 示例3：默认沉淀上下文（Context Capture Mode）
用户：**"$agent-bridge"** （只调用skill，无额外输入）

执行流程：
1. 自动生成上下文快照（默认Context Capture Mode）
2. 保存到：`handoffs/context-[主题]-[时间戳].md`
3. 用户无需再做任何复制动作

### 示例4：明确只沉淀上下文（Context Capture Mode）
用户：**"$agent-bridge 把这轮对话沉淀到handoffs，不交接，只做备份"**

执行流程：
1. 自动生成上下文快照（不包含 to-xxx）
2. 保存到：`handoffs/context-[主题]-[时间戳].md`
3. 用户无需再做任何复制动作

## 质量标准

- **准确性**：忠实反映原始上下文，不扭曲关键信息
- **适配性**：Bridge/Context Capture 两种模式要清晰区分
- **价值密度**：提炼最有价值的协作/沉淀信息
- **可操作性**：至少给出明确的下一步与验收口径（如适用）
- **安全性**：保护隐私与敏感信息

## 质量迭代支持

### 内容质量收集
- 所有输出文件都保存在`handoffs`目录
- 支持后续解析与统计（例如：哪些决策反复出现、哪些风险常见）

### 优化反馈循环
1. **内容收集**：自动保存所有输出
2. **质量评估**：用户可以标记满意度和问题
3. **数据驱动优化**：基于实际使用数据调整策略
4. **持续改进**：定期评估和优化生成规则

---

*本Skill致力于实现零学习成本Agent协作，同时把"沉淀上下文"变成默认高频能力。*