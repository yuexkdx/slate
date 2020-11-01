---
title: DxChain API

language_tabs: 
  - shell
  - go
  - python

toc_footers:
  - <a href='https://www.dxchain.com'>DxChain官网</a>
  - <a href='https://shequ.dxchain.com/'>DxChain社区</a>

search: true

code_clipboard: true
---

# 更新日志

修正时间    | 接口     | 类型    | 描述
---------  | ------- | -------|-----------
2020.10.28 |         | 新增    | 添加所有接口文档


# 简介

欢迎使用DxChain API。

DxChain API文档提供原生RPC和SDK两种访问方式，方便用户根据业务需求进行使用。

文档右侧提供了原生RPC和SDK调用相关方法的使用方式。

DxChain API的后续更新请参照此[文档](#)。

# SDK接口描述

目前我们提供Golang，Python两个语言版本的[SDK](https://github.com/DxChainNetwork/gdx-sdk)，并在SDK中内置了DxChain的所有API接口，具体接口名称参考SDK接口描述。

`Golang SDK` 返回值采用了**<font color="red">结构体</font>**返回，方便用户直接使用该值。

`Python SDK` 返回值采用的**<font color="red">json格式</font>**返回，用户可以根据返回JSON获取目标值。

## Golang SDK

Golang SDK 采用Golang实现DxChain RPC相关接口，具体使用方式参照右侧go代码区

<aside class="notice">
Golang SDK IP和Port的配置的配置位置为： <code>gdx-sdk-to/constant/constant.go</code> 生产环境中需要将该处的IP和端口换成成产环境中DxChain的IP和端口。
</aside>

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/account"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/constant"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
	"math/big"
	"strings"
)

func sendTransaction() {
	var from = strings.ToLower("0x515a9A17b41024A1E9a41DE21f90FA4cc76246C5")
	var sk = "0xc1c3597ad0a514a08200a5b4be5a6a7e88975e6fc1e09ef49c42cc07367bca1c"

	// 1. create a new account.
	gdxAccount, err := account.CreateAccount()
	if err != nil {
		panic("create the gdx account error， err = " + err.Error())
	}
	fmt.Println("gdx account: " +
		"\nAddress  = " + gdxAccount.Address,
		"\nPublicKey = " + gdxAccount.PublicKey,
		"\nPrivyKey =" + gdxAccount.PrivyKey)

	// 2. create the jsonRpc client.
	client := jsonrpc.NewClient()

	// 3. get from nonce.
	nonce, err := client.GetTxCount(strings.ToLower(from), nil)
	if err != nil {
		panic("get from nonce error, err = " + err.Error())
	}

	// 4. create the new transaction.
	tx, err := jsonrpc.NewTransaction(nonce, gdxAccount.Address, big.NewInt(1000000000000000000),
		constant.DefaultGasLimit, big.NewInt(constant.DefaultGasPrice), nil)
	if err != nil {
		panic("create the transaction error, err = " + err.Error())
	}

	// 5. signature the transaction according to the secret key.
	sigData, err := account.SignTx(tx, sk)
	if err != nil {
		panic("signature the transaction error, err = " + err.Error())
	}

	// 6. send the transaction and get transaction hash
	hash, err := client.SendTransaction(sigData)
	if err != nil {
		panic("send transaction error, err = " + err.Error())
	}

	fmt.Println("send transaction successful, hash = ", hash)
}

func main() {
	sendTransaction()
}
```

方法名       | 功能描述
---------   | ------- 
GetBalance  | [获取账户余额](#eth_getbalance)
GetAccounts | [获取客户端持有的地址清单](#eth_accounts)
GetGasPrice | [获取Gas价格](#eth_gasprice)
GetTxByHash | [根据Hash获取交易详情](#eth_gettransactionbyhash)
GetTxReceipt| [根据Hash获取交易收据](#eth_gettransactionreceipt)
GetTxCount  | [根据地址获取交易数量](#eth_gettransactioncount)
GetBlockNumber|[获取区块高度](#eth_getblocknumber)
GetBlockByHash|[根据区块Hash获取区块详情](#eth_getblockbyhash)
GetBlockByNumber|[根据区块高度获取区块详情](#eth_getblockbynumber)
GetBlockTxCountByHash|[根据区块Hash获取区块中交易数量](#eth_getblocktransactioncountbyhash)
GetBlockTxCountByNumber|[根据区块高度获取区块中交易数量](#eth_getblocktransactioncountbynumber)

## Python SDK

Python SDK采用python实现DxChain RPC相关接口， 具体使用方式参照右侧python代码区

方法名     | 功能描述
--------- | ------- 
get_balance|[获取账户余额](#eth_getbalance)



# 账户相关

## eth_getBalance

根据账户地址和区块高度获取账户余额，默认高度为(`latest`)最新高度

```python
from gdx.jsonrpc.account.account import Account

account = Account()
print(account.get_balance("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5"))
```

```shell
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x515a9a17b41024a1e9a41de21f90fa4cc76246c5","latest"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
	"math/big"
)

