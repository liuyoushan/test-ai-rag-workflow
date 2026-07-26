
# Test AI RAG Workflow

> AI辅助测试工具配置档案库 - 为Web3自动化测试框架赋能

## 项目简介

本仓库是一个AI辅助测试工具的配置档案库，旨在利用第三方AI软件（AnythingLLM）为现有测试框架提供智能辅助能力，提高测试效率和质量。

**核心价值**：
- 🚀 快速生成标准化测试用例
- 🔍 智能分析测试异常日志
- 📚 沉淀测试知识和故障案例
- 💼 面试演示完整AI辅助测试流程

## 技术架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        AI 外壳 (AnythingLLM)                    │
│                                                                 │
│  ┌──────────────────┐    ┌──────────────────┐    ┌───────────┐  │
│  │  1:knowledge_base│    │  2: skills       │    │3:workflow │  │
│  │  (RAG知识库)     │    │  (业务执行Skill) │    │(流程串联) │  │
│  │                  │    │                  │    │           │  │
│  │  • Golden Cases  │    │  • 用例生成Skill │    │• 用例生成 │  │
│  │  • Bad Cases     │    │  • 日志分析Skill │    │→日志分析  │  │
│  │  • 测试规范      │    │  • (后续扩展)    │    │           │  │
│  └────────┬─────────┘    └────────┬─────────┘    └─────┬─────┘  │
│           │                       │                    │        │
│           └──────────┬────────────┴────────────────────┘        │
│                      ▼                                         │
│              ┌──────────────────────┐                          │
│              │ Web3_Contract_Test   │                          │
│              │ _Automation_Workflow │                          │
│              │ (自动执行链路)        │                          │
│              └──────────────────────┘                          │
└─────────────────────────┬───────────────────────────────────────┘
                          │ 业务数据流
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                     测试框架 (ape-demo)                         │
│                                                                 │
│  ┌──────────────────┐    ┌──────────────────┐                  │
│  │  自动化测试用例   │    │  性能测试框架     │                  │
│  │  (pytest/ape)    │    │  (blockchain-    │                  │
│  │                  │    │   perf-test)     │                  │
│  │  • ERC20测试     │    │  • 压测执行       │                  │
│  │  • DEX测试       │    │  • 监控告警       │                  │
│  │  • 清算测试      │    │  • 故障排查       │                  │
│  │  • NFT测试       │    │                  │                  │
│  │  • 安全测试      │    │                  │                  │
│  └──────────────────┘    └──────────────────┘                  │
└─────────────────────────────────────────────────────────────────┘
```

### 实际运行效果

已在本地AnythingLLM中完成配置，三层架构（知识库 → Skill → Workflow）全部打通：

```
┌─────────────────────────────────────────────────────────┐
│  @Web3_Contract_Test_Automation_Workflow                │
│  粘贴你的合约ABI                                         │
├─────────────────────────────────────────────────────────┤
│  我需要先获取合约的ABI信息才能进行测试自动化流程。          │
│  请提供合约的ABI内容，以便我能够继续处理。                  │
├─────────────────────────────────────────────────────────┤
│  [{"inputs":[{"internalType":"uint256","name":"totalSupply",...}]}]
│  [{"inputs":[{"indexed":true,"name":"from","type":"address"},...]}]
│  [{"inputs":[{"name":"to","type":"address"},...],{"name":"transfer"},...}]
│  [{"type":"bool","stateMutability":"nonpayable","type":"function"}]
├─────────────────────────────────────────────────────────┤
│  ✅ Web3_Contract_Test_Automation_Workflow completed     │
│     successfully                                         │
└─────────────────────────────────────────────────────────┘
```

## 仓库关系

| 仓库 | 定位 | 职责 |
|------|------|------|
| **ape-demo** | 干活的发动机 | 真正执行自动化测试的核心程序 |
| **test-ai-rag-workflow** | AI配置档案库 | 记录如何使用AI给发动机减负提速 |

**文件层面**：完全独立，不需要代码互相导入
**业务层面**：数据互相流通，AI依托ape项目的数据工作，AI产出供给ape项目使用

## 目录结构

```
test-ai-rag-workflow/
├── .gitignore                    # Git忽略配置
├── README.md                     # 项目说明文档
├── knowledge_base/               # AI知识库原始数据
│   ├── golden_cases/             # 标准测试用例 (golden case)
│   │   ├── complete_case_list.md         # 68个用例完整清单
│   │   ├── erc20_test_cases.md           # ERC20测试用例
│   │   ├── dex_swap_test_cases.md        # DEX测试用例
│   │   ├── liquidation_test_cases.md     # 清算测试用例
│   │   ├── nft_test_cases.md             # NFT测试用例
│   │   ├── security_test_cases.md        # 安全测试用例
│   │   ├── contract_custom_test_cases.md # 自定义合约测试用例
│   │   └── test_format_standards.md      # 测试格式标准
│   └── bad_cases/                # 故障案例库 (bad case)
│       ├── exception_scenarios.md        # 异常场景汇总
│       ├── error_patterns.md             # 错误模式库
│       ├── troubleshooting_guide.md      # 排查指南
│       └── failure_classification.md     # 故障分类索引
├── skills/                       # Skill提示词模板
│   ├── case_generation_skill.txt         # 用例生成Skill
│   └── log_analysis_skill.txt            # 日志分析Skill
└── workflows/                    # Workflow配置文档
    └── test_automation_workflow.md       # 测试自动化工作流
