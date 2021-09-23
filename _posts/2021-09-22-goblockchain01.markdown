---
layout: post
title: "深入底层：Go语言从零构建区块链（一）: Hello, Blockchain"
subtitle: "Build Blockchain from Scratch with Golang"
date: 2021-09-22
author: Krad
category: Blockchain
tags: blockchain golang
finished: true
published: true
---

## 前言
有时觉得前言什么的可以省略，但本系列文章作为一个教程还是应该在最开始的时候说两句。

这个系列的教程目的是使用Golang由浅入深地还原PoW共识机制最基础区块链系统（参照比特币），适合想要快速入门区块链核心技术的读者，当然也适合刚学完Go基础语法希望练手的读者。相较于网络上其它Go语言实现区块链的教程，本系列教程以最终建立一个可以分布式运行的区块链系统为目标，使用新版本的Golang（v1.17）及相关包，在还原的过程中会尽量说明一些常规教程忽视的细节，给出所有的代码。

对于区块链技术的学习大部分人常常停留在表象，UTXO模型，共识机制，P2P网络背的是一套一套的，迫不及待地就想用区块链往所有涉及隐私与安全的问题里套。我始终持有的观点是，可以允许区块链在各种应用场景中试错，但不应该过于推崇区块链，它不是万能的，要学习区块链就应该了解其本质，从事物的两面性去研究它。举个例子，如果我们把区块链当作一个分布式的数据库看待，那么它的性能无疑是超拉跨的，但是如果充分理解区块链的本质，就能够明白比特币为什么仅仅用一段代码就能够在全球没有第三方机构的参与下实现资产信息的长久保存，稳定运行超过五年，感叹区块链核心技术的精妙之处。

如果想要了解区块链技术的本质，了解区块链技术的优缺点以及可能的研究方向，最直接的办法无疑是阅读比特币源码。但是比特币源码使用C进行编写，人们常常不知道从哪里开始进行学习理解，在耗费大量时间精力的过程中，渐渐消磨学习者对区块链的兴趣。本人在学习区块链的过程中发现，通过理解区块链原理一步一步构建区块链系统比直接阅读比特币源码要有乐趣的多，学习速度也更快，而且最终都能达到同样的目的，就像B树的建立往往比B树的查找容易学习理解一样。

为了达到使读者快速入门并理解区块链核心技术的目的，本教程不太注重对Go语言相关问题的讲解，关注的是代码背后的区块链原理与实现细节。读者需要做的只是创建一个空项目，从零开始跟随本教程敲下一行又一行的代码。对读者来说，重要的不是对我所写的代码进行改动，而是理解每一行代码的意义。我能够保证只需坚持阅读完本系列的前几章

*由于本人Go语言也还处于学习阶段，故一些部分代码可能较为冗余。区块链实现细节有误的地方也随时欢迎讨论指正。*

## 区块
区块链作为一个分布式账本，是由一个个的区块以链式结构进行存储的。作为本教程的开端，我想

区块链是有一个一个的区块以链式结构进行存储的

{% highlight python %}
package cli

import (
	"bytes"
	"flag"
	"fmt"
	"log"
	"os"
	"runtime"
	"strconv"

	"github.com/leo201313/Blockchain_with_Go/blockchain"
	"github.com/leo201313/Blockchain_with_Go/wallet"
)

type CommandLine struct{}

func (cli *CommandLine) printUsage() {
	fmt.Println("Welcome to Leo Cao's tiny blockchain system, usage is as follows:")
	fmt.Println("--------------------------------------------------------------------------------------------------------------")
	fmt.Println("All you need is to first create a wallet, and use the wallet address to init a blockchain.")
	fmt.Println("Then, you can add more wallets and try to make some transactions use 'send'.")
	fmt.Println("At this version, one block can contain multiple transactions. You should run mine to add block.")
	fmt.Println("The nickname of wallet is just used to refer the wallet instead of typing the full address.")
	fmt.Println("Nothing left to say, wish you good luck.")
	fmt.Println("--------------------------------------------------------------------------------------------------------------")
	fmt.Println("createwallet -nickname NICKNAME               ----> Creates a new wallet with the nick name")
	fmt.Println("listwallets                                   ----> Lists all the wallets in the wallet file")
	fmt.Println("walletinfo -nickname NICKNAME                 ----> Back all the information of a wallet")
	fmt.Println("createblockchain -nickname NICKNAME           ----> Creates a blockchain using the name of the user's wallet")
	fmt.Println("blockchaininfo                                ----> Prints the blocks in the chain")
	fmt.Println("send -from FROMNAME -to TONAME -amount AMOUNT ----> Make a transaction")
	fmt.Println("mine                                          ----> Mine and add a block to the chain")
	fmt.Println("--------------------------------------------------------------------------------------------------------------")
}

{% end highlight %}
