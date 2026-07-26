# DEX Swap 测试用例详细文档

## 文档信息
- **文档类型**: 测试用例详细文档
- **分类**: DEX 去中心化交易所
- **用例数量**: 10个主用例 + 1个V3用例
- **代码位置**: `/home/liuyoushan/ape-demo/tests/dex_swap/scenarios/`
- **测试数据**: 
  - V2: `/home/liuyoushan/ape-demo/tests/dex_swap/data/test_dex_swap.yaml`
  - V3: `/home/liuyoushan/ape-demo/tests/dex_swap/data/test_dex_swap_v3.yaml`
- **合约**: MiniSwapFactory, MiniSwapPair, MiniSwapRouter

---

## 测试数据配置

### 通用配置
| 配置项 | 值 | 说明 |
|-------|----|------|
| tokenA_name | "TokenA" | 交易对代币A名称 |
| tokenA_symbol | "TKA" | 交易对代币A符号 |
| tokenB_name | "TokenB" | 交易对代币B名称 |
| tokenB_symbol | "TKB" | 交易对代币B符号 |

---

## case_010 正向单池 Swap 兑换

### 基本信息
- **用例编号**: case_010
- **用例名称**: 正向单池 Swap 兑换
- **优先级**: P0
- **分类**: DEX / 功能测试 / Swap / 正向测试
- **代码参考**: `tests/dex_swap/scenarios/test_dex_swap_complex.py:30`

### 前置条件
1. TokenA 和 TokenB 合约已部署
2. Factory 和 Router 合约已部署
3. 用户持有足够的 TokenA

### 测试步骤
1. 给用户铸造两种代币
2. 用户授权 Router 使用代币
3. 添加流动性创建交易对
4. 记录兑换前余额和池子储备金
5. 授权兑换金额
6. 执行 TokenA → TokenB 兑换
7. 验证兑换后余额
8. 验证 K 值守恒

### 预期结果
- 发送方 TokenA 余额减少
- 接收方 TokenB 余额增加
- 池子储备金变化符合 AMM 算法
- K 值（reserveA * reserveB）不减少（手续费导致增加）

### 关联测试数据
- 数据键: `case_010_swap_tokenA_to_tokenB`
- 关键字段: `mint_amount` (10000 ether), `add_liquidity_amount` (1000 ether), `swap_amount` (100 ether)

### 断言规范
```python
assert balance_A_after == balance_A_before - swap_amount
assert balance_B_after > balance_B_before
assert k_after >= k_before
```

---

## case_011 反向单池 Swap 兑换

### 基本信息
- **用例编号**: case_011
- **用例名称**: 反向单池 Swap 兑换
- **优先级**: P0
- **分类**: DEX / Swap / 反向测试
- **代码参考**: `tests/dex_swap/scenarios/test_dex_swap_complex.py:110`

### 前置条件
1. TokenA 和 TokenB 合约已部署
2. Factory 和 Router 合约已部署
3. 交易对已创建并有流动性

### 测试步骤
1. 给用户铸造代币并添加流动性
2. 记录兑换前余额
3. 授权兑换金额
4. 执行 TokenB → TokenA 反向兑换
5. 验证兑换结果

### 预期结果
- TokenB 余额减少指定金额
- TokenA 余额增加
- 反向兑换逻辑与正向兑换一致

### 关联测试数据
- 数据键: `case_011_swap_tokenB_to_tokenA`
- 关键字段: `mint_amount` (10000 ether), `add_liquidity_amount` (2000 ether), `swap_amount` (200 ether)

### 断言规范
```python
assert balance_B_after == balance_B_before - swap_amount
assert balance_A_after > balance_A_before
```

---

## case_012 添加双边流动性测试

### 基本信息
- **用例编号**: case_012
- **用例名称**: 添加双边流动性测试
- **优先级**: P0
- **分类**: DEX / 流动性 / AddLiquidity
- **代码参考**: `tests/dex_swap/scenarios/test_dex_swap_complex.py:172`

### 前置条件
1. TokenA 和 TokenB 合约已部署
2. Factory 和 Router 合约已部署
3. 用户持有足够的两种代币

### 测试步骤
1. 给用户铸造两种代币
2. 记录添加前余额
3. 用户授权 Router 使用代币
4. 调用 addLiquidity 添加流动性
5. 验证余额变化
6. 验证 LP 代币数量
7. 验证池子储备金

### 预期结果
- 用户两种代币余额减少
- 用户获得 LP 代币
- LP 代币数量符合公式 sqrt(a*b)
- 池子储备金正确更新

### 关联测试数据
- 数据键: `case_012_add_liquidity`
- 关键字段: `mint_amount` (10000 ether), `add_liquidity_amount_a` (1000 ether), `add_liquidity_amount_b` (2000 ether)

