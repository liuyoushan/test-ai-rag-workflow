# 清算测试用例详细文档

## 文档信息
- **文档类型**: 测试用例详细文档
- **分类**: 清算系统
- **用例数量**: 11个
- **代码位置**: `/home/liuyoushan/ape-demo/tests/liquidation/scenarios/`
- **测试数据**: `/home/liuyoushan/ape-demo/tests/liquidation/data/test_liquidation.yaml`
- **合约**: Liquidation, MyERC20 (抵押品/债务代币)

---

## 测试数据配置

| 用例数据键 | 关键字段 | 说明 |
|-----------|---------|------|
| `case_048_liquidation_trigger` | health_factor_warning (1.2), health_factor_liquidation (1.0), collateral_ratio_initial (1.5), debt_amount (1000), collateral_amount (2000) | 清算触发条件测试 |
| `case_049_normal_liquidation` | liquidator_reward_pct (0.1), platform_fee_pct (0.02), debt_amount (1000), expected_liquidator_reward (100), collateral_amount (1100), adjusted_debt (1000) | 正常清算流程 |
| `case_052_liquidation_reward` | bonus_pct (0.1), platform_tax_pct (0.02), platform_address | 清算奖励计算 |
| `case_053_batch_liquidation` | user_count (3), total_debt (3000) | 批量清算 |
| `case_054_price_oracle_manipulation` | oracle_fake_price_high (2.0), oracle_fake_price_low (0.5) | 预言机操纵 |
| `case_055_reentrancy_attack` | debt_amount (1000), collateral_amount (1500), reentrancy_attempts (3) | 重入攻击防护 |
| `case_056_flash_loan_attack` | loan_amount (100000), target_collateral (2000), target_debt (1000), expected_protection (true) | 闪电贷攻击 |
| `case_057_duplicate_liquidation` | debt_amount (1000), collateral_amount (1500) | 重复清算防护 |

---

## case_048 清算触发条件测试

### 基本信息
- **用例编号**: case_048
- **用例名称**: 清算触发条件测试
- **优先级**: P0
- **分类**: 清算 / 功能测试 / 清算
- **代码参考**: `tests/liquidation/scenarios/test_liquidation_flow.py:28`

### 前置条件
1. 清算合约已部署
2. 抵押品代币和债务代币已部署
3. 用户账户存在

### 测试步骤
1. 用户授权清算合约使用抵押品
2. 用户存入抵押品
3. 用户借入债务
4. 计算并验证健康因子

### 预期结果
- 用户抵押品余额正确记录
- 用户债务余额正确记录
- 健康因子 = 抵押品 / 债务
- 健康因子高于阈值时不可清算
- 健康因子低于阈值时可清算

### 关联测试数据
- 数据键: `case_048_liquidation_trigger`
- 关键字段: `health_factor_warning` (1.2), `health_factor_liquidation` (1.0), `collateral_ratio_initial` (1.5), `debt_amount` (1000), `collateral_amount` (2000)

### 断言规范
```python
assert health_factor == expected_hf
# 健康因子 = 抵押品 / 债务 = 2000 / 1000 = 2
```

---

## case_049 正常清算流程测试

### 基本信息
- **用例编号**: case_049
- **用例名称**: 正常清算流程测试
- **优先级**: P0
- **分类**: 清算 / 功能测试
- **代码参考**: `tests/liquidation/scenarios/test_liquidation_scenarios.py:29`

### 前置条件
1. 清算合约已部署
2. 抵押品代币和债务代币已部署
3. 借款人有抵押品和债务
4. 健康因子低于清算阈值

### 测试步骤
1. 设置用户仓位（抵押品和债务）
2. 准备代币（给清算合约充值抵押品，给清算人充值债务代币）
3. 清算人授权清算合约使用债务代币
4. 执行清算操作
5. 验证借款人状态
6. 验证清算人收益

### 预期结果
- 借款人债务清零
- 借款人标记为已清算
- 清算人获得抵押品 = 债务 + 清算奖励（10%）
- 抵押品余额正确扣除

### 关联测试数据
- 数据键: `case_049_normal_liquidation`
- 关键字段: `liquidator_reward_pct` (0.1), `platform_fee_pct` (0.02), `debt_amount` (1000), `expected_liquidator_reward` (100), `collateral_amount` (1100), `adjusted_debt` (1000)

### 断言规范
```python
assert user_debt == 0
assert is_liquidated == True
assert actual_user2_collateral_gain == expected_user2_collateral
```

---

## case_050 清算后状态校验

### 基本信息
- **用例编号**: case_050
- **用例名称**: 清算后状态校验
- **优先级**: P0
- **分类**: 清算 / 功能测试
- **代码参考**: `tests/liquidation/scenarios/test_liquidation_scenarios.py:92`

