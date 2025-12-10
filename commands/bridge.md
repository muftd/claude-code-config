# Auto Bridge Load Command

自动检测并加载针对当前Agent的Agent Bridge Protocol文件。

## 功能
检测当前Agent身份，自动查找并加载最新的相关桥接文件。

## 触发方式
- `/bridge` - 自动检测当前Agent并加载相关桥接文件
- `/bridge [agent]` - 指定目标Agent加载桥接文件

## 检测逻辑
1. 识别当前Agent身份（Echo、ChatGPT、YouMind等）
2. 在`handoffs/`目录中查找最新的针对该Agent的桥接文件
3. 自动加载文件内容

## 使用示例
- `/bridge` - 自动加载当前Agent的相关桥接文件
- `/bridge echo` - 加载针对Echo的桥接文件
- `/bridge chatgpt` - 加载针对ChatGPT的桥接文件

## 实现逻辑
```bash
#!/bin/bash

# 检测当前Agent身份
CURRENT_AGENT=$(echo "$ROLE" | tr '[:upper:]' '[:lower:]' 2>/dev/null || echo "unknown")

# 根据参数或自动检测确定目标Agent
TARGET_AGENT=${1:-$CURRENT_AGENT}

# 查找最新的桥接文件（更精确的逻辑）
BRIDGE_FILE=$(find "/Users/wangmu/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault/handoffs/" -name "*to-$TARGET_AGENT*.md" -type f -exec ls -t {} + 2>/dev/null | head -1)

if [ -n "$BRIDGE_FILE" ] && [ -f "$BRIDGE_FILE" ]; then
    echo "🔍 发现Agent Bridge Protocol文件，正在自动加载..."
    echo ""
    cat "$BRIDGE_FILE"
    echo ""
    echo "📄 文件位置: $BRIDGE_FILE"
else
    echo "❌ 未找到针对 $TARGET_AGENT 的Agent Bridge Protocol文件"
    echo "💡 桥接文件通常保存在: handoffs/*to-$TARGET_AGENT*.md"
fi
```