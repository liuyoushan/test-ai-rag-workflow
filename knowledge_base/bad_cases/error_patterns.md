# 错误模式库

## 1. 合约 Revert 错误

### 错误类型
- EVM 执行失败（Revert）

### 错误信息特征
- `reverted with reason string`
- `Transaction reverted without a reason string`
- `VM Exception while processing transaction: revert`

### 触发原因
- 余额不足
- 权限不足
- 条件检查失败
- 数组长度不匹配
- 授权超限

### 修复建议
1. 检查调用者权限和角色配置
2. 验证账户余额是否充足
3. 确认授权额度是否足够
4. 检查函数参数条件
5. 使用 `reverts()` 上下文管理器捕获预期错误

### 代码示例
```python
from ape import reverts

# 捕获预期的 revert
with reverts():
    erc20_api.transfer(user1, "1000", sender=user2)
```


## 2. Gas 超限错误

### 错误类型
- GasLimitExceeded
- OutOfGas

### 错误信息特征
- `out of gas`
- `Gas limit exceeded`
- `Transaction ran out of gas`

### 触发原因
- `gas_limit` 设置过低
- 循环/递归执行次数过多
- 复杂计算超出 Gas 预算

### 修复建议
1. 增加 `gas_limit` 参数
2. 优化合约代码减少 Gas 消耗
3. 拆分大交易为多个小交易
4. 使用批量操作减少交易次数

### 代码示例
```python
# 设置足够的 Gas 限制
tx = erc20_token.transfer(user2, parse_ether("10"), sender=user1, gas_limit=100000)
```


## 3. 网络超时错误

### 错误类型
- NetworkTimeout
- ConnectionError

### 错误信息特征
- `TimeoutError`
- `Connection refused`
- `Unable to connect to provider`

### 触发原因
- RPC 节点不稳定
- 网络连接中断
- 节点响应过慢

### 修复建议
1. 切换到稳定的 RPC 节点
2. 增加超时配置
3. 实现重试机制
4. 检查网络连接状态

### 代码示例
```python
from ape import networks

# 使用备用 RPC 节点
with networks.parse_network_choice("ethereum:mainnet:infura"):
    # 执行操作
    pass
```


## 4. 数据断言失败

### 错误类型
- AssertionError

### 错误信息特征
- `AssertionError: expected X, got Y`
- `Assertion failed`

### 触发原因
- 合约返回值与预期不符
- 余额计算错误
- 事件参数不匹配
- 状态变更不符合预期

### 修复建议
1. 检查测试数据和预期值
2. 验证合约逻辑正确性
3. 增加调试日志
4. 检查链上状态

### 代码示例
```python
# 验证余额变更
balance_before = erc20_api.get_balance(user1)
erc20_api.transfer(user2, "100", sender=user1)
balance_after = erc20_api.get_balance(user1)

assert balance_after == balance_before - parse_ether("100"), \
    f"余额不符，预期: {format_ether(balance_before - parse_ether('100'))}, 实际: {format_ether(balance_after)}"
```


## 5. 合约部署失败

### 错误类型
- ContractDeploymentError

### 错误信息特征
- `Failed to deploy contract`
- `Invalid bytecode`
- `Insufficient funds for gas * price + value`

### 触发原因
- 构造函数参数错误
- 部署者余额不足
- 合约代码有问题
- 链上状态冲突

### 修复建议
1. 检查构造函数参数
2. 确保部署者有足够资金
3. 验证合约代码编译正确
4. 检查链上资源状态

### 代码示例
```python
# 确保部署者有足够资金
deployer_balance = deployer.balance
assert deployer_balance > parse_ether("1"), "部署者余额不足"

# 部署合约
contract = deployer.deploy(MyContract, constructor_arg1, constructor_arg2)
```


## 6. 事件缺失错误

### 错误类型
- EventNotFound

### 错误信息特征
- `Event not found`
- `Expected event not emitted`

### 触发原因
- 合约未触发预期事件
- 事件解码失败
- 交易未成功执行

### 修复建议
1. 检查合约代码是否触发事件
2. 验证事件签名和参数
3. 使用 `decode_logs` 正确解码
4. 确认交易已成功

### 代码示例
```python
# 解码事件
tx = erc20_api.transfer(user2, "100", sender=user1)
transfer_event = erc20_api.decode_transfer_event(tx)

assert transfer_event["from"] == user1
assert transfer_event["to"] == user2
assert transfer_event["value"] == parse_ether("100")
```


