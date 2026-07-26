# NFT 测试用例详细文档

## 文档信息
- **文档类型**: 测试用例详细文档
- **分类**: NFT 非同质化代币 / SFT 半同质化代币
- **用例数量**: 10个
- **代码位置**: `/home/liuyoushan/ape-demo/tests/nft/scenarios/test_nft_scenarios.py`
- **测试数据**: `/home/liuyoushan/ape-demo/tests/nft/data/test_nft.yaml`
- **实现状态**: 全部待实现 (NotImplementedError)

---

## 测试数据配置

### 通用配置
| 配置项 | 值 | 说明 |
|-------|----|------|
| nft_name | "TestNFT" | NFT 合约名称 |
| nft_symbol | "TNFT" | NFT 合约符号 |

---

## case_036 ERC721 基础元数据校验

### 基本信息
- **用例编号**: case_036
- **用例名称**: ERC721 基础元数据校验
- **优先级**: P0
- **分类**: NFT / 功能测试
- **代码参考**: `tests/nft/scenarios/test_nft_scenarios.py:25`
- **状态**: 待实现

### 前置条件
1. ERC721 NFT 合约已部署
2. 测试环境正常运行

### 测试步骤
1. 调用 `name()` 获取 NFT 集合名称
2. 调用 `symbol()` 获取 NFT 符号
3. 调用 `totalSupply()` 获取总发行量
4. 验证每枚 NFT 有唯一 TokenId

### 预期结果
- NFT 名称与配置一致
- NFT 符号与配置一致
- totalSupply 返回正确的总量
- 每枚 NFT 具有唯一的 TokenId

### 关联测试数据
- 数据键: `case_nft_common`
- 关键字段: `nft_name`, `nft_symbol`

---

## case_037 NFT 限量铸造测试

### 基本信息
- **用例编号**: case_037
- **用例名称**: NFT 限量铸造测试
- **优先级**: P0
- **分类**: NFT / 功能测试
- **代码参考**: `tests/nft/scenarios/test_nft_scenarios.py:41`
- **状态**: 待实现

### 前置条件
1. ERC721 NFT 合约已部署
2. 白名单配置已设置
3. 最大发行量已配置

### 测试步骤
1. 白名单用户执行铸造
2. 验证白名单铸造成功
3. 公售阶段执行铸造
4. 验证公售铸造成功
5. 达到最大发行量后尝试铸造
6. 验证超额铸造被拦截

### 预期结果
- 白名单用户可正常铸造
- 公售铸造正确执行
- 最大发行量限制生效
- 超额铸造被拦截 revert

### 关联测试数据
- 无特定测试数据

---

## case_038 NFT 独立转账持有测试

### 基本信息
- **用例编号**: case_038
- **用例名称**: NFT 独立转账持有测试
- **优先级**: P0
- **分类**: NFT / 功能测试
- **代码参考**: `tests/nft/scenarios/test_nft_scenarios.py:57`
- **状态**: 待实现

### 前置条件
1. ERC721 NFT 合约已部署
2. 发送方持有指定 TokenId 的 NFT
3. 接收方账户存在

### 测试步骤
1. 记录发送方和接收方的 NFT 持有数量
2. 验证发送方拥有该 TokenId
3. 执行 `safeTransferFrom` 转账
4. 验证发送方不再持有该 TokenId
5. 验证接收方获得该 TokenId
6. 解码 Transfer 事件

### 预期结果
- 发送者失去持有权（balance 减 1）
- 接收者获得持有权（balance 加 1）
- ownerOf 返回新持有者地址
- Transfer 事件正确触发

### 关联测试数据
- 无特定测试数据

---

## case_039 ERC721 授权与委托测试

### 基本信息
- **用例编号**: case_039
- **用例名称**: ERC721 授权与委托测试
- **优先级**: P1
- **分类**: NFT / 功能测试
- **代码参考**: `tests/nft/scenarios/test_nft_scenarios.py:73`
- **状态**: 待实现

### 前置条件
1. ERC721 NFT 合约已部署
2. 授权方持有 NFT
3. 被授权方/市场合约存在

### 测试步骤
1. 单 NFT 授权（approve）
2. 验证 approvedAddress 正确记录
3. 授权可撤销（approve to zero address）
4. 市场合约托管授权（setApprovalForAll）
5. 验证 isApprovedForAll 返回 true
6. 批量授权校验

### 预期结果
- 单 NFT 授权正确设置 approvedAddress
- 授权可被撤销
- 市场合约托管授权（isApprovedForAll）正确
- 授权方可以执行转账
- 未授权方无法执行转账

### 关联测试数据
- 无特定测试数据

---

## case_040 NFT 销毁逻辑测试

### 基本信息
- **用例编号**: case_040
- **用例名称**: NFT 销毁逻辑测试
- **优先级**: P1
- **分类**: NFT / 功能测试
- **代码参考**: `tests/nft/scenarios/test_nft_scenarios.py:89`
- **状态**: 待实现

### 前置条件
1. ERC721 NFT 合约已部署
2. 用户持有可销毁的 NFT

### 测试步骤
1. 记录销毁前持有者余额
2. 记录销毁前 totalSupply
3. 执行 burn 销毁指定 TokenId
4. 验证持有者余额减少
5. 验证 totalSupply 减少
6. 解码 Transfer 事件（to = address(0)）

