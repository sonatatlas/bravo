# 第九章 区块链

区块链的数据结构是由包含交易信息的区块按照从远及近的顺序有序链接起来的。它可以被存储为平面文件（flat file），或是存储在一个简单数据库中。  

区块被从远及近有序地链接在这个链条里，每个区块都指向前一个区块。区块链经常被视为一个垂直的栈，第一个区块作为栈底的首区块，随后每个区块都被放置在之前的区块之上。  

虽然每个区块只有一个父区块，但可以暂时拥有多个子区块。每个子区块都将同一区块作为其父区块，并且在“父区块哈希值”字段中具有相同的（父区块）哈希值。一个区块出现多个子区块的情况被称为“区块链分叉”。区块链分叉只是暂时状态，只有当多个不同区块几乎同时被不同的矿工发现时才会发生（参见“区块链分叉”）。最终，只有一个子区块会成为区块链的一部分，同时解决了“区块链分叉”的问题。尽管一个区块可能会有不止一个子区块，但每一区块只有 一个父区块，这是因为一个区块只有一个“父区块哈希值”字段可以指向它的唯一父区块。  

由于区块头里面包含“父区块哈希值”字段，所以当前区块的哈希值也受到该字段的影响。如果父区块的身份标识发生变化，子区块的身份标识也会跟着变化。当父区块有任何改动时，父区块的哈希值也发生变化。这将迫使子区块的“父区块哈希值”字段发生改变，从而又将导致子区块的哈希值发生改变。而子区块的哈希值发生改变又将迫使孙区块的“父区块哈希值”字段发生改变，又因此改变了孙区块哈希值，以此类推。  

### 区块结构

区块是一种被包含在公开账簿（区块链）里的聚合了交易信息的容器数据结构。它由一个包含元数据的区块头和紧跟其后的构成区块主体的一长串交易列表组成。区块头是80字节，而平均每个交易至少是250字节，而且平均每个区块至少包含超过500个交易。因此，一个包含所有交易的完整区块比区块头大1000倍。  

| Size                 | Field               | Description                                           |
|:--------------------:|:-------------------:|:-----------------------------------------------------:|
| 4 bytes              | Block Size          | The size of the block, in bytes, following this field |
| 80 bytes             | Block Header        | Several fields form the block header                  |
| 1 - 9 bytes (Varlnt) | Transaction Counter | How many transactions follow                          |
| Variable             | Transactions        | The transactions recorded in this block               |

### 区块头

区块头由三组区块元数据组成。  

首先是一组引用父区块哈希值的数据，这组元数据用于将该区块与区块链中前一区块相连接。  

第二组元数据，即难度、时间戳和 nonce，与挖矿竞争相关，详见挖矿章节。  

第三组元数据是 merkle 树根（一种用来有效地总结区块中所有交易的数据结构）。  

| Size     | field               | Description                                                           |
|:--------:|:-------------------:|:---------------------------------------------------------------------:|
| 4 bytes  | version             | A version number to track software / protocl upgrades                 |
| 32 bytes | Previous Block Hash | A reference to the hash of the previous (parennt) block in the chain  |
| 32 bytes | Merkle Root         | A hash of the root of the merkle tree of this block's transactions    |
| 4 bytes  | Timestamp           | The approximate creation time of this block (seconds from this block) |
| 4 bytes  | Difficulty Target   | The Proof-of-work algorithm difficulty target for this block          |
| 4 bytes  | Nonce               | A counter used for the Proof-of-Work algorithm                        |

### 区块标识符: 区块头哈希值和区块高度

区块主标识符是它的加密哈希值。  

一个通过SHA256算法对区块头进行二次哈希计算而得到的数字指纹。区块哈希值实际上并不包含在区块的数据结构里，不管是该区块在网络上传输时，抑或是它作为区块链的一部分被存储在某节点的永久性存储设备上时。 相反，区块哈希值是当该区块从网络被接收时由每个节点计算出来的。区块的哈希值可能会作为区块元数据的一部分被存储在一个独立的数据库表中，以便于索引和更快地从磁盘检索区块。  

第二种识别区块的方式是通过该区块在区块链中的位置，即“区块高度（block height）”。第一个区块，其区块高度为 0，和之前哈希值所引用的区块为同一个区块。  

因此，区块可以通过两种方式被识别：区块哈希值或者区块高度。每一个随后被存储在第一个区块之上的区块在区块链中都比前一区块“高”出一个位置，就像箱子一样,一个接一个堆叠在其他箱子之上。2017年1月1日的区块高度大约是 446,000，说明已经有446,000个区块被堆叠在2009年1月创建的第一个区块之上。  

> 一个区块的区块哈希值总是能唯一地识别出一个特定区块。一个区块也总是有特定的区块高度。但是，一 个特定的区块高度并不一定总是能唯一地识别出一个特定区块。更确切地说，两个或者更多数量的区块也许会为了区块链中的一个位置而竞争。

### 区块链接成为区块链

比特币的全节点在本地保存了区块链从创世区块起的完整副本。随着新的区块的产生，该区块链的本地副本会不断地更新用于扩展这个链条。当一个节点从网络接收传入的区块时，它会验证这些区块，然后链接到现有的区块链上。为建立一个连接，一个节点将检查传入的区块头并寻找该区块的“父区块哈希值”。  

从网上接受一个区块，描述如下:  
```
{
"size" : 43560,
"version" : 2,
"previousblockhash" :
    "00000000000000027e7ba6fe7bad39faf3b5a83daed765f05f7d1b71a1632249",
"merkleroot" :
    "5e049f4030e0ab2debb92378f53c0a6e09548aea083f3ab25e1d94ea1155e29d",
"time" : 1388185038,
"difficulty" : 1180923195.25802612,
"nonce" : 4215469401,
"tx" : [
    "257e7497fb8bc68421eb2c7b699dbab234831600e7352f0d9e6522c7cf3f6c77",

 [... many more transactions omitted ...]

    "05cfd38f6ae6aa83674cc99e4d75a1458c165b7ab84725eda41d018a09176634"
 ]
}
```
对于这一新的区块，节点会在“父区块哈希值”字段里找出包含它的父区块的哈希值。这是节点已知的哈希值，也就是第 277314块区块的哈希值。故这个新区块是这个链条里的最后一个区块的子区块，因此现有的区块链得以扩展。节点将新的区块添加至链条的尾端，使区块链变长到一个新的高度277,315。  

![blockchain](/assets/blockchain.png)

### Merkle 树

在比特币网络中，Merkle树被用来归纳一个区块中的所有交易，同时生成整个交易集合的数字指纹，且提供了一种校验区块是否存在某交易的高效途径。生成一棵完整的Merkle树需要递归地对哈希节点对进行哈希，并将新生成的哈希节点插入到Merkle树中，直到只剩一个哈希节点，该节点就是Merkle树的根。在比特币的Merkle树中两次使用到了SHA256 算法，因此其加密哈希算法也被称为double-SHA256。  

当N个数据元素经过加密后插入Merkle树时，你至多计算2*log~2~(N) 次就能检查出任意某数据元素是否在该树中，这使得该数据结构非常高效。

![merkle](/assets/merkle.png)

### Merkle 树和简单支付验证 (SPV)

Merkle树被SPV节点广泛使用。SPV节点不保存所有交易也不会下载整个区块，仅仅保存区块头。它们使用认证路径或者Merkle路径来验证交易存在于区块中，而不必下载区块中所有交易。


# Confuse  🤔️

#### 区块高度与深度


<style>img{padding: 5rem 10rem}</style>
