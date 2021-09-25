---
layout: post
title: "深入底层：Go语言从零构建区块链（二）：PoW工作证明机制"
subtitle: "Build Blockchain from Scratch with Golang"
date: 2021-09-25
author: Krad
category: Blockchain
tags: blockchain golang
finished: true
published: true
---

## 前言
在上一章中我们了解了区块是什么以及区块与区块链之间的关系。在这一章中我们将拓宽区块的头部信息，并讲解区块如何合法的被添加进区块链中。

## 项目重构
在上一章中，我们所有的代码都写在了main.go中，这显然不利于我们继续构建项目。我们希望main.go只用于最后启动我们的区块链系统，为此我们需要将设计的区块与区块链移植其它文件夹中，这也会帮你go语言的代码管理机制。

首先我们在goblockchain文件夹下建立一个新的文件夹blockchain，在blockchain下分别创建block.go,blockchain.go，之后我们会将相关结构体与函数放入其中。创建utils文件夹，在其下创建util.go用以存放一些龙套函数。创建constcoe文件夹在其下创建constcoe.go用于储存一些全局常量。

![pic1](../img/goblockchain2/pic1.png)

打开utils.go，引入以下包。

{% highlight go %}
//util.go

package utils

import (
	"bytes"
	"encoding/binary"
	"log"
)
{% endhighlight %}

然后构建一个简单的错误处理函数。

{% highlight go %}
//util.go

func Handle(err error) {
	if err != nil {
		log.Panic(err)
	}
}
{% endhighlight %}

然后将我们之前写的int64转字节串函数移动过来。

{% highlight go %}
//util.go

func ToHexInt(num int64) []byte {
	buff := new(bytes.Buffer)
	err := binary.Write(buff, binary.BigEndian, num)
	Handle(err)
	return buff.Bytes()
}
{% endhighlight %}

然后我们转至constcoe.go,设置一个后面会用到的全局常量，也就是实现PoW时的难度（后面会详细讲）。

{% highlight go %}
//constcoe.go

package constcoe

const (
	Difficulty = 12
)
{% endhighlight %}

打开block.go，将之前写的结构体与函数放入。

{% highlight go %}
//block.go

package blockchain

import (
	"bytes"
	"crypto/sha256"
	"goblockchain/utils"
	"time"
)

type Block struct {
	Timestamp int64
	Hash      []byte
	PrevHash  []byte
	Data      []byte
}

func (b *Block) SetHash() {
	information := bytes.Join([][]byte{utils.ToHexInt(b.Timestamp), b.PrevHash, b.Data}, []byte{})
	hash := sha256.Sum256(information)
	b.Hash = hash[:]
}

func CreateBlock(prevhash, data []byte) *Block {
	block := Block{time.Now().Unix(), []byte{}, prevhash, data}
	block.SetHash()
	return &block
}

func GenesisBlock() *Block {
	genesisWords := "Hello, blockchain!"
	return CreateBlock([]byte{}, []byte(genesisWords))
}
{% endhighlight %}

打开blockchain.go，将结构体与函数放入。

{% highlight go %}
//blockchain.go

package blockchain

type BlockChain struct {
	Blocks []*Block
}

func (bc *BlockChain) AddBlock(data string) {
	newBlock := CreateBlock(bc.Blocks[len(bc.Blocks)-1].Hash, []byte(data))
	bc.Blocks = append(bc.Blocks, newBlock)
}

func CreateBlockChain() *BlockChain {
	blockchain := BlockChain{}
	blockchain.Blocks = append(blockchain.Blocks, GenesisBlock())
	return &blockchain
}
{% endhighlight %}

回到main.go，只需要调用我们写的blockchain包，就可以启动之前写的区块链系统了。

{% highlight go %}
//main.go

package main

import (
	"fmt"
	"goblockchain/blockchain"
	"time"
)

func main() {
	blockchain := blockchain.CreateBlockChain()
	time.Sleep(time.Second)
	blockchain.AddBlock("After genesis, I have something to say.")
	time.Sleep(time.Second)
	blockchain.AddBlock("Leo Cao is awesome!")
	time.Sleep(time.Second)
	blockchain.AddBlock("I can't wait to follow his github!")
	time.Sleep(time.Second)

	for _, block := range blockchain.Blocks {
		fmt.Printf("Timestamp: %d\n", block.Timestamp)
		fmt.Printf("hash: %x\n", block.Hash)
		fmt.Printf("Previous hash: %x\n", block.PrevHash)
		fmt.Printf("data: %s\n", block.Data)

	}

}
{% endhighlight %}