### 断言规范
```python
assert balance_A_after == balance_A_before - add_liquidity_amount_a
assert balance_B_after == balance_B_before - add_liquidity_amount_b
assert abs(lp_balance - expected_lp_wei) < tolerance_wei
assert lp_balance > 0
assert reserves[0] == add_liquidity_amount_a
assert reserves[1] == add_liquidity_amount_b
```

---

## case_012_extend LP 占比校验

### 基本信息
- **用例编号**: case_012_extend
- **用例名称**: LP 占比校验
- **优先级**: P0
- **分类**: DEX / 流动性 / LP
- **代码参考**: `tests/dex_swap/scenarios/test_dex_swap_complex.py:250`

### 前置条件
1. TokenA 和 TokenB 合约已部署
2. Factory 和 Router 合约已部署
3. 两个用户账户存在

### 测试步骤
1. 给两个用户铸造代币
2. 用户1添加流动性
3. 验证用户1持有全部 LP
4. 用户2添加流动性
5. 验证 LP 按贡献比例分配

### 预期结果
- 第一个用户添加后获得全部 LP
- 第二个用户添加后 LP 按贡献比例分配
- 用户持有的 LP 占比与流动性贡献匹配（约 2:1）

### 关联测试数据
- 数据键: `case_012_1_lp_percentage_check`
- 关键字段: `mint_amount` (20000 ether), `user1_add_a` (1000 ether), `user1_add_b` (2000 ether), `user2_add_a` (500 ether), `user2_add_b` (1000 ether)

### 断言规范
```python
assert u1_lp_after_t1 == total_after_t1
assert pct1 in (66, 67)
assert pct2 in (33, 34)
assert pct1 + pct2 in (99, 100)
```

---

## case_013 移除流动性测试

### 基本信息
- **用例编号**: case_013
- **用例名称**: 移除流动性测试
- **优先级**: P0
- **分类**: DEX / 流动性 / RemoveLiquidity
- **代码参考**: `tests/dex_swap/scenarios/test_dex_swap_complex.py:337`

### 前置条件
1. TokenA 和 TokenB 合约已部署
2. Factory 和 Router 合约已部署
3. 用户已添加流动性并持有 LP

### 测试步骤
1. 给用户铸造代币
2. 记录添加前余额
3. 添加流动性
4. 记录 LP 余额和总供应量
5. 移除 50% 流动性
6. 验证部分移除结果
7. 移除剩余流动性
8. 验证全部移除结果

### 预期结果
- 用户销毁 LP 代币
- 按比例赎回两种资产
- LP 余额和总供应量正确减少
- 全部赎回后 LP 余额为 0

### 关联测试数据
- 数据键: `case_013_remove_liquidity`
- 关键字段: `mint_amount` (10000 ether), `add_liquidity_amount_a` (1500 ether), `add_liquidity_amount_b` (1500 ether), `remove_lp_percent` ("50%")

### 断言规范
```python
assert tot_after < tot_before_rem
assert b_a_after > b_a_before - add_a
assert b_b_after > b_b_before - add_b
assert pair.balanceOf(user1) == 0
```

---

## case_014 滑点控制边界测试

### 基本信息
- **用例编号**: case_014
- **用例名称**: 滑点控制边界测试
- **优先级**: P0
- **分类**: DEX / 滑点 / 边界测试
- **代码参考**: `tests/dex_swap/scenarios/test_dex_swap_complex.py:419`

### 前置条件
1. TokenA 和 TokenB 合约已部署
2. Factory 和 Router 合约已部署
3. 交易对已创建并有流动性

### 测试步骤
1. 铸造代币并添加流动性
2. 使用预期金额作为最小输出，交易成功
3. 设置不可能的最小输出（大于预期），交易失败
4. 使用 2% 滑点容忍度，交易成功

### 预期结果
- 正常滑点参数下交易成功
- 设置过高的最小输出金额时交易失败（revert）
- 设置合理的滑点容忍度（如 2%）时交易成功

### 关联测试数据
- 数据键: `case_014_slippage_control`
- 关键字段: `mint_amount` (10000 ether), `add_liquidity_amount` (3000 ether), `swap_amount` (200 ether), `max_slippage_allow` ("2%"), `max_slippage_block` ("0.1%")

### 断言规范
```python
# 正常滑点交易成功（无异常）
# 过高最小输出交易失败
assert "insufficient" in str(e).lower() or "amount" in str(e).lower()
# 2%滑点容忍度交易成功
assert balance_B_after > mint_amount - add_liquidity_amount
```

---

## case_015 DEX 手续费结算测试

### 基本信息
- **用例编号**: case_015
- **用例名称**: DEX 手续费结算测试
- **优先级**: P0
- **分类**: DEX / 手续费 / 结算
- **代码参考**: `tests/dex_swap/scenarios/test_dex_swap_complex.py:507`

