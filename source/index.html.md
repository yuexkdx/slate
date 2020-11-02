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
2020.11.2  |         | 新增    | 修正文档格式


# 简介

欢迎使用DxChain API。

DxChain API文档提供原生RPC和SDK两种访问方式，方便用户根据业务需求进行使用。

文档右侧提供了原生RPC和SDK调用相关方法的使用方式。

DxChain API的后续更新请参照此[文档](#)。

# 说明

本SDK返回金额相关的单位均为`Camel`

其中: 1 Dx = 10^9 Gcamle = 10^18 Camel

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

```python
import json
from gdx.jsonrpc.transaction.transaction import Transaction

if __name__ == "__main__":
    tx = Transaction()
    transaction = {
        # Note that the address must be in checksum format or native bytes:
        'to': '0x0000000000000000000000000000000000000000',
        'value': 10,
        'gas': 2000000,
        'gasPrice': 234567897654321,
        'nonce': 14,
        'chainId': 0,
        'data': ''
    }
    tr_json = tx.new_transaction(transaction)
    tr = json.loads(tr_json)
    key = '0xc1c3597ad0a514a08200a5b4be5a6a7e88975e6fc1e09ef49c42cc07367bca1c'
    signed_json = tx.sign_tx(tr["result"], key)
    signed_data = json.loads(signed_json)
    tx_hash = tx.send_raw_transaction(signed_data["result"])
    print(tx_hash)
```

<aside class="success">
Golang SDK实现了DxChain的账户创建、批量创建账户以及离线签名方法，具体实现过程详见：<code>gdx-sdk-go/account/account.go</code>
</aside>


方法       | 功能描述
---------   | ------- 
[GetBalance](#eth_getbalance)  | 获取账户余额
[GetAccounts](#eth_accounts) | 获取客户端持有的地址清单
[GetGasPrice](#eth_gasprice) | 获取Gas价格
[GetTxByHash](#eth_gettransactionbyhash) | 根据Hash获取交易详情
[GetTxReceipt](#eth_gettransactionreceipt)| 根据Hash获取交易收据
[GetTxCount](#eth_gettransactioncount)  | 根据地址获取交易数量
[GetBlockNumber](#eth_getblocknumber)|获取区块高度
[GetBlockByHash](#eth_getblockbyhash)|根据区块Hash获取区块详情
[GetBlockByNumber](#eth_getblockbynumber)|根据区块高度获取区块详情
[GetBlockTxCountByHash](#eth_getblocktransactioncountbyhash)|根据区块Hash获取区块中交易数量
[GetBlockTxCountByNumber](#eth_getblocktransactioncountbynumber)|根据区块高度获取区块中交易数量
[GetValidatorsByBlockNumber](#dpos_validators)|根据高区块度获取出块节点
[GetValidatorInfo](#dpos_validator)|根据出块节点地址和区块高度，获取出块节点详情
[GetCandidatesByBlockNumber](#dpos_candidates)|根据区块高度获取候选节点
[GetCandidateInfo](#dpos_candidate)|根据候选节点地址和区块高度，获取候选节点详情
[GetEpochID](#dpos_epochid)|根据区块高度获取周期号
[GetVoteDeposit](#dpos_votedeposit)|根据矿机地址和区块高度，获取质押数
[GetCandidateDeposit](#dpos_candidatedeposit)|根据候选节点和区块高度，获取候选节点质押数
[GetVotedCandidatesByAddress](#dpos_getvotedcandidatesbyaddress)|根据矿机地址和区块高度，获取投票的候选节点
[GetAllVotesOfCandidate](#dpos_getallvotesofcandidate)|根据候选节点地址和区块高度，获取候选节点的投票信息
[GetAllVotesOfValidator](#dpos_getallvotesofvalidator)|根据出块节点地址和区块高度，获取出块节点的投票信息
[GetValidatorDistribution](#dpos_getvalidatordistribution)|根据出块节点地址和起止高度，获取高度间出块节点的奖励分配信息
[GetValidatorReward](#dpos_getvalidatorreward)|根据出块节点地址和起止高度，获取高度间出块节点的总奖励
[GetBlockReward](#dpos_getblockreward)|根据区块高度，获取区块的奖励值
[GetEpochInitDepositByNumber](#dpos_getepochinitdepositbynumber)|根据区块高度，查询对应周期的质押数
[GetConfirmedBlockNumber](#dpos_getconfirmedblocknumber)|获取不可逆块高


## Python SDK

Python SDK采用python实现DxChain RPC相关接口， 具体使用方式参照右侧python代码区

方法     | 功能描述
--------- | ------- 
[get_balance](#eth_getbalance)  | 获取账户余额
[accounts](#eth_accounts) | 获取客户端持有的地址清单
[get_gas_price](#eth_gasprice) | 获取Gas价格
[get_transaction_by_hash](#eth_gettransactionbyhash) | 根据Hash获取交易详情
[get_transaction_receipts_by_hash](#eth_gettransactionreceipt)| 根据Hash获取交易收据
[get_transaction_count](#eth_gettransactioncount) | 根据地址获取交易数量
[get_block_height](#eth_getblocknumber)|获取区块高度
[get_block_by_hash](#eth_getblockbyhash)|根据区块Hash获取区块详情
[get_block_by_number](#eth_getblockbynumber)|根据区块高度获取区块详情
[get_block_transaction_count_by_hash](#eth_getblocktransactioncountbyhash)|根据区块Hash获取区块中交易数量
[get_block_transaction_count_by_number](#eth_getblocktransactioncountbynumber)|根据区块高度获取区块中交易数量
[get_validators](#dpos_validators)|根据高区块度获取出块节点
[get_validator](#dpos_validator)|根据出块节点地址和区块高度，获取出块节点详情
[get_candidates](#dpos_candidates)|根据区块高度获取候选节点
[get_candidate](#dpos_candidate)|根据候选节点地址和区块高度，获取候选节点详情
[get_epoch_id](#dpos_epochid)|根据区块高度获取周期号
[get_vote_deposit](#dpos_votedeposit)|根据矿机地址和区块高度，获取质押数
[get_candidate_deposit](#dpos_candidatedeposit)|根据候选节点和区块高度，获取候选节点质押数
[get_voted_candidates_by_address](#dpos_getvotedcandidatesbyaddress)|根据矿机地址和区块高度，获取投票的候选节点
[get_all_votes_of_candidate](#dpos_getallvotesofcandidate)|根据候选节点地址和区块高度，获取候选节点的投票信息
[get_all_votes_of_validator](#dpos_getallvotesofvalidator)|根据出块节点地址和区块高度，获取出块节点的投票信息
[get_validator_distribution](#dpos_getvalidatordistribution)|根据出块节点地址和起止高度，获取高度间出块节点的奖励分配信息
[get_validator_reward](#dpos_getvalidatorreward)|根据出块节点地址和起止高度，获取高度间出块节点的总奖励
[get_block_reward](#dpos_getblockreward)|根据区块高度，获取区块的奖励值
[get_epoch_init_deposit](#dpos_getepochinitdepositbynumber)|根据区块高度，查询对应周期的质押数
[get_confirmed_block_number](#dpos_getconfirmedblocknumber)|获取不可逆块高

# 账户相关

## eth_getBalance

获取账户地址在指定高度的余额，默认高度为(`latest`)最新高度

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

```python
from gdx.jsonrpc.account.account import Account

account = Account()
print(account.get_balance("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5"))
```

#### 请求参数

参数名称    | 是否必需    | 描述
--------- | ------- | -----------
address   |  是    | 账户地址
height    | 否     | 区块高度

#### 返回值

参数名称       |  描述
--------- |  -----------
total_balance| 账户总金额
available_balance|可用金额
frozen_assets|冻结金额

## eth_accounts

获取客户端持有的地址列表

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"eth_accounts","params":[],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()

	accounts, err := client.GetAccounts()
	if err != nil {
		panic("get accounts error, err = "+ err.Error())
	}

	fmt.Println("accounts = ", accounts)
}
```

```python
from gdx.jsonrpc.account.account import Account

account = Account()
print(account.accounts())
```

#### 请求参数

#### 返回值

参数名称    |  描述
--------- | -----------
result |  客户端持有的地址列表

## eth_gasPrice

返回当前链的gas price

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	gasPrice, err := client.GetGasPrice()
	if err != nil {
		panic("get gas price error, err = " + err.Error())
	}
	fmt.Println("gasPrice = ", gasPrice)
}
```

```python
from gdx.jsonrpc.account.account import Account

account = Account()
print(account.get_gas_price())
```

#### 请求参数

#### 返回值

参数名称    |  描述
--------- |  -----------
result | 当前链的gas price

# 交易相关

## eth_getTransactionByHash

根据hash获取交易详情


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

```python
from gdx.jsonrpc.transaction.transaction import Transaction

transaction = Transaction()
print(transaction.get_transaction_by_hash("0x856ba402ba84232e1d32a569262978272b5179fb14d812bca9e42480c8ed3606"))
```

#### 请求参数

| 参数名称 | 是否必需 | 描述       |
| -------- | -------- | ---------- |
| hash     | 是       | 交易哈希值 |

#### 返回参数

| 参数名称    | 描述                                                       |
| ----------- | ---------------------------------------------------------- |
| BlockHash   | 区块哈希值                                                 |
| BlockNumber | 区块高度                                                     |
| Gas         | 所有计算量的计价单位                                       |
| GasPrice    | 表示交易发送方对每单位 Gas 愿意支付的价格（以 `camel` 计量）。 |
| Hash        | 交易哈希值                                                 |
| Input       | 填充数据                                                   |
| Nonce       | 此账户发出的交易序号数                                     |
| R、S、V     | 在交易的密码学签名中用到的值，可以用于确定交易的发送方。   |
| To          | 交易方                                                     |
| TxIndex     | 交易序号                                                   |
| Value       | 转账金额                                                 |

## eth_getTransactionReceipt

根据Hash获取交易收据

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
		panic("get transaction receipts error, err = " + err.Error())
	}
	jsonTx, err := json.Marshal(tx)
	if err != nil {
		panic("convert the transaction receipts json error, err = " + err.Error())
	}
	fmt.Println(string(jsonTx))
}
```

```python
from gdx.jsonrpc.transaction.transaction import Transaction

transaction = Transaction()
print(transaction.get_transaction_receipts_by_hash("0x856ba402ba84232e1d32a569262978272b5179fb14d812bca9e42480c8ed3606"))
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

根据地址获取交易数量， 默认值为`latest`表示最新高度

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
	txCount, err := client.GetTxCount("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5", "latest")
	if err != nil {
		panic("get transaction count error, err = " + err.Error())
	}
	fmt.Println("transactionCount = ", txCount)
}
```

```python
from gdx.jsonrpc.transaction.transaction import Transaction

transaction = Transaction()
print(transaction.get_transaction_count("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5", "latest"))
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
	blockHeight, err := client.GetBlockNumber()
	if err != nil {
		panic("get block height error, err = " + err.Error())
	}
	fmt.Println("blockHeight = ", blockHeight)
}
```

```python
from gdx.jsonrpc.block.block import Block

block = Block()
print(block.get_block_height())
```

#### 请求参数


#### 返回参数

| 参数名称 | 描述   |
| -------- | ------ |
| result   | 区块高度 |


## eth_getBlockByHash

根据区块Hash获取区块详情

<aside class="notice">
参数中flag控制区块中交易的返回形式; flag为true 返回交易详情， flag为talse 返回交易Hash
</aside>

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
		panic("get block info error, err = " + err.Error())
	}
	jsonBlock, err := json.Marshal(block)
	if err != nil {
		panic("convert the block json error, err = " + err.Error())
	}
	fmt.Println(string(jsonBlock))
}
```

```python
from gdx.jsonrpc.block.block import Block

block = Block()
print(block.get_block_by_hash("0x0456e6f0d2942f866356e142a681b13974ec328ac05ebb14f17571e6019c758a", True))
```

#### 请求参数

| 参数名称 | 是否必需 | 描述                                        |
| -------- | -------- | ------------------------------------------- |
| hash     | 是       | 区块哈希                                    |
| flag     | 是       | true返回交易详情，false返回交易哈希 |

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
		panic("get block info error, err = " + err.Error())
	}
	jsonBlock, err := json.Marshal(block)
	if err != nil {
		panic("convert the block json error, err = " + err.Error())
	}
	fmt.Println(string(jsonBlock))
}
```

```python
from gdx.jsonrpc.block.block import Block

block = Block()
print(block.get_block_by_number("1024", True))
```

#### 请求参数

| 参数名称 | 是否必需 | 描述                                        |
| -------- | -------- | ------------------------------------------- |
| hash     | 是       | 区块哈希                                    |
| flag     | 是       | true返回交易详情，false返回交易哈希|

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
		panic("get block transaction count error, err = " + err.Error())
	}
	fmt.Println("txCount = ", txCount)
}
```

```python
from gdx.jsonrpc.block.block import Block

block = Block()
print(block.get_block_transaction_count_by_hash("0x0456e6f0d2942f866356e142a681b13974ec328ac05ebb14f17571e6019c758a"))
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
		panic("get block transaction count error, err = " + err.Error())
	}
	fmt.Println("txCount = ", txCount)
}
```

```python
from gdx.jsonrpc.block.block import Block

block = Block()
print(block.get_block_transaction_count_by_number("latest"))
```

#### 请求参数

| 参数名称 | 是否必需 | 描述   |
| -------- | -------- | ------ |
| height   | 是       | 区块高度 |

#### 返回参数

| 参数名称 | 描述     |
| -------- | -------- |
| result   | 交易数量 |

# 共识相关

## dpos_validators

根据区块高度查询对应的超级节点列表，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_validators","params":[],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	validators, err := client.GetValidatorsByBlockNumber(1000)
	if err != nil {
		panic("get validators error, err = " + err.Error())
	}
	fmt.Println("validators = ", validators)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_validators())
```


#### 请求参数

| 参数名称 | 是否必需 | 描述   |
| -------- | -------- | ------ |
| height   | 否       | 区块高度 |

#### 返回参数

| 参数名称 | 描述         |
| -------- | ------------ |
| result   | 超级节点列表 |

## dpos_validator

根据地址和区块高度查询对应的超级节点详细信息，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_validator","params":["0xc476f174ce3b5e6b7928d9faa153b824502c19ac"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	validator, err := client.GetValidatorInfo("0xc476f174ce3b5e6b7928d9faa153b824502c19ac", 1000)
	if err != nil {
		panic("get validator error, err = " + err.Error())
	}
	fmt.Printf("validator = %v\n", validator)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_validator("0xc476f174ce3b5e6b7928d9faa153b824502c19ac"))
```

#### 请求参数

| 参数名称 | 是否必需 | 描述         |
| -------- | -------- | ------------ |
| address  | 是       | 出块节点地址 |
| height   | 否       | 区块高度       |

#### 返回参数

| 参数名称            | 描述                   |
| ------------------- | ---------------------- |
| validator           | 出块节点地址           |
| total_vote          | 总票数                 |
| current_epoch       | 查询高度对应的周期     |
| epoch_mined_blocks  | 节点在该周期的出块数量 |
| reward_distribution | 奖励分配比例           |

## dpos_candidates

根据区块高度查询对应的候选节点列表，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_candidates","params":[],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	candidates, err := client.GetCandidatesByBlockNumber(1000)
	if err != nil {
		panic("get candidates error, err = " + err.Error())
	}
	fmt.Println("candidates = ", candidates)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_candidates())
```

#### 请求参数

| 参数名称 | 是否必需 | 描述   |
| -------- | -------- | ------ |
| height   | 否       | 区块高度 |

#### 返回参数

| 参数名称 | 描述         |
| -------- | ------------ |
| result   | 候选节点列表 |

## dpos_candidate

根据地址和区块高度查询对应的候选节点详细信息，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_candidate","params":["0xc476f174ce3b5e6b7928d9faa153b824502c19ac"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	candidate, err := client.GetCandidateInfo("0xc476f174ce3b5e6b7928d9faa153b824502c19ac", 1000)
	if err != nil {
		panic("get candidate error, err = " + err.Error())
	}
	fmt.Printf("candidate = %v\n", candidate)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_candidate("0xc476f174ce3b5e6b7928d9faa153b824502c19ac"))
```


#### 请求参数

| 参数名称 | 是否必需 | 描述         |
| -------- | -------- | ------------ |
| address  | 是       | 候选节点地址 |
| height   | 否       | 区块高度       |

#### 返回参数

| 参数名称            | 描述               |
| ------------------- | ------------------ |
| candidate           | 候选节点地址       |
| deposit             | 节点质押数         |
| total_vote          | 总票数             |
| reward_distribution | 声明的奖励分配比例 |

## dpos_epochID

根据区块高度查询对应的周期编号，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_epochID","params":[],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	epochID, err := client.GetEpochID(1000)
	if err != nil {
		panic("get epochID error, err = " + err.Error())
	}
	fmt.Println("epochID = ", epochID)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_epoch_id())
```

#### 请求参数

| 参数名称 | 是否必需 | 描述   |
| -------- | -------- | ------ |
| height   | 否       | 区块高度 |

#### 返回参数

| 参数名称 | 描述     |
| -------- | -------- |
| result   | 周期编号 |

## dpos_voteDeposit

查询投票节点在指定高度的质押信息，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_voteDeposit","params":["0x515a9a17b41024a1e9a41de21f90fa4cc76246c5"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	voteDeposit, err := client.GetVoteDeposit("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5", 1000)
	if err != nil {
		panic("get deposit error, err = " + err.Error())
	}
	fmt.Println("voteDeposit = ", voteDeposit)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_vote_deposit("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5"))
```

#### 请求参数

| 参数名称 | 是否必需 | 描述         |
| -------- | -------- | ------------ |
| address  | 是       | 投票节点地址 |
| height   | 否       | 区块高度       |

#### 返回参数

| 参数名称 | 描述   |
| -------- | ------ |
| result   | 质押数 |

## dpos_candidateDeposit

查询候选节点在指定高度的质押信息，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_candidateDeposit","params":["0xc476f174ce3b5e6b7928d9faa153b824502c19ac"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	candidateDeposit, err := client.GetCandidateDeposit("0xc476f174ce3b5e6b7928d9faa153b824502c19ac", 1000)
	if err != nil {
		panic("get candidateDeposit error, err = " + err.Error())
	}
	fmt.Println("candidateDeposit = ", candidateDeposit)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_candidate_deposit("0xc476f174ce3b5e6b7928d9faa153b824502c19ac"))
```

#### 请求参数

| 参数名称 | 是否必需 | 描述         |
| -------- | -------- | ------------ |
| address  | 是       | 候选节点地址 |
| height   | 否       | 区块高度       |

#### 返回参数

| 参数名称 | 描述   |
| -------- | ------ |
| result   | 质押数 |

## dpos_getVotedCandidatesByAddress

查询投票节点在指定块高时投了哪些候选节点，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_getVotedCandidatesByAddress","params":["0xc476f174ce3b5e6b7928d9faa153b824502c19ac"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	votedCandidates, err := client.GetVotedCandidatesByAddress("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5", nil)
	if err != nil {
		panic("get votedCandidates error, err = " + err.Error())
	}
	fmt.Println("votedCandidates = ", votedCandidates)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_voted_candidates_by_address("0x515a9a17b41024a1e9a41de21f90fa4cc76246c5"))
```

#### 请求参数

| 参数名称 | 是否必需 | 描述         |
| -------- | -------- | ------------ |
| address  | 是       | 投票节点地址 |
| height   | 否       | 区块高度       |

#### 返回参数

| 参数名称 | 描述             |
| -------- | ---------------- |
| result   | 投的候选节点列表 |

## dpos_getAllVotesOfCandidate

查询候选节点在指定块高被哪些投票节点投了，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_getAllVotesOfCandidate","params":["0xc476f174ce3b5e6b7928d9faa153b824502c19ac"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	allVotes, err := client.GetAllVotesOfCandidate("0xc476f174ce3b5e6b7928d9faa153b824502c19ac", 1000)
	if err != nil {
		panic("get allVotes error, err = " + err.Error())
	}
	fmt.Printf("allVotes = %v\n", allVotes)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_all_votes_of_candidate("0xc476f174ce3b5e6b7928d9faa153b824502c19ac"))
```

#### 请求参数

| 参数名称 | 是否必需 | 描述         |
| -------- | -------- | ------------ |
| address  | 是       | 候选节点地址 |
| height   | 否       | 区块高度       |

#### 返回参数

| 参数名称  | 描述     |
| --------- | -------- |
| delegator | 投票节点 |
| vote      | 投票数   |

## dpos_getAllVotesOfValidator

查询超级节点在指定块高被哪些投票节点投了，这里查的是当前周期当选时的投票节点，同一周期查询结果相同，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_getAllVotesOfValidator","params":["0xc476f174ce3b5e6b7928d9faa153b824502c19ac"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	allVotes, err := client.GetAllVotesOfValidator("0xc476f174ce3b5e6b7928d9faa153b824502c19ac", nil)
	if err != nil {
		panic("get allVotes error, err = " + err.Error())
	}
	fmt.Printf("allVotes = %v\n", allVotes)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_all_votes_of_validator("0xc476f174ce3b5e6b7928d9faa153b824502c19ac"))
```

#### 请求参数

| 参数名称 | 是否必需 | 描述         |
| -------- | -------- | ------------ |
| address  | 是       | 超级节点地址 |
| height   | 否       | 区块高度       |

#### 返回参数

| 参数名称  | 描述     |
| --------- | -------- |
| delegator | 投票节点 |
| vote      | 投票数   |

## dpos_getValidatorDistribution

查询某个超级节点，在指定区块期间的所有奖励分配情况，结束区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

<aside class="success">
奖励分配的使用一般是计算一个周期内的奖励分配情况，其起止区块高度为(startEpochBlockHeight, endEpochBlockHeight]
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_getValidatorDistribution","params":["0xc476f174ce3b5e6b7928d9faa153b824502c19ac", "0x2d599"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	distribution, err := client.GetValidatorDistribution("0xc476f174ce3b5e6b7928d9faa153b824502c19ac", 185753, nil)
	if err != nil {
		panic("get distribution error, err = " + err.Error())
	}
	fmt.Printf("distribution = %v\n", distribution)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_validator_distribution("0xc476f174ce3b5e6b7928d9faa153b824502c19ac", 999))
```

#### 请求参数

| 参数名称    | 是否必需 | 描述         |
| ----------- | -------- | ------------ |
| address     | 是       | 超级节点地址 |
| startHeight | 是       | 开始区块高度   |
| endHeight   | 否       | 结束区块高度   |

#### 返回参数

| 参数名称  | 描述                   |
| --------- | ---------------------- |
| delegator | 投票节点               |
| reward    | 投票节点得到的奖励分配 |

## dpos_getValidatorReward

查询某个超级节点，在指定区块期间得到的总奖励，结束区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_getValidatorReward","params":["0xc476f174ce3b5e6b7928d9faa153b824502c19ac", "0x2d599"],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	validatorReward, err := client.GetValidatorReward("0xc476f174ce3b5e6b7928d9faa153b824502c19ac", 999, 1000)
	if err != nil {
		panic("get validatorReward error, err = " + err.Error())
	}
	fmt.Printf("validatorReward = %v\n", validatorReward)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_validator_reward("0xc476f174ce3b5e6b7928d9faa153b824502c19ac", 1000))
```

#### 请求参数

| 参数名称    | 是否必需 | 描述         |
| ----------- | -------- | ------------ |
| address     | 是       | 超级节点地址 |
| startHeight | 是       | 开始区块高度   |
| endHeight   | 否       | 结束区块高度   |

#### 返回参数

| 参数名称 | 描述   |
| -------- | ------ |
| result   | 总奖励 |

## dpos_getBlockReward

查询指定区块高度时的出块奖励，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_getBlockReward","params":[],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	blockReward, err := client.GetBlockReward(1000)
	if err != nil {
		panic("get blockReward error, err = " + err.Error())
	}
	fmt.Println("blockReward = ", blockReward)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_block_reward())
```

#### 请求参数

| 参数名称 | 是否必需 | 描述   |
| -------- | -------- | ------ |
| height   | 否       | 区块高度 |

#### 返回参数

| 参数名称    | 描述     |
| ----------- | -------- |
| totalReward | 总出块奖励 |

## dpos_getEpochInitDepositByNumber

根据区块高度查询对应周期的初始质押数，区块高度默认为最新高度

<aside class="notice">
高度为空值时，查询的结果为最新区块对应的结果
</aside>

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_getEpochInitDepositByNumber","params":[],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	initDeposit, err := client.GetEpochInitDepositByNumber(1000)
	if err != nil {
		panic("get initDeposit error, err = " + err.Error())
	}
	fmt.Printf("initDeposit = %v\n", initDeposit)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_epoch_init_deposit())
```

#### 请求参数

| 参数名称 | 是否必需 | 描述   |
| -------- | -------- | ------ |
| height   | 否       | 区块高度 |

#### 返回参数

| 参数名称              | 描述       |
| --------------------- | ---------- |
| Epoch                 | 所在周期   |
| totalCandidateDeposit | 初始质押数 |


## dpos_getConfirmedBlockNumber

查询最新不可逆的块

```shell


curl -X POST --data '{"jsonrpc":"2.0","method":"dpos_getConfirmedBlockNumber","params":[],"id":3}' -H 'Content-Type: application/json' http://127.0.0.1:11688
```

```go
package main

import (
	"fmt"
	"github.com/DxChainNetwork/gdx-sdk/gdx-sdk-go/jsonrpc"
)

func main() {
	client := jsonrpc.NewClient()
	block, err := client.GetConfirmedBlockNumber()
	if err != nil {
		panic("get block error, err = " + err.Error())
	}
	fmt.Println("block = ", block)
}
```

```python
from gdx.jsonrpc.dpos.dpos import Dpos

dpos = Dpos()
print(dpos.get_confirmed_block_number())
```

#### 请求参数

| 参数名称 | 是否必需 | 描述 |
| -------- | -------- | ---- |


#### 返回参数

| 参数名称 | 描述 |
| -------- | ---- |
| result   | 不可逆区块高度 |