注意此时调用函数CreateBlockChain需要在前面加blockchain的前缀。

## 共识机制

我们常说区块链是一个分布式系统，系统中每个节点都有机会储存数据信息构造一个区块然后追加到区块链尾部。这里就存在一个问题，那就是当区块链系统中有多个节点都想将自己的区块追加到区块链是我们该怎么办？我们将这些等待添加的区块统称为候选区块，显然我们不能对候选区块全盘照收，否则区块链就不再是一条链而是不同分叉成区块树。那么我们如何确定一种方法来从候选区块中选择一个加入到区块链中了？这里就需要用到区块链的共识机制，后文将以比特币使用的最经典PoW共识机制进行讲解。

共识机制说的通俗明白一点就是要在相对公平的条件下让想要添加区块进区块链的节点内卷，通过竞争选择出一个大家公认的节点添加它的区块进入区块链。整个共识机制被分为两部分，首先是竞争，然后是共识。中本聪在比特币中设计了如下的一个Game来实现竞争：每个节点去寻找一个随机值（也就是nonce），将这个随机值作为候选区块的头部信息属性之一，要求候选区块对自身信息（注意这里是包含了nonce的）进行哈希后表示为数值要小于一个难度目标值（也就是Target），最先寻找到nonce的节点即为卷王，可以将自己的候选区块发布并添加到区块链尾部。这个Game设计的非常巧妙，首先每个节点要寻找到的nonce只对自己候选区块有效，防止了其它节点同学抄答案；其次，nonce的寻找是完全随机的没有技巧，寻找到nonce的时间与目标难度值与节点本身计算性能有关，但不妨碍性能较差的节点也有机会获胜；最后寻找nonce可能耗费大量时间与资源，但是验证卷王是否真的找到了nonce却非常却能够很快完成并几乎不需要耗费资源，这个寻找到的nonce可以说就是卷王真的是卷王的证据。现在我们就来一步一步实现这个Game。

## 添加Nonce
如前文所说，我们要先增加一些区块的头部信息。

{% highlight go %}
//block.go

import (
	"bytes"
	"crypto/sha256"
	"goblockchain/utils"
	"time"
)

type Block struct {
	Timestamp int64
	Hash      []byte
	PrevHash  []byte
	Target    []byte //This line is new
	Nonce     int64  //This line is new
	Data      []byte
}
{% endhighlight %}

Nonce就是节点寻找到的作为卷王的证据。Target就是我们前文说到的目标难度值，将它保存到区块中便于其他节点快速验证Nonce是否正确。

这样一来之前创建的几个函数会报错，我们先暂时不理会。

## PoW实现
在blockchain文件夹下创建proofofwork.go，我们来实现之前说到的Game。首先引入以下包。

{% highlight go %}
//proofofwork.go

package blockchain

import (
	"bytes"
	"crypto/sha256"
	"goblockchain/constcoe"
	"goblockchain/utils"
	"math"
	"math/big"
)
{% endhighlight %}

我们现在来构建一个可以返回目标难度值的函数。我们这里使用的之前设定的一个常量Difficulty来构造目标难度值，但是在实际的区块链中目标难度值会根据网络情况定时进行调整，且能够保证各节点在同一时间在同一难度下进行竞争，故这里的GetTarget可以理解为预留API，期待一下之后的分布式网络实现。

{% highlight go %}
//proofofwork.go

func (b *Block) GetTarget() []byte {
	target := big.NewInt(1)
	target.Lsh(target, uint(256-constcoe.Difficulty))
	return target.Bytes()
}
{% endhighlight %}

Lsh函数就是向左移位，移的越多目标难度值越大，哈希取值落在的空间就更多就越容易找到符合条件的nonce。

每次我们输入一个nonce对应的区块的哈希值都会改变，如下。

{% highlight go %}
//proofofwork.go

func (b *Block) GetBase4Nonce(nonce int64) []byte {
	data := bytes.Join([][]byte{
		utils.ToHexInt(b.Timestamp),
		b.PrevHash,
		utils.ToHexInt(int64(nonce)),
		b.Target,
		b.Data,
	},
		[]byte{},
	)
	return data
}
{% endhighlight %}

现在对于任意一个区块，我们都能去寻找一个合适的nonce了。

{% highlight go %}
//proofofwork.go

func (b *Block) FindNonce() int64 {
	var intHash big.Int
	var intTarget big.Int
	var hash [32]byte
	var nonce int64
	nonce = 0
	intTarget.SetBytes(b.Target)

	for nonce < math.MaxInt64 {
		data := b.GetBase4Nonce(nonce)
		hash = sha256.Sum256(data)
		intHash.SetBytes(hash[:])
		if intHash.Cmp(&intTarget) == -1 {
			break
		} else {
			nonce++
		}
	}
	return nonce
}
{% endhighlight %}

