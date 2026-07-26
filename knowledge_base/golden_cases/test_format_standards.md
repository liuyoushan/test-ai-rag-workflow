# 测试用例编写规范

## 文档信息
- **文档类型**: 编写规范
- **适用范围**: Web3 自动化测试项目所有测试用例
- **代码框架**: pytest + ape + allure
- **提炼来源**: `/home/liuyoushan/ape-demo/tests/` 目录下所有测试代码

---

## 一、文件组织结构

### 1.1 目录结构规范

每个测试模块采用统一的四层目录结构：

```
tests/{module_name}/
├── scenarios/           # 测试场景（测试用例代码）
│   └── test_*.py
├── data/               # 测试数据
│   └── test_*.yaml
├── apis/               # API 封装层
│   └── *_api.py
├── fixtures/           # Fixture 定义
│   ├── conftest.py
│   └── *_fixtures.py
└── conftest.py         # 模块级配置
```

### 1.2 目录职责说明

| 目录 | 职责 | 说明 |
|-----|------|------|
| `scenarios/` | 测试场景代码 | 存放具体的测试用例，按功能分类组织文件 |
| `data/` | 测试数据 | 存放 YAML 格式的测试数据，与代码分离 |
| `apis/` | API 封装层 | 封装合约交互逻辑，提供易用的 API 接口 |
| `fixtures/` | Fixture 定义 | 定义测试前置条件和共享资源 |

---

## 二、命名规范

### 2.1 文件命名

| 文件类型 | 命名规范 | 示例 |
|---------|---------|------|
| 测试场景文件 | `test_{模块}_{功能}.py` | `test_erc20_transfer.py` |
| 测试数据文件 | `test_{模块}.yaml` | `test_erc20.yaml` |
| API 文件 | `{模块}_api.py` | `erc20_api.py` |
| Fixture 文件 | `{模块}_fixtures.py` | `erc20_fixtures.py` |

### 2.2 测试函数命名

**格式**: `test_{模块}_{编号}_{描述}`

```python
# 正确示例
def test_erc20_001_metadata_verification(...):
def test_dex_010_swap_tokenA_to_tokenB(...):
def test_liquidation_049_normal_workflow(...):

# 说明
# - 模块前缀：erc20 / dex / liquidation / nft / security / custom
# - 编号：三位数字，与 case_list 对应
# - 描述：下划线分隔的功能描述
```

### 2.3 变量命名

| 类型 | 规范 | 示例 |
|-----|------|------|
| 合约实例 | 小写蛇形 | `erc20_token`, `dex_router` |
| 用户账户 | user + 数字 | `deployer`, `user1`, `user2` |
| 测试数据 | 语义化命名 | `mint_amount`, `transfer_amount`, `expected_out` |
| 事件变量 | 事件名 + 后缀 | `transfer_event`, `approval_event` |

---

## 三、Allure 标注规范

### 3.1 必选装饰器

每个测试函数必须包含以下三个装饰器：

```python
@allure.title("case_001 代币基础信息校验")           # 用例标题
@allure.description("验证 MyERC20 合约的元数据")    # 用例描述
@allure.tag("ERC20", "P0", "功能测试")               # 用例标签
def test_erc20_001_metadata_verification(...):
    ...
```

### 3.2 Title 规范

**格式**: `case_{编号} {用例名称}`

```python
@allure.title("case_002 正常转账功能测试")
@allure.title("case_010 正向单池 Swap 兑换")
@allure.title("case_049 正常清算流程测试")
```

### 3.3 Description 规范

简要描述用例的核心验证点，一句话概括：

```python
@allure.description("普通地址间转账，校验余额变更、链上事件、交易状态")
@allure.description("TokenA 兑换 TokenB，校验余额、手续费、池子库存、K值守恒")
```

### 3.4 Tag 规范

**格式**: `[分类, 优先级, 测试类型, 补充标签...]`

| 标签类型 | 可选值 | 说明 |
|---------|--------|------|
| 分类 | `ERC20`, `DEX`, `Liquidation`, `NFT`, `Security`, `功能测试` | 业务分类 |
| 优先级 | `P0`, `P1` | P0 核心，P1 进阶 |
| 测试类型 | `功能测试`, `安全测试`, `边界测试`, `反向测试`, `性能测试` | 测试类型 |
| 补充标签 | `转账`, `授权`, `铸币`, `流动性`, `Swap` 等 | 具体功能点 |

**示例**:
```python
@allure.tag("ERC20", "P0", "功能测试", "转账")
@allure.tag("DEX", "P0", "功能测试", "Swap", "正向测试")
@allure.tag("Security", "P1", "安全测试", "重入攻击")
```

