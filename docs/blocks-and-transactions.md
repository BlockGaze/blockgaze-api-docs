# Blocks and transactions

[Back to overview](../README.md)

- [List blocks](#list-blocks)
- [Get a block](#get-a-block)
- [List transactions](#list-transactions)
- [Get a transaction](#get-a-transaction)

## List blocks

`GET /v1/blocks`

Blocks, newest first, with producer, gas, transaction count, fees and reward. `minerLabel` and `minerTags` give the producer's label and tags. Tags are those [Get an address](addresses.md#get-an-address) lists, in the same order and shape; an untagged address has an empty list.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Blocks per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). `from` takes a block number or a `yyyy-MM-dd` date. |

```bash
curl "https://fsnex.net/api/v1/blocks?pageSize=2"
```

Response `200`

```json
{
  "blocks": [
    {
      "number": 15130080,
      "hash": "0xe93ffded087a79097d4309c7831690161db6ed136f4b1a22c4c83a99db80f99f",
      "timestamp": 1759826630,
      "miner": "0x0b5d3768772ece935b5743f1be1a1304b7d47699",
      "minerLabel": null,
      "minerTags": [],
      "gasUsed": 21224,
      "gasLimit": 30000000,
      "transactionCount": 1,
      "totalFees": "132318515417984",
      "blockReward": "312632318515417984"
    },
    {
      "number": 15130079,
      "hash": "0x3f714878f91d2fce6c95e2f0a5a7265c7b8998ef48d83840ac0826140e93c5f7",
      "timestamp": 1759826617,
      "miner": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
      "minerLabel": null,
      "minerTags": [],
      "gasUsed": 21224,
      "gasLimit": 30000000,
      "transactionCount": 1,
      "totalFees": "132318515417984",
      "blockReward": "312632318515417984"
    }
  ],
  "nextCursor": "15130079",
  "previousCursor": null,
  "total": null,
  "indexedFromBlock": 0,
  "indexedToBlock": 15130080
}
```

## Get a block

`GET /v1/blocks/{numberOrHash}`

One block by number or by hash, with its header fields, fees, issuance and reward. `minerLabel` and `minerTags` give the producer's label and tags, as in [List blocks](#list-blocks). `annotations` carries chain-specific notes recorded with the block, such as the Fusion ticket snapshot below.

| Parameter | Description |
| --- | --- |
| `numberOrHash` (path) | A block number, or a 32-byte block hash. |

Returns `400` for anything that is neither, and `404` when no indexed block matches.

```bash
curl "https://fsnex.net/api/v1/blocks/15130080"
```

Response `200` (trimmed: `extraData` cut from 272 characters)

```json
{
  "annotations": [
    {
      "title": "Fusion ticket staking",
      "description": "Historical ticket snapshot recorded in this block. This count describes this block alone, not the newest block the ticket index has reached.",
      "fields": [
        {
          "label": "Tickets at this block",
          "value": "491",
          "format": "text"
        },
        {
          "label": "Selected ticket",
          "value": "0x65932c901cbc1e0f0bc8521585797f530d4bfbc178d279f710269b637a68d6b7",
          "format": "ticket"
        },
        {
          "label": "Retreated tickets",
          "value": "0",
          "format": "text"
        }
      ]
    }
  ],
  "totalFees": "132318515417984",
  "gasFees": "132318515417984",
  "issuance": "312500000000000000",
  "blockReward": "312632318515417984",
  "averageGasPrice": "6234381616",
  "number": 15130080,
  "hash": "0xe93ffded087a79097d4309c7831690161db6ed136f4b1a22c4c83a99db80f99f",
  "parentHash": "0x3f714878f91d2fce6c95e2f0a5a7265c7b8998ef48d83840ac0826140e93c5f7",
  "timestamp": 1759826630,
  "miner": "0x0b5d3768772ece935b5743f1be1a1304b7d47699",
  "minerLabel": null,
  "minerTags": [],
  "gasLimit": 30000000,
  "gasUsed": 21224,
  "baseFeePerGas": "1000000000",
  "difficulty": "499",
  "totalDifficulty": "63370514513",
  "extraData": "0xd883050000846566736e88676f312e31392e33856c696e757800000000000000…",
  "size": 779,
  "nonce": "0x0000000000000000",
  "mixHash": "0xe4e079c525c189eb30ca22ba57bbd4af8b44050e2fc1618c2157107b00820e2c",
  "sha3Uncles": "0xafce7a60c626fc6f076965853a339f14f1ac3373665d92e2e9f90cd7c9b087fe",
  "stateRoot": "0x1526799d6f10f4a4467e4dd4f3b1f81ca1479822bb2c24aa541ac66f889ccffc",
  "transactionsRoot": "0xf6aeefa908d60f34dacc080cb0468633981af4008a3afbce51737e2ed3ca6900",
  "receiptsRoot": "0x25404575e27a7090c3b4cc474357b85f69a33901f443daf80190b275a919deb7",
  "transactionCount": 1
}
```

## List transactions

`GET /v1/transactions`

Transactions, newest first. Without filters this lists every indexed transaction. Each entry has sender and recipient with their labels and tags (`fromTags`, `toTags`), value, fee, gas, outcome, `kind` (`Transfer`, `ContractInteraction` and so on), and `actionName` where the explorer recognises the call. With an `address` filter, `direction` gives which way the transaction went for that address, such as `Out`; it is `null` without one. Tags are those [Get an address](addresses.md#get-an-address) lists, in the same order and shape; an untagged address has an empty list. A contract creation's `toTags` is empty.

| Parameter | Default | Description |
| --- | --- | --- |
| `block` | | Only transactions in this block number. |
| `address` | | Only transactions sent or received by this address. |
| `operation` | | Only transactions of this native operation kind, from `transactionOperationKinds` in [network status](network.md#network-status). FSNEX has Fusion operation kinds such as `swap-take`; BNB Smart Chain has none. Cannot be combined with `block` or `address`. |
| `pageSize` | `25` | Transactions per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

Returns `503` when `page` asks for a page too deep in a long list to reach quickly; use `from` with a block number or date instead.

```bash
curl "https://bnb.blockgaze.com/api/v1/transactions?pageSize=2"
```

Response `200`

```json
{
  "transactions": [
    {
      "fee": "0",
      "blockNumber": 128208994,
      "transactionIndex": 58,
      "hash": "0x02e1514aee227bc4ea22d2a1d62fece25431623836746b9e405644e66fc8052c",
      "timestamp": 1792257496,
      "from": "0xbdcc079bbb23c1d9a6f36aa31309676c258abac7",
      "to": "0x0000000000000000000000000000000000001000",
      "fromLabel": "Fuji",
      "fromTags": [
        {
          "title": "Validator",
          "colour": "Accent",
          "description": "A validator's consensus address: it produces blocks and sends each block's system transactions."
        }
      ],
      "toLabel": "Validator Set",
      "toTags": [],
      "value": "1212097178310679",
      "gasUsed": 55345,
      "effectiveGasPrice": "0",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": null,
      "direction": null
    },
    {
      "fee": "0",
      "blockNumber": 128208994,
      "transactionIndex": 57,
      "hash": "0xefb2d001ddb55566cb46f4cde6ffdb9a4e13a439670a7140533b365745695502",
      "timestamp": 1792257496,
      "from": "0x4848489f0b2bedd788c696e2d79b6b69d7484848",
      "to": "0x487e5dfe70119c1b320b8219b190a6fa95a5bb48",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": null,
      "toTags": [],
      "value": "0",
      "gasUsed": 21000,
      "effectiveGasPrice": "0",
      "succeeded": true,
      "kind": "Transfer",
      "actionName": null,
      "direction": null
    }
  ],
  "nextCursor": "128208994-57",
  "previousCursor": null,
  "total": null,
  "indexedFromBlock": 127800000,
  "indexedToBlock": 128208994
}
```

Filtered by address:

```bash
curl "https://fsnex.net/api/v1/transactions?address=0xc2c007352c00fefa1a4752a28658705168abd873&pageSize=2"
```

Response `200`

```json
{
  "transactions": [
    {
      "fee": "3328935851064514",
      "blockNumber": 14613670,
      "transactionIndex": 2,
      "hash": "0x1d5a3573fef26f15a3a035718e3b21e10c688a0035277086abd52471fdbbea05",
      "timestamp": 1753101293,
      "from": "0xc2c007352c00fefa1a4752a28658705168abd873",
      "to": "0xc1e9125dca7fc864af4bd731526a5b8c298571ed",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": null,
      "toTags": [],
      "value": "0",
      "gasUsed": 237746,
      "effectiveGasPrice": "14002068809",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": null,
      "direction": "Out"
    },
    {
      "fee": "2649320000000000",
      "blockNumber": 14613669,
      "transactionIndex": 2,
      "hash": "0x57f418ff092f7014ea880b8f1182868195c198eafa5f9563cf7e3c3469322f74",
      "timestamp": 1753101280,
      "from": "0xc2c007352c00fefa1a4752a28658705168abd873",
      "to": "0xc1e9125dca7fc864af4bd731526a5b8c298571ed",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": null,
      "toTags": [],
      "value": "0",
      "gasUsed": 264932,
      "effectiveGasPrice": "10000000000",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": null,
      "direction": "Out"
    }
  ],
  "nextCursor": "14613669-2",
  "previousCursor": null,
  "total": null,
  "indexedFromBlock": 0,
  "indexedToBlock": 15130080
}
```

Filtered by Fusion operation:

```bash
curl "https://fsnex.net/api/v1/transactions?operation=swap-take&pageSize=2"
```

Response `200`

```json
{
  "transactions": [
    {
      "fee": "216080000000000",
      "blockNumber": 13630056,
      "transactionIndex": 1,
      "hash": "0xa6705b3154472bce55caac7d3033657b9d7248668312761755f7da061adbbaeb",
      "timestamp": 1740254022,
      "from": "0xfb5801158426bb143a5ce43dba0eae4eae2f1fc9",
      "to": "0xffffffffffffffffffffffffffffffffffffffff",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": "Fusion Native Calls",
      "toTags": [],
      "value": "0",
      "gasUsed": 21608,
      "effectiveGasPrice": "10000000000",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": "Fusion swap take",
      "direction": null
    },
    {
      "fee": "216080000000000",
      "blockNumber": 13630022,
      "transactionIndex": 1,
      "hash": "0xeb4f757a82eab837f41dcd3b658fd0bef4792fca71050a583f5cbc2f4afe5269",
      "timestamp": 1740253578,
      "from": "0xfb5801158426bb143a5ce43dba0eae4eae2f1fc9",
      "to": "0xffffffffffffffffffffffffffffffffffffffff",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": "Fusion Native Calls",
      "toTags": [],
      "value": "0",
      "gasUsed": 21608,
      "effectiveGasPrice": "10000000000",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": "Fusion swap take",
      "direction": null
    }
  ],
  "nextCursor": "13630022-1",
  "previousCursor": null,
  "total": null,
  "indexedFromBlock": 0,
  "indexedToBlock": 15130080
}
```

With a total count:

```bash
curl "https://fsnex.net/api/v1/transactions?pageSize=2&count=true"
```

Response `200`

```json
{
  "transactions": [
    {
      "fee": "132318515417984",
      "blockNumber": 15130080,
      "transactionIndex": 0,
      "hash": "0xd85a2c32d72c0ece382abc181574a666b6c75e99a5cb19861633e24f33bd706d",
      "timestamp": 1759826630,
      "from": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
      "to": "0xffffffffffffffffffffffffffffffffffffffff",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": "Fusion Native Calls",
      "toTags": [],
      "value": "0",
      "gasUsed": 21224,
      "effectiveGasPrice": "6234381616",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": "Fusion ticket purchase",
      "direction": null
    },
    {
      "fee": "132318515417984",
      "blockNumber": 15130079,
      "transactionIndex": 0,
      "hash": "0x1b4630a0905ae8e85e6f347db59532bb9f88a5f1d9dafe90a6efa7a02b823ef8",
      "timestamp": 1759826617,
      "from": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
      "to": "0xffffffffffffffffffffffffffffffffffffffff",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": "Fusion Native Calls",
      "toTags": [],
      "value": "0",
      "gasUsed": 21224,
      "effectiveGasPrice": "6234381616",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": "Fusion ticket purchase",
      "direction": null
    }
  ],
  "nextCursor": "15130079-0",
  "previousCursor": null,
  "total": {
    "count": 100000,
    "capped": true
  },
  "indexedFromBlock": 0,
  "indexedToBlock": 15130080
}
```

Starting from a date:

```bash
curl "https://fsnex.net/api/v1/transactions?pageSize=2&from=2024-01-01"
```

Response `200`

```json
{
  "transactions": [
    {
      "fee": "3667270000000000",
      "blockNumber": 10862511,
      "transactionIndex": 9,
      "hash": "0xee8ca5617333b71f3bbe79b61f2eb0cd12fc6be66bc2d07453d0e7acfbf7a3fe",
      "timestamp": 1704153597,
      "from": "0x4cf46a2bd39133c8ca1e41bd4548eb3a07000000",
      "to": "0xc6df7b5e384db0b45b18935c60c6ebc293950311",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": null,
      "toTags": [],
      "value": "0",
      "gasUsed": 366727,
      "effectiveGasPrice": "10000000000",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": null,
      "direction": null
    },
    {
      "fee": "22732040000000000",
      "blockNumber": 10862511,
      "transactionIndex": 8,
      "hash": "0x33682e1c6edd3e155d3a824db86ec8bfc994b3201cb892eb01a56d8e272d59b8",
      "timestamp": 1704153597,
      "from": "0x030eef5e64d71ca3022dd0f37443a7b5e8222222",
      "to": "0xc6df7b5e384db0b45b18935c60c6ebc293950311",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": null,
      "toTags": [],
      "value": "0",
      "gasUsed": 2273204,
      "effectiveGasPrice": "10000000000",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": null,
      "direction": null
    }
  ],
  "nextCursor": "10862511-8",
  "previousCursor": "10862511-9-back",
  "total": null,
  "indexedFromBlock": 0,
  "indexedToBlock": 15130080
}
```

The next page, using the `nextCursor` of the first page:

```bash
curl "https://fsnex.net/api/v1/transactions?pageSize=2&cursor=15130079-0"
```

Response `200`

```json
{
  "transactions": [
    {
      "fee": "132318515417984",
      "blockNumber": 15130078,
      "transactionIndex": 0,
      "hash": "0x6f087f69fb2cde7e9345ceaad17bd3b4f7994656f318fcfc87e80e7a3753150b",
      "timestamp": 1759826604,
      "from": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
      "to": "0xffffffffffffffffffffffffffffffffffffffff",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": "Fusion Native Calls",
      "toTags": [],
      "value": "0",
      "gasUsed": 21224,
      "effectiveGasPrice": "6234381616",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": "Fusion ticket purchase",
      "direction": null
    },
    {
      "fee": "132318515417984",
      "blockNumber": 15130077,
      "transactionIndex": 0,
      "hash": "0x667a0340c3f4305c8237de0b1c638c57e175d5e051c500bd7f1a270342310cff",
      "timestamp": 1759826591,
      "from": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
      "to": "0xffffffffffffffffffffffffffffffffffffffff",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": "Fusion Native Calls",
      "toTags": [],
      "value": "0",
      "gasUsed": 21224,
      "effectiveGasPrice": "6234381616",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": "Fusion ticket purchase",
      "direction": null
    }
  ],
  "nextCursor": "15130077-0",
  "previousCursor": "15130078-0-back",
  "total": null,
  "indexedFromBlock": 0,
  "indexedToBlock": 15130080
}
```

## Get a transaction

`GET /v1/transactions/{hash}`

One transaction with everything the explorer shows for it:

- the transaction's own fields: sender, recipient, value, gas, fee, nonce, type, input and the decoded method where the contract is verified, with the sender's and recipient's labels and tags in `fromLabel`, `fromTags`, `toLabel` and `toTags`, and those of the contract a contract creation made in `createdContractLabel` and `createdContractTags`
- `feeBreakdown`, which separates the gas fee from any chain-specific fee
- `tokenTransfers`: the token and NFT transfers the transaction made, each with `fromLabel`, `fromTags`, `toLabel` and `toTags` for its sender and recipient
- `logs`: the event logs, decoded into `eventSignature` and `arguments` where the event is known, each with its emitter's `addressLabel` and `addressTags`, the tags [Get an address](addresses.md#get-an-address) lists
- `actions`: what the transaction did in plain terms, such as the swaps below, each with the tokens given and received, and `accountLabel`, `accountTags`, `recipientLabel` and `recipientTags` for the account it names and its recipient
- `operations`: Fusion native operations, on FSNEX, each field with its label, value and `format`; a field whose `format` is `address` carries that address's label and tags in `addressLabel` and `addressTags`, and every other field a `null` label and no tags
- `internalTransactions`: the call frames the transaction made, when the deployment traces them, each frame with `fromLabel`, `fromTags`, `toLabel` and `toTags`. `isTraced` is `false` when the transaction's block lies outside `coveredFrom`–`coveredThrough`, and the field is `null` on deployments without tracing.

Tags are those [Get an address](addresses.md#get-an-address) lists, in the same order and shape; an untagged address has an empty list, and an address with no label has a `null` label.

| Parameter | Description |
| --- | --- |
| `hash` (path) | A 32-byte transaction hash. |

Returns `400` for a malformed hash and `404` when no indexed transaction matches.

```bash
curl "https://fsnex.net/api/v1/transactions/0x1d5a3573fef26f15a3a035718e3b21e10c688a0035277086abd52471fdbbea05"
```

Response `200` (trimmed: `actions` shows 2 of 3; `input` cut from 320 characters; `tokenTransfers` shows 2 of 5; `logs` shows 2 of 11)

```json
{
  "operations": [],
  "actions": [
    {
      "accountLabel": null,
      "accountTags": [],
      "recipientLabel": null,
      "recipientTags": [],
      "logIndex": 16,
      "position": 0,
      "kind": 0,
      "contract": "0x16ee31ac8020ee5421994b3e5afae7fa49f2bf43",
      "account": null,
      "recipient": null,
      "pool": null,
      "crossChainReference": null,
      "given": [
        {
          "token": {
            "address": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
            "isNative": false,
            "name": "Wrapped Fusion",
            "symbol": "WFSN",
            "decimals": 18
          },
          "value": "2162060962723682439"
        }
      ],
      "received": [
        {
          "token": {
            "address": "0xab1f7e5bf2587543fe41f268c59d35da95f046e0",
            "isNative": false,
            "name": "Chainge",
            "symbol": "XCHNG",
            "decimals": 18
          },
          "value": "7193341152092284389"
        }
      ],
      "tokens": [],
      "sliceStart": null,
      "sliceEnd": null,
      "item": null
    },
    {
      "accountLabel": null,
      "accountTags": [],
      "recipientLabel": null,
      "recipientTags": [],
      "logIndex": 19,
      "position": 0,
      "kind": 0,
      "contract": "0x5c76aff0709927b571fcaccf1cec5b21429b84f4",
      "account": null,
      "recipient": null,
      "pool": null,
      "crossChainReference": null,
      "given": [
        {
          "token": {
            "address": "0xab1f7e5bf2587543fe41f268c59d35da95f046e0",
            "isNative": false,
            "name": "Chainge",
            "symbol": "XCHNG",
            "decimals": 18
          },
          "value": "7193341152092284389"
        }
      ],
      "received": [
        {
          "token": {
            "address": "0x9b9c34f603723ed9aec108a51edfc1b60cd2d7cc",
            "isNative": false,
            "name": "TRON",
            "symbol": "TRX",
            "decimals": 6
          },
          "value": "232345"
        }
      ],
      "tokens": [],
      "sliceStart": null,
      "sliceEnd": null,
      "item": null
    }
  ],
  "inputEncoding": null,
  "methodSource": null,
  "feeBreakdown": {
    "gasFee": "3328935851064514",
    "additionalFee": "0",
    "additionalFeeLabel": "Native operation fee",
    "totalFee": "3328935851064514"
  },
  "internalTransactions": {
    "isTraced": false,
    "coveredFrom": 0,
    "coveredThrough": 7901400,
    "error": null,
    "revertReason": null,
    "frames": []
  },
  "fromLabel": null,
  "fromTags": [],
  "toLabel": null,
  "toTags": [],
  "createdContractLabel": null,
  "createdContractTags": [],
  "kind": "ContractInteraction",
  "fee": "3328935851064514",
  "blockNumber": 14613670,
  "transactionIndex": 2,
  "hash": "0x1d5a3573fef26f15a3a035718e3b21e10c688a0035277086abd52471fdbbea05",
  "blockHash": "0x92dfa9abe482dec2c4369eeac3e598a113ee57082f546e6a1f3e5ea1b07003c7",
  "timestamp": 1753101293,
  "from": "0xc2c007352c00fefa1a4752a28658705168abd873",
  "to": "0xc1e9125dca7fc864af4bd731526a5b8c298571ed",
  "createdContract": null,
  "value": "0",
  "gas": 423318,
  "gasUsed": 237746,
  "gasPrice": null,
  "effectiveGasPrice": "14002068809",
  "maxFeePerGas": "14002068809",
  "maxPriorityFeePerGas": "14002068809",
  "nonce": 137488,
  "input": "0xff030000000000000000000c5a482ac95f6e16ee31ac8020ee5421994b3e5afa…",
  "method": null,
  "methodArguments": null,
  "type": 2,
  "chainId": 32659,
  "succeeded": true,
  "logCount": 11,
  "tokenTransfers": [
    {
      "fromLabel": null,
      "fromTags": [],
      "toLabel": null,
      "toTags": [],
      "logIndex": 12,
      "token": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
      "tokenName": "Wrapped Fusion",
      "tokenSymbol": "WFSN",
      "tokenDecimals": 18,
      "standard": "Erc20",
      "from": "0x32a201389d96e98c895f5315b943ca89f8f921ab",
      "to": "0xc1e9125dca7fc864af4bd731526a5b8c298571ed",
      "value": "2172416572650443455",
      "tokenId": null,
      "sliceStart": null,
      "sliceEnd": null,
      "sliceOf": null
    },
    {
      "fromLabel": null,
      "fromTags": [],
      "toLabel": null,
      "toTags": [],
      "logIndex": 13,
      "token": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
      "tokenName": "Wrapped Fusion",
      "tokenSymbol": "WFSN",
      "tokenDecimals": 18,
      "standard": "Erc20",
      "from": "0xc1e9125dca7fc864af4bd731526a5b8c298571ed",
      "to": "0x16ee31ac8020ee5421994b3e5afae7fa49f2bf43",
      "value": "2162060962723682439",
      "tokenId": null,
      "sliceStart": null,
      "sliceEnd": null,
      "sliceOf": null
    }
  ],
  "logs": [
    {
      "removed": false,
      "eventSource": "Catalog",
      "logIndex": 12,
      "address": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
      "addressLabel": null,
      "addressTags": [],
      "topics": [
        "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "0x00000000000000000000000032a201389d96e98c895f5315b943ca89f8f921ab",
        "0x000000000000000000000000c1e9125dca7fc864af4bd731526a5b8c298571ed"
      ],
      "data": "0x0000000000000000000000000000000000000000000000001e25f95ce50052bf",
      "eventSignature": "Transfer(address,address,uint256)",
      "arguments": [
        {
          "name": "from",
          "type": "address",
          "value": "0x32a201389d96e98c895f5315b943ca89f8f921ab"
        },
        {
          "name": "to",
          "type": "address",
          "value": "0xc1e9125dca7fc864af4bd731526a5b8c298571ed"
        },
        {
          "name": "value",
          "type": "uint256",
          "value": "2172416572650443455"
        }
      ]
    },
    {
      "removed": false,
      "eventSource": "Catalog",
      "logIndex": 13,
      "address": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
      "addressLabel": null,
      "addressTags": [],
      "topics": [
        "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "0x000000000000000000000000c1e9125dca7fc864af4bd731526a5b8c298571ed",
        "0x00000000000000000000000016ee31ac8020ee5421994b3e5afae7fa49f2bf43"
      ],
      "data": "0x0000000000000000000000000000000000000000000000001e012efd91e25887",
      "eventSignature": "Transfer(address,address,uint256)",
      "arguments": [
        {
          "name": "from",
          "type": "address",
          "value": "0xc1e9125dca7fc864af4bd731526a5b8c298571ed"
        },
        {
          "name": "to",
          "type": "address",
          "value": "0x16ee31ac8020ee5421994b3e5afae7fa49f2bf43"
        },
        {
          "name": "value",
          "type": "uint256",
          "value": "2162060962723682439"
        }
      ]
    }
  ]
}
```

A plain transfer on BNB Smart Chain:

```bash
curl "https://bnb.blockgaze.com/api/v1/transactions/0xc714b944c97eaccce94814efb588f11cd6f3a9279e319cca0d1bb45d98df4c4e"
```

Response `200`

```json
{
  "operations": [],
  "actions": [],
  "inputEncoding": null,
  "methodSource": null,
  "feeBreakdown": null,
  "internalTransactions": null,
  "fromLabel": null,
  "fromTags": [],
  "toLabel": null,
  "toTags": [],
  "createdContractLabel": null,
  "createdContractTags": [],
  "kind": "Transfer",
  "fee": "0",
  "blockNumber": 128033625,
  "transactionIndex": 63,
  "hash": "0xc714b944c97eaccce94814efb588f11cd6f3a9279e319cca0d1bb45d98df4c4e",
  "blockHash": "0x78494a76d0f1d82bab73f374e83960008397d1e2fbf60eb4b71d2b59ca56e1b9",
  "timestamp": 1792178554,
  "from": "0xc7d4187cbda07f4b5be04418b9723b5faf65db5c",
  "to": "0x5532cdb3c0c4278f9848fc4560b495b70ba67455",
  "createdContract": null,
  "value": "0",
  "gas": 21000,
  "gasUsed": 21000,
  "gasPrice": "0",
  "effectiveGasPrice": "0",
  "maxFeePerGas": null,
  "maxPriorityFeePerGas": null,
  "nonce": 405691,
  "input": "0x",
  "method": null,
  "methodArguments": null,
  "type": 0,
  "chainId": 56,
  "succeeded": true,
  "logCount": 0,
  "tokenTransfers": [],
  "logs": []
}
```