func main() {
	client := jsonrpc.NewClient()
	account, err := client.GetBalance("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5", nil)
	if err != nil {
		panic("get balance error, err = " + err.Error())
	}
	fmt.Println("AvailableBalance = ", account.AvailableBalance.Div(account.AvailableBalance, big.NewInt(1e18)), "dx")
}
```

#### 请求参数

参数名称    | 是否必需    | 描述
--------- | ------- | -----------
address   |  是    | 账户地址
height    | 否     | 区块高度

#### 返回值

参数名称       | 是否必需 | 描述
--------- | ------- | -----------
total_balance|      | 账户总金额
available_balance||可用金额
frozen_assets||冻结金额

## eth_accounts

获取客户端持有的地址列表

## eth_gasPrice

返回当前链的gas price 单位为`camel`

```python
from gdx.jsonrpc.account.account import Account

account = Account()
print(account.get_balance("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5"))
```

```shell
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
	"math/big"
)

func main() {
	client := jsonrpc.NewClient()
	account, err := client.GetBalance("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5", nil)
	if err != nil {
		panic("get balance error, err = " + err.Error())
	}
	fmt.Println("AvailableBalance = ", account.AvailableBalance.Div(account.AvailableBalance, big.NewInt(1e18)), "dx")
}
```

# 交易相关

## eth_getTransactionByHash

根据hash获取交易详情

```python
from gdx.jsonrpc.transaction.transaction import Transaction

transaction = Transaction()
print(transaction.get_transaction_by_hash("0x856ba402ba84232e1d32a569262978272b5179fb14d812bca9e42480c8ed3606"))
```

```shell
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionByHash","params":["0x856ba402ba84232e1d32a569262978272b5179fb14d812bca9e42480c8ed3606"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"encoding/json"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
	"strings"
)

func main() {
	client := jsonrpc.NewClient()
	tx, err := client.GetTxByHash("0x856ba402ba84232e1d32a569262978272b5179fb14d812bca9e42480c8ed3606")
	if err != nil {
		panic("get transaction error, err = " + err.Error())
	}
	jsonTx, err := json.Marshal(tx)
	if err != nil {
		panic("convert the transaction json error, err = " + err.Error())
	}
	fmt.Println(string(jsonTx))
}
```

#### 请求参数

| 参数名称 | 是否必需 | 描述       |
| -------- | -------- | ---------- |
| hash     | 是       | 交易哈希值 |

#### 返回参数

| 参数名称    | 描述                                                       |
| ----------- | ---------------------------------------------------------- |
| BlockHash   | 区块哈希值                                                 |
| BlockNumber | 区块号                                                     |
| Gas         | 所有计算量的计价单位                                       |
| GasPrice    | 表示交易发送方对每单位 Gas 愿意支付的价格（以 Wei 计量）。 |
| Hash        | 交易哈希值                                                 |
| Input       | 填充数据                                                   |
| Nonce       | 此账户发出的交易序号数                                     |
| R、S、V     | 在交易的密码学签名中用到的值，可以用于确定交易的发送方。   |
| To          | 交易方                                                     |
| TxIndex     | 交易序号                                                   |
| Value       | 代币数量                                                   |

## eth_getTransactionReceipt

根据Hash获取交易收据

```python
from gdx.jsonrpc.transaction.transaction import Transaction

transaction = Transaction()
print(transaction.get_transaction_by_hash("0x856ba402ba84232e1d32a569262978272b5179fb14d812bca9e42480c8ed3606"))
```

```shell
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionReceipt","params":["0x856ba402ba84232e1d32a569262978272b5179fb14d812bca9e42480c8ed3606"],"id":3}' -H 'Content-Type: application/json' http://13.212.17.162:11688
```

```go
package main

import (
	"fmt"
	"encoding/json"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
	"strings"
)

