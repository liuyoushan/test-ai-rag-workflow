# 安全测试用例详细文档

## 文档信息
- **文档类型**: 测试用例详细文档
- **分类**: 安全测试 / Web3 高阶安全
- **用例数量**: 11个
- **代码位置**: `/home/liuyoushan/ape-demo/tests/security/scenarios/test_security_scenarios.py`
- **测试数据**: `/home/liuyoushan/ape-demo/tests/security/data/test_security_advanced.yaml`
- **涉及合约**: MyERC20, StakingContract, TimeLockContract, ReentrancyVault, UpgradeableProxy, IntegerMath

---

## 测试数据配置

| 用例数据键 | 关键字段 | 说明 |
|-----------|---------|------|
| `case_026_approve_security` | initial_approve_amount (1000), second_approve_amount (500), infinite_approve_amount (MAX_UINT256), zero_approve_amount (0) | 授权安全高阶测试 |
| `case_027_batch_operations` | batch_size (5), transfer_amount (100) | 批量操作接口测试 |
| `case_028_staking_mining` | stake_amount (1000), reward_rate (100), stake_duration_blocks (100) | 质押/挖矿收益测算 |
| `case_029_timelock_blocklock` | lock_duration (86400), lock_blocks (100) | 时间锁/区块锁控制 |
| `case_030_reentrancy_guard` | test_amount (1000) | 重入攻击防护测试 |
| `case_031_integer_overflow_underflow` | max_uint256 (MAX_UINT256) | 整数溢出/下溢边界 |
| `case_032_proxy_upgrade` | test_value (1000) | 合约升级代理测试 |
| `case_033_event_completeness` | test_amount (1000) | 链上事件完整校验 |
| `case_035_gas_tx_exception` | test_amount (100) | Gas与交易异常兼容 |

---

## case_026 授权安全高阶测试

### 基本信息
- **用例编号**: case_026
- **用例名称**: 授权安全高阶测试
- **优先级**: P0
- **分类**: 安全测试 / 授权安全
- **代码参考**: `tests/security/scenarios/test_security_scenarios.py:29`

### 前置条件
1. ERC20 代币合约已部署
2. 授权方和被授权方账户存在

### 测试步骤
1. 执行初始授权，验证授权额度正确
2. 执行重复授权，验证覆盖旧值
3. 尝试无限授权（MAX_UINT256），验证被拦截
4. 授权清零，验证清零后无法操作

### 预期结果
- 初始授权 allowance 正确记录
- 重复授权覆盖旧值
- 无限授权（type(uint256).max）被拦截
- 授权清零后 transferFrom 被拒绝
- Approval 事件正确触发

### 关联测试数据
- 数据键: `case_026_approve_security`
- 关键字段: `initial_approve_amount` (1000), `second_approve_amount` (500), `infinite_approve_amount` (MAX_UINT256), `zero_approve_amount` (0)

### 断言规范
```python
assert allowance == initial_amount
assert allowance == second_amount
# 无限授权抛异常
# 清零后操作抛异常
```

---

## case_027 批量操作接口测试

### 基本信息
- **用例编号**: case_027
- **用例名称**: 批量操作接口测试
- **优先级**: P1
- **分类**: 安全测试 / 效率测试
- **代码参考**: `tests/security/scenarios/test_security_scenarios.py:90`

### 前置条件
1. ERC20 代币合约已部署
2. 合约支持批量操作接口
3. 调用方有足够余额/权限

### 测试步骤
1. 执行批量转账（batchTransfer）
2. 验证所有接收方余额正确增加
3. 验证 Transfer 事件数量正确
4. 执行批量授权（batchApprove）
5. 验证所有 allowance 正确设置
6. 数组长度不匹配时验证被拒绝

### 预期结果
- 批量转账：多地址同时转账，余额正确扣减
- 批量授权：多地址同时授权，allowance 正确设置
- Transfer/Approval 事件正确触发
- 数组长度不匹配时被拒绝

### 关联测试数据
- 数据键: `case_027_batch_operations`
- 关键字段: `batch_size` (5), `transfer_amount` (100)

### 断言规范
```python
assert len(transfer_events) == len(recipients)
assert allowance0 == approve_amounts[0]
assert allowance1 == approve_amounts[1]
# 数组长度不匹配抛异常
```

---

## case_028 质押/挖矿收益测算

### 基本信息
- **用例编号**: case_028
- **用例名称**: 质押/挖矿收益测算
- **优先级**: P0
- **分类**: 安全测试 / 核心业务
- **代码参考**: `tests/security/scenarios/test_security_scenarios.py:144`

### 前置条件
1. 质押合约已部署
2. 奖励代币合约已部署
3. 用户持有质押代币

### 测试步骤
1. 用户1质押代币
2. 验证质押余额正确
3. 用户2质押触发奖励计算
4. 验证用户1有待领取奖励
5. 用户1解押
6. 验证质押余额清零
7. 用户1领取奖励
8. 验证奖励余额增加