可以看到，神秘的nonce不过是从0开始取的整数而已，随着不断尝试，每次失败nonce就加1直到由当前nonce得到的区块哈希转化为数值小于目标难度值为止。

我们再来实现一个快速验证卷王是卷王的函数。

{% highlight go %}
//proofofwork.go

func (b *Block) ValidatePoW() bool {
	var intHash big.Int
	var intTarget big.Int
	var hash [32]byte
	intTarget.SetBytes(b.Target)
	data := b.GetBase4Nonce(b.Nonce)
	hash = sha256.Sum256(data)
	intHash.SetBytes(hash[:])
	if intHash.Cmp(&intTarget) == -1 {
		return true
	}
	return false
}
{% endhighlight %}

好了，PoW我们已经实现了。回到block.go，调整以下函数。

{% highlight go %}
//block.go

func (b *Block) SetHash() {
	information := bytes.Join([][]byte{utils.ToHexInt(b.Timestamp), b.PrevHash, b.Target, utils.ToHexInt(b.Nonce), b.Data}, []byte{})
	hash := sha256.Sum256(information)
	b.Hash = hash[:]
}

func CreateBlock(prevhash, data []byte) *Block {
	block := Block{time.Now().Unix(), []byte{}, prevhash, []byte{}, 0, data}
	block.Target = block.GetTarget()
	block.Nonce = block.FindNonce()
	block.SetHash()
	return &block
}
{% endhighlight %}

## 调试带PoW的区块链系统
现在打开main.go，我们可以编写程序启动我们的区块链系统了。

{% highlight go %}
//main.go

package main

import (
	"fmt"
	"goblockchain/blockchain"
	"time"
)

func main() {
	chain := blockchain.CreateBlockChain()
	time.Sleep(time.Second)
	chain.AddBlock("After genesis, I have something to say.")
	time.Sleep(time.Second)
	chain.AddBlock("Leo Cao is awesome!")
	time.Sleep(time.Second)
	chain.AddBlock("I can't wait to follow his github!")
	time.Sleep(time.Second)

	for _, block := range chain.Blocks {
		fmt.Printf("Timestamp: %d\n", block.Timestamp)
		fmt.Printf("hash: %x\n", block.Hash)
		fmt.Printf("Previous hash: %x\n", block.PrevHash)
		fmt.Printf("nonce: %d\n", block.Nonce)
		fmt.Printf("data: %s\n", block.Data)
		fmt.Println("Proof of Work validation:", block.ValidatePoW())
	}
}
{% endhighlight %}

老样子在terminal中敲下go run main.go，得到如下结果。

    D:\learngo\goblockchain>go run main.go
    Timestamp: 1632558933
    hash: 5ee3e13ce051362c7fa4a5c0dcf4883fec97affb66521d02dcd1bfc5ac14c142
    Previous hash:
    nonce: 1623
    data: Hello, blockchain!
    Proof of Work validation: true
    Timestamp: 1632558934
    hash: 611739fb51de47b7bb472444517a4a9c7d590f9f17fc54ef13fe79ddebefa0e2
    Previous hash: 5ee3e13ce051362c7fa4a5c0dcf4883fec97affb66521d02dcd1bfc5ac14c142
    nonce: 3369
    data: After genesis, I have something to say.
    Proof of Work validation: true
    Timestamp: 1632558935
    hash: eca9fe17530d867ee8212084eae6d54581d152fd01f0c0c64f2143adbbf6a9cb
    Previous hash: 611739fb51de47b7bb472444517a4a9c7d590f9f17fc54ef13fe79ddebefa0e2
    nonce: 4541
    data: Leo Cao is awesome!
    Proof of Work validation: true
    Timestamp: 1632558936
    hash: edaff517ca77b1f6705227b1bf40f311aeb0207014581318f2c98a0fe3842723
    Previous hash: eca9fe17530d867ee8212084eae6d54581d152fd01f0c0c64f2143adbbf6a9cb
    nonce: 1325
    data: I can't wait to follow his github!
    Proof of Work validation: true

可以看到我们的PoW验证运行正常。你可以尝试将全局变量Difficulty改大来增加难度值，然后观察一下各个区块的timestamp变化。

## 总结
本章讲解了PoW共识机制，需要重点理解nonce与目标难度值，以及卷王。在一章中我们将会讲解区块中的数据信息存储方式，涉及UTXO模型。