```

## 核心功能

### 1. 用例生成

**功能描述**：输入接口文档，AI参考知识库中的标准用例格式，自动生成高质量测试用例

**使用流程**：
```
接口文档 → AI生成用例 → 复制到ape项目 → 运行自动化测试
```

**Skill文件**：`skills/case_generation_skill.txt`

### 2. 日志分析

**功能描述**：输入测试异常日志，AI匹配故障案例库，快速定位问题根因并提供修复建议

**使用流程**：
```
测试报错 → AI分析根因 → 给出修复方案 → 验证修复
```

**Skill文件**：`skills/log_analysis_skill.txt`

### 3. 工作流串联

**功能描述**：配置自动化工作流，串联用例生成、执行、日志分析的完整流程

**工作流链路**：
```
接口文档输入 → AI产出标准化用例 → 写入ape自动化执行 → 程序抛出异常日志 → AI自动解析日志给出根因
```

**Workflow文件**：`workflows/test_automation_workflow.md`

## 数据来源声明

本仓库的知识库数据来源于以下自研测试工程：

1. **ape-demo** - Web3自动化测试框架
   - 位置：`/home/liuyoushan/ape-demo`
   - 内容：ERC20/DEX/清算/NFT/安全/V3等68个测试用例
   - 用途：提供Golden Case标准用例和Bad Case异常场景

2. **blockchain-perf-test** - 性能测试平台（后续补充）
   - 位置：待接入
   - 内容：压测报错、资金对账不一致、服务告警日志
   - 用途：提供真实故障案例

## 使用指南

### 前置准备

1. 安装第三方AI软件 **AnythingLLM**
2. 本地已有 **ape-demo** 测试框架

### 配置步骤

1. **上传知识库**：将 `knowledge_base/` 目录下的所有文件上传到AnythingLLM的RAG知识库

2. **配置Skill**：将 `skills/` 目录下的提示词粘贴到AnythingLLM的Skill配置中

3. **配置Workflow**：参考 `workflows/test_automation_workflow.md` 在AnythingLLM中配置工作流

### 日常使用

#### 场景1：新接口测试用例生成
```
1. 在AnythingLLM中输入新接口文档
2. 调用"用例生成"Skill
3. AI生成标准化测试用例
4. 将用例复制到ape项目执行
```

#### 场景2：测试异常排查
```
1. 将测试报错日志复制到AnythingLLM
2. 调用"日志分析"Skill
3. AI给出根因分析和修复建议
4. 修复代码并验证
```

### 面试演示

```
1. 打开本仓库，展示知识库结构
2. 打开本地AnythingLLM软件
3. 演示用例生成功能：输入接口文档 → 生成用例
4. 演示日志分析功能：输入报错日志 → 分析根因
5. 展示工作流自动化配置
```

## 三层架构使用示例

### Layer 1: knowledge_base（RAG知识库）

**作用**：AI回答问题时的"记忆"，只会调取这里的内部资料，不会网上乱编

**配置步骤**：
```
1. 打开AnythingLLM → 进入工作区
2. 点击"上传文件"
3. 选择 knowledge_base/ 目录下所有文件
4. 等待文件解析完成（约1-2分钟）
5. 验证：在聊天框输入"ERC20转账测试用例有哪些？"，AI应能准确回答
```

**文件说明**：
| 文件 | 用途 | AI如何使用 |
|------|------|-----------|
| `golden_cases/` | 标准测试用例 | AI参考格式生成新用例 |
| `bad_cases/` | 故障案例库 | AI匹配报错给出排查方案 |
| `test_format_standards.md` | 测试规范 | AI保证输出格式统一 |

**使用示例**：
```
用户提问："帮我生成ERC20授权测试用例"
AI检索：自动查找 knowledge_base/golden_cases/erc20_test_cases.md 中授权相关用例
AI输出：按照知识库中的标准格式，生成完整的授权测试用例文档
```

---

### Layer 2: skills（业务执行Skill）

**作用**：固定模板提示词，告诉AI"遇到什么情况该怎么做"

**当前配置方式**：两份Skill规则已写入全局系统Prompt，不管是普通聊天还是跑Workflow，AI都会遵循这份行为规范。**注：当前缺少"单独唤起单个Skill"的快捷调用入口（即`@用例生成Skill`这种方式暂不可用）**。

**配置步骤**：
```
1. 打开AnythingLLM → 进入工作区设置
2. 找到"系统Prompt"配置项
3. 将 skills/case_generation_skill.txt 和 skills/log_analysis_skill.txt 的内容合并写入
4. 保存设置
5. 验证：直接在聊天框输入"帮我生成测试用例"或"分析报错日志"，AI会自动遵循Skill规则
```

**Skill说明**：
| Skill名称 | 触发方式 | 功能 |
|-----------|----------|------|
| 用例生成Skill | 自动匹配（输入含"生成用例"、"测试用例"等关键词） | 根据接口文档生成测试用例 |
| 日志分析Skill | 自动匹配（输入含"报错"、"日志"、"排查"等关键词） | 分析报错日志给出根因 |

**使用示例1 - 用例生成**：
```
用户输入：
请根据以下ABI生成测试用例：
[{"inputs":[{"name":"to","type":"address"},{"name":"value","type":"uint256"}],
"name":"transfer","type":"function"}]