---

## 四、日志规范

### 4.1 日志导入

使用项目统一的 logger：

```python
from framework.core.logger import log
```

### 4.2 日志级别使用

| 级别 | 使用场景 | 示例 |
|-----|---------|------|
| `log.step` | 测试用例开始 | `log.step("case_001: 代币基础信息校验")` |
| `log.info` | 测试步骤说明 | `log.info("步骤1: 查询初始授权额度")` |
| `log.debug` | 详细数据输出 | `log.debug("转账金额: %s", transfer_amount)` |
| `log.success` | 用例通过 | `log.success("✅ case_001 测试通过")` |

### 4.3 步骤日志规范

每个测试步骤使用 `log.info` 标记，格式为 `步骤{N}: {步骤描述}`：

```python
log.info("步骤1: 记录铸造前状态")
log.info("步骤2: 执行铸造操作")
log.info("步骤3: 验证余额和总供应量变化")
log.info("步骤4: 验证 Transfer 事件")
```

### 4.4 数据日志规范

关键数据使用 `log.debug` 输出，带上下文说明：

```python
log.debug("deployer 余额: %s", format_ether(balance_deployer_before))
log.debug(f"初始授权额度 (deployer->user1): {format_ether(initial_allowance)}")
log.debug(f"Transfer 事件 - from: {transfer_event['from']}, to: {transfer_event['to']}")
```

---

## 五、断言规范

### 5.1 断言原则

1. **每个验证点独立断言**：不合并多个验证点到一个断言
2. **断言信息完整**：失败时能快速定位问题
3. **正向+反向结合**：既验证成功场景，也验证失败场景

### 5.2 断言格式

**数值比较断言**:
```python
assert balance_after == balance_before + expected_mint
assert total_supply_after > total_supply_before
assert abs(actual - expected) < tolerance
```

**事件断言**:
```python
assert transfer_event["from"] == deployer
assert transfer_event["to"] == user1
assert transfer_event["value"] == expected_transfer
```

**反向测试断言**（异常场景）:
```python
from ape import reverts

with reverts():
    erc20_api.transfer(user1, transfer_amount, sender=deployer)
```

**带消息的断言**:
```python
assert balance == expected, f"余额不符，预期: {expected}, 实际: {balance}"
```

### 5.3 断言分类

| 断言类型 | 用途 | 示例 |
|---------|------|------|
| 相等断言 | 精确匹配 | `assert a == b` |
| 范围断言 | 区间判断 | `assert a > b`, `assert a < c` |
| 包含断言 | 字符串/列表包含 | `assert "error" in str(e)` |
| 异常断言 | 预期失败 | `with reverts(): ...` |
| 容差断言 | 浮点/近似 | `assert abs(a - b) < tolerance` |

---

## 六、Fixture 使用规范

### 6.1 Fixture 层级

| 层级 | 位置 | 作用域 | 示例 |
|-----|------|--------|------|
| 全局 | `tests/conftest.py` | 所有测试 | 基础账户、公共配置 |
| 模块级 | `tests/{module}/conftest.py` | 模块内测试 | 模块通用 fixture |
| 文件级 | `tests/{module}/fixtures/*.py` | 按需引用 | 细分功能 fixture |

### 6.2 常用 Fixture 列表

| Fixture 名称 | 类型 | 说明 |
|-------------|------|------|
| `deployer` | 账户 | 部署者/管理员账户 |
| `user1` | 账户 | 普通用户1 |
| `user2` | 账户 | 普通用户2 |
| `{module}_test_data` | 数据 | 模块测试数据（从 YAML 加载） |
| `{module}_api` | API | 模块 API 封装实例 |

### 6.3 Fixture 命名规范

**格式**: `{模块}_{描述}`

```python
# 正确示例
@pytest.fixture
def erc20_api(...):
    ...

@pytest.fixture
def dex_test_data(...):
    ...

@pytest.fixture
def liquidation_env(...):
    ...
```

### 6.4 测试数据 Fixture

测试数据从 YAML 文件加载，通过 fixture 注入测试函数：

```python
# fixture 定义
@pytest.fixture
def erc20_test_data():
    with open("tests/erc20/data/test_erc20.yaml") as f:
        return yaml.safe_load(f)

# 使用
def test_xxx(erc20_test_data):
    data = erc20_test_data["case_002_transfer"]
    transfer_amount = data["transfer_amount"]
```

---

## 七、测试数据规范

### 7.1 数据格式

使用 YAML 格式，按用例组织：

