# 选题助手开发日志

## 2026-04-19 v3.1 重大更新

### 概述
本次更新将选题助手从v3.0升级到v3.1，核心改进包括：
1. 新增三种定位入口方式，降低用户使用门槛
2. 融合6大选题理论到选题生成流程
3. 简化选题输出格式，聚焦用户核心需求
4. 优化模型选择和API配置

---

## 一、用户反馈与问题分析

### 1.1 定位门槛问题
**用户反馈**：用户可能无法清晰回答定位问题

**分析**：
- 原v3.0版本只有完整诊断一种入口（11个问题）
- 对于定位不清晰的用户，门槛过高
- 需要提供多种入口，适应不同清晰度的用户

**解决方案**：设计三种定位入口
- **完整诊断**：11个问题，适合定位清晰的用户
- **对标分析**：通过分析对标账号推导定位，适合有参考对象的用户
- **快速启动**：3个核心问题，适合想快速体验的用户

### 1.2 对标分析优化
**用户反馈**：对标分析中需要让用户描述"喜欢这个账号的什么"

**分析**：
- 从自由描述中可以提取用户偏好
- 结合结构化选项，更准确理解用户需求

**实现**：
- 添加自由文本输入框
- 添加结构化偏好选项（内容选题/语言风格/人设感觉）
- 实现`extractBenchmarkPreferences()`函数提取偏好

### 1.3 选题输出复杂问题
**用户反馈**：选题输出太复杂，包含太多分析字段

**用户真正关心**：
1. 这个选题是什么（标题）
2. 为什么会爆（爆款原因）
3. 怎么讲更好（内容框架）

**原输出格式**（v3.0）：
```
### 选题1：[标题]
**基本信息**
- 象限归属
- 人群指向
- 应用理论
**选题分析**
- 场景还原
- 痛点分析
- 情绪价值
**产品连接**
- 连接方式
- 转化路径
```

**新输出格式**（v3.1）：
```
### 选题1：[标题]
🔥 为什么会爆
[用了什么元素/角度/逻辑，戳中什么痛点，引发什么情绪]

📝 怎么讲更好
- 开头：[如何吸引注意力]
- 中间：[内容框架，3-5点]
- 结尾：[如何引导行动/连接产品]
```

### 1.4 网页版vs对话版选题质量差异
**用户反馈**：网页上的选题和对话的选题相差很远

**原因分析**：
- 网页版使用 `glm-4-flash` 模型（快速版）
- 对话版使用更强大的模型
- Flash版响应快但质量略低

**尝试的解决方案**：
1. 升级到 `glm-4` 标准版 → 可能存在API key权限问题
2. 改进prompt结构，添加更具体的指导
3. 添加选题自检清单

**最终方案**：保持 `glm-4-flash`，通过优化prompt弥补质量差距

### 1.5 加载状态问题
**用户反馈**：一直转圈"正在基于你的定位生成选题"，不需要这么复杂

**解决**：
- 简化加载提示为"AI正在思考..."
- 添加更好的错误处理和调试信息

---

## 二、6大选题理论整合

### 2.1 理论来源
用户提供以下文档作为理论基础：

1. **06.爆款选题基本逻辑.doc**
   - 两大原则：颠覆认知、冲突矛盾

2. **07.选题模版和选题裂变.doc**
   - 模板裂变方法论

3. **08.正确理解选题意图 快速输出模版结构.doc**
   - 选题意图理解流程

4. **第4节课：账号策划-8大「爆款元素」.doc**
   - 8大爆款元素

5. **009_方向篇：创意选题的新角度.doc**
   - 14种创意角度

6. **SKILL.md（原有）**
   - 选题四象限方法论

### 2.2 理论映射关系

```
选题四象限（核心方法论）
├── Q1: 泛人群+泛选题 = 流量型
├── Q2: 泛人群+垂直选题 = 权威型
├── Q3: 垂直人群+泛选题 = 信任型
└── Q4: 垂直人群+垂直选题 = 转化型
        ↓
爆款选题逻辑（增强吸引力）
├── 颠覆认知：反常识、反惯性、反预设、反刻板、反权威、反经验、反情绪
└── 冲突矛盾：强弱冲突、新旧冲突、正负冲突
        ↓
8大爆款元素（注入传播力）
├── 人群元素：锁定特定人群
├── 成本元素：时间/金钱/精力成本
├── 奇葩元素：反常识、罕见
├── 负面元素：负面情绪、恐惧
├── 反向元素：反常规操作
├── 怀旧元素：回忆、情怀
├── 激素元素：情绪刺激
└── 顶级元素：顶级资源、顶级结果
        ↓
14种创意角度（提供切入点）
多维解法、跨学科、前沿方法、认知盲区、矛盾点、反向思考、
极端假设、时间维度、特殊情境、小题大做、深度思考、
个人化角度、多视角、叙述语气
        ↓
选题模板裂变（批量生产）
关键词提取 → 公式总结 → 反向验证 → 批量生成
        ↓
选题意图理解（快速成稿）
分解选题 → 构建结构 → 完善开头结尾 → 完整输出
```

