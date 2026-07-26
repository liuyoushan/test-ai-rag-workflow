# 异常场景汇总

## 1. 余额不足 Revert

### 场景描述
用户尝试转账的金额超过其账户余额，交易被拒绝并 revert。

### 触发条件
- 转账金额 > 账户余额
- 调用 `transfer` 或 `transferFrom` 方法

### 预期行为
- 交易 revert，状态完全回滚
- 发送方和接收方余额保持不变
- 无任何状态变更

### 代码参考
```python
# 测试用例: case_003 余额不足转账失败测试
from ape import reverts

balance_deployer = erc20_api.get_balance(deployer)
transfer_amount = balance_deployer + 1

with reverts():
    erc20_api.contract.transfer(user1, transfer_amount, sender=deployer)
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/erc20/scenarios/test_erc20_transfer.py`


## 2. 权限拒绝

### 场景描述
无权限的用户尝试执行受限操作（如铸币、暂停合约、授权角色），操作被拒绝。

### 触发条件
- 调用者没有对应角色（MINTER_ROLE、PAUSER_ROLE、ADMIN_ROLE）
- 执行需要权限的操作

### 预期行为
- 交易 revert
- 合约状态保持不变
- 权限控制机制生效

### 代码参考
```python
# 测试用例: case_008 RBAC角色控制测试
from ape import reverts

with reverts():
    erc20_api.mint(user1, "100", user1)  # 需要 MINTER_ROLE

with reverts():
    erc20_api.pause(user1)  # 需要 PAUSER_ROLE

MINTER_ROLE = erc20_api.get_minter_role()
with reverts():
    erc20_api.grant_role(MINTER_ROLE, user2, user1)  # 需要 ADMIN_ROLE
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/erc20/scenarios/test_erc20_rbac.py`


## 3. 授权超限

### 场景描述
授权方尝试使用超出已授权额度的代币进行转账，操作被拒绝。

### 触发条件
- `allowance` 额度不足
- 调用 `transferFrom` 超过授权额度

### 预期行为
- 交易 revert
- allowance 额度保持不变
- 余额不发生变化

### 代码参考
```python
# 测试用例: case_005 超出授权额度转账测试
from ape import reverts

approve_amount = "100"
erc20_api.approve(user1, approve_amount, deployer)

transfer_amount = "150"  # 超出授权额度
with reverts():
    erc20_api.transfer_from(deployer, user1, transfer_amount, user1)
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/erc20/scenarios/test_erc20_approval.py`


## 4. 重复清算

### 场景描述
已被清算的用户仓位再次被尝试清算，操作被拒绝。

### 触发条件
- 用户已处于清算状态（`isLiquidated = true`）
- 再次调用 `liquidate` 方法

### 预期行为
- 第二次清算被拒绝
- 用户状态保持已清算
- 债务保持为 0

### 代码参考
```python
# 测试用例: case_057 重复清算防护测试
liquidation_contract.liquidate(user1, sender=deployer)  # 第一次清算成功

is_liquidated = liquidation_contract.isLiquidated(user1)
assert is_liquidated == True

# 第二次清算（预期失败）
try:
    liquidation_contract.liquidate(user1, sender=deployer)
    assert False, "第二次清算应该失败"
except Exception as e:
    pass  # 预期行为
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/liquidation/scenarios/test_liquidation_scenarios.py`


## 5. 重入攻击

### 场景描述
恶意合约尝试通过递归调用窃取合约资产，被重入锁拦截。

### 触发条件
- 恶意合约在回调中重复调用敏感函数
- 合约未实现重入防护机制

### 预期行为
- 攻击被重入锁拦截
- 交易 revert
- 合约资产安全

### 代码参考
```python
# 测试用例: case_055 重入攻击防护测试
attacker_contract = deployer.deploy(ape.project.MaliciousAttacker, 
    liquidation_contract.address, collateral_token.address, debt_token.address)

try:
    attacker_contract.attack(sender=deployer)
except Exception as e:
    pass  # 攻击被拦截

attack_success, _ = attacker_contract.getAttackResult()
assert attack_success == False
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/liquidation/scenarios/test_liquidation_scenarios.py`


## 6. 闪电贷价格操纵

### 场景描述
攻击者尝试通过闪电贷操纵价格预言机，进行恶意清算。

### 触发条件
- 攻击者通过闪电贷获取大量代币
- 尝试操纵市场价格触发非目标清算

### 预期行为
- 价格操纵被防护机制拦截
- 恶意清算失败
- 用户状态保持正常

### 代码参考
```python
# 测试用例: case_056 闪电贷价格操纵测试
flash_loan_contract = deployer.deploy(ape.project.SimpleFlashLoan, debt_token.address)
attacker_contract = deployer.deploy(ape.project.FlashLoanAttacker)

try:
    callback_data = attacker_contract.onFlashLoanReceived.encode_input()
    flash_loan_contract.flashLoan(attacker_contract.address, parse_ether("10000"), callback_data, sender=deployer)
except Exception as e:
    pass  # 攻击被拦截

user_debt = liquidation_contract.userDebt(user2)
assert user_debt in (0, debt_amount)  # 状态正常
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/liquidation/scenarios/test_liquidation_scenarios.py`


