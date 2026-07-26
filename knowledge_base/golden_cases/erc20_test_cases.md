# ERC20 测试用例详细文档

## 文档信息
- **文档类型**: 测试用例详细文档
- **分类**: ERC20 同质化代币
- **用例数量**: 10个主用例 + 2个子用例
- **代码位置**: `/home/liuyoushan/ape-demo/tests/erc20/scenarios/`
- **测试数据**: `/home/liuyoushan/ape-demo/tests/erc20/data/test_erc20.yaml`
- **合约**: MyERC20 (基于 OpenZeppelin 的 ERC20 + RBAC)

---

## 测试数据配置

### 通用配置
| 配置项 | 值 | 说明 |
|-------|----|------|
| token_name | "MyToken" | 代币名称 |
| token_symbol | "MTK" | 代币符号 |
| expected_decimals | 18 | 预期小数位数 |
| expected_initial_supply | 0 | 预期初始供应量 |

---

## case_001 代币基础信息校验

### 基本信息
- **用例编号**: case_001
- **用例名称**: 代币基础信息校验
- **优先级**: P0
- **分类**: ERC20 / 功能测试
- **代码参考**: `tests/erc20/scenarios/test_erc20_metadata.py:28`

### 前置条件
1. ERC20 代币合约已部署
2. 测试环境正常运行

### 测试步骤
1. 调用 `name()` 方法获取代币名称
2. 调用 `symbol()` 方法获取代币符号
3. 调用 `decimals()` 方法获取小数位数
4. 调用 `totalSupply()` 方法获取总发行量

### 预期结果
- 代币名称与配置的 `token_name` 一致
- 代币符号与配置的 `token_symbol` 一致
- 小数位数为 18
- 初始总供应量为 0

### 关联测试数据
- 数据键: `common`
- 关键字段: `token_name`, `token_symbol`, `expected_decimals`, `expected_initial_supply`

### 断言规范
```python
assert name == data["token_name"]
assert symbol == data["token_symbol"]
assert decimals == data["expected_decimals"]
assert total_supply == data["expected_initial_supply"]
```

---

## case_002 正常转账功能测试

### 基本信息
- **用例编号**: case_002
- **用例名称**: 正常转账功能测试
- **优先级**: P0
- **分类**: ERC20 / 功能测试 / 转账
- **代码参考**: `tests/erc20/scenarios/test_erc20_transfer.py:29`

### 前置条件
1. ERC20 代币合约已部署
2. 发送方账户有足够余额
3. 接收方账户存在

### 测试步骤
1. 记录发送方和接收方的初始余额
2. 记录总供应量
3. 调用 `transfer()` 执行转账
4. 记录转账后的余额和总供应量
5. 解码 Transfer 事件

### 预期结果
- 发送方余额减少指定转账金额
- 接收方余额增加指定转账金额
- 总供应量保持不变
- Transfer 事件正确触发，参数（from、to、value）正确

### 关联测试数据
- 数据键: `case_002_transfer`
- 关键字段: `mint_amount` (1000 ether), `transfer_amount` (200 ether)

### 断言规范
```python
assert balance_deployer_after == balance_deployer_before - expected_transfer
assert balance_user1_after == balance_user1_before + expected_transfer
assert total_supply_after == total_supply_before
assert transfer_event["from"] == deployer
assert transfer_event["to"] == user1
assert transfer_event["value"] == expected_transfer
```

---

## case_002_002 用户自转账测试

### 基本信息
- **用例编号**: case_002_002
- **用例名称**: 用户自转账测试
- **优先级**: P1
- **分类**: ERC20 / 边界测试 / 转账
- **代码参考**: `tests/erc20/scenarios/test_erc20_transfer.py:78`

### 前置条件
1. ERC20 代币合约已部署
2. 用户账户有余额

### 测试步骤
1. 给用户铸造代币
2. 记录用户转账前余额
3. 用户向自己转账
4. 记录转账后余额
5. 解码 Transfer 事件

### 预期结果
- 用户余额保持不变
- Transfer 事件正常触发（from = to）

### 关联测试数据
- 无特定测试数据，使用硬编码值 50

### 断言规范
```python
assert balance_after == balance_before
assert transfer_event["from"] == user1
assert transfer_event["to"] == user1
assert transfer_event["value"] == parse_ether(transfer_amount)
```

---

## case_003 余额不足转账失败测试

### 基本信息
- **用例编号**: case_003
- **用例名称**: 余额不足转账失败测试
- **优先级**: P0
- **分类**: ERC20 / 安全测试 / 反向测试
- **代码参考**: `tests/erc20/scenarios/test_erc20_transfer.py:113`

### 前置条件
1. ERC20 代币合约已部署
2. 发送方账户有一定余额

### 测试步骤
1. 查询发送方当前余额
2. 尝试转账金额超过余额
3. 观察交易是否 revert

### 预期结果
- 交易 revert，抛出异常
- 发送方余额保持不变
- 状态完全回滚