AI输出（自动遵循用例生成Skill规则）：
## case_001 正常转账测试
- 前置条件：部署合约，mint代币给用户
- 测试步骤：调用transfer(to, value)
- 预期结果：余额变更，Transfer事件触发
- 代码参考：def test_transfer(erc20_token, deployer, user1): ...
```

**使用示例2 - 日志分析**：
```
用户输入：
分析以下测试报错：
AssertionError: balance_deployer_after == balance_deployer_before - expected_transfer
Expected: 800 ether
Actual: 1000 ether

AI输出（自动遵循日志分析Skill规则）：
## 问题分析：转账未生效
- 根因：transfer调用失败但未捕获异常
- 解决方案：使用with reverts()上下文管理器
- 代码参考：with reverts(): erc20_api.transfer(user1, amount, deployer)
```

---

### Layer 3: workflow（流程串联）

**作用**：按顺序把多个Skill串起来自动执行，不用手动一次次发消息

**配置步骤**：
```
1. 打开AnythingLLM → 点击左侧"工作流"图标
2. 点击"新建工作流"
3. 命名："Web3_Contract_Test_Automation_Workflow"
4. 添加步骤：
   Step 1: 接收用户输入（合约ABI）
   Step 2: 根据系统Prompt中的用例生成规则，生成标准化测试用例
   Step 3: 根据系统Prompt中的日志分析规则，分析潜在风险
