---
layout: post
title: "ParaSpace NFT借贷平台介绍"
date: 2023-2-15 09:00
author: "Tiny Hill"
categories:
  - DeFi
tags:
  - 区块链
  - DeFi
  - NFT
---

ParaSpace是一个非常强大的NFT借贷平台，可以看成是一个加强版的Benddao或者支持NFT抵押物的AAVE。

<!-- more -->

## 0x01
它支持NFT和主流token(ETH/USDC/USDT/APE/stETH等)作为抵押物，同样，也支持借出所有这些ERC20的Token。

除了借贷功能外，ParaSpace还有一个NFT的聚合交易功能，支持用户使用在借贷平台的信贷额度，直接购买NFT，免去借出再购买的两次操作。除此之外，还有APE Staking的功能模块。

https://docs.para.space/para-space/external-audits 能够看到项目的审计机构阵容强大，特别是难道一见的Trail of Bits。

## 0x02
https://github.com/para-space/paraspace-core paraspace的合约代码也是一个很大的工程，使用hardhat的开发框架，不过也看到了foundry的一点影子，看来目前foundry还是在成熟度方面，特别是部署管理这块不够强大。

## 0x03
借贷功能的入口合约是 PoolCore.sol 
https://etherscan.io/address/0x638a98bbb92a7582d07c52ff407d49664dc8b3ee ，主要借贷逻辑如，存入token/nft，借出token，清算，拍卖的入口都在这个合约，代码结构比较清晰，入口function内会继续调用SupplyLogic，BorrowLogic，LiquidationLogic，AuctionLogic这几个library。

## 0x04
看代码里面，很多地方都能够看出这个项目有不错的技术实力，在代码优化层面做的很多，在token存入的时候，针对DAI这样支持permit授权的token，做了特别的支持，这样存入DAI就不用花费approve的gas费。还有在存储配置内容的时候，通过掩码的方式，把很多配置项都存在一个uint256里面，节省了存储

## 0x05
除了常用的NFT，该项目还支持uniswap v3的LP，也是一种ERC721 形式的NFT，来作为抵押物，目前支持8个主流token的LP NFT，这部分的价值估算和token一样，从合约接口读到底层资产的数量，使用chainlink的oracle来计算价值。

## 0x06
supply和withdraw ERC20 token的逻辑和compound差不多，每个token会有一个对应pToken来存放资产，给用户pToken的存款凭证。withdraw和borrow一样，需要检查可提现额度和健康因子。

## 0x07
supplyERC721 也是类似的，每个collection会对应一个NToken的存款凭证合约，用户存入NFT后，会对应mint一个相同tokenId的存款凭证给到用户。

## 0x08 
NFT抵押物定价一直是NFT借贷项目一个非常核心的关切点，因为诸如地板价这样的价值很多时候都并不能非常准确的体现NFT的公允价值。ParaSpace使用的是Chainlink的数据源，这个数据源是Chainlink和Coinbase一起做的，但是算法是不公开的。

## 0x09 
borrow功能会先调用ValidationLogic去检查借出后的健康因子是否满足要求，其他诸如withdraw的功能也是需要先检查，然后给用户mint 不可转的债务token VariableDebtToken，这个是借鉴了aave 里面的写法，最后把要借的资产转给用户。

## 0x0a 
repay功能除了可以使用借出token还款之外，还可以使用pToken存款凭证来还款，具体什么场景会这么干呢？大概在循环贷还款的时候可以这么干，节约了取款再还款的步骤。

## 0x0b 
清算会先从ERC20的抵押物开始，先清算完了ERC20再启动ERC721的清算。这部分和传统的借贷协议一致。NFT的清算则会使用荷兰拍卖的方式，拍卖达成后还会对借款人收取一定罚金进入项目的保险池。拍卖NFT只能使用ETH，这些ETH会转成借款人的抵押物。清算对所有人公开，没有白名单。

## 0x0c 
他们目前的NFT shop感觉还不是很好用，我随意试了几个，都是sold，说明从opensea同步状态还不够及时，目前他们前端只能看到对opensea和自家市场的支持，但是在代码里面已经体现了多家的接口，这块应该会在以后去接入。我查了下历史记录，在12000+条交易中，和buyWithCredit相关的只有大概30条。

## 0x0d 
buyWithCredit 的执行步骤大概是，借出ETH，使用这些ETH到Opensea市场去买NFT，然后再把NFT存回借贷合约。操作后，用户的抵押物增加了一个NFT，债务增加了购买所需的ETH。所有操作都在一个交易中执行，用户自己的账户没有余额的变化。所以操作时候还是要小心一些。

## 0x0e 
虽然耗费gas有点多，但是这一套的用户体验应该是很不错的，方向是对的，当然需要不断优化。NFTfi的乐高应该也会慢慢叠起来的，非常期待之后还有其他类型的产品。关于 @ParaSpace_NFT的学习贴就写这么多了，这个帖子收获了一些赞，很开心，虽然炒币很差，但是写些简单的项目体验还是可以的。

## 0x0f
最后也说一些我对NFTfi的一些看法。确定性的产品类型，比如借贷池，已经很成熟了，拼的就是产品的优化。另外还有衍生品交易， 有
@nftperp和@nfex_official，我是很看好的，NFT的现货交易因为版税的原因，有诸多的问题，但衍生品就没有这个问题，而且衍生品天然就碎片化，可以降低交易的价格门槛

## 0x10 
很有可能，未来蓝筹NFT的交易量会慢慢迁移到衍生品，现货交易更倾向于收藏性质。但是目前阻碍NFT衍生品的问题还是来自于NFT的定价，地板价或者是成交价都太容易被操控了，这个问题暂时没有看到很好的方案，可以期待一下。
## 0x11 
除此之外，我对点对点的NFT借贷也有一些期待，目前点对点的模式，量非常小，因为匹配难度很大，之前我想到了collection offer的时候还兴奋了一下，后来发现
@NFTfi
 也已经适配了，出资人对整个collection出价明显比借款人单个抵押物出家的效率高。如果把NFT点对点的借贷和房产进行一些类比

## 0x12 
就能发现还有很多事情可以做的，对点对点债务进行再打包，再销售，再流转，ETF化。都是可以做的事情，但是目前的资产规模和用户体量似乎还不需要做这些事情。我喜欢点对点这样没有系统性风险的方式，但是从产品和市场角度，似乎点对池才是可以work的模式，期待一下，就这样了。