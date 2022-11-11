## BTC密码学原理

 **BTC用到了密码学的两个功能分别是HASH和签名**

### HASH

密码学中用到的哈希函数被称为cryptographic hash function:  它有两个重要的性质:

- 哈希碰撞

	- 例如x≠y H(x)=H(y) 两个不同的输入，输出却是相等的，这就称哈希碰撞。它是不可避免的，因为输入空间总大于输出空间。给出x，很难找到y，除非蛮力求解(brute-force)。

	- 性质：对一个message求digest
比如message取m m的哈希值是H(m)=digest 如果有人想篡改m值而H(m)不变，则无法做到。哈希碰撞无法人为制造，无法验证，是根据实践经验得来的。


	- 场景：对比两个文件hash是否一致，防止文件被篡改、
	- 区块链块的hash长度是256位

- hiding 哈希函数的计算过程是单向的，不可逆的。

	- (从H(x)无法推导出x) hiding性质前提是输入空间足够大，分布比较均匀。如果不是足够大，一般在x后面拼接一个随机数，如H(x||nonce)。
	
	- 作用：和collision resistance 结合在一起，用来实现digital commitment(又称为digital equivalent of a sealed envelope)
把预测结果作为输入x，算出一个哈希值，将哈希值公布，hiding让人们知道哈希值而不知道预测值，最后再将x公布，因为有collision resistance的性质，预测结果是不可篡改的。

**区块链中用到了密码学的两个性质，并且还有第三个性质：**
	
- puzzle friendly 指哈希值的预算事先是不可预测的。假如哈希值是00...0XX...X，一样事先无法知道哪个值更容易算出这个结果，还是要一个一个带入。


 比特币挖矿的过程中实际就是找一个nonce，nonce跟区块的块头里的其他信息合一起作为输入，得出的哈希值要小于等于某个指定的目标预值。H(block header)≤target。block header 指块头，块头里有很多域，其中一个域是我们可以设置的随机数nonce，挖矿的过程是不停的试随机数，使得block header取哈希后落在指定的范围之内。

	puzzle friendly是指挖矿过程中没有捷径，为了使输出值落在指定范围，只能一个一个去试。所以这个过程还可以作为工作量证明(proof of work)。
挖矿很难，验证很容易。(difficult to solve ,but easy to verify)

**比特币中用的哈希函数叫作SHA-256(secure hash algorithm )以上三个性质它都是满足的。**
	
### 签名

在比特币系统中开账户:

- 在本地创立一个公私钥匙对(public key ,private key)，这就是一个账户。公私钥匙对是来自于非对称的加密技术(asymmetric encryption algorithm)。

- 作用：
	- 私钥解密，保存本地，公钥是公开的用来加密。
	- 用来做签名
		- 签名用的是私钥，验证签名是公钥。 
		
- 公私钥会不会产生相同呢

	- 256位hash值，产生出来的概率微乎其微，超级计算机每天都生成公私钥对，产生出来相同的概率比地球爆炸概率还要低。
	
- 生成公私钥是随机的，依赖随机源

**比特币中对message取hash，然后对hash进行签名。**
 