## MeatMask API

### 一、钱包提供的API

MetaMask 将全局 API 注入其用户访问的网站window.ethereum。该 API 允许网站请求用户的以太坊账户，从用户连接的区块链读取数据，并建议用户签署消息和交易。提供者对象的存在表明以太坊用户。我们建议使用[@metamask/detect-provider](https://npmjs.com/package/@metamask/detect-provider)在任何平台或浏览器上检测我们的提供商。

- [@metamask/detect-provider](https://npmjs.com/package/@metamask/detect-provider)用来做网站检测，不一定包括ETH的全量API
	- [EIP-1193](https://eips.ethereum.org/EIPS/eip-1193)指定了 Ethereum JavaScript provider API 
	- 连接方式
		
		```
		// This function detects most providers injected at window.ethereum
		import detectEthereumProvider from '@metamask/detect-provider';
		
		const provider = await detectEthereumProvider();
		
		if (provider) {
		  // From now on, this should always be true:
		  // provider === window.ethereum
		  startApp(provider); // initialize your app
		} else {
		  console.log('Please install MetaMask!');
		}
		``` 
- [ethers库](https://www.npmjs.com/package/ethers)，包含了eth的API。一个完整的以太坊钱包实现和 JavaScript（和 TypeScript）实用程序。满足您所有以太坊需求的完整功能。方便DAPP使用

- [MetaMask JSON-RPC API](https://metamask.github.io/api-playground/api-documentation/#wallet_addEthereumChain) MetaMask的api

### 二、如何区分MeatMask是否调用了以太坊API

- MetaMask 使用该ethereum.request(args)方法来包装 RPC API。
- 该 API 基于所有以太坊客户端公开的接口，以及越来越多的其他钱包可能支持或不支持的方法。
- 所有 RPC 方法请求都可以返回错误。确保处理每次调用的错误ethereum.request(args)。
- ethereum.request(args)
	
	```
	interface RequestArguments {
	  method: string;
	  params?: unknown[] | object;
	}
	
	ethereum.request(args: RequestArguments): Promise<unknown>;
	
	```
	- 用于request通过 MetaMask 向以太坊提交 RPC 请求。它返回Promise解析为 RPC 方法调用结果的params和返回值将RPC方法而变化。在实践中，如果一个方法有任何params，它们几乎总是类型的Array<any>
	- 如果请求因任何原因失败，Promise 将拒绝并返回 [Ethereum RPC Error](https://docs.metamask.io/guide/ethereum-provider.html#errors)


### 三、远程调用以太坊API(JSON-RPC)

重要的方法如下:

- [eth_accounts](https://eth.wiki/json-rpc/API#eth_accounts)
- [eth_call](https://eth.wiki/json-rpc/API#eth_call)
- [eth_getBalance](https://eth.wiki/json-rpc/API#eth_getbalance)
- [eth_sendTransaction](https://eth.wiki/json-rpc/API#eth_sendtransaction)
- [eth_sign](https://eth.wiki/json-rpc/API#eth_sign)

### 四、钱包提供的API

[MetaMask JSON-RPC API](https://metamask.github.io/api-playground/api-documentation/#wallet_addEthereumChain)

### 五、Accounts

- 用户帐户在以太坊的各种环境中使用，包括作为标识符和用于签署交易。为了请求用户签名或让用户批准交易，必须能够访问用户的帐户。在wallet methods下面涉及的签名或交易的批准，所有需要发送帐户作为函数参数。

- MetaMask提供程序[Node.jsEventEmitter](https://nodejs.org/api/events.html)实现了应用程序接口。用来监听用户事件。------ **有用的话让前端研究**

	```
	ethereum.on('accountsChanged', function (accounts) {
	  // Time to reload your interface with accounts[0]!
	});
	
	ethereum.on('accountsChanged', (accounts) => {
	  // Handle the new accounts, or lack thereof.
	  // "accounts" will always be an array, but it can be empty.
	});
	
	ethereum.on('chainChanged', (chainId) => {
	  // Handle the new chain.
	  // Correctly handling chain changes can be complicated.
	  // We recommend reloading the page unless you have good reason not to.
	  window.location.reload();
	});
	```
	- 第一个参数ethereum.removeListener是事件名称，第二个参数是对传递给ethereum.on第一个参数中提到的事件名称的相同函数的引用。

- 连接以太坊(和账户没关系。不是dapp连接钱包)

	- 当 MetaMask 提供程序首次能够向链提交 RPC 请求时，它会发出此事件。我们建议使用connect事件处理程序和ethereum.isConnected()方法来确定提供程序何时/是否已连接。

- 链改变

	```
	ethereum.on('chainChanged', handler: (chainId: string) => void);
	```	
	- 当前连接的链发生变化时，MetaMask 提供程序会发出此事件。

	- 所有 RPC 请求都提交到当前连接的链。因此，通过侦听此事件来跟踪当前链 ID 至关重要。我们强烈建议在链更改时重新加载页面。
	
- **总结**

	- **MeatMask封装的方法有很多，大部分功能都可以满足，具体功能的逻辑需要研究代码。文档中只给了部分的示例起到引导作用，无法参透整体逻辑。**


### 六、 Permissions

MetaMask 通过[EIP-2255](https://eips.ethereum.org/EIPS/eip-2255)引入了 Web3 钱包权限 （打开新窗口）. 在这个权限系统中，每个 RPC 方法要么是受限的，要么是开放的。如果某个方法受到限制，则外部域（如 web3 站点）必须具有相应的权限才能调用它。同时，打开方法不需要调用权限，但可能需要用户确认才能成功（例如eth_sendTransaction）。

目前，唯一的权限是eth_accounts，它允许您访问用户的以太坊地址。将来会添加更多权限。

在幕后，权限是简单的、与 JSON 兼容的对象，具有许多主要由 MetaMask 内部使用的字段。以下界面列出了消费者可能感兴趣的字段：

```
interface Web3WalletPermission {
  // The name of the method corresponding to the permission
  parentCapability: string;

  // The date the permission was granted, in UNIX epoch time
  date?: number;
}
```

权限系统在包中实现[rpc-cap](https://github.com/MetaMask/rpc-cap). 如果您有兴趣了解有关此功能启发的权限系统背后的理论的更多信息，我们鼓励您查看[EIP-2255](https://eips.ethereum.org/EIPS/eip-2255).

- eth_requestAccounts
	- 要求用户提供一个以太坊地址以供识别。返回一个解析为单个以太坊地址字符串数组的承诺。如果用户拒绝请求，Promise 将拒绝并返回4001错误。

	- 该请求会导致出现 MetaMask 弹出窗口。您应该只在响应用户操作（例如单击按钮）时请求用户的帐户。当请求仍然挂起时，您应该始终禁用导致请求被分派的按钮。

	- 如果您无法检索用户的帐户，您应该鼓励用户发起帐户请求。 
- wallet_getPermissions
	- 获取调用者的当前权限。返回解析为Web3WalletPermission对象数组的 Promise 。如果调用者没有权限，则数组将为空。 
- wallet_requestPermissions
	- 向用户请求给定的权限。返回一个解析为非空Web3WalletPermission对象数组的 Promise ，对应于调用者的当前权限。如果用户拒绝请求，Promise 将拒绝并返回4001错误。
	- 该请求会导致出现 MetaMask 弹出窗口。您应该只在响应用户操作（例如单击按钮）时请求权限。


### 七、SendTransaction

交易是区块链上的正式行动。它们总是通过调用eth_sendTransaction方法在 MetaMask 中启动。它们可能涉及简单的以太发送，可能导致发送代币、创建新的智能合约或以多种方式更改区块链上的状态。它们总是由来自外部帐户的签名或简单的密钥对发起。


在 MetaMask 中，ethereum.request直接使用该方法，发送交易将涉及组成这样的选项对象：

```
const transactionParameters = {
  nonce: '0x00', // ignored by MetaMask
  gasPrice: '0x09184e72a000', // customizable by user during MetaMask confirmation.
  gas: '0x2710', // customizable by user during MetaMask confirmation.
  to: '0x0000000000000000000000000000000000000000', // Required except during contract publications.
  from: ethereum.selectedAddress, // must match user's active address.
  value: '0x00', // Only required to send ether to the recipient from the initiating external account.
  data:
    '0x7f7465737432000000000000000000000000000000000000000000000000000000600057', // Optional, but used for defining smart contract creation and interaction.
  chainId: '0x3', // Used to prevent transaction reuse across blockchains. Auto-filled by MetaMask.
};

// txHash is a hex string
// As with any RPC call, it may throw an error
const txHash = await ethereum.request({
  method: 'eth_sendTransaction',
  params: [transactionParameters],
});
```
 - 许多事务参数由 MetaMask 为您处理，但最好知道所有参数的作用。

### 八、sign（略）
### 九、call （本地方法调用，立即执行新消息调用，无需在区块链上创建交易。）