func main() {
	client := jsonrpc.NewClient()
	tx, err := client.GetTxReceipt("0x856ba402ba84232e1d32a569262978272b5179fb14d812bca9e42480c8ed3606")
	if err != nil {
		panic("get transaction error, err = " + err.Error())
	}
	jsonTx, err := json.Marshal(tx)
	if err != nil {
		panic("convert the transaction json error, err = " + err.Error())
	}
	fmt.Println(string(jsonTx))
}
```

#### 请求参数

| 参数名称 | 是否必需 | 描述   |
| -------- | -------- | ------ |
| hash     | 是       | hash值 |

#### 返回参数

| 参数名称          | 描述                                             |
| ----------------- | ------------------------------------------------ |
| BlockHash         | 区块哈希值                                       |
| BlockNumber       | 区块高度                                           |
| CumulativeGasUsed | 区块中交易预计消耗的 gas 总量                  |
| From              | 发起这个交易的账户或者用户                     |
| GasUsed           | 区块中交易所实际消耗的 gas 总量。                |
| LogsBloom         | 布隆过滤器，用于判断某区块的交易是否产生了某日志 |
| Status            | 交易状态                                         |
| To                | 交易方                                           |
| TxHash            | 交易哈希                                         |
| TxIndex           | 交易在区块中索引                                         |
| Logs              | 交易事件日志                                     |

## eth_getTransactionCount

根据地址获取交易数量

```python
from gdx.jsonrpc.transaction.transaction import Transaction

transaction = Transaction()
print(transaction.get_transaction_count("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5", "latest"))
```

```shell
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionCount","params":["0x515a9a17b41024a1e9a41de21f90fa4cc76246c5","latest"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	nonce, err := client.GetTxCount("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5", "latest")
	if err != nil {
		panic("get nonce error, err = " + err.Error())
	}
	fmt.Println("nonce = ", nonce)
}
```

#### 请求参数

| 参数名称 | 是否必需 | 描述     |
| -------- | -------- | -------- |
| address  | 是       | 账户地址 |
| height   | 是       | 区块高度 |

#### 返回参数

| 参数名称 | 描述     |
| -------- | -------- |
| result   | 交易数量 |

# 区块相关

## eth_getBlockNumber

获取最新区块高度

```python
from gdx.jsonrpc.block.block import Block

block = Block()
print(block.get_block_height())
```

```shell
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688 
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	number, err := client.GetBlockNumber()
	if err != nil {
		panic("get nonce error, err = " + err.Error())
	}
	fmt.Println("number = ", number)
}
```

#### 请求参数


#### 返回参数

| 参数名称 | 描述   |
| -------- | ------ |
| result   | 区块高度 |

## eth_getBlockByHash

根据区块Hash获取区块详情

```python
from gdx.jsonrpc.block.block import Block

block = Block()
print(block.get_block_height_by_hash("0x0456e6f0d2942f866356e142a681b13974ec328ac05ebb14f17571e6019c758a", True))
```

```shell
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByHash","params":["0x0456e6f0d2942f866356e142a681b13974ec328ac05ebb14f17571e6019c758a",true],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	block, err := client.GetBlockByHash("0x0456e6f0d2942f866356e142a681b13974ec328ac05ebb14f17571e6019c758a",true)
	if err != nil {
		panic("get nonce error, err = " + err.Error())
	}
	jsonBlock, err := json.Marshal(block)
	if err != nil {
		panic("convert the block json error, err = " + err.Error())
	}
	fmt.Println(string(jsonBlock))
}
```

#### 请求参数

| 参数名称 | 是否必需 | 描述                                        |
| -------- | -------- | ------------------------------------------- |
| hash     | 是       | 区块哈希                                    |
| flag     | 是       | true返回区块交易详情，false返回区块交易哈希 |

#### 返回参数

| 参数名称         | 描述                                                        |
| ---------------- | ----------------------------------------------------------- |
| CoinBase         | 矿工收取奖励地址                                                    |
| Difficulty       | 此区块的难度值                                              |
| ExtraData        | 填充数据                                                    |
| GasLimit         | gas limit 标示了该区块所记录的所有交易可以使用的 gas 总量 |
| GasUsed          | 区块中各条交易所实际消耗的 gas 总量                        |
| Hash             | 区块哈希值                                                  |
| LogsBloom        | 布隆过滤器                                                  |
| MixHash          | 用于验证一个区块是否被真正记录到链上的哈希值                |
| Nonce            | 和 mixHash 一样，用于验证区块是否被真正记录到链上的值       |
| Number           | 区块高度                                                      |
| ParentHash       | 父区块哈希值                                                |
| ReceiptsRoot     | 交易收据根节点哈希值                                        |
| Sha3Uncles       | 叔块哈希值                                                  |
| Size             | 当前区块的字节大小                                          |
| StateRoot        | 世界状态根哈希值                                            |
| TimeStamp        | 时间戳                                                      |
| TotalDifficulty  | 挖矿难度                                                    |
| Transactions     | 交易详情                                                    |
| TransactionsRoot | 交易根节点哈希值                                            |
| Uncles           | 叔块信息，一般为空                                          |
| Validator        | 矿工地址                                                  |

## eth_getBlockByNumber

根据区块高度获取区块详情

```python
from gdx.jsonrpc.block.block import Block

