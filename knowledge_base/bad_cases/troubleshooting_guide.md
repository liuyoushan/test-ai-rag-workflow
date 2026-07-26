# 排查指南

## 1. 资金对账不一致

### 问题现象
- 链上余额与预期不符
- 转账后余额变化异常
- 代币数量计算错误

### 排查步骤

#### 步骤1: 确认初始状态
```python
# 记录操作前的余额
balance_before = token.balanceOf(account)
log.debug(f"操作前余额: {format_ether(balance_before)}")
```

#### 步骤2: 执行操作并验证
```python
# 执行转账操作
tx = token.transfer(recipient, amount, sender=sender)

# 记录操作后的余额
balance_after = token.balanceOf(account)
log.debug(f"操作后余额: {format_ether(balance_after)}")

# 验证余额变化
expected_balance = balance_before - parse_ether(amount)
assert balance_after == expected_balance, \
    f"余额不符，预期: {format_ether(expected_balance)}, 实际: {format_ether(balance_after)}"
```

#### 步骤3: 检查事件日志
```python
# 解码 Transfer 事件
transfer_events = tx.decode_logs(token.Transfer)
for event in transfer_events:
    log.debug(f"Transfer 事件: from={event['from']}, to={event['to']}, value={format_ether(event['value'])}")
```

### 定位方法
- 使用 `balanceOf` 检查各账户余额
- 对比 `Transfer` 事件参数与实际余额变化
- 检查是否有其他合约交互影响余额

### 解决方案
1. 确认转账金额计算正确
2. 检查是否存在授权未释放的情况
3. 验证合约是否有手续费或扣减逻辑
4. 使用 `format_ether`/`parse_ether` 确保单位正确


## 2. Gas 消耗异常

### 问题现象
- 交易 Gas 消耗远超预期
- Gas 超限导致交易失败
- Gas 费用过高

### 排查步骤

#### 步骤1: 检查 Gas 使用情况
```python
# 获取交易 Gas 消耗
tx_receipt = tx.receipt
gas_used = tx_receipt.gas_used
log.debug(f"Gas 消耗: {gas_used}")

# 获取 Gas 价格
gas_price = tx_receipt.gas_price
log.debug(f"Gas 价格: {format_ether(gas_price)}")

# 计算交易费用
tx_cost = gas_used * gas_price
log.debug(f"交易费用: {format_ether(tx_cost)}")
```

#### 步骤2: 分析 Gas 超限原因
```python
# 设置不同的 Gas 限制测试
try:
    tx = contract.function(arg1, arg2, sender=user, gas_limit=500000)
except Exception as e:
    log.debug(f"Gas 限制 500000 失败: {type(e).__name__}")

try:
    tx = contract.function(arg1, arg2, sender=user, gas_limit=1000000)
    log.debug("Gas 限制 1000000 成功")
except Exception as e:
    log.debug(f"Gas 限制 1000000 失败: {type(e).__name__}")
```

#### 步骤3: 优化 Gas 消耗
```python
# 使用批量操作减少交易次数
recipients = [user1, user2, user3]
amounts = [parse_ether("100"), parse_ether("200"), parse_ether("300")]
tx = token.batchTransfer(recipients, amounts, sender=deployer)
```

### 定位方法
- 使用 `tx.receipt.gas_used` 查看实际消耗
- 对比不同参数下的 Gas 消耗
- 检查是否有循环或递归导致 Gas 激增

### 解决方案
1. 拆分大交易为多个小交易
2. 使用批量操作接口
3. 优化合约代码减少存储操作
4. 设置合理的 `gas_limit`


## 3. 交易失败

### 问题现象
- 交易 revert
- 交易超时
- 交易状态为失败

### 排查步骤

#### 步骤1: 检查交易状态
```python
# 获取交易状态
tx_receipt = tx.receipt
status = tx_receipt.status
log.debug(f"交易状态: {status}")  # 1 = 成功, 0 = 失败

if status == 0:
    # 获取 revert 原因
    revert_reason = tx_receipt.revert_reason
    log.debug(f"Revert 原因: {revert_reason}")
```