### 预期结果
- 质押：代币转入合约，质押余额增加
- 奖励计算：基于区块高度计算待领取奖励
- 解押：代币返还用户，质押余额清零
- 领取奖励：奖励代币正确发放
- Staked/Unstaked/RewardClaimed 事件正确触发

### 关联测试数据
- 数据键: `case_028_staking_mining`
- 关键字段: `stake_amount` (1000), `reward_rate` (100), `stake_duration_blocks` (100)

### 断言规范
```python
assert stake_balance == stake_amount
assert pending_reward > 0
assert stake_balance == 0
assert reward_after > reward_before
```

---

## case_029 时间锁/区块锁控制

### 基本信息
- **用例编号**: case_029
- **用例名称**: 时间锁/区块锁控制
- **优先级**: P0
- **分类**: 安全测试 / 时间敏感
- **代码参考**: `tests/security/scenarios/test_security_scenarios.py:210`

### 前置条件
1. 时间锁合约已部署
2. 用户账户存在
3. 管理员账户存在

### 测试步骤
1. 用户1执行时间锁锁定
2. 验证锁定信息正确记录
3. 时间锁未到期尝试释放，验证被拒绝
4. 用户2执行区块锁锁定
5. 区块锁未到期尝试释放，验证被拒绝
6. 重复锁定验证被拒绝
7. 零金额锁定验证被拒绝
8. 管理员更新锁定参数

### 预期结果
- 锁定操作正确记录金额和时间戳
- 时间锁未到期时拒绝释放
- 区块锁未到期时拒绝释放
- 重复锁定被拒绝
- 零金额锁定被拒绝
- 管理员可更新锁定参数

### 关联测试数据
- 数据键: `case_029_timelock_blocklock`
- 关键字段: `lock_duration` (86400), `lock_blocks` (100)

### 断言规范
```python
assert lock_info[0] == lock_amount
assert not timelock_contract.isTimeLockExpired(user1)
# 未到期释放抛异常
# 重复锁定抛异常
# 零金额锁定抛异常
assert duration == 172800
```

---

## case_030 重入攻击防护测试

### 基本信息
- **用例编号**: case_030
- **用例名称**: 重入攻击防护测试
- **优先级**: P1
- **分类**: 安全测试 / 重入攻击
- **代码参考**: `tests/security/scenarios/test_security_scenarios.py:287`

### 前置条件
1. 防护合约（带重入锁）已部署
2. 漏洞合约（无重入锁）已部署
3. 攻击者合约可部署

### 测试步骤
1. 防护合约正常存款/取款测试
2. 验证正常存取款功能正常
3. 漏洞合约测试攻击（验证漏洞存在）
4. 防护合约测试攻击
5. 验证重入攻击被拦截

### 预期结果
- 正常存款/取款功能正常
- 漏洞合约易受重入攻击（攻击者可提取超额资金）
- 防护合约成功拦截重入攻击（reentrant lock 生效）
- 攻击后攻击者余额不超过存款金额

### 关联测试数据
- 数据键: `case_030_reentrancy_guard`
- 关键字段: `test_amount` (1000)

### 断言规范
```python
assert balance == parse_ether("1")
assert balance == 0
# 漏洞合约攻击可执行
# 防护合约攻击抛异常
```

---

## case_031 整数溢出/下溢边界测试

### 基本信息
- **用例编号**: case_031
- **用例名称**: 整数溢出/下溢边界测试
- **优先级**: P1
- **分类**: 安全测试 / 数值安全
- **代码参考**: `tests/security/scenarios/test_security_scenarios.py:344`

### 前置条件
1. IntegerMath 合约已部署
2. Solidity 版本 >= 0.8

### 测试步骤
1. 基本算术运算测试（add/sub/mul/div）
2. 验证基本运算正确
3. 无符号整数溢出测试
4. 验证溢出触发 revert
5. 无符号整数下溢测试
6. 验证下溢触发 revert
7. SafeMath 安全函数测试
8. 除以零测试

### 预期结果
- 基本算术运算正常工作
- 无符号整数溢出触发 revert
- 无符号整数下溢触发 revert
- SafeMath 安全函数正确执行
- 除以零触发 revert

### 关联测试数据
- 数据键: `case_031_integer_overflow_underflow`
- 关键字段: `max_uint256` (MAX_UINT256)

### 断言规范
```python
assert add_result == 300
assert sub_result == 100
assert mul_result == 200
# 溢出抛异常
# 下溢抛异常
assert safe_sub_result == 50
# safeSub 下溢抛异常
# 除以零抛异常
```

---

## case_032 合约升级代理测试

### 基本信息
- **用例编号**: case_032
- **用例名称**: 合约升级代理测试
- **优先级**: P1
- **分类**: 安全测试 / 升级测试
- **代码参考**: `tests/security/scenarios/test_security_scenarios.py:430`