## 7. 无限授权拦截

### 场景描述
用户尝试授权无限额度（`type(uint256).max`），被安全机制拦截。

### 触发条件
- 授权金额设置为 `type(uint256).max`
- 调用 `approve` 方法

### 预期行为
- 无限授权被拒绝
- allowance 保持不变
- 防止潜在安全风险

### 代码参考
```python
# 测试用例: case_026 授权安全高阶测试
infinite_amount = data["infinite_approve_amount"]  # type(uint256).max

try:
    erc20_token.approve(user2, infinite_amount, sender=user1)
    assert False, "未拦截无限授权，安全缺陷"
except Exception as e:
    pass  # 无限授权被拦截
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/security/scenarios/test_security_scenarios.py`


## 8. 时间锁未到期

### 场景描述
用户在时间锁或区块锁未到期时尝试释放资产，操作被拒绝。

### 触发条件
- 时间锁/区块锁尚未过期
- 调用 `releaseByTime` 或 `releaseByBlock` 方法

### 预期行为
- 释放操作被拒绝
- 资产保持锁定状态
- 时间锁机制生效

### 代码参考
```python
# 测试用例: case_029 时间锁/区块锁控制
timelock_contract.lock(lock_amount, sender=user1)

assert not timelock_contract.isTimeLockExpired(user1)

try:
    timelock_contract.releaseByTime(sender=user1)
    assert False, "时间锁未到期时应该拒绝释放"
except Exception as e:
    pass  # 预期行为
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/security/scenarios/test_security_scenarios.py`


## 9. 整数溢出/下溢

### 场景描述
在 Solidity 0.8+ 中，算术运算超出边界时自动触发 revert。

### 触发条件
- 无符号整数加法超过 `uint256.max`
- 无符号整数减法小于 0
- 除以零

### 预期行为
- 溢出/下溢操作 revert
- 状态保持不变
- 内置检查机制生效

### 代码参考
```python
# 测试用例: case_031 整数溢出/下溢边界测试
max_uint256 = 2**256 - 1
math_contract = deployer.deploy(project.IntegerMath)

try:
    math_contract.incrementMax()  # max_uint256 + 1
    assert False, "应该触发溢出 revert"
except Exception as e:
    pass  # 溢出被拦截

try:
    math_contract.decrementZero()  # 0 - 1
    assert False, "应该触发下溢 revert"
except Exception as e:
    pass  # 下溢被拦截

try:
    math_contract.divide(100, 0)
    assert False, "应该触发除以零 revert"
except Exception as e:
    pass  # 除以零被拦截
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/security/scenarios/test_security_scenarios.py`


## 10. 零地址/黑洞地址转账

### 场景描述
用户尝试向零地址（0x0）或黑洞地址（0x...dEaD）转账，操作被拦截。

### 触发条件
- 目标地址为零地址 `0x0000000000000000000000000000000000000000`
- 目标地址为黑洞地址 `0x000000000000000000000000000000000000dEaD`

### 预期行为
- 危险地址转账被拦截
- 余额保持不变
- 资产安全得到保护

### 代码参考
```python
# 测试用例: case_034 零地址/黑洞地址防护测试
try:
    erc20_token.transfer("0x0000000000000000000000000000000000000000", parse_ether("100"), sender=user1)
    assert False, "零地址转账应该被拦截"
except Exception as e:
    pass  # 零地址被拦截

try:
    erc20_token.transfer("0x000000000000000000000000000000000000dEaD", parse_ether("100"), sender=user1)
    assert False, "黑洞地址转账应该被拦截"
except Exception as e:
    pass  # 黑洞地址被拦截
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/security/scenarios/test_security_scenarios.py`


## 11. Gas 超限

### 场景描述
交易设置的 Gas 上限不足以完成执行，交易失败。

### 触发条件
- `gas_limit` 设置过低
- 交易执行需要的 Gas 超过限制

### 预期行为
- 交易失败
- 状态完全回滚
- Gas 按实际消耗扣除

### 代码参考
```python
# 测试用例: case_035 Gas与交易异常兼容测试
balance_before = erc20_token.balanceOf(user1)

try:
    erc20_token.transfer(user2, parse_ether("10"), sender=user1, gas_limit=21000)
except Exception as e:
    pass  # 低Gas导致失败

balance_after = erc20_token.balanceOf(user1)
assert balance_after == balance_before  # 状态回滚
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/security/scenarios/test_security_scenarios.py`


## 12. 非清算条件拒绝

### 场景描述
用户健康因子高于阈值时，清算操作被拒绝。

### 触发条件
- 用户健康因子 >= 安全阈值
- 调用 `liquidate` 方法

### 预期行为
- 清算操作被拒绝
- 用户状态保持不变
- 防止恶意清算

### 代码参考
```python
# 测试用例: case_051 非清算条件拒绝测试
# 当用户健康因子正常时，清算操作被拒绝
# 保护用户免受恶意清算
with reverts():
    liquidation_contract.liquidate(user1, sender=liquidator)
```

**文件路径**: `/home/liuyoushan/ape-demo/tests/liquidation/scenarios/test_liquidation_scenarios.py`
