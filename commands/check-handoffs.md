# /check-handoffs 命令 - 检查待加载的摘要文件

⚠️ **执行类型**：此命令需要 Claude 根据以下步骤主动执行和验证，而不是仅显示文档说明。

## 功能

扫描 `handoffs/` 目录，检查是否有新的摘要文件等待加载。支持两种使用模式：
1. **传统模式**：跨 Worktree 的上下文移交检测（向后兼容）
2. **Handover模式**：Agent 间上下文交接文档检查（Sprint 3 新增）

## 用法

```bash
# 检查所有新摘要文件（自动检测模式）
/check-handoffs

# 检查特定日期的摘要
/check-handoffs 2025-11-11

# 列出所有摘要文件（包括已加载的）
/check-handoffs all

# 检查特定 Agent 的 handover（Handover 模式）
/check-handoffs echo
/check-handoffs claude
```

## 执行步骤

### 1. 扫描 handoffs 目录

- 优先扫描 `./handoffs/` 目录（根目录下的 handoffs）
- 如果不存在，则扫描 `../../handoffs/` 目录（传统 Worktree 模式）
- 记录文件名、大小和修改时间
- 如果目录都不存在，创建 `./handoffs/` 目录

### 2. 检测模式和识别"新文件"

#### 模式自动检测
- 如果文件名格式为 `[项目名]-[timestamp]-to-[target].md`，进入 **Handover 模式**
- 如果文件名格式为 `[日期]_[文件名].md`，进入 **传统模式**

#### Handover 模式的文件识别
- 检查是否有针对当前 Agent 的 handover 文件
- "新文件"定义为：`-to-[当前agent].md` 文件且未被加载过
- 如果用户指定了 Agent（如 `/check-handoffs echo`），只显示目标为该 Agent 的文件

#### 传统模式的文件识别
- 比较文件修改时间与最后一次加载时间
- "新文件"定义为：修改时间 > 上次会话的启动时间
- 如果无法确定启动时间，显示最新的 3 个文件

### 3. Handover模式的智能处理

#### 自动检测和加载
- 如果发现针对当前Agent的handover文件，**自动加载**而不是仅显示
- 优先级：针对当前Agent的文件 > 其他Agent的文件 > 传统摘要文件
- 自动选择最新的相关handover文件进行加载

#### 加载确认和过渡
- 简要确认已加载的文档信息
- 直接开始基于上下文的对话，无需用户额外操作
- 平滑过渡从"文件发现"到"上下文理解"

### 4. 分类显示（仅当没有自动加载时）

当没有需要自动加载的文档时，按以下格式显示：

#### 🆕 Handover 文件（待加载）
- 文件名
- 生成时间
- 文件大小
- 来源项目（从文件名提取）
- 加载命令：`@handoffs/[文件名].md`

#### 📋 传统摘要文件（待加载）
- 文件名
- 修改时间
- 文件大小
- 来源 Coach（如果能从文件内容推断）
- 加载命令：`@handoffs/[文件名].md`

#### 已加载的文件（仅供参考）
- 按时间倒序列出

### 5. 加载建议（仅当没有自动加载时）

- 如果有新文件，提供相应的 `@handoffs/[文件名].md` 加载命令
- 如果没有新文件，告知用户：当前无待加载的摘要

### 5. 显示摘要统计

- 总摘要数量
- 最新摘要时间
- 最旧摘要时间
- 总存储大小

## 实现细节

```bash
# 基础扫描逻辑（优先根目录，后备Worktree）
if [ -d "./handoffs" ]; then
    echo "扫描目录: ./handoffs/"
    # 自动加载逻辑：检查是否有针对当前Agent的handover文件
    CURRENT_AGENT=$(echo "$ROLE" | tr '[:upper:]' '[:lower:]' 2>/dev/null || echo "unknown")
    HANDOVER_FILE=$(ls -t ./handoffs/*-to-$CURRENT_AGENT.md 2>/dev/null | head -1)

    if [ -n "$HANDOVER_FILE" ] && [ -f "$HANDOVER_FILE" ]; then
        echo "🔍 发现针对你的handover文档，正在自动加载..."
        echo "📄 文件：$(basename "$HANDOVER_FILE")"
        echo "📅 时间：$(stat -f "%Sm" -t "%Y-%m-%d %H:%M" "$HANDOVER_FILE" 2>/dev/null || stat -c "%y" "$HANDOVER_FILE" | cut -d' ' -f1-2)"
        echo "📊 大小：$(du -h "$HANDOVER_FILE" | cut -f1)"
        echo ""
        echo "✅ Handover文档已自动加载，现在开始基于上下文对话："
        echo ""
        # 输出文件内容
        cat "$HANDOVER_FILE"
        echo ""
        echo "---"
        echo "🤝 上下文交接完成。基于以上文档，我已经了解了当前状况。"
    else
        # 没有自动加载文件时，显示所有文件列表
        ls -lh ./handoffs/*.md 2>/dev/null | sort -k6,7 -r
    fi
elif [ -d "../../handoffs" ]; then
    ls -lh ../../handoffs/*.md 2>/dev/null | sort -k6,7 -r
fi

# Handover 模式识别（备用）
# 匹配格式：[项目名]-[timestamp]-to-[target].md
ls ./handoffs/*-to-*.md 2>/dev/null

# 传统模式识别（可选）
# 新文件 = 修改时间在过去 24 小时内的文件
find ./handoffs -name "*.md" -type f -mtime -1 2>/dev/null
```

