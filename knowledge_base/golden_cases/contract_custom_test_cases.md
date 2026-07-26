# 自定义合约测试用例文档

## 文档信息
- **文档类型**: 测试用例详细文档
- **分类**: 项目自定义自研合约
- **用例数量**: 8个
- **代码位置**: `/home/liuyoushan/ape-demo/tests/contract_custom/scenarios/test_contract_custom_scenarios.py`
- **测试数据**: `/home/liuyoushan/ape-demo/tests/contract_custom/data/test_contract_custom.yaml`
- **涉及合约**: MyERC20, HelloWorld, MiniSwapRouter, MiniSwapFactory, MiniSwapPair

---

## 测试数据配置

| 用例数据键 | 关键字段 | 说明 |
|-----------|---------|------|
| `case_018_admin_permission` | token_name ("TestToken"), token_symbol ("TT") | 管理员权限测试 |
| `case_019_global_parameter_rw` | initial_message ("Hello ApeWorX!"), first_update_message, second_update_message | 全局参数读写 |
| `case_020_custom_business_logic` | amount_in_ether (200) | 自定义业务逻辑 |
| `case_021_pause_unpause` | (预留) | 暂停恢复 |
| `case_022_blacklist_whitelist` | (预留) | 黑白名单 |
| `case_023_dynamic_parameter_update` | (预留) | 动态参数更新 |
| `case_024_external_contract_call` | (预留) | 外部合约调用 |
| `case_025_custom_error_revert` | (预留) | 自定义异常 |

---

## case_018 管理员权限接口校验

### 基本信息
- **用例编号**: case_018
- **用例名称**: 管理员权限接口校验
- **优先级**: P0
- **分类**: 自定义合约 / 权限控制
- **代码参考**: `tests/contract_custom/scenarios/test_contract_custom_scenarios.py:29`

### 前置条件
1. MyERC20 合约已部署
2. deployer 具有 ADMIN_ROLE
3. user1 为普通用户，user2 为被授权用户

### 测试步骤
1. 管理员执行 mint，验证成功
2. 管理员执行 pause，验证成功
3. 管理员执行 unpause，验证成功
4. 管理员授权角色，验证成功
5. 普通用户尝试 mint，验证被拒绝
6. 普通用户尝试 pause，验证被拒绝

### 预期结果
- MINTER_ROLE：仅该角色可 mint
- PAUSER_ROLE：仅该角色可 pause
- ADMIN_ROLE：仅该角色可 grant/revoke
- 普通用户调用以上接口 revert
- 错误信息包含 "Missing required role"

### 关联测试数据
- 数据键: `case_018_admin_permission`
- 关键字段: `token_name` ("TestToken"), `token_symbol` ("TT")

### 断言规范
```python
# 管理员操作应有权限（无异常）
assert is_paused == True
assert is_paused == False
# 普通用户操作应被拒绝
assert "Missing required role" in str(e)
```

---

## case_019 自定义全局参数读写

### 基本信息
- **用例编号**: case_019
- **用例名称**: 自定义全局参数读写
- **优先级**: P0
- **分类**: 自定义合约 / 参数管理
- **代码参考**: `tests/contract_custom/scenarios/test_contract_custom_scenarios.py:87`

### 前置条件
1. HelloWorld 合约已部署
2. 初始消息已在构造函数中设置

### 测试步骤
1. 读取初始值，验证与构造器值一致
2. 第一次修改参数，验证修改后立即生效
3. 第二次修改参数，验证再次生效

### 预期结果
- 初始构造器值正确读取
- set 修改后立即生效
- 多轮修改：读取值 = 写入值（双向断言）

### 关联测试数据
- 数据键: `case_019_global_parameter_rw`
- 关键字段: `initial_message` ("Hello ApeWorX!"), `first_update_message`, `second_update_message`

### 断言规范
```python
assert initial_msg == data["initial_message"]
assert msg_after_first == data["first_update_message"]
assert msg_after_second == data["second_update_message"]
```

---

## case_020 项目独有业务接口测试

