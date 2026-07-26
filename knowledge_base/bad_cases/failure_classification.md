# 故障分类索引

## 分类体系概述

本分类体系将 Web3 测试中的故障分为四个维度：合约层面、网络层面、测试层面、业务逻辑层面。每个维度下包含具体故障类型、特征描述和处理策略。

---

## 一、合约层面故障

### 1.1 权限控制故障

#### 故障类型
- 未授权访问
- 权限提升
- 角色管理异常

#### 特征描述
- 无权限用户执行了受限操作
- 权限验证逻辑缺失或错误
- 角色授权/撤销流程异常

#### 处理策略
```python
# 验证权限
MINTER_ROLE = contract.get_minter_role()
assert contract.has_role(MINTER_ROLE, caller), "调用者缺少权限"

# 使用 reverts 捕获未授权操作
from ape import reverts
with reverts():
    contract.mint(user, amount, sender=unauthorized_user)
```

#### 关联文档
- `exception_scenarios.md` - 权限拒绝场景
- `troubleshooting_guide.md` - 权限控制问题排查


### 1.2 资产安全故障

#### 故障类型
- 重入攻击
- 闪电贷攻击
- 整数溢出/下溢
- 零地址/黑洞地址转账

#### 特征描述
- 合约资产被盗或锁定
- 算术运算超出边界
- 危险地址转账未被拦截

#### 处理策略
```python
# 重入攻击防护
# 使用 ReentrancyGuard 或 Check-Effects-Interaction 模式

# 整数溢出防护（Solidity 0.8+ 内置）
# 使用 safeAdd/safeSub 等安全函数

# 零地址检查
require(to != address(0), "Zero address not allowed")
```

#### 关联文档
- `exception_scenarios.md` - 重入攻击、整数溢出、零地址场景
- `error_patterns.md` - 状态不一致错误


### 1.3 逻辑执行故障

#### 故障类型
- 条件判断错误
- 状态更新遗漏
- 事件触发缺失
- 数据不一致

#### 特征描述
- 合约状态不符合预期
- 事件未正确触发
- 条件分支逻辑错误

#### 处理策略
```python
# 验证状态变更
balance_before = token.balanceOf(account)
contract.transfer(recipient, amount, sender=account)
balance_after = token.balanceOf(account)
assert balance_after == balance_before - amount

# 验证事件触发
tx = contract.transfer(recipient, amount, sender=account)
events = tx.decode_logs(contract.Transfer)
assert len(events) == 1
```

#### 关联文档
- `error_patterns.md` - 事件缺失错误、状态不一致错误
- `troubleshooting_guide.md` - 事件缺失、资金对账不一致排查


### 1.4 升级代理故障

#### 故障类型
- 数据丢失
- 版本不兼容
- 升级权限绕过

#### 特征描述
- 合约升级后数据丢失
- 新逻辑与旧数据不兼容
- 非管理员执行了升级

#### 处理策略
```python
# 验证升级后数据完整性
value_before = proxy.getValue()
proxy.upgradeTo(new_logic.address, sender=admin)
value_after = proxy.getValue()
assert value_after == value_before

# 验证升级权限
with reverts():
    proxy.upgradeTo(new_logic.address, sender=non_admin)
```

#### 关联文档
- `error_patterns.md` - 合约版本不兼容
- `troubleshooting_guide.md` - 合约升级问题排查


---

## 二、网络层面故障

### 2.1 连接故障

#### 故障类型
- RPC 连接失败
- 网络超时
- 节点不可用

#### 特征描述
- 无法连接到区块链网络
- 请求超时
- RPC 节点响应异常

#### 处理策略
```python
# 检查网络连接
from ape import networks

provider = networks.active_provider()
if not provider.is_connected():
    log.error("网络连接失败")

# 切换备用节点
with networks.parse_network_choice("ethereum:goerli:alchemy"):
    # 执行操作
    pass
```

#### 关联文档
- `error_patterns.md` - 网络超时错误、网络配置错误
- `troubleshooting_guide.md` - 网络连接问题排查


### 2.2 交易故障

#### 故障类型
- Gas 超限
- 交易卡住
- 交易失败