## 🚀 核心优化要点

### 自动检测逻辑
1. **识别当前Agent**：通过环境变量或参数确定当前AI角色
2. **查找相关文档**：扫描`-to-[当前agent].md`格式的文件
3. **按时间排序**：选择最新的相关handover文档
4. **自动加载内容**：直接输出文件内容并开始对话

### 用户体验改进
- **零操作交接**：用户无需手动复制@handoffs命令
- **信息确认**：简要显示加载的文档信息
- **无缝过渡**：从文件发现平滑过渡到基于上下文的对话
- **向下兼容**：当没有自动加载文档时，仍显示文件列表供手动选择

## 成功标准

- [ ] 正确扫描 handoffs 目录
- [ ] 自动识别当前Agent并加载相关handover文档
- [ ] 实现零操作交接体验
- [ ] 平滑过渡到基于上下文的对话
- [ ] 向下兼容传统模式
- [ ] 提供清晰的加载确认信息

## 注意事项

- **双模式支持**：自动检测 Handover 模式和传统模式，向后兼容
- **路径优先级**：优先扫描 `./handoffs/`，后备 `../../handoffs/`
- **Agent 感知**：Handover 模式下能识别当前 Agent，只显示相关文件
- **轻量级设计**：不维护状态文件（零 SOP 污染）
- **用户控制**：用户主动触发，不自动打扰
- **时区处理**：使用本地时区显示时间
- **可读性优先**：优先清晰的输出格式，而非完全自动化

## 使用场景

1. **Agent 间交接**：在 Echo 中运行 `/check-handoffs`，查看 Claude Code 生成的 handover
2. **跨 Worktree 协作**：在 Coach B 中运行 `/check-handoffs`，查看 Coach A 生成的新摘要
3. **定期检查**：在新会话启动时运行，快速了解是否有待加载的上下文
4. **特定 Agent 检查**：使用 `/check-handoffs echo` 只查看给 Echo 的 handover
5. **摘要审计**：查看所有历史摘要，了解协作记录

## 示例输出

```
=== Handoffs 检查结果 ===

🆕 待加载的新文件（过去 24 小时内生成）
────────────────────────
✅ 2025-11-11_phase_2_test_simple.md
   生成时间：2025-11-11 17:38 UTC+8
   大小：6.2K
   来源：Coach B (学习快跑教练)
   加载命令：@../../handoffs/2025-11-11_phase_2_test_simple.md

📋 已加载的摘要（供参考，按时间倒序）
────────────────────────
2025-11-11_mva_7_plan.md (7.2K, 17:13)

📊 统计信息
────────────────────────
总摘要数：2
最新摘要：2025-11-11 17:38
最旧摘要：2025-11-11 17:13
总大小：13.4K
```

## 何时使用

- ✅ 在新的 Claude Code 会话启动时运行一次
- ✅ 从另一个 Worktree 回来时检查是否有新摘要
- ✅ 在 `/handoff` 执行后，检查文件是否正确生成
- ❌ 不需要频繁运行（数据不会实时变化）

## 与 /handoff 的配合

| 命令 | 场景 | 方向 |
|------|------|------|
| `/handoff` | 生成摘要 | 当前 Worktree → handoffs 目录 |
| `/check-handoffs` | 检查摘要 | handoffs 目录 → 当前 Worktree |

两个命令构成了**跨 Worktree 上下文移交的完整流程**。

---

## 🆕 Handover 模式输出示例

```
=== Handoffs 检查结果（Handover 模式）===

🆕 Handover 文件（待加载给你的）
────────────────────────
✅ coach_os_sprint-2025-12-09-130649-to-echo.md
   生成时间：2025-12-09 13:06 UTC+8
   大小：4.8K
   来源项目：coach_os_sprint
   加载命令：@handoffs/coach_os_sprint-2025-12-09-130649-to-echo.md

📋 其他 Handover 文件（非目标）
────────────────────────
other_project-2025-12-08-150000-to-claude.md (3.2K, 昨天)

📊 统计信息
────────────────────────
总 Handover 数：2
最新 Handover：2025-12-09 13:06
最旧 Handover：2025-12-08 15:00
总大小：8.0K
```

**Handover 模式特点**：
- 自动识别当前 Agent，只显示相关的 handover 文件
- 文件名格式：`[项目名]-[timestamp]-to-[target].md`
- 加载路径统一为：`@handoffs/[文件名].md`
- 支持 Agent 过滤：`/check-handoffs echo` 只显示给 Echo 的文件