## 7. 授权未设置错误

### 错误类型
- AllowanceError

### 错误信息特征
- `ERC20: insufficient allowance`
- `Transfer amount exceeds allowance`

### 触发原因
- 未调用 `approve` 设置授权
- 授权额度已用完
- 授权已被清零

### 修复建议
1. 确保先调用 `approve` 设置授权
2. 检查 allowance 是否足够
3. 处理授权过期情况

### 代码示例
```python
# 设置授权
erc20_token.approve(spender, parse_ether("1000"), sender=owner)

# 检查授权额度
allowance = erc20_token.allowance(owner, spender)
assert allowance >= parse_ether("500"), "授权额度不足"

# 使用授权
erc20_token.transferFrom(owner, recipient, parse_ether("500"), sender=spender)
```


## 8. 合约交互超时

### 错误类型
- ContractCallTimeout

### 错误信息特征
- `Contract call timed out`
- `Execution took too long`

### 触发原因
- 复杂计算耗时过长
- 链上拥堵
- RPC 节点响应慢

### 修复建议
1. 优化合约计算逻辑
2. 使用异步调用
3. 增加超时配置
4. 选择低拥堵时段执行

### 代码示例
```python
# 使用自定义超时配置
with networks.active_provider() as provider:
    provider.set_timeout(60)  # 设置超时为 60 秒
    result = my_contract.complexCalculation(sender=user1)
```


## 9. 状态不一致错误

### 错误类型
- StateInconsistency

### 错误信息特征
- `State mismatch`
- `Data inconsistency detected`

### 触发原因
- 并发操作导致竞态条件
- 交易部分执行失败
- 合约逻辑缺陷

### 修复建议
1. 使用重入锁保护关键代码
2. 实现状态回滚机制
3. 增加状态校验逻辑
4. 使用 Check-Effects-Interaction 模式

### 代码示例
```python
# Check-Effects-Interaction 模式
def withdraw(amount):
    # Check
    require(balances[msg.sender] >= amount, "Insufficient balance")
    
    # Effects
    balances[msg.sender] -= amount
    
    # Interaction
    msg.sender.transfer(amount)
```


## 10. 类型转换错误

### 错误类型
- TypeError
- ConversionError

### 错误信息特征
- `Cannot convert X to Y`
- `Type mismatch`

### 触发原因
- 数值类型不匹配
- 地址格式错误
- 编码/解码失败

### 修复建议
1. 确保类型一致
2. 使用正确的格式化工具
3. 验证数据编码格式
4. 使用 `parse_ether`/`format_ether` 处理金额

### 代码示例
```python
from framework.core.formatters import parse_ether, format_ether

# 正确的数值转换
amount = parse_ether("100")  # 字符串转 Wei
formatted = format_ether(amount)  # Wei 转字符串

# 错误示例：直接使用整数
wrong_amount = 100  # 这是 100 Wei，不是 100 ETH
```


## 11. 合约版本不兼容

### 错误类型
- VersionMismatch

### 错误信息特征
- `ABI version mismatch`
- `Function not found`
- `Invalid contract interface`

### 触发原因
- ABI 与合约实际代码不一致
- 合约已升级但 ABI 未更新
- 使用错误的合约地址

### 修复建议
1. 更新 ABI 文件
2. 验证合约地址正确性
3. 使用正确的合约接口
4. 检查代理合约实现

### 代码示例
```python
# 使用正确的合约接口
proxy_v2 = project.LogicV2.at(proxy.address)

# 验证版本
version = proxy_v2.getVersion()
assert version == "V2", f"版本不符，预期: V2, 实际: {version}"
```


## 12. 网络配置错误

### 错误类型
- NetworkConfigError

### 错误信息特征
- `Unknown network`
- `Invalid network configuration`
- `Provider not found`

### 触发原因
- 网络名称拼写错误
- 缺少网络配置
- 依赖的插件未安装

### 修复建议
1. 检查 `ape-config.yaml` 配置
2. 安装必要的网络插件
3. 使用正确的网络标识
4. 验证 RPC 配置

### 代码示例
```yaml
# ape-config.yaml 配置示例
ethereum:
  mainnet:
    default_provider: infura
  goerli:
    default_provider: alchemy
```
