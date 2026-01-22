# 心声（Mind State）生成失败诊断与修复

## 问题描述
用户反馈：在点击心声按钮时，显示的内容为空或显示"尚未生成"。

## 根本原因分析

### 1. 系统提示词问题
- **原因**：系统提示词中包含了【MSG】和【WAIT】多消息格式说明，可能导致AI将心声分割到单独的消息中
- **影响**：心声数据可能被分割，导致提取失败

### 2. 正则表达式过于复杂
- **原因**：`extractMindStateFromText` 中的正则表达式复杂且脆弱，无法处理AI可能的多种输出格式
- **影响**：即使心声数据存在，也可能因为格式微小差异而无法提取

### 3. 调试信息不足
- **原因**：当心声提取失败时，系统没有提供足够的诊断信息
- **影响**：用户无法了解问题所在，无法自我调试

## 实施的修复方案

### 修复 1：改进系统提示词
**文件**：`app.js` 第 5856-5887 行

**改进内容**：
- 明确要求AI不要使用【MSG】多消息格式来分割心声
- 添加英文强调说明（CRITICAL REQUIREMENTS）
- 提供更清晰的格式示例
- 强调心声必须在同一个回复中，而不是分割到多条消息

```javascript
systemPrompts.push(`【重要】必须每次在回复最后添加以下格式的心声信息，不能省略、不能变更格式、不能使用多消息格式：
...
IMPORTANT REQUIREMENTS FOR 心声:
1. 心声MUST be placed at the very end of your response on a separate line
2. Do NOT split this into multiple [MSG] blocks - 心声 must be in the SAME response as your main dialogue
...
`);
```

### 修复 2：改进心声提取函数
**文件**：`app.js` 第 6193-6269 行

**改进内容**：
- 使用更灵活的多模式正则表达式
- 支持多种标签格式（中文/英文冒号）
- 更好的错误处理和日志输出
- 逐个字段提取，而不是整体匹配

```javascript
// 使用三层模式尝试提取，逐个字段处理
for (const label of fieldDef.labels) {
    const patterns = [
        // 模式1：标签：内容（到下一个标签或结尾）
        new RegExp(`${label}[：:]+\\s*([^\\n]*?)(?=\\n(?:穿搭|心情|...)|$)`, 'i'),
        // 模式2：标签：内容（到空行）
        new RegExp(`${label}[：:]+\\s*([^\\n]+)`, 'i'),
        // 模式3：标签: 内容（支持多行到下一个标签）
        new RegExp(`${label}[：:]+([\\s\\S]*?)(?=(?:穿搭|心情|...)[：:]+|$)`, 'i')
    ];
    // 尝试每个模式...
}
```

### 修复 3：改进错误处理和诊断
**文件**：`app.js` 第 6370-6445 行

**改进内容**：
- 在心声提取失败时输出详细的诊断信息
- 创建失败记录，便于调试
- 保存失败原因到 `mindStates` 数组

```javascript
if (!mindStateData) {
    console.warn('⚠️ 心声提取失败 - 可能的原因：');
    console.warn('  1. AI没有在回复末尾添加【心声】标记');
    console.warn('  2. 【心声】后面的格式不符合预期');
    console.warn('  3. 心声被分割到多条[MSG]消息中');
    console.warn('  API响应文本（前500字）:', text.substring(0, 500));
}
```

### 修复 4：改进用户界面反馈
**文件**：`app.js` 第 8370-8382 行

**改进内容**：
- 在心声窗口顶部显示警告提示
- 用不同颜色标记失败的字段
- 显示失败原因

```javascript
${isFailedState ? `<div style="padding:12px;background:#fff3cd;border-bottom:1px solid #ffc107;color:#856404;font-size:12px;">⚠️ 心声提取失败：请确保API已配置正确，且AI在回复末尾添加了完整的【心声】标记。</div>` : ''}
```

## 问题排查清单

如果心声仍然显示为空，请按以下步骤排查：

### 1. 检查浏览器控制台
打开开发者工具（F12），查看 Console 标签：
- 查找 `⚠️ 心声提取失败` 的相关日志
- 检查 `API响应文本` 是否包含 `【心声】` 标记
- 查看具体的错误信息

### 2. 检查API配置
- 确保API端点正确
- 确保API Key有效
- 确保选择的模型支持长文本输出（心声需要更多token）

### 3. 检查AI是否遵循指示
- 向角色发送一条消息
- 查看API返回的完整文本是否包含 `【心声】` 标记
- 如果没有，可能需要更强的系统提示或更好的模型

### 4. 检查心声格式
正确的格式应该是：
```
【心声】穿搭：... 心情：... 动作：... 心声：... 坏心思：... 好感度：75 好感度变化：+5 好感度原因：...
```

## 技术细节

### 心声数据结构
```javascript
{
    outfit: "description...",        // 穿搭
    mood: "description...",          // 心情
    action: "description...",        // 动作
    thought: "description...",       // 心声
    badThought: "description...",    // 坏心思
    affinity: 75,                    // 好感度（0-100）
    affinityChange: 5,               // 好感度变化
    affinityReason: "理由",          // 好感度原因
    timestamp: "2026-01-21T...",     // 时间戳
    messageId: "msg_...",            // 关联的消息ID
    failed: false,                   // 是否失败
    reason: "failure reason"         // 失败原因
}
```

### 提取流程
1. 原始API回复 → `appendSingleAssistantMessage`
2. 提取心声：`extractMindStateFromText` （在清理之前）
3. 清理回复：`cleanAIResponse` （移除【心声】标记）
4. 保存消息和心声数据
5. 刷新UI显示

## 预期改进

这些修复应该能解决大多数心声提取失败的问题。如果仍然有问题，请：

1. 检查浏览器控制台的详细错误信息
2. 检查API返回的完整文本
3. 考虑更新AI的系统提示词以增加强制性
4. 可能需要使用更强大的模型（如 GPT-4）

## 相关代码位置

- 系统提示词：`app.js` 第 5856-5887 行
- 心声提取函数：`app.js` 第 6193-6269 行
- 消息处理函数：`app.js` 第 6370-6445 行
- 心声显示UI：`app.js` 第 8315-8450 行
- 清理函数：`app.js` 第 6263-6342 行
