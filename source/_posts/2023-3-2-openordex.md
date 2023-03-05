---
layout: post
title: "比特币链上的NFT DEX"
date: 2023-3-2 21:40
author: "Tiny Hill"
categories:
  - DeFi
tags:
  - 区块链
  - DeFi
  - NFT
---

> 只要设计得好，不需要图灵完备的智能合约运行环境也可以做很多的事情。

<!-- more -->

openordex.org 是一个为了交易 Ordinals 协议的铭文(inscription) 所设计的去中心化交易所。

## Ordinals

Ordinals 是一个为Bitcoin最小单位sat进行编号的协议，它是一个Bitcoin的外挂协议，每个sat被挖出来后，就会关联一个唯一的编号。

此外，Ordinals协议使用Bitcoin最新的Taproot升级提供的能力，可以将文件、图片等信息写入Bitcoin交易，完成上链，该过程被称为inscribe(铭刻)。而图片等内容通过铭刻与ordinal序号关联了起来，成为铭文(inscription) ，可以简单理解为Bitcoin上的NFT。

## 铭文交易现状

因为比特币并不支持图灵完备的智能合约，所以想要交易铭文还是比较困难。目前除了OTC交易之外，用的比较多的是使用托管的方式，把铭文映射到以太坊进行交易，这样多少让人感觉很违和。

除了交易不便，铭文同ETH NFT最大的不同在于，其没有链上保证的collection集合，所有集合所包含的铭文由项目方链下维护。

## [openordex](https://openordex.org/)

openordex是我看到的第一个用去中心化的方式来进行铭文交易的平台。该项目是由 [fireblocks](https://twitter.com/fireblockshq)的研究员[orenyomtov](https://twitter.com/orenyomtov) 开发的开源产品。

其原理是构造一个交易，在同一个交易中把铭文转给买家，同时把btc转给卖家，交易需要买卖双方进行签名，因为是在同一交易中，所以交易或者一起成功，或者一起失败，不会有付钱买不到资产的情况。

而构造交易的难点来自于，挂单的人，事先不知道买家的地址，但是他需要对交易进行预签名。如果不是这样的话，买卖双方就需要同时在线，增加了成交的难度。

这里就需要引出比特币中的一个特性，Partially Signed Bitcoin Transaction (PSBT)，也就是部分签名，参与者可以只对交易的部分内容进行签名。

在openordex的流程中，交易构造流程如下：

### 交易输入

- A: taker 的一个笔小额(dummy)btc(1000 sat)
- B: maker 的铭文所关联的BTC，数量M
- C: taker 购买铭文所需要的BTC，数量N，如果utxo值大于N，则会产生退款
  
### 交易输出

- X: 铭文所关联的BTC，数量M，转移到 taker 的地址
- Y: 数量N的BTC转移到 maker的地址
- Z: taker转数量N的BTC的退款（可能出现）

### 交易构造步骤

- maker 使用部分签名对 B 和 Y 部分进行签名，保证卖出铭文后会收到自己挂单的BTC数量
- maker 将签名广播
- taker 获得部分签名的交易，将自己的地址加入交易中，并签名
- taker 广播交易
- 交易落块，兑换完成

前两步对应了opensea上的挂单步骤，后面对应吃单的步骤。

## 问题

目前这套交易机制，对于挂单的保护可能不够。体现在订单没有失效时间，不能撤单。如果要撤单，则需要卖家将铭文转移到其他地址来阻止交易。

## 小结

openordex的交易方式无疑是去中心化的，目前项目是开源的，前端也比较潦草，相信很快会有更多更好的产品出现。