5. 保存工作流
6. 验证：在聊天框输入"@Web3_Contract_Test_Automation_Workflow"
```

**注**：Skill规则已写入全局系统Prompt，Workflow执行时AI会自动遵循规则，无需单独调用`@用例生成Skill`或`@日志分析Skill`。

**工作流链路**：
```
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│  用户输入ABI     │ → │ 用例生成Skill    │ → │ 日志分析Skill    │
│  接口文档        │    │ 生成标准化用例   │    │ 分析潜在风险     │
└──────────────────┘    └──────────────────┘    └──────────────────┘
                                                        │
                                                        ▼
                                              ┌──────────────────┐
                                              │ 输出完整测试方案 │
                                              │ + 风险提示       │
                                              └──────────────────┘
```

**使用示例**：
```
用户输入：
@Web3_Contract_Test_Automation_Workflow
请帮我处理这个合约的测试：
[{"inputs":[{"name":"spender","type":"address"},{"name":"amount","type":"uint256"}],
"name":"approve","type":"function"},
{"inputs":[{"name":"from","type":"address"},{"name":"to","type":"address"},
{"name":"amount","type":"uint256"}],"name":"transferFrom","type":"function"}]

AI自动执行：
1. → 解析ABI，识别approve和transferFrom两个函数
2. → 调用用例生成Skill，生成授权+代付转账测试用例
3. → 调用日志分析Skill，分析重入攻击、授权超限等风险
4. → 输出完整测试方案：用例文档 + pytest代码 + 安全测试建议

最终输出：
✅ Web3_Contract_Test_Automation_Workflow completed successfully

## 测试方案汇总

### 生成的测试用例
1. case_001: approve基础授权测试
2. case_002: approve授权覆盖测试
3. case_003: transferFrom代付转账测试
4. case_004: transferFrom授权不足测试
...

### 潜在风险提示
1. ⚠️ 无限授权风险：建议添加授权额度校验
2. ⚠️ 重入攻击风险：建议添加防重入锁
3. ⚠️ 零地址风险：建议添加零地址校验
...
```

---

### 三层协同工作原理

```
用户提问
    │
    ▼
┌─────────────────────────────────────────────────────┐
│              Workflow (流程调度)                     │
│   "用户要生成测试用例，先调哪个Skill？"               │
└──────────────────┬──────────────────────────────────┘
                   │ 调用
                   ▼
┌─────────────────────────────────────────────────────┐
│              Skill (业务执行)                        │
│   "收到，我来生成用例，但需要查知识库"                │
└──────────────────┬──────────────────────────────────┘
                   │ 检索
                   ▼
┌─────────────────────────────────────────────────────┐
│              knowledge_base (知识检索)               │
│   "找到ERC20测试用例模板，返回给Skill"               │
└──────────────────┬──────────────────────────────────┘
                   │ 返回
                   ▼
              AI生成答案
                   │
                   ▼
              返回给用户
```

## 后续规划

### 短期目标 ✅ 已完成
- [x] 完成知识库搭建（Golden Case + Bad Case）
- [x] 配置核心Skill（用例生成、日志分析）
- [x] 设计工作流文档
- [x] 在AnythingLLM中完成实际配置，三层架构全部打通

### 中期目标
- [ ] 增加测试数据生成Skill
- [ ] 增加测试报告生成Skill
- [ ] 接入blockchain-perf-test故障日志

### 长期目标
- [ ] 增加合约安全审计Skill
- [ ] 实现CI/CD集成
- [ ] 建立测试知识图谱

## 技术栈

- **AI平台**：AnythingLLM（第三方成品工具）
- **测试框架**：ape（Python Web3测试框架）
- **测试工具**：pytest、foundry
- **文档格式**：Markdown

## 许可证

MIT License

## 联系方式

如有问题或建议，欢迎提交Issue或PR。