### 2.3 在Prompt中的体现

**选题生成Prompt结构**：
```
【用户定位】
- 身份/方向、目标人群、核心产品、当前阶段、选题目标

【用户输入素材】

【生成要求】
1. 选题标题：必须符合爆款格式
   - 痛点式：为什么XX总是XX？
   - 数字式：3个方法/5个信号
   - 对比式：同样是XX，为什么XX？
   - 悬念式：意想不到，XX才是关键
   - 结果式：30天，我从XX到XX
   - 情绪式：35岁我才明白...

2. 爆款原因分析
   - 用了什么爆款元素
   - 用了什么创意角度
   - 戳中什么痛点，引发什么情绪

3. 怎么讲更好
   - 开头：如何吸引注意力
   - 中间：内容结构，3-5个要点
   - 结尾：如何引导行动

【选题自检】
- 人群贴近度
- 场景真实性
- 产品连接度
- 爆款格式符合度
```

---

## 三、技术实现细节

### 3.1 三种定位入口实现

#### 完整诊断（11问）
```javascript
const fullDiagnosisQuestions = [
    { id: 'profession', question: '你的职业/专业身份是什么？' },
    { id: 'background', question: '你有什么独特的背景/故事/经历？' },
    { id: 'differentiation', question: '你和同行最大的不同是什么？' },
    { id: 'audience', question: '你的用户具体是谁？' },
    { id: 'product', question: '你的核心产品/服务是什么？' },
    { id: 'productValue', question: '它解决了用户的什么问题？' },
    { id: 'stage', question: '你的IP目前处于哪个阶段？', isChoice: true },
    { id: 'goal', question: '现阶段你做内容的首要目标是什么？', isChoice: true },
    { id: 'platform', question: '你主要在哪个平台做内容？', isChoice: true },
    { id: 'format', question: '你更擅长哪种内容形式？', isChoice: true },
    { id: 'style', question: '你的内容风格是什么？', isChoice: true }
];
```

#### 对标分析
```javascript
function extractBenchmarkPreferences() {
    const preferences = [];
    // 从自由文本提取
    if (prefText.includes('接地气')) preferences.push('语言接地气');
    // 从选项提取
    if (document.getElementById(`topic1${i}`)?.checked) preferences.push('选题贴近生活');
    return [...new Set(preferences)]; // 去重
}
```

#### 快速启动
```javascript
// 3个核心问题
1. 你是做什么的？
2. 内容主要给谁看？（职场人/学生党/宝妈/创业者）
3. 大概有多少粉丝？（0-1万/1-10万/10万+）
```

### 3.2 状态管理

**localStorage存储结构**：
```javascript
{
    positioning: {
        method: 'full' | 'benchmark' | 'quick',
        profession: string,
        audience: string,
        product: string,
        stage: 'start' | 'growth' | 'mature',
        goal: 'traffic' | 'trust' | 'conversion',
        // ... 其他定位信息
    },
    method: string,
    topics: Array<{input, result, timestamp}>,
    lastUpdate: ISOString,
    topicCount: number
}
```

### 3.3 模型选择

**智谱AI模型对比**：
| 模型 | 速度 | 质量 | 价格 | 使用场景 |
|------|------|------|------|----------|
| glm-4-flash | 快 | 中 | 0.1元/百万tokens | 当前使用 |
| glm-4 | 中 | 高 | 0.5元/百万tokens | 尝试升级（权限问题） |
| glm-4-plus | 慢 | 最高 | 1元/百万tokens | 备选升级 |
| glm-4-air | 快 | 中高 | 1元/百万tokens | 轻量级 |

**最终选择**：保持 `glm-4-flash`，通过优化prompt提升质量

### 3.4 输出格式化

```javascript
function formatResult(text) {
    return text
        .replace(/### 选题(\d+)：(.*)/g,
            '<div class="topic-header"><span class="topic-num">选题 $1</span><span class="topic-title">$2</span></div>')
        .replace(/🔥 为什么会爆/g, '<div class="section-title">🔥 为什么会爆</div>')
        .replace(/📝 怎么讲更好/g, '<div class="section-title">📝 怎么讲更好</div>')
        .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
        .replace(/---/g, '<hr class="topic-divider">');
}
```