#### 步骤2: 验证调用者权限
```python
# 检查角色权限
MINTER_ROLE = contract.get_minter_role()
has_role = contract.has_role(MINTER_ROLE, caller)
log.debug(f"{caller} 是否有 MINTER_ROLE: {has_role}")

if not has_role:
    log.error("调用者缺少必要权限")
```

#### 步骤3: 检查前置条件
```python
# 检查余额
balance = token.balanceOf(sender)
log.debug(f"发送方余额: {format_ether(balance)}")

# 检查授权
allowance = token.allowance(sender, spender)
log.debug(f"授权额度: {format_ether(allowance)}")

# 检查合约状态
is_paused = contract.is_paused()
log.debug(f"合约是否暂停: {is_paused}")
```

### 定位方法
- 使用 `tx_receipt.status` 判断成功/失败
- 使用 `tx_receipt.revert_reason` 获取失败原因
- 检查调用者权限、余额、授权等前置条件

### 解决方案
1. 根据 revert 原因修复问题
2. 确保调用者有足够权限
3. 验证账户余额和授权额度
4. 检查合约状态（如暂停状态）


## 4. 事件缺失

### 问题现象
- 预期事件未触发
- 事件参数不正确
- 事件数量不足

### 排查步骤

#### 步骤1: 检查事件触发情况
```python
# 获取交易日志
logs = tx.decode_logs(contract.Transfer)
log.debug(f"Transfer 事件数量: {len(logs)}")

if len(logs) == 0:
    log.warning("未触发 Transfer 事件")
```

#### 步骤2: 验证事件参数
```python
# 解码事件并验证
transfer_event = contract.decode_transfer_event(tx)

expected_from = sender
expected_to = recipient
expected_value = parse_ether(amount)

assert transfer_event["from"] == expected_from, \
    f"事件 from 不符，预期: {expected_from}, 实际: {transfer_event['from']}"
assert transfer_event["to"] == expected_to, \
    f"事件 to 不符，预期: {expected_to}, 实际: {transfer_event['to']}"
assert transfer_event["value"] == expected_value, \
    f"事件 value 不符，预期: {format_ether(expected_value)}, 实际: {format_ether(transfer_event['value'])}"
```

#### 步骤3: 检查合约代码
```python
# 确认合约是否正确触发事件
# Solidity 合约中应包含:
# emit Transfer(from, to, value);
```

### 定位方法
- 使用 `decode_logs` 获取所有事件
- 对比事件参数与预期值
- 检查合约代码是否 emit 事件

### 解决方案
1. 修复合约代码确保触发事件
2. 验证事件解码逻辑正确性
3. 确认交易已成功执行
4. 检查事件签名是否匹配


## 5. 授权问题

### 问题现象
- `transferFrom` 失败
- 授权额度不更新
- 授权被意外清零

### 排查步骤

#### 步骤1: 检查授权状态
```python
# 查询当前授权额度
allowance = token.allowance(owner, spender)
log.debug(f"授权额度: {format_ether(allowance)}")

# 检查是否已授权
if allowance == 0:
    log.warning("授权额度为0，请先授权")
```

#### 步骤2: 执行授权操作
```python
# 设置授权
tx = token.approve(spender, parse_ether("1000"), sender=owner)

# 验证授权结果
allowance_after = token.allowance(owner, spender)
assert allowance_after == parse_ether("1000"), \
    f"授权失败，预期: 1000, 实际: {format_ether(allowance_after)}"

# 检查 Approval 事件
approval_event = token.decode_approval_event(tx)
assert approval_event["owner"] == owner
assert approval_event["spender"] == spender
```

#### 步骤3: 使用授权
```python
# 使用授权转账
transfer_amount = parse_ether("500")
assert allowance_after >= transfer_amount, "授权额度不足"

tx = token.transferFrom(owner, recipient, transfer_amount, sender=spender)

# 验证授权扣减
remaining_allowance = token.allowance(owner, spender)
expected_remaining = parse_ether("1000") - transfer_amount
assert remaining_allowance == expected_remaining, \
    f"授权扣减错误，预期: {format_ether(expected_remaining)}, 实际: {format_ether(remaining_allowance)}"
```

### 定位方法
- 使用 `allowance` 查询授权状态
- 检查 `Approval` 事件
- 验证授权扣减逻辑