```yaml
# 通用配置
common:
  token_name: "MyToken"
  token_symbol: "MTK"

# 各用例数据
case_002_transfer:
  mint_amount: "1000 ether"
  transfer_amount: "200 ether"

case_003_insufficient_transfer:
  balance: "100 ether"
  transfer_amount: "200 ether"
```

### 7.2 数据命名

**格式**: `case_{编号}_{描述}`

```yaml
case_002_transfer:           # 转账测试数据
case_010_swap_tokenA_to_tokenB:  # 兑换测试数据
case_049_normal_liquidation: # 清算测试数据
```

### 7.3 金额表示

使用带单位的字符串表示，增强可读性：

```yaml
# 推荐
mint_amount: "1000 ether"
transfer_amount: "200 ether"

# 不推荐（纯数字，含义不明确）
mint_amount: 1000000000000000000000
```

---

## 八、API 封装规范

### 8.1 API 层职责

API 层封装合约交互，提供语义化的方法：

```python
class ERC20API:
    def __init__(self, contract):
        self.contract = contract
    
    def get_name(self):
        return self.contract.name()
    
    def get_balance(self, account):
        return self.contract.balanceOf(account)
    
    def transfer(self, to, amount, sender):
        return self.contract.transfer(to, amount, sender=sender)
    
    def decode_transfer_event(self, tx):
        ...
```

### 8.2 API 方法命名

| 前缀 | 用途 | 示例 |
|-----|------|------|
| `get_` | 读取数据 | `get_name()`, `get_balance()`, `get_total_supply()` |
| 动词 | 写入操作 | `transfer()`, `approve()`, `mint()`, `burn()` |
| `decode_` | 事件解码 | `decode_transfer_event()`, `decode_approval_event()` |

---

## 九、测试函数结构规范

### 9.1 标准结构

每个测试函数应包含以下部分：

```python
@allure.title("case_xxx 用例标题")
@allure.description("用例简要描述")
@allure.tag("分类", "优先级", "类型")
def test_xxx(fixture1, fixture2, ...):
    """
    case_xxx 用例标题
    
    用例详细说明：
    - 验证点1
    - 验证点2
    - 验证点3
    """
    log.step("case_xxx: 用例标题")
    
    # 1. 准备测试数据
    log.info("步骤1: 准备测试数据")
    ...
    
    # 2. 执行操作
    log.info("步骤2: 执行操作")
    ...
    
    # 3. 验证结果
    log.info("步骤3: 验证结果")
    assert ...
    
    log.success("✅ case_xxx 测试通过")
```

### 9.2 文档字符串规范

函数 docstring 包含：
1. 用例编号和标题（第一行）
2. 空行
3. 详细说明（验证点列表）

---

## 十、异常处理规范

### 10.1 预期异常

使用 `ape.reverts` 或 try-except 处理预期异常：

```python
# 方式1：使用 reverts 上下文（推荐）
from ape import reverts

with reverts():
    erc20_api.transfer(user1, amount, sender=user1)

# 方式2：使用 try-except（需要检查异常内容）
try:
    erc20_api.transfer(user1, amount, sender=user1)
    assert False, "应该 revert"
except Exception as e:
    assert "Missing required role" in str(e)
```

### 10.2 异常断言

验证异常信息包含特定内容：

```python
assert "insufficient" in str(e).lower() or "amount" in str(e).lower()
assert "Missing required role" in str(e)
```

---

## 十一、工具函数使用规范

### 11.1 格式转换

使用 `framework.core.formatters` 中的工具函数：

```python
from framework.core.formatters import format_ether, parse_ether

# 解析字符串为 wei
amount_wei = parse_ether("100 ether")

# 格式化 wei 为可读字符串
amount_str = format_ether(balance)
```

### 11.2 可用工具

| 函数 | 用途 | 示例 |
|-----|------|------|
| `format_ether(wei_value)` | wei 转 ether 字符串 | `format_ether(balance)` |
| `parse_ether(ether_str)` | ether 字符串转 wei | `parse_ether("100 ether")` |

---

## 十二、最佳实践

### 12.1 测试隔离

- 每个测试用例独立运行，不依赖其他用例的状态
- 使用 fixture 准备测试环境，确保每个用例有干净的起始状态

### 12.2 数据驱动

- 测试数据与代码分离，存放在 YAML 文件中
- 相同逻辑不同参数的测试用参数化测试

### 12.3 可读性优先

- 测试代码要清晰易读，如同文档
- 变量名、函数名要语义化
- 关键步骤加日志说明

### 12.4 全面覆盖

- 正向测试：验证正常流程
- 反向测试：验证异常场景
- 边界测试：验证边界条件
- 安全测试：验证安全防护

### 12.5 事件验证

- 状态变更必须验证对应的链上事件
- 事件参数必须与实际状态一致
