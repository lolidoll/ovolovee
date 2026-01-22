# 语音条功能使用指南

## 功能概述

语音条是一个模拟真实社交软件（如QQ）语音消息功能的模块。用户和AI都可以发送语音条，语音条支持文字展示和转录功能。

## 功能特性

### 1. 用户发送语音条
- **操作步骤**：
  1. 打开任意对话
  2. 点击工具栏中的**语音条按钮**（🎙️ 图标）
  3. 在弹出的黑白简约风弹窗中输入文字内容
  4. 可以使用 `Ctrl+Enter` 快速发送，或点击"发送"按钮
  5. 语音条显示在对话页面，包含波形动画

### 2. 点击查看语音转文字
- **操作步骤**：
  1. 点击已发送或接收的语音条
  2. 弹出语音转文字显示框
  3. 可以复制文字内容
  4. 再次点击隐藏转文字显示

### 3. AI发送语音条
- AI可以通过 `VoiceMessageModule.sendAIVoiceMessage()` 方法发送语音条
- 语音条样式与用户发送的相同，支持同样的转文字功能

## 代码结构

### 文件组织

```
voice-message.js      - 语音条功能模块（子JS文件）
index.html           - HTML引入voice-message.js
style.css            - 语音条样式
app.js              - 在renderChatMessages中集成语音条渲染
```

### 核心模块 (VoiceMessageModule)

VoiceMessageModule 是一个自执行函数（IIFE），提供以下公开接口：

#### 公开方法

```javascript
// 初始化语音条功能
VoiceMessageModule.init()

// 打开语音条输入弹窗
VoiceMessageModule.openVoiceModal()

// 关闭语音条输入弹窗
VoiceMessageModule.closeVoiceModal()

// 发送用户语音消息
VoiceMessageModule.sendVoiceMessage()

// AI发送语音消息到指定对话
VoiceMessageModule.sendAIVoiceMessage(conversationId, text)

// 显示语音转文字内容
VoiceMessageModule.showVoiceTranscript(text, msgElement)

// 获取指定消息的语音内容
VoiceMessageModule.getVoiceContent(messageId)

// 检查消息是否为语音类型
VoiceMessageModule.isVoiceMessage(message)

// 获取对话中的所有语音消息
VoiceMessageModule.getVoiceMessagesForConversation(convId)

// 导出语音条的完整转录记录
VoiceMessageModule.exportVoiceTranscripts(convId)

// 清空所有语音消息数据
VoiceMessageModule.clearVoiceMessages()
```

## 数据结构

### 语音消息对象

```javascript
{
    id: string,                    // 唯一消息ID
    conversationId: string,        // 所属对话ID
    type: 'voice',                // 消息类型固定为'voice'
    content: string,              // 语音文字内容
    sender: 'sent' | 'received',  // 'sent'=用户, 'received'=AI
    timestamp: string,            // ISO格式时间戳
    senderName: string,           // 发送者名称
    senderAvatar: string          // 发送者头像URL
}
```

## 样式设计

### 特点
- **黑白简约风**：弹窗采用白色背景、黑色文字的简约设计
- **蓝色强调**：操作按钮采用蓝色主题色
- **动画效果**：语音条显示波形动画，转文字显示有平滑过渡
- **响应式设计**：在移动设备上自动调整大小

### 主要样式类
- `.voice-modal` - 语音条输入弹窗容器
- `.voice-bubble` - 对话中的语音气泡
- `.voice-bubble-user` - 用户发送的语音气泡（蓝色）
- `.voice-bubble-ai` - AI发送的语音气泡（灰色）
- `.voice-transcript` - 语音转文字显示框
- `.voice-waveform` - 波形动画

## 集成说明

### 与app.js的集成

1. **消息渲染**（renderChatMessages 函数）
   - 添加了对 `type === 'voice'` 的判断
   - 在 bubble 中渲染语音气泡
   - 为语音气泡绑定点击事件

2. **点击事件处理**
   - 语音气泡点击时调用 `VoiceMessageModule.showVoiceTranscript()`
   - 显示语音文字转录

3. **存储兼容性**
   - 语音消息与普通消息一同存储
   - `saveToStorage()` 自动保存语音消息数据

### 与app.js通信