#### 特征描述
- 交易 Gas 消耗超出限制
- 交易长时间未确认
- 交易执行失败

#### 处理策略
```python
# 设置合理的 Gas 限制
tx = contract.function(args, sender=user, gas_limit=1000000)

# 检查交易状态
tx_receipt = tx.receipt
if tx_receipt.status == 0:
    revert_reason = tx_receipt.revert_reason
    log.error(f"交易失败: {revert_reason}")
```

#### 关联文档
- `error_patterns.md` - Gas 超限错误
- `troubleshooting_guide.md` - Gas 消耗异常、交易失败排查


### 2.3 数据同步故障

#### 故障类型
- 区块同步延迟
- 数据不一致
- 链上状态查询异常

#### 特征描述
- 查询数据与实际链上状态不符
- 区块高度异常
- 事件查询缺失

#### 处理策略
```python
# 等待区块确认
provider.set_block_timeout(12)
tx.wait()

# 验证链上状态
balance = token.balanceOf(account)
assert balance > 0, "链上数据异常"
```

#### 关联文档
- `error_patterns.md` - 数据断言失败
- `troubleshooting_guide.md` - 资金对账不一致排查


---

## 三、测试层面故障

### 3.1 数据准备故障

#### 故障类型
- 测试数据错误
- 环境配置不当
- 状态初始化失败

#### 特征描述
- 测试数据不符合预期
- 测试环境未正确设置
- 合约状态初始化失败

#### 处理策略
```python
# 使用 fixtures 准备测试数据
@pytest.fixture
def erc20_token_with_balance(deployer, user1):
    token = deployer.deploy(ERC20Token)
    token.mint(user1, parse_ether("1000"), sender=deployer)
    return token

# 验证测试数据
balance = token.balanceOf(user1)
assert balance == parse_ether("1000"), "测试数据准备错误"
```

#### 关联文档
- `error_patterns.md` - 数据断言失败


### 3.2 断言逻辑故障

#### 故障类型
- 断言条件错误
- 预期值计算错误
- 类型转换错误

#### 特征描述
- 断言失败但实际逻辑正确
- 预期值与实际值类型不匹配
- 数值计算错误

#### 处理策略
```python
# 使用正确的类型转换
from framework.core.formatters import parse_ether, format_ether

amount = parse_ether("100")  # 字符串转 Wei
balance_after = balance_before - amount

# 清晰的断言消息
assert balance_after == expected_balance, \
    f"余额不符，预期: {format_ether(expected_balance)}, 实际: {format_ether(balance_after)}"
```

#### 关联文档
- `error_patterns.md` - 数据断言失败、类型转换错误


### 3.3 测试执行故障

#### 故障类型
- 测试超时
- 测试顺序依赖
- 资源竞争

#### 特征描述
- 测试执行时间过长
- 测试结果依赖执行顺序
- 并发测试资源冲突

#### 处理策略
```python
# 设置测试超时
@pytest.mark.timeout(60)
def test_long_running_test():
    # 测试逻辑
    pass

# 使用隔离的测试环境
@pytest.fixture(autouse=True)
def isolate(fn_isolation):
    pass

# 避免测试顺序依赖
def test_independent_1():
    pass

def test_independent_2():
    pass
```

#### 关联文档
- `error_patterns.md` - 合约交互超时


---

## 四、业务逻辑层面故障

### 4.1 清算逻辑故障

#### 故障类型
- 健康因子计算错误
- 清算条件判断错误
- 重复清算
- 清算奖励计算错误

#### 特征描述
- 健康因子计算不正确
- 非清算条件下执行了清算
- 已清算仓位再次被清算
- 清算奖励分配错误

#### 处理策略
```python
# 验证健康因子
health_factor = liquidation_contract.get_health_factor(user)
threshold = liquidation_contract.HEALTH_FACTOR_THRESHOLD()
assert health_factor < threshold, "不满足清算条件"

# 验证重复清算防护
liquidation_contract.liquidate(user, sender=liquidator)
with reverts():
    liquidation_contract.liquidate(user, sender=liquidator)  # 预期失败
```