### 解决方案
1. 确保先调用 `approve` 设置授权
2. 检查授权额度是否足够
3. 处理授权过期或被清零的情况
4. 使用安全授权模式


## 6. 清算异常

### 问题现象
- 清算未触发
- 清算结果不符合预期
- 重复清算问题

### 排查步骤

#### 步骤1: 检查清算条件
```python
# 计算健康因子
health_factor = liquidation_contract.get_health_factor(user)
log.debug(f"健康因子: {format_ether(health_factor)}")

# 检查是否可清算
can_liquidate = liquidation_contract.canLiquidate(user)
log.debug(f"是否可清算: {can_liquidate}")

# 获取清算阈值
threshold = liquidation_contract.HEALTH_FACTOR_THRESHOLD()
log.debug(f"清算阈值: {format_ether(threshold)}")
```

#### 步骤2: 执行清算并验证
```python
# 执行清算
debt_token.approve(liquidation_contract, debt_amount, sender=liquidator)
tx = liquidation_contract.liquidate(user, sender=liquidator)

# 验证清算状态
is_liquidated = liquidation_contract.isLiquidated(user)
user_debt = liquidation_contract.userDebt(user)

assert is_liquidated == True, "清算状态未更新"
assert user_debt == 0, f"债务未清零，实际: {format_ether(user_debt)}"
```

#### 步骤3: 检查重复清算防护
```python
# 尝试再次清算（预期失败）
try:
    liquidation_contract.liquidate(user, sender=liquidator)
    assert False, "重复清算应该被拒绝"
except Exception as e:
    log.debug(f"重复清算被正确拒绝: {type(e).__name__}")
```

### 定位方法
- 检查健康因子是否低于阈值
- 验证清算前后状态变化
- 测试重复清算防护机制

### 解决方案
1. 确认健康因子计算正确
2. 检查清算条件逻辑
3. 验证重复清算防护
4. 确保清算奖励计算正确


## 7. 权限控制问题

### 问题现象
- 无权限用户执行了受限操作
- 有权限用户操作被拒绝
- 角色管理异常

### 排查步骤

#### 步骤1: 检查角色配置
```python
# 获取角色标识
ADMIN_ROLE = erc20_api.get_admin_role()
MINTER_ROLE = erc20_api.get_minter_role()

# 检查用户角色
has_admin = erc20_api.has_role(ADMIN_ROLE, user)
has_minter = erc20_api.has_role(MINTER_ROLE, user)

log.debug(f"{user} 是否有 ADMIN_ROLE: {has_admin}")
log.debug(f"{user} 是否有 MINTER_ROLE: {has_minter}")
```

#### 步骤2: 测试权限边界
```python
# 无权限用户尝试受限操作（预期失败）
with reverts():
    erc20_api.mint(user1, "100", user2)  # user2 无 MINTER_ROLE

# 授权后再次尝试（预期成功）
erc20_api.grant_role(MINTER_ROLE, user2, admin)
erc20_api.mint(user1, "100", user2)  # 应成功
```

#### 步骤3: 验证角色管理
```python
# 撤销角色
erc20_api.revoke_role(MINTER_ROLE, user2, admin)
assert not erc20_api.has_role(MINTER_ROLE, user2)

# 验证撤销后权限失效
with reverts():
    erc20_api.mint(user1, "100", user2)
```

### 定位方法
- 使用 `has_role` 检查角色分配
- 测试权限边界情况
- 验证角色授权/撤销流程

### 解决方案
1. 确保角色正确分配
2. 修复权限检查逻辑
3. 验证角色管理功能
4. 使用最小权限原则


## 8. 合约升级问题

### 问题现象
- 升级后数据丢失
- 升级后功能异常
- 非管理员执行了升级

### 排查步骤

#### 步骤1: 验证升级前状态
```python
# 获取升级前数据
proxy_v1 = project.LogicV1.at(proxy.address)
value_before = proxy_v1.getValue()
version_before = proxy_v1.getVersion()

log.debug(f"升级前 - 值: {value_before}, 版本: {version_before}")
```

#### 步骤2: 执行升级
```python
# 部署新逻辑合约
logic_v2 = deployer.deploy(project.LogicV2)

# 执行升级
proxy.upgradeTo(logic_v2.address, sender=admin)
log.debug("合约升级完成")
```