---

## 四、UI/UX改进

### 4.1 定位历史显示

**功能**：
- 显示上次使用的定位方式
- 显示定位摘要（身份/人群/产品/阶段）
- 显示更新时间和选题数量
- 提供快速操作按钮

**实现**：
```javascript
function loadPositioningHistory() {
    const saved = localStorage.getItem('topic_config_v31');
    if (saved) {
        const config = JSON.parse(saved);
        // 显示历史信息
        document.getElementById('lastPositioningHistory').classList.remove('hidden');
    }
}
```

### 4.2 视觉优化

**新增CSS类**：
```css
.topic-header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 15px 20px;
    border-radius: 10px;
    display: flex;
    align-items: center;
}

.section-title {
    font-size: 15px;
    font-weight: 600;
    color: #667eea;
    margin: 20px 0 12px 0;
}
```

---

## 五、部署信息

### 5.1 GitHub仓库
- **仓库地址**：https://github.com/luckyluucc/xuanti
- **GitHub Pages**：https://luckyluucc.github.io/xuanti/

### 5.2 版本历史

| 版本 | 日期 | 主要变更 |
|------|------|----------|
| v1.0 | - | Initial commit |
| v3.0 | 2026-03 | Two-step diagnosis workflow |
| v3.1 | 2026-04-19 | Three positioning methods, 6 theories integration |

### 5.3 提交记录

```
8636356 Fix loading state and error handling
5f108a6 Enhance topic generation prompt with more specific guidance
84c9c4c Simplify topic output format
d8ba327 Upgrade to glm-4 model for better topic quality
5692368 Add positioning history feature
7fc8821 Upgrade to v3.1: Three positioning methods + 6 theories integration
```

---

## 六、待优化项

### 6.1 功能增强
- [ ] 支持导出选题到文档
- [ ] 支持保存历史选题记录
- [ ] 支持自定义知识库上传
- [ ] 支持多用户配置切换
- [ ] 角度启发功能
- [ ] 模板裂变功能
- [ ] 文案生成功能

### 6.2 技术优化
- [ ] 探索模型升级方案（API key权限）
- [ ] 添加选题收藏功能
- [ ] 优化prompt以提升选题质量
- [ ] 添加选题评分/排序功能

### 6.3 用户体验
- [ ] 添加选题使用反馈
- [ ] 添加选题效果追踪
- [ ] 优化移动端适配
- [ ] 添加快捷键支持

---

## 七、关键决策记录

### 决策1：为什么选择三种定位入口？
**问题**：用户可能无法清晰回答定位问题
**方案**：提供完整诊断、对标分析、快速启动三种方式
**理由**：适应不同清晰度的用户，降低使用门槛

### 决策2：为什么简化选题输出？
**问题**：原输出包含太多分析字段，用户不关心
**方案**：聚焦于"是什么、为什么爆、怎么讲"三个核心问题
**理由**：用户最关心的是可执行的选题建议

### 决策3：为什么保持glm-4-flash模型？
**问题**：升级到glm-4可能有API权限问题
**方案**：保持flash模型，通过优化prompt提升质量
**理由**：确保稳定可用，响应速度快

### 决策4：为什么添加定位历史？
**问题**：用户每次使用需要重新定位
**方案**：保存上次定位，提供快速恢复功能
**理由**：提升复用效率，改善用户体验

---

## 八、用户反馈与迭代方向

### 当前用户反馈
1. 选题输出格式更清晰了 ✅
2. 加载状态简化了 ✅
3. 定位历史很方便 ✅

### 待观察反馈
1. 选题质量是否满足需求
2. 哪种定位入口最受欢迎
3. 爆款原因分析是否准确

### 迭代方向
1. 根据用户反馈持续优化prompt
2. 可能考虑模型升级方案
3. 添加更多辅助功能（角度启发、模板裂变）

---

## 九、文档维护

### 相关文档
- `README.md`：项目说明和使用指南
- `SKILL.md`：选题助手的Skill定义（v3.0）
- `CHANGELOG.md`：本文档，开发日志

### 理论文档（用户提供）
1. `06.爆款选题基本逻辑.doc`
2. `07.选题模版和选题裂变.doc`
3. `08.正确理解选题意图 快速输出模版结构.doc`
4. `第4节课：账号策划-8大「爆款元素」.doc`
5. `009_方向篇：创意选题的新角度.doc`

---

*最后更新：2026-04-19*
