---
title: DxChain API

language_tabs: 
  - shell
  - python
  - javascript

toc_footers:
  - <a href='https://www.dxchain.com'>官网</a>
  - <a href='https://shequ.dxchain.com/'>DxChain社区</a>

search: true

code_clipboard: true
---

# 简介

欢迎使用DxChain API，你可以通过DxChain API获取DxChain中的账户、交易、区块和共识相关的信息，并能通过API进行投票和转账操作。此文档在后续工作中会持续更新。

DxChain API包括：原生API和SDK的调用方式。

DxChain API按照接口功能的不同划分为：账户相关、交易相关、区块相关、共识相关和状态相关的接口。

## SDK

开发者可根据自身偏好和使用场景选择合适自己的方式来使用DxChain API。目前我们提供Golang，Python两个语言版本的[SDK](https://github.com/DxChainNetwork/gdx-sdk)，并在SDK中内置了DxChain的所有API接口，具体使用方式可以参考接口的Golang和Python调用。

## 调用方式

`原生API使用方式`:

`Golang SDK使用方式`:

`Python SDK使用方式`:

# 更新说明

修正时间 | 接口| 类型| 描述|
--------- | ------- | ------- |-----------
2020.10.28 | |新增|添加所有接口文档

# API接口说明

DxChain提供测试网和主网的测试地址：

**测试网**：

**主网**：

# 账户相关

# 交易相关

## 根据Hash获取交易详情


## 根据Hash获取交易收据

## 根据地址获取交易数量

## 

# 区块相关

# 共识相关

# 状态相关

## 获取账户余额

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

根据账户地址和高度信息获取账户余额，默认高度为最新高度

### 请求参数

Parameter | Default | Description
--------- | ------- | -----------
ADDRESS||账户地址
BLOCK|latest|区块高度

### 返回值

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>