### 关联测试数据
- 数据键: `case_003_insufficient_transfer`
- 关键字段: `balance` (100 ether), `transfer_amount` (200 ether)

### 断言规范
```python
with reverts():
    erc20_api.contract.transfer(user1, transfer_amount, sender=deployer)
```

---

## case_004 授权功能测试

### 基本信息
- **用例编号**: case_004
- **用例名称**: 授权功能测试
- **优先级**: P0
- **分类**: ERC20 / 功能测试 / 授权
- **代码参考**: `tests/erc20/scenarios/test_erc20_approval.py:29`

### 前置条件
1. ERC20 代币合约已部署
2. 授权方和被授权方账户存在

### 测试步骤
1. 查询初始授权额度
2. 执行 approve 授权操作
3. 验证授权额度
4. 验证 Approval 事件
5. 使用授权额度执行 transferFrom
6. 验证余额变化
7. 验证授权额度扣减

### 预期结果
- 授权后 allowance 等于授权金额
- Approval 事件正确触发
- transferFrom 后发送方余额减少，接收方余额增加
- 授权额度相应扣减

### 关联测试数据
- 数据键: `case_004_approve`
- 关键字段: `approve_amount` (500 ether)

### 断言规范
```python
assert allowance_after == parse_ether(approve_amount)
assert approval_event["owner"] == deployer
assert approval_event["spender"] == user1
assert approval_event["value"] == parse_ether(approve_amount)
assert balance_deployer_after == balance_deployer_before - parse_ether(transfer_amount)
assert balance_user1_after == balance_user1_before + parse_ether(transfer_amount)
assert remaining_allowance == expected_remaining
```

---

## case_005 超出授权额度转账测试

### 基本信息
- **用例编号**: case_005
- **用例名称**: 超出授权额度转账测试
- **优先级**: P0
- **分类**: ERC20 / 安全测试 / 反向测试
- **代码参考**: `tests/erc20/scenarios/test_erc20_approval.py:96`

### 前置条件
1. ERC20 代币合约已部署
2. 已设置一定的授权额度

### 测试步骤
1. 设置授权额度
2. 尝试超出授权额度执行 transferFrom
3. 验证交易失败
4. 验证授权额度未变化

### 预期结果
- 超出授权额度的 transferFrom 被拒绝
- 交易 revert
- 授权额度保持不变

### 关联测试数据
- 数据键: `case_005_transfer_from`
- 关键字段: `mint_amount` (1000 ether), `approve_amount` (300 ether), `transfer_amount` (200 ether)

### 断言规范
```python
with reverts():
    erc20_api.transfer_from(deployer, user1, transfer_amount, user1)
assert allowance_after == parse_ether(approve_amount)
```

---

## case_006 铸币功能测试

### 基本信息
- **用例编号**: case_006
- **用例名称**: 铸币功能测试
- **优先级**: P0
- **分类**: ERC20 / 功能测试 / 铸币
- **代码参考**: `tests/erc20/scenarios/test_erc20_mint_burn.py:28`

### 前置条件
1. ERC20 代币合约已部署
2. 调用方具有 MINTER_ROLE 权限

### 测试步骤
1. 记录铸造前接收方余额
2. 记录铸造前总供应量
3. 执行 mint 操作
4. 记录铸造后余额和总供应量
5. 解码 Transfer 事件

### 预期结果
- 接收方余额增加铸造金额
- 总供应量增加铸造金额
- Transfer 事件 from 为零地址（0x0）
- Transfer 事件 to 为接收方地址
- Transfer 事件 value 为铸造金额

### 关联测试数据
- 数据键: `case_006_mint`
- 关键字段: `mint_amount` (500 ether)

### 断言规范
```python
assert balance_user1_after == balance_user1_before + expected_mint
assert total_supply_after == total_supply_before + expected_mint
assert transfer_event["from"] == "0x" + "0" * 40
assert transfer_event["to"] == user1
assert transfer_event["value"] == expected_mint
```

---

## case_007 销毁代币功能测试

### 基本信息
- **用例编号**: case_007
- **用例名称**: 销毁代币功能测试
- **优先级**: P0
- **分类**: ERC20 / 功能测试 / 销毁
- **代码参考**: `tests/erc20/scenarios/test_erc20_mint_burn.py:76`

### 前置条件
1. ERC20 代币合约已部署
2. 销毁方持有足够代币

### 测试步骤
1. 先给用户铸造代币
2. 记录销毁前余额和总供应量
3. 执行 burn 操作
4. 记录销毁后余额和总供应量
5. 解码 Transfer 事件

### 预期结果
- 用户余额减少销毁金额
- 总供应量减少销毁金额
- Transfer 事件 from 为用户地址
- Transfer 事件 to 为零地址（0x0）
- Transfer 事件 value 为销毁金额

### 关联测试数据
- 数据键: `case_007_burn`
- 关键字段: `mint_amount` (1000 ether), `burn_amount` (200 ether), `approve_amount` (300 ether), `burn_from_amount` (100 ether), `excess_burn_amount` (1000 ether)