### 基本信息
- **用例编号**: case_020
- **用例名称**: 项目独有业务接口测试
- **优先级**: P1
- **分类**: 自定义合约 / 业务逻辑
- **代码参考**: `tests/contract_custom/scenarios/test_contract_custom_scenarios.py:130`

### 前置条件
1. MiniSwapFactory 合约可部署
2. MiniSwapRouter 合约可部署
3. 两种 ERC20 代币可部署

### 测试步骤
1. 部署 DEX 合约（Factory + Router + TokenA + TokenB）
2. 添加流动性创建交易对
3. 获取池子储备金
4. 调用链上 getAmountOut 计算输出
5. 本地公式计算预期输出
6. 比对链上结果与本地计算

### 预期结果
- Uniswap 风格定制化公式：扣除 0.3% 手续费后计算输出
- 链上计算结果 = 本地公式计算结果
- 公式：amountOut = (amountInWithFee * reserveOut) / (reserveIn * 1000 + amountInWithFee)

### 关联测试数据
- 数据键: `case_020_custom_business_logic`
- 关键字段: `amount_in_ether` (200)

### 断言规范
```python
assert amount_out_chain == expected_local
# amountInWithFee = swap_amount * 997
# expected_local = (amountInWithFee * reserveOut) // (reserveIn * 1000 + amountInWithFee)
```

---

## case_021 合约暂停/恢复功能测试

### 基本信息
- **用例编号**: case_021
- **用例名称**: 合约暂停/恢复功能测试
- **优先级**: P0
- **分类**: 自定义合约 / 状态控制
- **代码参考**: `tests/contract_custom/scenarios/test_contract_custom_scenarios.py:182`

### 前置条件
1. MyERC20 合约已部署
2. deployer 具有 PAUSER_ROLE

### 测试步骤
1. pause 前执行业务操作（mint），验证成功
2. 普通用户尝试 pause，验证被拒绝
3. 管理员执行 pause
4. pause 后尝试 mint，验证被拒绝
5. 管理员执行 unpause
6. unpause 后再次 mint，验证成功

### 预期结果
- pause 前业务正常
- 只有 PAUSER 能 pause/unpause
- pause 后核心业务被锁住 revert
- unpause 后业务恢复正常

### 关联测试数据
- 数据键: `case_021_pause_unpause` (预留)
- 使用硬编码测试数据

### 断言规范
```python
assert balance == test_mint_amt
# 普通用户 pause 抛异常
assert is_paused == True
# pause 后 mint 抛异常
assert is_paused == False
assert balance == test_mint_amt * 2
```

---

## case_022 黑白名单控制接口测试

### 基本信息
- **用例编号**: case_022
- **用例名称**: 黑白名单控制接口测试
- **优先级**: P0
- **分类**: 自定义合约 / 访问控制
- **代码参考**: `tests/contract_custom/scenarios/test_contract_custom_scenarios.py:247`

### 前置条件
1. MyERC20 合约已部署
2. user2 为白名单用户（有 MINTER_ROLE）
3. user1 为黑名单外用户（无角色）

### 测试步骤
1. 给白名单用户（user2）授权 MINTER_ROLE
2. 白名单用户执行 mint，验证成功
3. 验证黑名单外用户（user1）没有角色
4. 黑名单外用户尝试 mint，验证被拒绝

### 预期结果
- 白名单地址（有角色）：可享受特权操作 mint
- 黑名单外地址（无角色）：操作被强制拦截 revert
- 错误信息包含 "Missing required role"

### 关联测试数据
- 数据键: `case_022_blacklist_whitelist` (预留)
- 使用硬编码测试数据

### 断言规范
```python
assert balance == test_mint_amt
assert has_role == False
# 名单外用户 mint 抛异常
assert "Missing required role" in str(e)
```

---

## case_023 动态参数修改接口测试

### 基本信息
- **用例编号**: case_023
- **用例名称**: 动态参数修改接口测试
- **优先级**: P0
- **分类**: 自定义合约 / 参数管理
- **代码参考**: `tests/contract_custom/scenarios/test_contract_custom_scenarios.py:292`