### 前置条件
1. 清算合约已部署
2. 借款人已被清算

### 测试步骤
1. 设置仓位并执行清算
2. 验证借款人债务状态
3. 验证清算标记
4. 验证抵押品扣除
5. 验证清算人收益

### 预期结果
- 借款人债务为 0
- 借款人标记为已清算
- 抵押品按规则扣除
- 清算人获得相应奖励

### 关联测试数据
- 数据键: `case_049_normal_liquidation`
- 关键字段: `collateral_amount` (1100), `adjusted_debt` (1000)

### 断言规范
```python
assert user_debt == 0
assert is_liquidated == True
assert actual_collateral == expected_collateral
assert user2_collateral == expected_user2_collateral
```

---

## case_051 非清算条件拒绝测试

### 基本信息
- **用例编号**: case_051
- **用例名称**: 非清算条件拒绝测试
- **优先级**: P1
- **分类**: 清算 / 功能测试
- **代码参考**: `tests/liquidation/scenarios/test_liquidation_scenarios.py:346`
- **状态**: 待实现 (NotImplementedError)

### 前置条件
1. 清算合约已部署
2. 用户健康因子正常（高于阈值）

### 测试步骤
1. 设置健康的用户仓位
2. 尝试执行清算
3. 验证清算被拒绝

### 预期结果
- 健康仓位无法被清算
- 交易 revert
- 用户状态保持不变

### 关联测试数据
- 无特定测试数据

---

## case_052 清算奖励计算测试

### 基本信息
- **用例编号**: case_052
- **用例名称**: 清算奖励计算测试
- **优先级**: P1
- **分类**: 清算 / 功能测试
- **代码参考**: `tests/liquidation/scenarios/test_liquidation_scenarios.py:360`
- **状态**: 待实现 (NotImplementedError)

### 前置条件
1. 清算合约已部署
2. 清算奖励参数已配置

### 测试步骤
1. 设置可清算的仓位
2. 执行清算
3. 验证清算人奖励金额
4. 验证平台抽成金额
5. 验证借款人罚金

### 预期结果
- 清算人获得适当的清算奖励（债务的 10%）
- 平台收取手续费（债务的 2%）
- 计算精度和边界条件正确

### 关联测试数据
- 数据键: `case_052_liquidation_reward`
- 关键字段: `bonus_pct` (0.1), `platform_tax_pct` (0.02), `platform_address`

---

## case_053 批量清算场景测试

### 基本信息
- **用例编号**: case_053
- **用例名称**: 批量清算场景测试
- **优先级**: P1
- **分类**: 清算 / 功能测试
- **代码参考**: `tests/liquidation/scenarios/test_liquidation_scenarios.py:375`
- **状态**: 待实现 (NotImplementedError)

### 前置条件
1. 清算合约已部署
2. 多个用户仓位可清算

### 测试步骤
1. 设置多个可清算的用户仓位
2. 执行批量清算
3. 验证各用户状态
4. 验证资产不冲突、不超额

### 预期结果
- 多用户同时触发清算
- 系统性能和状态一致性
- 并发清算时的资源分配正确

### 关联测试数据
- 数据键: `case_053_batch_liquidation`
- 关键字段: `user_count` (3), `total_debt` (3000)

---

## case_054 价格预言机操纵边界测试

### 基本信息
- **用例编号**: case_054
- **用例名称**: 价格预言机操纵边界测试
- **优先级**: P1
- **分类**: 清算 / 安全测试
- **代码参考**: `tests/liquidation/scenarios/test_liquidation_scenarios.py:390`
- **状态**: 待实现 (NotImplementedError)

### 前置条件
1. 清算合约已部署
2. 预言机合约已部署

### 测试步骤
1. 设置正常用户仓位
2. 尝试操纵预言机价格
3. 验证正常用户不被恶意清算
4. 验证价格偏差检测

### 预期结果
- 防止预言机价格被操纵导致恶意清算
- 价格偏差检测和保护生效
- 恶意价格无法非法清算正常用户

### 关联测试数据
- 数据键: `case_054_price_oracle_manipulation`
- 关键字段: `oracle_fake_price_high` (2.0), `oracle_fake_price_low` (0.5)

---

## case_055 重入攻击防护测试

### 基本信息
- **用例编号**: case_055
- **用例名称**: 重入攻击防护测试
- **优先级**: P0
- **分类**: 清算 / 安全测试
- **代码参考**: `tests/liquidation/scenarios/test_liquidation_scenarios.py:150`

### 前置条件
1. 清算合约已部署
2. 恶意攻击合约可部署
3. 用户有可清算的仓位