### 前置条件
1. TokenA 和 TokenB 合约已部署
2. Factory 和 Router 合约已部署
3. 交易对已创建并有流动性

### 测试步骤
1. 给用户铸造代币
2. 用户授权并添加流动性
3. 获取初始储备金和 K 值
4. 执行 3 次交易累积手续费
5. 获取交易后储备金和 K 值
6. 验证储备金增加和 K 值增长
7. 验证 K 值增长率在理论范围内

### 预期结果
- 交易手续费正确抽取
- 手续费计入池子储备金
- K 值随手续费累积而增长
- K 值增长率在理论范围内

### 关联测试数据
- 数据键: `case_015_fee_settlement`
- 关键字段: `mint_amount` (10000 ether), `add_liquidity_amount` (5000 ether), `swap_amount` (500 ether)

### 断言规范
```python
assert reserves_after[0] > reserves_before[0]
assert k_after >= k_before
assert k_growth_pct > theory_min and k_growth_pct < theory_max
```

---

## case_017 大额/极值交易边界测试

### 基本信息
- **用例编号**: case_017
- **用例名称**: 大额/极值交易边界测试
- **优先级**: P1
- **分类**: DEX / 大额交易 / 边界
- **代码参考**: `tests/dex_swap/scenarios/test_dex_swap_complex.py:670`

### 前置条件
1. 部署新的代币和 DEX 合约
2. 添加初始流动性

### 测试步骤
1. 部署代币合约
2. 给用户铸造代币
3. 部署 Factory 和 Router
4. 用户授权并添加流动性
5. 获取初始状态
6. 执行大额交易（占流动性的 75%）
7. 验证安全边界

### 预期结果
- 超大额交易不会导致池子枯竭
- 交易后双方储备金仍大于 0
- K 值保持守恒
- 防止整数溢出和价格操纵

### 关联测试数据
- 数据键: `case_017_large_trade_boundary`
- 关键字段: `mint_amount` (10000 ether), `add_liquidity_amount` (2000 ether), `large_swap_amount` (1500 ether)

### 断言规范
```python
assert reserves_after[0] > 0 and reserves_after[1] > 0
assert k_after >= k_before
assert balance_B_after > expected_min_balance
```

---

## case_055 V3 集中流动性添加测试

### 基本信息
- **用例编号**: case_055
- **用例名称**: V3 集中流动性添加测试
- **优先级**: P0
- **分类**: DEX / V3 / 集中流动性
- **代码参考**: `tests/dex_swap/scenarios/test_dex_swap_complex.py:588`

### 前置条件
1. V3 流动性环境已准备
2. TokenA 和 TokenB 合约已部署
3. Factory 和 Router 合约已部署

### 测试步骤
1. 加载测试数据和环境
2. 设置流动性金额
3. 用户授权 Router 使用代币
4. 第一轮添加全范围流动性
5. 验证第一轮添加成功
6. 第二轮添加窄范围流动性
7. 验证第二轮添加成功，流动性正确叠加

### 预期结果
- 第一轮添加后储备金正确
- LP 代币数量随流动性增加而增长
- 第二轮添加后储备金正确叠加
- 总流动性 = 全范围 + 窄范围

### 关联测试数据
- 数据键: `case_055_concentrated_liquidity_add`
- 关键字段: `add_liquidity_full_range` (1000), `add_liquidity_narrow_range` (500)

### 断言规范
```python
assert reserves_1[0] == add_full
assert reserves_1[1] == add_full
assert lp_balance_1 > 0
assert reserves_2[0] == expected_reserve
assert reserves_2[1] == expected_reserve
assert lp_balance_2 > lp_balance_1
```

---

## 测试文件结构

```
tests/dex_swap/
├── scenarios/
│   ├── test_dex_swap.py             # 基础交易测试
│   ├── test_dex_liquidity.py        # 流动性管理测试
│   └── test_dex_swap_complex.py     # 复杂交易测试
├── data/
│   ├── test_dex_swap.yaml           # V2 测试数据
│   └── test_dex_swap_v3.yaml        # V3 测试数据
├── apis/
│   └── dex_api.py                   # API 封装
├── fixtures/
│   └── dex_fixtures.py              # Fixture
└── conftest.py                      # 配置
```

---

## 核心 Fixture 列表

| Fixture 名称 | 说明 |
|-------------|------|
| `dex_token_a` | TokenA 合约实例 |
| `dex_token_b` | TokenB 合约实例 |
| `dex_factory` | Factory 合约实例 |
| `dex_router` | Router 合约实例 |
| `dex_pair_api` | 交易对 API 实例 |
| `dex_test_data` | DEX V2 测试数据 |
| `swap_v3_test_data` | DEX V3 测试数据 |
| `v3_liquidity_environment` | V3 流动性环境 |
