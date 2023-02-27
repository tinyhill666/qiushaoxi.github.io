---
layout: post
title: "Perpetual Protocol V2学习"
date: 2023-1-20 09:58
author: "Tiny Hill"
categories:
  - DeFi
tags:
  - 区块链
  - DeFi
---

Perpetual Protocol 是除了@dydx之外另一个明星的链上永续合约交易所，目前V2版本运行在 @optimismFND上。主要的特点是协议主要负责利润计算和爆仓清算，交易的部分和流动性的部分都集成了 @Uniswap V3

 <!-- more -->

# 2 Vault模块 
https://optimistic.etherscan.io/address/0xad7b4c162707e0b2b5f6fddbd3f8538a5fba0d60 该模块负责存放用户的保证金的充值和提现，还有清算。结算的币种是USDC，利润和亏损都是计算在USDC上。如果爆仓或关仓后USDC为负数，则必须充值进去补上才能提现其他币种。有5个币种可以作为保证金，权重和折算如图。
![](https://twitter.com/JosQiu/status/1616458374098587654/photo/1)

# 3 CollateralManager 模块
https://optimistic.etherscan.io/address/0x8Ac835C05530f10595C8015467339523154b4D85 
管理抵押物的币种、计价来源、权重、封顶和折算比例，这个模块是一个配置的模块，上线后暂时没有调整过配置。

# 4 DelegateApproval模块
https://optimistic.etherscan.io/address/0xfd7bB5F6844a43c5469c972640Eddfa99597a547 ，该模块用来支持用户将操作权限，比如开单权限授权给第三方。这样的方式可以吸引更多生态项目建立，也方便DeFi乐高的堆叠。


# 5 ClearingHouse模块
https://optimistic.etherscan.io/address/0x82ac2CE43e33683c58BE4cDc40975E73aA50f459，是最重要的一个模块，也是用户交互最多的模块。开单、关仓、增加减少流动性都通过这个合约进行。开单和关仓实际上是在uniswap上交易perp发行的虚拟token，vETH/vUSD等。该模块另一重要的功能就是清算爆仓，计算用户的亏损和保证金余额，不足则爆仓。

# 6 InsuranceFund模块
https://optimistic.etherscan.io/address/0x1C9a192DF3936cBF093d8afDc352718bCF834EB6，该模块提供协议的保险资金，初始资金100,000 USDC。用来支付协议的清算缺口。该合约还有一个分发手续费收入的功能，可以把这部分钱从vault中转到 SurplusBeneficiary


# 7 SurplusBeneficiary模块
https://optimistic.etherscan.io/address/0x78120c1ca337007323de2226d677e7fcf42d6ee7，该合约接受手续费收入，然后根据比例分发给treasury和vePERP持有人

# 8  MarketRegistry模块
https://optimistic.etherscan.io/address/0xd5820eE0F55205f6cdE8BB0647072143b3060067，该合约负责交易币种的管理，上架新币种、设置交易费率等。


# 9 VirtualToken模块
在perp上交易的所有token，都会由ClearingHouse模块发行相应的vToken，包括vUSD、vETH等，然后vToken实际的交易撮合发生在Uniswap V3。vToken的作用更多是为了记账，vToken只会出现在ClearingHouse、和Uniswap Pool里面，并不会转到用户的地址里。

# 10 Exchange模块
https://optimistic.etherscan.io/address/0xBd7a3B7DbEb096F0B832Cf467B94b091f30C34ec，用户通过ClearingHouse模块发起开仓和关仓的操作，ClearingHouse会调用Exchange模块来将对应的vToken通过Uniswap交易成另外一个vToken。比如开多ETH，就会把vUSD swap 成为 vETH。

# 11 OrderBook模块
https://optimistic.etherscan.io/address/0xDfcaEBe8f6ea5E022BeFAFaE8c6Cdae8D4E1094b，用来记录通过ClearingHouse往Uniswap添加流动性的行为。在实际开单关单的交易中会利用这些流动性记录来预计算交易的金额和滑点的检查。

# 12  AccountBalance模块
https://optimistic.etherscan.io/address/0xA7f3FC32043757039d5e13d790EE43edBcBa8b7c，前面讲到的vToken模块并不代表用户在系统内的余额，更多是与uniswap之间的账户记录。实际计算用户余额、利润是在AccountBalance模块。该模块可以查询用户的系统内余额、未结算利润和仓位等信息。

# 13 小结
该贴只是个很简单过了一下各个合约是干什么的，要理解还是需要更多的研究。ClearingHouse是该协议最重要的模块，基本操作的入口都是该模块。很久没有学习DeFi的合约代码了，现在的项目确实复杂度越来越高。所有逻辑堆在一个合约里的小作坊时代一去不复返了。https://github.com/perpetual-protocol/perp-curie-contract