### 测试步骤
1. 用户存入抵押品并借入债务
2. 调整仓位满足清算条件
3. 部署恶意攻击合约
4. 执行重入攻击
5. 验证攻击被拦截
6. 验证清算正常完成

### 预期结果
- 重入攻击被拦截（使用重入锁和 Check-Effects-Interaction 模式）
- 恶意攻击者无法通过递归调用窃取资产
- 清算仍能正常完成
- 用户债务清零

### 关联测试数据
- 数据键: `case_055_reentrancy_attack`
- 关键字段: `debt_amount` (1000), `collateral_amount` (1500), `reentrancy_attempts` (3)

### 断言规范
```python
assert attack_success == False
assert is_liquidated == True
assert user_debt == 0
```

---

## case_056 闪电贷价格操纵测试

### 基本信息
- **用例编号**: case_056
- **用例名称**: 闪电贷价格操纵测试
- **优先级**: P1
- **分类**: 清算 / 安全测试
- **代码参考**: `tests/liquidation/scenarios/test_liquidation_scenarios.py:214`

### 前置条件
1. 清算合约已部署
2. 闪电贷合约可部署
3. 攻击者合约可部署

### 测试步骤
1. 设置正常用户仓位
2. 部署闪电贷和攻击者合约
3. 执行闪电贷攻击
4. 验证用户状态正常

### 预期结果
- 攻击者无法通过闪电贷操纵价格进行恶意清算
- 清算条件判断不受临时价格波动影响
- 用户状态保持正常

### 关联测试数据
- 数据键: `case_056_flash_loan_attack`
- 关键字段: `loan_amount` (100000), `target_collateral` (2000), `target_debt` (1000), `expected_protection` (true)

### 断言规范
```python
assert user_debt in (0, debt_amount)
```

---

## case_057 重复清算防护测试

### 基本信息
- **用例编号**: case_057
- **用例名称**: 重复清算防护测试
- **优先级**: P0
- **分类**: 清算 / 安全测试
- **代码参考**: `tests/liquidation/scenarios/test_liquidation_scenarios.py:276`

### 前置条件
1. 清算合约已部署
2. 用户有可清算的仓位

### 测试步骤
1. 设置用户仓位
2. 执行第一次清算
3. 验证第一次清算成功
4. 尝试第二次清算
5. 验证第二次清算被拒绝
6. 验证状态保持一致

### 预期结果
- 已清算的仓位不能再次被清算
- 第二次清算尝试被拒绝
- 清算状态保持一致
- canLiquidate 返回 False

### 关联测试数据
- 数据键: `case_057_duplicate_liquidation`
- 关键字段: `debt_amount` (1000), `collateral_amount` (1500)

### 断言规范
```python
# 第一次清算后
assert is_liquidated == True
assert user_debt == 0
# 第二次清算尝试失败（抛异常）
# 状态不变
assert is_liquidated == True
assert user_debt == 0
assert can_liquidate == False
```

---

## case_058 漏洞合约攻击测试

### 基本信息
- **用例编号**: case_058
- **用例名称**: 漏洞合约攻击测试
- **优先级**: P0
- **分类**: 清算 / 安全测试
- **代码参考**: `tests/liquidation/scenarios/test_liquidation_scenarios.py:404`

### 前置条件
1. 清算合约已部署
2. 恶意攻击合约可部署
3. 用户有可清算的仓位

### 测试步骤
1. 设置用户仓位
2. 部署恶意攻击合约
3. 执行漏洞攻击
4. 验证攻击失败
5. 验证清算正常完成
6. 重置状态验证

### 预期结果
- 漏洞合约攻击被拦截
- 重入攻击防护生效
- 恶意回调防护生效
- 清算仍能正常完成
- 状态重置功能正常

### 关联测试数据
- 数据键: `case_055_reentrancy_attack`
- 关键字段: `collateral_amount` (1500), `debt_amount` (1000)

### 断言规范
```python
assert attack_success == False
assert is_liquidated == True
```

---

## 测试文件结构

```
tests/liquidation/
├── scenarios/
│   ├── test_liquidation_flow.py       # 清算流程测试
│   └── test_liquidation_scenarios.py  # 完整清算场景
├── data/
│   └── test_liquidation.yaml          # 测试数据
├── apis/
│   └── liquidation_api.py             # API 封装
├── fixtures/
│   └── liquidation_fixtures.py        # Fixture
└── conftest.py                        # 配置
```

---

## 核心 Fixture 列表

| Fixture 名称 | 说明 |
|-------------|------|
| `liquidation_env` | 清算环境（包含 API、代币） |
| `liquidation_api` | 清算 API 实例 |
| `collateral_token` | 抵押品代币 |
| `debt_token` | 债务代币 |
| `liquidation_contract` | 清算合约实例 |
| `liquidation_test_data` | 清算测试数据 |