#### 步骤3: 验证升级后状态
```python
# 获取升级后数据
proxy_v2 = project.LogicV2.at(proxy.address)
value_after = proxy_v2.getValue()
version_after = proxy_v2.getVersion()

log.debug(f"升级后 - 值: {value_after}, 版本: {version_after}")

# 验证数据不丢失
assert value_after == value_before, f"数据丢失，升级前: {value_before}, 升级后: {value_after}"
assert version_after == "V2", f"版本不符，预期: V2, 实际: {version_after}"
```

#### 步骤4: 验证权限控制
```python
# 非管理员尝试升级（预期失败）
try:
    proxy.upgradeTo(logic_v1.address, sender=non_admin)
    assert False, "非管理员不应能升级"
except Exception as e:
    log.debug(f"非管理员升级被正确拒绝: {type(e).__name__}")
```

### 定位方法
- 对比升级前后数据
- 验证新功能是否正常
- 检查升级权限控制

### 解决方案
1. 使用代理模式保留数据
2. 验证升级后数据完整性
3. 确保升级权限控制正确
4. 进行升级兼容性测试


## 9. 安全攻击防护

### 问题现象
- 重入攻击成功
- 闪电贷攻击成功
- 价格预言机被操纵

### 排查步骤

#### 步骤1: 检查重入防护
```python
# 部署攻击者合约
attacker = deployer.deploy(project.ReentrancyAttacker, vault.address)

# 执行攻击（预期被拦截）
try:
    attacker.attack(parse_ether("1"), sender=attacker)
    assert False, "重入攻击应该被拦截"
except Exception as e:
    log.debug(f"重入攻击被拦截: {type(e).__name__}")

# 验证资产安全
vault_balance = vault.balance()
assert vault_balance >= 0, "合约资产被盗"
```

#### 步骤2: 检查闪电贷防护
```python
# 部署闪电贷攻击者
flash_loan_attacker = deployer.deploy(project.FlashLoanAttacker)

# 执行攻击（预期失败）
try:
    flash_loan_attacker.executeAttack(sender=attacker)
except Exception as e:
    log.debug(f"闪电贷攻击被拦截: {type(e).__name__}")

# 验证用户状态正常
user_debt = liquidation_contract.userDebt(user)
assert user_debt in (0, expected_debt), "用户状态异常"
```

#### 步骤3: 检查预言机操纵
```python
# 验证价格预言机数据
price = oracle.getPrice(token_address)
log.debug(f"当前价格: {format_ether(price)}")

# 检查价格偏差
deviation = abs(price - expected_price) / expected_price
assert deviation < 0.05, f"价格偏差过大: {deviation * 100}%"
```

### 定位方法
- 模拟攻击验证防护机制
- 检查重入锁和 Check-Effects-Interaction 模式
- 验证价格预言机安全性

### 解决方案
1. 实现重入锁保护
2. 使用 Check-Effects-Interaction 模式
3. 增加价格预言机保护
4. 进行安全审计


## 10. 网络连接问题

### 问题现象
- 无法连接到 RPC 节点
- 交易超时
- 网络不稳定

### 排查步骤

#### 步骤1: 检查网络状态
```python
# 检查当前网络
from ape import networks

current_network = networks.active_network
log.debug(f"当前网络: {current_network}")

# 检查提供者状态
provider = networks.active_provider()
is_connected = provider.is_connected()
log.debug(f"是否连接: {is_connected}")
```

#### 步骤2: 测试网络连接
```python
# 获取区块信息
try:
    block_number = provider.get_block_number()
    log.debug(f"当前区块: {block_number}")
except Exception as e:
    log.error(f"网络连接失败: {type(e).__name__}")
```

#### 步骤3: 切换备用节点
```python
# 切换到备用网络
try:
    with networks.parse_network_choice("ethereum:goerli:alchemy"):
        # 执行操作
        pass
except Exception as e:
    log.error(f"切换网络失败: {type(e).__name__}")
```

### 定位方法
- 使用 `is_connected` 检查连接状态
- 测试区块查询验证网络通畅
- 尝试切换不同 RPC 节点

### 解决方案
1. 检查网络配置
2. 切换稳定的 RPC 节点
3. 增加超时配置
4. 实现网络重试机制
