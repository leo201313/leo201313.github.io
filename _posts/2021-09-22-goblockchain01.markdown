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

有时觉得前言什么的可以省略，但作为一个教程还是应该在最开始的时候说两句。

这个系列的教程目的是使用Golang由浅入深地还原PoW共识机制最基础区块链系统（参照比特币），适合想要快速入门区块链核心技术的读者，当然也适合刚学完Go基础语法希望练手的读者。相较于网络上其它Go语言实现区块链的教程，本系列教程以最终建立一个可以分布式运行的区块链系统为目标，使用新版本的Golang（v1.17）及相关包，在还原的过程中会尽量说明一些常规教程忽视的细节，给出所有的代码。

大部分人对于区块链技术的学习常常停留在表象，了解了UTXO模型，共识机制，P2P网络后，总是迫不及待地就想用区块链往所有涉及隐私与安全的问题里套。我始终持有的观点是，可以允许区块链在各种应用场景中试错，但不应该过于推崇区块链，它不是万能的，要学习区块链就应该了解其本质，从事物的两面性去研究它。举个例子，如果我们把区块链当作一个分布式的数据库看待，那么它的性能无疑是拉跨的，但是如果充分理解区块链的本质，就能够明白比特币为什么仅仅用一段代码就能够在全球没有第三方机构的参与下实现资产信息的长久保存，稳定运行超过五年，感叹其精妙之处。

如果想要了解区块链技术的本质，了解区块链技术的优缺点以及可能的研究方向，最直接的办法无疑是阅读比特币源码。但是比特币源码使用C进行编写，人们常常不知道从哪里开始进行学习理解，在耗费大量时间与精力的过程中，渐渐消磨学习者对区块链的兴趣。本人在学习区块链的过程中发现，通过理解区块链原理一步一步构建区块链系统比直接阅读比特币源码要有乐趣的多，学习速度也更快，而且最终都能达到同样的目的，就像B树的建立往往比B树的查找容易学习理解一样。

为了达到使读者快速入门并理解区块链核心技术的目的，本教程不太注重对Go语言相关问题的讲解，关注的是代码背后的区块链原理与实现细节。读者需要做的只是创建一个空项目，从零开始跟随本教程敲下一行又一行的代码。对读者来说，重要的不是对我所写的代码进行改动，而是理解每一行代码的意义。

*由于本人Go语言也还处于学习阶段，故一些部分代码可能较为冗余。区块链实现细节有误的地方也随时欢迎讨论指正。*

## 创建项目
首先我们需要创建一个项目。创建一个文件夹命名为goblockchain（当然你也可以取一个霸气的名字，比如lighteningchain,goXchain等等），然后使用VS（Visual Studio Code，推荐使用VS作为IDE）打开文件夹，如下图

![pic1](../img/goblockchain1/pic1.png)

此时文件中什么都没有，我们使用go mod来初始化项目，点击VS左下方的小三角，在terminal中输入go mod init goblockchain.

![pic1](../img/goblockchain1/pic2.jpg)
此时文件夹中将会多出一个go.mod文件，证明项目已经初始化成功。在goblockchain文件夹下创建main.go文件。
![pic1](../img/goblockchain1/pic3.png)

## 区块
区块链以区块（block）的形式储存数据信息，一个区块记录了一段时间内系统或网络中产生的重要数据信息，区块通过引用上一个区块的hash值来连接上一个区块这样区块就按时间顺序排列形成了一条链。每个区块应该包含头部（head）信息用于总结性的描述这个区块，然后在区块的数据存放区（body）中存放要保存的重要数据。首先我们需要初始化main.go，并导入一些基本的包。

{% highlight go %}
//main.go
package main

import (
	"bytes"
	"crypto/sha256"
	"encoding/binary"
	"fmt"
	"log"
	"time"
)

func main {
	
}
{% endhighlight %}
然后定义区块的结构体。
{% highlight go %}
//main.go
type Block struct{
	Timestamp int64
	Hash []byte
	PrevHash []byte
	Data []byte
}
{% endhighlight %}
我们定义的区块中有时间戳，本身的哈希值，指向上一个区块的哈希这三个属性构成头部信息，而区块中的数据以Data属性表示。在获得了区块后，我们可以定义区块链。