### 前置条件
1. HelloWorld 合约已部署
2. 初始消息已设置

### 测试步骤
1. 读取初始值，验证等于默认值
2. set 新值 A，验证读取值 = A
3. 再 set 新值 B，验证读取值 = B

### 预期结果
- 读初始值 = 默认值
- set 新值 A 后读回来 = A
- 再 set 新值 B 后读回来 = B
- 参数修改闭环验证

### 关联测试数据
- 数据键: `case_023_dynamic_parameter_update` (预留)
- 使用 case_019 的 initial_message

### 断言规范
```python
assert msg == initial_msg
assert msg == "FeeRate: 0.5%"
assert msg == "RewardRate: 10%, PlatformTax: 2%"
```

---

## case_024 外部合约依赖调用测试

### 基本信息
- **用例编号**: case_024
- **用例名称**: 外部合约依赖调用测试
- **优先级**: P1
- **分类**: 自定义合约 / 跨合约调用
- **代码参考**: `tests/contract_custom/scenarios/test_contract_custom_scenarios.py:337`

### 前置条件
1. 可部署多个外部合约
2. 预言机、外部池、第三方代币合约可用

### 测试步骤
1. 部署外部合约（预言机、外部池、第三方代币）
2. 写入预言机数据，验证读取正确
3. 写入外部池数据，验证读取正确
4. 读取第三方代币信息，验证正确
5. 更新预言机价格，验证更新生效

### 预期结果
- 业务模式：合约A → 调用 → 合约B（外部依赖）的只读接口
- 预言机数据读取正确
- 外部池数据读取正确
- 第三方代币信息正确
- 数据更新后立即生效

### 关联测试数据
- 数据键: `case_024_external_contract_call` (预留)
- 使用硬编码测试数据

### 断言规范
```python
assert oracle_msg == "ETH/USD: 3456.78"
assert pool_msg == "DAI Pool Reserve: 1.2M"
assert symbol == "LINK"
assert name == "ChainLink"
assert oracle_msg == "ETH/USD: 3680.00"
```

---

## case_025 自定义业务异常拦截测试

### 基本信息
- **用例编号**: case_025
- **用例名称**: 自定义业务异常拦截测试
- **优先级**: P0
- **分类**: 自定义合约 / 异常处理
- **代码参考**: `tests/contract_custom/scenarios/test_contract_custom_scenarios.py:387`

### 前置条件
1. MyERC20 合约已部署
2. 用户账户存在

### 测试步骤
1. 权限校验：普通用户尝试 mint，验证被拒绝
2. 状态校验：暂停合约后尝试 mint，验证被拒绝
3. 参数校验：转账至零地址，验证被拒绝
4. 业务规则：超额转账，验证被拒绝

### 预期结果
- 权限校验：非授权地址调用权限接口，预期 revert
- 状态校验：暂停合约无法执行写操作，预期 revert
- 参数校验：零值、负值、超限等非法输入，预期 revert
- 业务规则：余额不足、流动性为零等边界条件，预期 revert

### 关联测试数据
- 数据键: `case_025_custom_error_revert` (预留)
- 使用硬编码测试数据

### 断言规范
```python
# 权限校验 - 抛异常
# 状态校验 - 抛异常
# 参数校验 - 抛异常
# 业务规则 - 抛异常
```

---

## 测试文件结构

```
tests/contract_custom/
├── scenarios/
│   └── test_contract_custom_scenarios.py  # 自定义合约场景
├── data/
│   └── test_contract_custom.yaml          # 测试数据
├── apis/
│   └── (待补充)                           # API 封装
├── fixtures/
│   └── contract_custom_fixtures.py        # Fixture
└── conftest.py                            # 配置
```

---

## 核心 Fixture 列表

| Fixture 名称 | 说明 |
|-------------|------|
| `myerc20_token` | MyERC20 代币实例 |
| `role_constants` | 角色常量字典 |
| `contract_custom_test_data` | 自定义合约测试数据 |