### 预期结果
- 指定 TokenId 可被销毁
- 持有者 balance 减少 1
- totalSupply 减少 1
- Transfer 事件 to 为零地址
- ownerOf 查询 revert 或返回零地址

### 关联测试数据
- 无特定测试数据

---

## case_041 ERC1155 半同质化基础校验

### 基本信息
- **用例编号**: case_041
- **用例名称**: ERC1155 半同质化基础校验
- **优先级**: P0
- **分类**: NFT / 功能测试
- **代码参考**: `tests/nft/scenarios/test_nft_scenarios.py:105`
- **状态**: 待实现

### 前置条件
1. ERC1155 合约已部署
2. 测试环境正常运行

### 测试步骤
1. 铸造不同 TokenId 的 SFT
2. 验证同 TokenId 可叠加数量
3. 验证多品类资产隔离存储
4. 验证 balanceOf 正确返回数量
5. 验证不同 TokenId 独立管理

### 预期结果
- 同 TokenId 数量可叠加
- 不同 TokenId 资产隔离
- balanceOf(account, id) 返回正确数量
- balanceOfBatch 批量查询正确

### 关联测试数据
- 无特定测试数据

---

## case_042 SFT 批量铸造/批量转账测试

### 基本信息
- **用例编号**: case_042
- **用例名称**: SFT 批量铸造/批量转账测试
- **优先级**: P0
- **分类**: NFT / 功能测试
- **代码参考**: `tests/nft/scenarios/test_nft_scenarios.py:121`
- **状态**: 待实现

### 前置条件
1. ERC1155 合约已部署
2. 调用方有铸造权限
3. 发送方有足够余额

### 测试步骤
1. 批量铸造指定数量的 SFT
2. 验证铸造后数量正确
3. 批量转账（safeBatchTransferFrom）
4. 验证发送方余额扣减
5. 验证接收方数量增加
6. 数据一致性校验

### 预期结果
- 批量铸造指定数量正确
- 批量转账正确扣减和增加
- 接收者数量正确增加
- TransferBatch 事件正确触发
- 数组长度不匹配时 revert

### 关联测试数据
- 无特定测试数据

---

## case_043 链游道具交易场景测试

### 基本信息
- **用例编号**: case_043
- **用例名称**: 链游道具交易场景测试
- **优先级**: P0
- **分类**: NFT / 功能测试
- **代码参考**: `tests/nft/scenarios/test_nft_scenarios.py:137`
- **状态**: 待实现

### 前置条件
1. 游戏道具合约已部署
2. 市场/交易合约已部署
3. 卖家持有道具

### 测试步骤
1. 卖家挂单（设置价格和授权）
2. 买家购买道具
3. 验证道具转移到买家
4. 验证资金转移到卖家
5. 点对点私下交易场景
6. 资产扣减与交割校验

### 预期结果
- 道具挂单功能正常
- 点对点交易正常执行
- 资产扣减正确
- 交割完整性验证
- 交易双方状态一致

### 关联测试数据
- 无特定测试数据

---

## case_044 权益类 SFT 权限控制测试

### 基本信息
- **用例编号**: case_044
- **用例名称**: 权益类 SFT 权限控制测试
- **优先级**: P1
- **分类**: NFT / 功能测试
- **代码参考**: `tests/nft/scenarios/test_nft_scenarios.py:153`
- **状态**: 待实现

### 前置条件
1. 权益 SFT 合约已部署
2. 权益验证功能已实现

### 测试步骤
1. 会员凭证权限校验
2. 门票核销功能
3. 游戏 Buff 生效验证
4. 权益与持有状态绑定验证
5. 失去持有权后权益失效

### 预期结果
- 会员凭证权限校验正确
- 门票核销功能正常
- 游戏 Buff 正常生效
- 权益与持有状态绑定
- 失去持有权后权益自动失效

### 关联测试数据
- 无特定测试数据

---

## case_045 NFT/SFT 链上链下一致性测试

### 基本信息
- **用例编号**: case_045
- **用例名称**: NFT/SFT 链上链下一致性测试
- **优先级**: P1
- **分类**: NFT / 功能测试
- **代码参考**: `tests/nft/scenarios/test_nft_scenarios.py:169`
- **状态**: 待实现

### 前置条件
1. NFT/SFT 合约已部署
2. 元数据 URI 可访问
3. 前端/后端服务可用

### 测试步骤
1. 链上 tokenURI 查询
2. 验证元数据 JSON 格式
3. 验证属性值正确存储
4. 验证图片 URI 可访问
5. 前端展示与链上数据对齐
6. 权益信息同步验证

### 预期结果
- 链上元数据与前端匹配
- 属性值正确存储
- 权益信息同步
- 图片 URI 可访问
- JSON 格式符合标准

### 关联测试数据
- 无特定测试数据

---

## 测试文件结构

```
tests/nft/
├── scenarios/
│   └── test_nft_scenarios.py       # NFT 场景测试
├── data/
│   └── test_nft.yaml               # 测试数据
├── apis/
│   └── (待实现)                    # API 封装
├── fixtures/
│   └── nft_fixtures.py             # Fixture
└── conftest.py                     # 配置
```

---

## 核心 Fixture 列表

| Fixture 名称 | 说明 |
|-------------|------|
| (待补充) | NFT 合约实例 |
| (待补充) | ERC1155 合约实例 |
| `nft_test_data` | NFT 测试数据 |