### 前置条件
1. LogicV1 合约已部署
2. LogicV2 合约已部署
3. UpgradeableProxy 合约可部署
4. 管理员和普通用户账户存在

### 测试步骤
1. 部署 V1 逻辑合约和代理
2. 初始化 V1，验证数据和版本
3. 更新 V1 数据
4. 升级到 V2
5. 验证升级后数据不丢失
6. 验证 V2 新增功能正常
7. 非管理员尝试升级，验证被拒绝
8. 管理员变更权限

### 预期结果
- 初始化 V1 逻辑合约
- 数据读写在代理中正常工作
- 升级到 V2 后数据不丢失
- V2 新增功能正常工作
- 非管理员无法执行升级
- 管理员可变更升级权限

### 关联测试数据
- 数据键: `case_032_proxy_upgrade`
- 关键字段: `test_value` (1000)

### 断言规范
```python
assert value == 1000
assert version == "V1"
assert value == 2000
assert version == "V2"
# 非管理员升级抛异常
assert admin == user1.address
```

---

## case_033 事件完整性测试

### 基本信息
- **用例编号**: case_033
- **用例名称**: 事件完整性测试
- **优先级**: P1
- **分类**: 安全测试 / 事件测试
- **代码参考**: `tests/security/scenarios/test_security_scenarios.py:414`
- **状态**: 待实现 (NotImplementedError)

### 前置条件
1. 测试合约已部署
2. 事件定义完整

### 测试步骤
1. 执行状态变更操作
2. 收集触发的所有事件
3. 验证事件数量正确
4. 验证事件参数与实际状态一致
5. 验证事件顺序与操作顺序一致
6. 验证无遗漏或冗余事件

### 预期结果
- 每次状态变更都正确触发事件
- 事件参数与实际状态一致
- 事件顺序与操作顺序一致
- 无遗漏或冗余事件

### 关联测试数据
- 数据键: `case_033_event_completeness`
- 关键字段: `test_amount` (1000)

---

## case_034 零地址/黑洞地址防护测试

### 基本信息
- **用例编号**: case_034
- **用例名称**: 零地址/黑洞地址防护测试
- **优先级**: P1
- **分类**: 安全测试 / 安全防护
- **代码参考**: `tests/security/scenarios/test_security_scenarios.py:504`

### 前置条件
1. ERC20 代币合约已部署
2. 用户持有代币

### 测试步骤
1. 尝试转账至零地址（0x0）
2. 验证被拦截
3. 尝试转账至黑洞地址（0x...dEaD）
4. 验证被拦截
5. 正常地址间转账
6. 验证正常执行

### 预期结果
- 转账至零地址被拦截
- 转账至黑洞地址被拦截
- 正常地址间转账正常执行

### 关联测试数据
- 无特定测试数据

### 断言规范
```python
# 零地址转账抛异常
# 黑洞地址转账抛异常
assert balance_after == balance_before - parse_ether("100")
```

---

## case_035 Gas与交易异常兼容测试

### 基本信息
- **用例编号**: case_035
- **用例名称**: Gas与交易异常兼容测试
- **优先级**: P1
- **分类**: 安全测试 / 交易异常
- **代码参考**: `tests/security/scenarios/test_security_scenarios.py:551`

### 前置条件
1. ERC20 代币合约已部署
2. 用户持有代币

### 测试步骤
1. 正常 Gas 下执行交易
2. 验证交易成功
3. 低 Gas（如 21000）执行交易
4. 验证交易失败
5. 验证状态完全回滚

### 预期结果
- 正常 Gas 下交易成功执行
- 低 Gas 导致交易失败
- 交易失败时状态完全回滚

### 关联测试数据
- 数据键: `case_035_gas_tx_exception`
- 关键字段: `test_amount` (100)

### 断言规范
```python
# 正常交易成功
# 低 Gas 交易失败
assert balance_after == balance_before
```

---

## 测试文件结构

```
tests/security/
├── scenarios/
│   └── test_security_scenarios.py    # 安全场景测试
├── data/
│   └── test_security_advanced.yaml   # 测试数据
├── apis/
│   └── security_api.py               # API 封装
├── fixtures/
│   └── security_fixtures.py          # Fixture
└── conftest.py                       # 配置
```

---

## 核心 Fixture 列表

| Fixture 名称 | 说明 |
|-------------|------|
| `erc20_token` | ERC20 代币实例 |
| `staking_contract` | 质押合约实例（元组：质押合约 + 奖励代币） |
| `timelock_contract` | 时间锁合约实例 |
| `reentrancy_vault` | 防护型 Vault（带重入锁） |
| `vulnerable_vault` | 漏洞型 Vault（无重入锁） |
| `security_test_data` | 安全测试数据 |