{% highlight go %}
//main.go
type BlockChain struct{
	Blocks []*Block
}
{% endhighlight %}
可以看到我们这里的区块链就是区块的一个集合。好了，现在你已经掌握了区块与区块链了，现在就可以去搭建自己的区块链系统了。

QVQ，好吧，我们现在来给我们的区块增加点细节，来看看它们是怎么连接起来的。对于一个区块而言，可以通过哈希算法概括其所包含的所有信息，哈希值就相当于区块的ID值，同时也可以用来检查区块所包含信息的完整性。哈希函数构造如下。

{% highlight go %}
//main.go
func (b *Block) SetHash() {
	information := bytes.Join([][]bytes{ToHexInt(b.Timestamp),b.PrevHash,b.Data},[]byte{})
	hash := sha256.Sum256(information)
	b.Hash = hash[:]
}

func ToHexInt(num int64) []byte {
	buff := new(bytes.Buffer)
	err := binary.Write(buff, binary.BigEndian, num)
	if err != nil {
		log.Panic(err)
	}
	return buff.Bytes()
}
{% endhighlight %}
information变量是将区块的各项属性串联之后的字节串。这里提醒一下bytes.Join可以将多个字节串连接，第二个参数是将字节串连接时的分隔符，这里设置为[]byte{}即为空，ToHexInt将int64转换为字节串类型。然后我们对information做哈希就可以得到区块的哈希值了。既然我们可以获得区块的哈希值了，我们就能够创建区块了。

{% highlight go %}
//main.go
func CreateBlock(prevhash, data []byte) *Block {
	block := Block{time.Now().Unix(), []byte{}, prevhash, data}
	block.SetHash()
	return &block
}
{% endhighlight %}
可以看到在创建一个区块时一定要引用前一个区块的哈希值，这里会有一个问题，那就是区块链中的第一个区块怎么创建？其实，在区块链中有一个创世区块，随着区块链的创建而添加，它指向的上一个区块的哈希值为空。

{% highlight go %}
//main.go
func GenesisBlock() *Block {
	genesisWords := "Hello, blockchain!"
	return CreateBlock([]byte{}, []byte(genesisWords))
}
{% endhighlight %}
可以看到我们在创始区块中存放了 *Hello, blockchain!* 这段信息。现在我们来构建函数，使得区块链可以根据其它信息创建区块进行储存。
{% highlight go %}
//main.go
func (bc *BlockChain) AddBlock(data string) {
	newBlock := CreateBlock(bc.Blocks[len(bc.Blocks)-1].PrevHash, []byte(data))
	bc.Blocks = append(bc.Blocks, newBlock)
}
{% endhighlight %}
最后我们构建一个区块链初始化函数，使其返回一个包含创始区块的区块链。
{% highlight go %}
//main.go
func CreateBlockChain() *BlockChain {
	blockchain := BlockChain{}
	blockchain.Blocks = append(blockchain.Blocks, GenesisBlock())
	return &blockchain
}
{% endhighlight %}
现在我们已经拥有了所有创建区块链需要的函数了，来看看我们的区块链是怎么运作的。
{% highlight go %}
//main.go
func main() {
	blockchain := CreateBlockChain()
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
输出如下。
{% highlight %}
D:\learngo\goblockchain>go run main.go
Timestamp: 1632470217
hash: 681e1d81f71f523fb523f806a9656c70c3d05a9e0504ce754f6cf9d8a1633976
Previous hash:
data: Hello, blockchain!
Timestamp: 1632470218
hash: 8ea5e3d6d94b88fe4c4ce5ec2d4fcd3497ee72033f4c17b114ee75f98a55437e
Previous hash:
data: After genesis, I have something to say.
Timestamp: 1632470219
hash: 0368630f6e6f9361663b001913e9e8138ffff1d1c273e939e2e3c28bce5c25bc
Previous hash:
data: Leo Cao is awesome!
Timestamp: 1632470220
hash: e7f82ea9803abff2bb7a5068eeabad5b427d404f21c246c455ba31ed10c8ee42
Previous hash:
data: I can't wait to follow his github!
{% endhighlight %}

