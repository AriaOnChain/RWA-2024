# 项目名称：房地产通证化与自动化房地产借贷

## 目录
1. [房地产通证化](#房地产通证化)
2. [自动化房地产借贷](#自动化房地产借贷)

## 房地产通证化
通过使用 Zillow 的测试API接口，创建实现房地产细分权益化的代币。所选资产将会是一项现实世界的房产，而相关数据则是通过使用 Chainlink Functions 调用 API 来获取的。

## 通证规范
将创建的是 ERC-1155 标准的通证，这使得能够融合 ERC-20 通证同质化的优点来进行资产细分，同时又可通过统一资源标识符 URI 为代币集合赋予独特的属性，类似于 ERC-721 标准所提供的能力。

## 区块链的选择
代币化资产将使用CCIP以实现在任意链上的可用性。将把 issuer 合约部署在 Avalanche Fuji 区块链上。这意味着初始发行将在 Avalanche Fuji 上进行，但随后该通证可通过 Chainlink CCIP 转移到与Avalanche Fuji 相连的任何其他区块链上。

## 链下数据获取
为了创建一个与现实世界资产关联并反映其价值的链上资产，我们需要将区块链与现实世界进行连接。为实现这一目标，将使用 Chainlink Functions 和 Automation 服务。 Functions 将使我们能够从 Zillow API 上读取数据，而 Automation 将自动化这一调用过程实现数据每日更新。

## 概述
此智能合约，用于房地产通证化。它集成了 Chainlink 的 Functions 服务，用于获取外部数据（例如房地产详情、市场价格），以便为每个房地产 NFT 生成唯一的元数据。

## 核心功能
### NFT 发行:
合约允许所有者发行新的房地产 NFT。
使用 Chainlink Functions 获取外部数据来生成 NFT 的元数据（例如房地产价格、税务评估值）。

### 外部数据请求:
使用 Chainlink Functions 获取房地产数据（如挂牌价格、评估价值）。
获取的外部数据被用于动态生成 NFT 的元数据 (tokenURI)。

### 自动化 NFT 铸造:
根据接收到的数据，合约会铸造新的房地产 NFT，代表分割的房地产资产。
NFT 会分配唯一的标识符和元数据（例如，tokenURI 和分割的所有权比例）。

### 请求管理:
确保一次只能处理一个 NFT 发行请求。
允许通过 cancelPendingRequest 函数取消挂起的请求。

### 安全性和权限:
只有合约所有者或指定的自动化转发器可以发行新的 NFT 或取消挂起的请求。
跨链转移功能允许 NFT 在不同的区块链网络之间进行转移。

## 合约结构

### Issuer 合约:
主要合约，用于发行新的房地产 NFT，并与 Chainlink Functions 交互。
继承了 FunctionsClient、FunctionsSource 和 OwnerIsCreator。

### RealEstateToken 合约:
基于 ERC1155 的代币合约，用于铸造和管理房地产 NFT。
集成了 Chainlink 的跨链烧铸功能。

### RealEstatePriceDetails 合约:
通过 Chainlink Functions 获取房地产价格数据（如挂牌价格、税务评估值）。

## 函数
### issue
描述: 通过请求外部数据（元数据）来发行新的房地产 NFT。
参数:
to: 接收铸造 NFT 的地址。
amount: 房地产的分割份额（即 NFT 的数量）。
subscriptionId: Chainlink 的订阅 ID，用于数据请求。
gasLimit: Chainlink 请求的 gas 限额。
donID: 数据预言机网络 ID，用于请求。
返回值: 返回一个请求 ID，用于跟踪数据请求。

### cancelPendingRequest
描述: 如果有挂起的 NFT 发行请求，允许取消该请求。

### fulfillRequest
描述: Chainlink 数据请求的回调函数。它处理接收到的数据（房地产价格），并铸造 NFT。
参数:
requestId: Chainlink 请求的 ID。
response: Chainlink 返回的数据（房地产数据）。
err: 如果请求失败，返回的错误信息。

### setAutomationForwarder
描述: 允许合约所有者设置自动化转发器地址。自动化转发器可以用来发行 NFT。

### getPriceDetails
描述: 返回特定 tokenId 的价格详情（挂牌价格、原始挂牌价格、税务评估值）。

## 设置和部署
### 部署 RealEstateToken:

首先部署 RealEstateToken 合约。该合约将用于铸造 NFT。

### 部署 Issuer 合约:
部署 Issuer 合约，并传入 RealEstateToken 合约地址和 Chainlink Functions 路由器地址。

### 设置自动化转发器:
部署后，调用 setAutomationForwarder 函数，设置可用于发行 NFT 的自动化转发器地址。

### 发行 NFT:
一旦设置了自动化转发器，就可以通过调用 issue 函数来发行 NFT，提供必要的参数。



## 自动化房地产借贷
RwaLending 是一个去中心化的房地产借贷平台，允许用户将房地产代币作为抵押借取稳定币（如 USDC）。合约实现了贷款发放、还款、清算等功能，并基于链上数据和价格预言机来确保贷款和清算条件的自动化和透明化。

## 概述
RwaLending 合约结合了房地产通证化与去中心化金融（DeFi）概念。通过将房地产代币化为 **ERC1155** 代币，平台允许用户将其作为抵押物向合约借款。贷款的额度和清算门槛基于房地产代币的市场估值，并且通过 Chainlink 预言机获取最新的市场价格。

### 核心功能：
- **房地产代币作为抵押物**：用户可以通过 **ERC1155** 代币提交房地产资产作为贷款的抵押物。
- **贷款发放**：根据房地产估值，平台自动计算并发放贷款金额。
- **贷款偿还**：用户可以按期偿还贷款，并取回抵押的房地产代币。
- **自动清算**：当贷款的抵押物价值低于设定的清算阈值时，平台会自动清算抵押物。

## 功能说明
- **借款（Borrow）**：用户将房地产代币作为抵押物，通过合约借取 USDC 稳定币，借款额度和清算阈值会根据房地产代币的市场估值动态计算。
- **还款（Repay）**：借款用户按期偿还贷款，清算抵押物并归还借款金额。
- **清算（Liquidate）**：当借款抵押物的市场价值低于设定的清算阈值时，任何人都可以调用清算功能来回收抵押物。
- **价格获取（Valuation）**：合约通过 Chainlink 预言机获取最新的 USDC 价格，进而计算房地产代币的市场估值。
- **风险控制**：平台使用“贷款价值比（LTV）”控制借款的额度，并设置清算阈值防止借款人违约。