### 断言规范
```python
assert balance_user1_after == balance_user1_before - expected_burn
assert total_supply_after == total_supply_before - expected_burn
assert transfer_event["from"] == user1
assert transfer_event["to"] == "0x" + "0" * 40
assert transfer_event["value"] == expected_burn
```

---

## case_008 RBAC 角色控制测试

### 基本信息
- **用例编号**: case_008
- **用例名称**: RBAC 角色控制测试
- **优先级**: P0
- **分类**: ERC20 / 安全测试 / RBAC / 反向测试
- **代码参考**: `tests/erc20/scenarios/test_erc20_rbac.py:29`

### 前置条件
1. ERC20 代币合约已部署
2. 普通用户账户存在

### 测试步骤
1. 普通用户尝试 mint
2. 普通用户尝试 pause
3. 普通用户尝试授权角色

### 预期结果
- 普通用户 mint 被拒绝（需要 MINTER_ROLE）
- 普通用户 pause 被拒绝（需要 PAUSER_ROLE）
- 普通用户授权角色被拒绝（需要 ADMIN_ROLE）

### 关联测试数据
- 无特定测试数据

### 断言规范
```python
with reverts():
    erc20_api.mint(user1, "100", user1)
with reverts():
    erc20_api.pause(user1)
with reverts():
    erc20_api.grant_role(MINTER_ROLE, user2, user1)
```

---

## case_009 RBAC 角色正常操作测试

### 基本信息
- **用例编号**: case_009
- **用例名称**: RBAC 角色正常操作测试
- **优先级**: P0
- **分类**: ERC20 / 功能测试 / RBAC / 正向测试
- **代码参考**: `tests/erc20/scenarios/test_erc20_rbac.py:59`

### 前置条件
1. ERC20 代币合约已部署
2. deployer 具有 ADMIN_ROLE

### 测试步骤
1. 授予用户1 MINTER_ROLE
2. 验证用户1有该角色
3. 用户1执行 mint
4. 验证 mint 成功
5. 授予用户2 PAUSER_ROLE
6. 验证用户2有该角色
7. 用户2执行 pause
8. 验证合约已暂停
9. 暂停期间尝试转账
10. 验证转账失败
11. 执行 unpause
12. 验证合约已恢复

### 预期结果
- 角色授予成功
- 有权限的用户可以执行对应操作
- 暂停期间转账被拒绝
- 恢复后功能正常

### 关联测试数据
- 无特定测试数据

### 断言规范
```python
assert erc20_api.has_role(MINTER_ROLE, user1)
assert balance == parse_ether("500")
assert erc20_api.has_role(PAUSER_ROLE, user2)
assert erc20_api.is_paused()
with reverts():
    erc20_api.transfer(user2, "10", user1)
assert not erc20_api.is_paused()
```

---

## case_010 权限升级测试

### 基本信息
- **用例编号**: case_010
- **用例名称**: 权限升级测试
- **优先级**: P1
- **分类**: ERC20 / 功能测试 / RBAC / 权限管理
- **代码参考**: `tests/erc20/scenarios/test_erc20_rbac.py:96`

### 前置条件
1. ERC20 代币合约已部署
2. deployer 具有 ADMIN_ROLE

### 测试步骤
1. 验证 deployer 有 ADMIN_ROLE
2. 授予用户1 MINTER_ROLE
3. 验证用户1有该角色
4. 用户1执行 mint
5. 验证 mint 成功
6. 撤销用户1的 MINTER_ROLE
7. 验证用户1不再有该角色
8. 用户1尝试 mint
9. 验证 mint 失败

### 预期结果
- 角色授予成功
- 角色撤销成功
- 撤销后用户无法执行受限操作

### 关联测试数据
- 无特定测试数据

### 断言规范
```python
assert erc20_api.has_role(ADMIN_ROLE, deployer)
assert erc20_api.has_role(MINTER_ROLE, user1)
assert not erc20_api.has_role(MINTER_ROLE, user1)
with reverts():
    erc20_api.mint(user1, "100", user1)
```

---

## 测试文件结构

```
tests/erc20/
├── scenarios/
│   ├── test_erc20_metadata.py       # 元数据校验
│   ├── test_erc20_transfer.py       # 转账功能
│   ├── test_erc20_approval.py       # 授权功能
│   ├── test_erc20_mint_burn.py      # 铸币销毁
│   └── test_erc20_rbac.py           # RBAC 权限
├── data/
│   └── test_erc20.yaml              # 测试数据
├── apis/
│   └── erc20_api.py                 # API 封装
├── fixtures/
│   └── erc20_fixtures.py            # Fixture
└── conftest.py                      # 配置
```

---

## 核心 Fixture 列表

| Fixture 名称 | 说明 |
|-------------|------|
| `erc20_api` | ERC20 API 实例 |
| `erc20_token_with_balance` | 带有初始余额的 ERC20 API |
| `erc20_test_data` | ERC20 测试数据 |