```javascript
// 获取当前对话ID
const convId = AppState.currentChat

// 保存到本地存储
saveToStorage()

// 重新渲染消息
renderChatMessages()

// 调用AI API（可选）
callApiWithConversation(messageObj)
```

## 使用示例

### 示例1：用户发送语音条

```javascript
// 用户点击语音按钮后自动触发
VoiceMessageModule.openVoiceModal()

// 用户输入内容并点击发送按钮
// VoiceMessageModule.sendVoiceMessage() 自动被调用
```

### 示例2：AI发送语音条回复

```javascript
// 在AI回复函数中调用
const conversationId = AppState.currentChat
const voiceText = '这是AI的语音回复内容'

VoiceMessageModule.sendAIVoiceMessage(conversationId, voiceText)
```

### 示例3：获取对话中的所有语音消息

```javascript
const convId = AppState.currentChat
const voiceMessages = VoiceMessageModule.getVoiceMessagesForConversation(convId)

console.log(voiceMessages) // 返回数组
```

### 示例4：导出语音转录

```javascript
const convId = AppState.currentChat
const transcripts = VoiceMessageModule.exportVoiceTranscripts(convId)

// 返回格式：
// [
//   {
//     id: 'msg_123',
//     sender: 'sent',
//     senderName: '用户',
//     timestamp: '2026-01-21T...',
//     voiceText: '这是我说的话'
//   },
//   ...
// ]
```

## AI集成建议

如果需要让AI能够识别和回复语音条：

1. **在发送给AI的消息中标记**
   ```javascript
   const messageToAI = {
       content: `[语音条] 用户说的内容`,
       isVoiceMessage: true,
       voiceContent: '用户说的内容'
   }
   ```

2. **AI响应中发送语音条**
   ```javascript
   // 在AI回复处理函数中
   VoiceMessageModule.sendAIVoiceMessage(
       AppState.currentChat,
       aiResponseText
   )
   ```

3. **在对话上下文中保留语音标记**
   - 这样AI能够知道某条消息是通过语音条发送的
   - AI可以调整回复风格（例如，对语音条回复也用语音条）

## 浏览器兼容性

- ✅ Chrome/Edge (推荐)
- ✅ Firefox
- ✅ Safari
- ✅ 移动浏览器

## 已知限制

1. 语音条显示的时长固定为"1秒"（可修改）
2. 实际音频文件不支持（仅文字模拟）
3. 语音条不支持语音识别（由用户手动输入）

## 扩展功能建议

### 可能的扩展
1. **真实音频支持** - 集成Web Audio API
2. **语音识别** - 使用Web Speech API
3. **语音合成** - 使用TTS实现文字转语音
4. **时长计算** - 根据内容长度动态计算语音时长
5. **播放动画** - 播放时显示实时波形
6. **速度调节** - 支持调节回放速度

## 常见问题

### Q: 语音条消息保存到哪里？
A: 语音条消息与所有其他消息一样，保存在 `AppState.messages[convId]` 中，通过 `saveToStorage()` 持久化。

### Q: 语音条支持多媒体内容吗？
A: 目前只支持文字内容。语音条的"audio"是模拟效果（波形动画），实际内容是用户输入的文字。

### Q: 如何自定义语音条样式？
A: 修改 `style.css` 中的以下样式类：
   - `.voice-bubble-user` - 修改用户语音气泡样式
   - `.voice-bubble-ai` - 修改AI语音气泡样式
   - `.voice-waveform` - 修改波形动画

### Q: 语音条可以转发吗？
A: 目前转发功能需要在app.js的转发逻辑中额外处理，可以检查 `msg.type === 'voice'` 来特殊处理。

## 调试技巧

### 查看语音消息数据
```javascript
// 在浏览器控制台执行
VoiceMessageModule.getVoiceMessagesForConversation(AppState.currentChat)
```

### 清空所有语音消息
```javascript
VoiceMessageModule.clearVoiceMessages()
```

### 检查消息类型
```javascript
const msg = AppState.messages[AppState.currentChat][0]
VoiceMessageModule.isVoiceMessage(msg) // true or false
```

## 后续维护

- 定期检查样式在不同设备上的显示效果
- 监控用户反馈并优化交互体验
- 可根据需求添加更多语音条功能（如支持实际音频、语音识别等）