#### 关联文档
- `exception_scenarios.md` - 重复清算、非清算条件拒绝场景
- `troubleshooting_guide.md` - 清算异常排查


### 4.2 交易逻辑故障

#### 故障类型
- 余额不足
- 授权超限
- 滑点超限
- 价格预言机异常

#### 特征描述
- 转账金额超过余额
- 授权额度不足
- 交易滑点超出允许范围
- 预言机价格异常

#### 处理策略
```python
# 检查余额
balance = token.balanceOf(sender)
assert balance >= amount, "余额不足"

# 检查授权
allowance = token.allowance(sender, spender)
assert allowance >= amount, "授权不足"

# 设置滑点保护
min_amount_out = int(amount_out * 0.95)  # 5% 滑点保护
```

#### 关联文档
- `exception_scenarios.md` - 余额不足、授权超限场景
- `error_patterns.md` - 授权未设置错误


### 4.3 质押/挖矿故障

#### 故障类型
- 质押余额错误
- 奖励计算错误
- 解押失败
- 时间锁异常

#### 特征描述
- 质押余额与实际不符
- 奖励计算不正确
- 解押操作失败
- 时间锁未正确生效

#### 处理策略
```python
# 验证质押余额
staking.stake(amount, sender=user)
stake_balance = staking.userInfo(user)[0]
assert stake_balance == amount, "质押余额错误"

# 验证时间锁
timelock_contract.lock(amount, sender=user)
with reverts():
    timelock_contract.releaseByTime(sender=user)  # 未到期时预期失败
```

#### 关联文档
- `exception_scenarios.md` - 时间锁未到期场景
- `troubleshooting_guide.md` - 资金对账不一致排查


### 4.4 授权管理故障

#### 故障类型
- 授权设置失败
- 授权额度异常
- 无限授权风险
- 授权清零异常

#### 特征描述
- 授权操作未生效
- 授权额度与预期不符
- 无限授权未被拦截
- 授权清零后仍可操作

#### 处理策略
```python
# 设置授权并验证
token.approve(spender, amount, sender=owner)
allowance = token.allowance(owner, spender)
assert allowance == amount, "授权失败"

# 拦截无限授权
infinite_amount = 2**256 - 1
with reverts():
    token.approve(spender, infinite_amount, sender=owner)

# 验证授权清零
token.approve(spender, 0, sender=owner)
with reverts():
    token.transferFrom(owner, recipient, 1, sender=spender)
```

#### 关联文档
- `exception_scenarios.md` - 授权超限、无限授权拦截场景
- `troubleshooting_guide.md` - 授权问题排查


---

## 故障分类速查表

| 故障类型 | 分类维度 | 处理策略 | 关联文档 |
|---------|---------|---------|---------|
| 权限拒绝 | 合约层面 | 检查角色配置 | exception_scenarios.md |
| 重入攻击 | 合约层面 | 重入锁保护 | exception_scenarios.md |
| 整数溢出 | 合约层面 | Solidity 0.8+ 内置检查 | exception_scenarios.md |
| Gas 超限 | 网络层面 | 增加 Gas 限制 | error_patterns.md |
| 网络超时 | 网络层面 | 切换 RPC 节点 | error_patterns.md |
| 数据断言失败 | 测试层面 | 验证测试数据 | error_patterns.md |
| 清算异常 | 业务逻辑层面 | 检查健康因子 | troubleshooting_guide.md |
| 余额不足 | 业务逻辑层面 | 验证余额 | exception_scenarios.md |
| 授权问题 | 业务逻辑层面 | 设置正确授权 | troubleshooting_guide.md |
| 合约升级 | 合约层面 | 验证数据完整性 | troubleshooting_guide.md |

---

## 故障处理流程

### 1. 故障发现
- 测试失败
- 监控告警
- 用户反馈

### 2. 故障分类
- 根据上述分类体系定位故障维度
- 识别具体故障类型

### 3. 故障定位
- 参考对应的排查指南
- 收集相关日志和数据
- 复现故障场景

### 4. 故障修复
- 根据处理策略实施修复
- 验证修复效果
- 编写回归测试

### 5. 故障记录
- 更新故障案例文档
- 添加到测试用例库
- 分享经验教训
