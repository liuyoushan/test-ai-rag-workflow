
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
│  ┌──────────────────┐    ┌──────────────────┐                  │
│  │  RAG 知识库      │    │   Skill 技能库    │                  │
│  │  (本仓库提供)    │    │  (本仓库提供)     │                  │
│  │                  │    │                  │                  │
│  │  • Golden Cases  │    │  • 用例生成       │                  │
│  │  • Bad Cases     │    │  • 日志分析       │                  │
│  │  • 测试规范      │    │  • (后续扩展)     │                  │
│  └────────┬─────────┘    └────────┬─────────┘                  │
│           │                       │                            │
│           └──────────┬────────────┘                            │
│                      ▼                                         │
│              ┌──────────────┐                                  │
│              │  Workflow    │                                  │
│              │  (本仓库提供)│                                  │
│              │              │                                  │
│              │ • 用例生成   │                                  │
│              │ → 执行 →     │                                  │
│              │ 日志分析     │                                  │
│              └──────────────┘                                  │
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

## 后续规划

### 短期目标
- [x] 完成知识库搭建（Golden Case + Bad Case）
- [x] 配置核心Skill（用例生成、日志分析）
- [x] 设计工作流文档
- [ ] 在AnythingLLM中完成实际配置

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