block = Block()
print(block.get_block_height_by_number("1024", True))
```

```shell
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x400",true],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	block, err := client.GetBlockByNumber(1024,true)
	if err != nil {
		panic("get nonce error, err = " + err.Error())
	}
	jsonBlock, err := json.Marshal(block)
	if err != nil {
		panic("convert the block json error, err = " + err.Error())
	}
	fmt.Println(string(jsonBlock))
}
```

#### 请求参数

| 参数名称 | 是否必需 | 描述                                        |
| -------- | -------- | ------------------------------------------- |
| hash     | 是       | 区块哈希                                    |
| flag     | 是       | true返回区块交易详情，false返回区块交易哈希 |

#### 返回参数

| 参数名称         | 描述                                                        |
| ---------------- | ----------------------------------------------------------- |
| CoinBase         | 矿工收取奖励地址                                                    |
| Difficulty       | 此区块的难度值                                              |
| ExtraData        | 填充数据                                                    |
| GasLimit         | gas limit 标示了该区块所记录的所有交易可以使用的 gas 总量。 |
| GasUsed          | 区块中各条交易所实际消耗的 gas 总量。                       |
| Hash             | 区块哈希值                                                  |
| LogsBloom        | 布隆过滤器                                                  |
| MixHash          | 用于验证一个区块是否被真正记录到链上的哈希值                |
| Nonce            | 和 mixHash 一样，用于验证区块是否被真正记录到链上的值       |
| Number           | 区块高度                                                      |
| ParentHash       | 父区块哈希值                                                |
| ReceiptsRoot     | 交易收据根节点哈希值                                        |
| Sha3Uncles       | 叔块哈希值                                                  |
| Size             | 当前区块的字节大小                                          |
| StateRoot        | 世界状态根哈希值                                            |
| TimeStamp        | 时间戳                                                      |
| TotalDifficulty  | 挖矿难度                                                    |
| Transactions     | 交易详情                                                    |
| TransactionsRoot | 交易根节点哈希值                                            |
| Uncles           | 叔块信息，一般为空                                          |
| Validator        | 矿工地址                                                  |

## eth_getBlockTransactionCountByHash

根据区块Hash获取区块中的交易数量

```python
from gdx.jsonrpc.block.block import Block

block = Block()
print(block.get_block_transaction_count_by_hash("0x0456e6f0d2942f866356e142a681b13974ec328ac05ebb14f17571e6019c758a"))
```

```shell
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockTransactionCountByHash","params":["0x0456e6f0d2942f866356e142a681b13974ec328ac05ebb14f17571e6019c758a"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	txCount, err := client.GetBlockTxCountByHash("0x0456e6f0d2942f866356e142a681b13974ec328ac05ebb14f17571e6019c758a")
	if err != nil {
		panic("get nonce error, err = " + err.Error())
	}
	fmt.Println("txCount:", txCount)
}
```

#### 请求参数

| 参数名称 | 是否必需 | 描述     |
| -------- | -------- | -------- |
| hash     | 是       | 区块哈希 |

#### 返回参数

| 参数名称 | 描述     |
| -------- | -------- |
| result   | 交易数量 |

## eth_getBlockTransactionCountByNumber

根据区块高度获取区块中的交易数量，默认区块高度为`latest`

```python
from gdx.jsonrpc.block.block import Block

block = Block()
print(block.get_block_transaction_count_by_number("latest"))
```

```shell
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockTransactionCountByNumber","params":["latest"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	txCount, err := client.GetBlockTxCountByNumber("latest")
	if err != nil {
		panic("get nonce error, err = " + err.Error())
	}
	fmt.Println("txCount:", txCount)
}
```

#### 请求参数

| 参数名称 | 是否必需 | 描述   |
| -------- | -------- | ------ |
| height   | 是       | 区块号 |

#### 返回参数

| 参数名称 | 描述     |
| -------- | -------- |
| result   | 交易数量 |

# 共识相关

## eth_getTransactionByHash

根据交易Hash获取交易详情

## eth_getTransactionReceiptsByHash

根据交易Hash获取交易副本

## eth_getTransactionByHash

根据交易Hash获取交易详情

## eth_getTransactionReceiptsByHash

根据交易Hash获取交易副本

## eth_getTransactionByHash

根据交易Hash获取交易详情

## eth_getTransactionReceiptsByHash

根据交易Hash获取交易副本

# 状态相关

## eth_getTransactionByHash

根据交易Hash获取交易详情

## eth_getTransactionReceiptsByHash

根据交易Hash获取交易副本

## eth_getTransactionByHash

根据交易Hash获取交易详情

## eth_getTransactionReceiptsByHash

根据交易Hash获取交易副本

## eth_getTransactionByHash

根据交易Hash获取交易详情

## eth_getTransactionReceiptsByHash

根据交易Hash获取交易副本


<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>
