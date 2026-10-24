# Blocks and transactions

[Back to overview](../README.md)

- [List blocks](#list-blocks)
- [Get a block](#get-a-block)
- [Count down to a block](#count-down-to-a-block)
- [List transactions](#list-transactions)
- [Get a transaction](#get-a-transaction)
- [List internal transactions](#list-internal-transactions)

## List blocks

`GET /v1/blocks`

Blocks, newest first, with producer, gas, transaction count, fees and reward. `totalFees` is what the block's transactions paid in wei, blob transactions' blob fees included. `minerLabel` and `minerTags` give the producer's label and tags. Tags are those [Get an address](addresses.md#get-an-address) lists, in the same order and shape; an untagged address has an empty list.

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

One block by number or by hash, with its header fields, fees, issuance and reward. `totalFees` is what its transactions paid in wei, blob transactions' blob fees included, and `gasFees` what they paid for gas alone, leaving out blob fees and any chain-specific fee. `blobGas` gives the blob gas the block's blob transactions `used`, the header's `excess` blob gas and the `blobCount` they make (blob gas used over 131,072), and is `null` for a block whose header does not report both, as on FSNEX and on BNB Smart Chain before its Cancun fork. `minerLabel` and `minerTags` give the producer's label and tags, as in [List blocks](#list-blocks). `annotations` carries chain-specific notes recorded with the block, such as the Fusion ticket snapshot below.

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
      "description": "The ticket snapshot recorded in this block's header. Its count is for this block, not for the newest block the ticket index has reached.",
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
  "blobGas": null,
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
A BNB Smart Chain block that carries one blob:

```bash
curl "https://bnb.blockgaze.com/api/v1/blocks/129169995"
```

Response `200` (trimmed: `extraData` cut from 562 characters)

```json
{
  "annotations": [],
  "totalFees": "825027494867330",
  "gasFees": "825027494736258",
  "issuance": "0",
  "blockReward": "825027494867330",
  "averageGasPrice": "69689916",
  "number": 129169995,
  "hash": "0x769dcd587ea291e39ceecb8745604269c3c139c441f1a7cc2b127b77ff7b61a8",
  "parentHash": "0xd020072ecabad1a63eefbf589cdbde9ad642de09b1482b2d04e5d2685cee6e15",
  "timestamp": 1792690124,
  "miner": "0x38944092685a336cb6b9ea58836436709a2adc89",
  "minerLabel": "Shannon",
  "minerTags": [
    {
      "title": "Validator",
      "colour": "Accent",
      "description": "Produces blocks and sends each block's system transactions."
    }
  ],
  "gasLimit": 70000000,
  "gasUsed": 11838549,
  "baseFeePerGas": "0",
  "difficulty": "2",
  "totalDifficulty": "257754238",
  "blobGas": {
    "used": 131072,
    "excess": 0,
    "blobCount": 1
  },
  "extraData": "0xda8301070888336434656363636288676f312e32372e31836c696e00d38cc282…",
  "size": 51261,
  "nonce": "0x0000000000000000",
  "mixHash": "0x00000000000000000000000000000000000000000000000000000000000002bc",
  "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  "stateRoot": "0xe1a6b4eb5b71f4609bec1cf3e6dff79e302cf5ed666c5ebb0ccd4fbe037c35b7",
  "transactionsRoot": "0xc16b986b238bc47b5994fc28849a3de75de5b03d72d12e1c8693859670a83313",
  "receiptsRoot": "0x6051ff17772a2193f85b6f72aaeb98c723f4ddc8420615b6ff0d2f9ff443c1d6",
  "transactionCount": 84
}
```


## Count down to a block

`GET /v1/blocks/{number}/countdown`

How far off a block number is, an estimate. The chain's head is estimated from the newest indexed block's age and the mean time per block between it and the oldest of the newest 1,000 indexed blocks, and only while the newest block is fresh by the deployment's sync health (no more than two minutes old by default). `state` says which case applies:

| `state` | Meaning |
| --- | --- |
| `Countdown` | The block is expected: `remainingBlocks` are left, `estimatedTimestamp` (Unix seconds, rounded up) says when, and `remainingTime` how long that is from the answer; the two are `null` for a number too far ahead to date. |
| `Reached` | By the estimate the chain has reached or passed the number, whether or not the block is indexed yet. |
| `Stale` | The newest indexed block is too old to estimate from. |
| `Halted` | The number is past the block where the chain's data ends, so it is not expected (FSNEX). Numbers up to that block are `Reached`. |
| `Unavailable` | Too few blocks are indexed to measure a block time. |

`averageBlockTime` is the time per block used, as a [duration](../README.md#conventions), or `null` where none could be measured or none is needed. A number that is not a whole, non-negative block number answers `400`. Count down from `remainingTime` rather than from `estimatedTimestamp` if your clock may be off. `remainingTime` is measured when the answer is made, and answers are cached for about 2 seconds, so a cached answer's `remainingTime` can be up to 2 seconds too long.

```bash
curl "https://bnb.blockgaze.com/api/v1/blocks/130000000/countdown"
```

Response `200`

```json
{
  "targetBlock": 130000000,
  "state": "Countdown",
  "remainingBlocks": 1603325,
  "estimatedTimestamp": 1793064195,
  "remainingTime": "8.08:36:58.8782874",
  "averageBlockTime": "00:00:00.4504505"
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
      "blockNumber": 129445619,
      "transactionIndex": 58,
      "hash": "0xe342735048fdab6295c3187a21f783d78428356fdaf341a6216705468c311e09",
      "timestamp": 1792814184,
      "from": "0x9f1b7fae54be07f4fee34eb1aacb39a1f7b6fc92",
      "to": "0x0000000000000000000000000000000000001000",
      "fromLabel": "TWStaking",
      "fromTags": [
        {
          "title": "Validator",
          "colour": "Accent",
          "description": "Produces blocks and sends each block's system transactions."
        }
      ],
      "toLabel": "Validator Set",
      "toTags": [],
      "value": "1144118968376969",
      "gasUsed": 55345,
      "effectiveGasPrice": "0",
      "succeeded": true,
      "kind": "ContractInteraction",
      "actionName": null,
      "direction": null
    },
    {
      "fee": "0",
      "blockNumber": 129445619,
      "transactionIndex": 57,
      "hash": "0xd95bb3e1580eb1703b302e4972fa90f5c750d8efb774d7ddd6cd5f34c868649e",
      "timestamp": 1792814184,
      "from": "0xc7d4187cbda07f4b5be04418b9723b5faf65db5c",
      "to": "0x539e24781f616f0d912b60813ab75b7b80b75c53",
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
  "nextCursor": "129445619-57",
  "previousCursor": null,
  "total": null,
  "indexedFromBlock": 129040000,
  "indexedToBlock": 129445619
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
- `inputText`: the input read as UTF-8 text, for a message sent in it, or `null`. It is sent only when nothing decodes the input as a call (`method` is `null` and so is `inputEncoding`), the transaction creates no contract and its bytes are valid UTF-8, contain at least one visible character, and contain no control character other than a tab, a carriage return or a line feed, so a call with arguments, whose ABI encoding almost always holds zero bytes, is practically never read as text, though an unknown selector sent alone occasionally is. Every line break is sent as a line feed, tabs are kept, and any other invisible character, such as a zero-width space or a right-to-left override, is written as a `\uXXXX` escape
- `feeBreakdown`, which separates the gas fee from any chain-specific fee
- `blobs`: for a blob transaction (type 3), its blobs' `versionedHashes`, its `maxFeePerBlobGas` in wei, and from its receipt the `blobGasUsed`, the `blobGasPrice` in wei and the `blobFee` in wei, their product, paid beside `fee` rather than in it; the receipt's three are `null` when the receipt is not indexed yet or did not report them, and `blobs` is `null` for every other transaction. The blobs themselves are not kept.
- `tokenTransfers`: the token and NFT transfers the transaction made, each with `fromLabel`, `fromTags`, `toLabel` and `toTags` for its sender and recipient
- `logs`: the event logs, decoded into `eventSignature` and `arguments` where the event is known, each with its emitter's `addressLabel` and `addressTags`, the tags [Get an address](addresses.md#get-an-address) lists
- `actions`: what the transaction did in plain terms, such as the swaps below, each with the tokens given and received, and `accountLabel`, `accountTags`, `recipientLabel` and `recipientTags` for the account it names and its recipient
- `operations`: Fusion native operations, on FSNEX, each field with its label, value and `format`; a field whose `format` is `address` carries that address's label and tags in `addressLabel` and `addressTags`, and every other field a `null` label and no tags; an `amount` field's `unit` gives the asset's symbol and decimals, and every other field's is `null`
- `operations` entries and where they come from: a native call usually appears twice, as the parameters read from its input, whose `source` is `Transaction input`, and as the operation its receipt log recorded, whose `source` is `Receipt log N`, `N` being that log's `logIndex`; `status` says whether the input decoded and, for a log, whether the chain recorded the operation. A call that writes no log appears once, and from block 8,250,000 a transaction, a native call or not, can also carry a `Fusion time-lock maturity conversion` that efsn wrote, which has no input counterpart.
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
  "inputText": null,
  "methodSource": null,
  "feeBreakdown": {
    "gasFee": "3328935851064514",
    "additionalFee": "0",
    "additionalFeeLabel": "Native operation fee",
    "totalFee": "3328935851064514"
  },
  "blobs": null,
  "internalTransactions": {
    "isTraced": true,
    "coveredFrom": 0,
    "coveredThrough": 15130080,
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

A blob transaction on BNB Smart Chain, which carries one blob:

```bash
curl "https://bnb.blockgaze.com/api/v1/transactions/0xb79c7d563ec8e2ac55acc4709540b93684215178ca1056f69d32abfd2698b353"
```

Response `200`

```json
{
  "operations": [],
  "actions": [],
  "inputEncoding": null,
  "inputText": null,
  "methodSource": null,
  "feeBreakdown": null,
  "blobs": {
    "versionedHashes": [
      "0x01b3b2b7c6ff1a4cabd7e536301ea1ccef5c763851d5191e1bd7a98397e4ad4a"
    ],
    "maxFeePerBlobGas": "2",
    "blobGasUsed": 131072,
    "blobGasPrice": "1",
    "blobFee": "131072"
  },
  "internalTransactions": null,
  "fromLabel": null,
  "fromTags": [],
  "toLabel": null,
  "toTags": [],
  "createdContractLabel": null,
  "createdContractTags": [],
  "kind": "Transfer",
  "fee": "4200000000000",
  "blockNumber": 129169995,
  "transactionIndex": 32,
  "hash": "0xb79c7d563ec8e2ac55acc4709540b93684215178ca1056f69d32abfd2698b353",
  "blockHash": "0x769dcd587ea291e39ceecb8745604269c3c139c441f1a7cc2b127b77ff7b61a8",
  "timestamp": 1792690124,
  "from": "0xef8783382ef80ec23b66c43575a6103deca909c3",
  "to": "0xff00000000000000000000000000000000000204",
  "createdContract": null,
  "value": "0",
  "gas": 21000,
  "gasUsed": 21000,
  "gasPrice": null,
  "effectiveGasPrice": "200000000",
  "maxFeePerGas": "200000000",
  "maxPriorityFeePerGas": "200000000",
  "nonce": 16570384,
  "input": "0x",
  "method": null,
  "methodArguments": null,
  "type": 3,
  "chainId": 56,
  "succeeded": true,
  "logCount": 0,
  "tokenTransfers": [],
  "logs": []
}
```

A Fusion native call, a send of the DAV asset to an address labelled Bittrex:

```bash
curl "https://fsnex.net/api/v1/transactions/0x82c9c71b35439e3c5b4502eab3ee6b6dec9bdd60c979795da636b2cf592737d2"
```

Response `200` (trimmed: `logs.data` cut from 326 characters)

```json
{
  "operations": [
    {
      "name": "Fusion asset transfer",
      "source": "Transaction input",
      "status": "Attempted parameters",
      "fields": [
        {
          "addressLabel": null,
          "addressTags": [],
          "label": "Asset ID",
          "value": "0x46992f33b0cce4fbee9af575cb4de6c5cb748b535670477f834780db851ed0ac",
          "format": "asset",
          "unit": null
        },
        {
          "addressLabel": "Bittrex",
          "addressTags": [],
          "label": "Recipient",
          "value": "0x9996c35cdb692de8a76397e83b9a30982f1b0692",
          "format": "address",
          "unit": null
        },
        {
          "addressLabel": null,
          "addressTags": [],
          "label": "Amount",
          "value": "22482752736800000000000",
          "format": "amount",
          "unit": {
            "symbol": "DAV",
            "decimals": 18
          }
        }
      ]
    },
    {
      "name": "Fusion asset transfer",
      "source": "Receipt log 11",
      "status": "Recorded native operation",
      "fields": [
        {
          "addressLabel": null,
          "addressTags": [],
          "label": "Asset ID",
          "value": "0x46992f33b0cce4fbee9af575cb4de6c5cb748b535670477f834780db851ed0ac",
          "format": "asset",
          "unit": null
        },
        {
          "addressLabel": "Bittrex",
          "addressTags": [],
          "label": "Recipient",
          "value": "0x9996c35cdb692de8a76397e83b9a30982f1b0692",
          "format": "address",
          "unit": null
        },
        {
          "addressLabel": null,
          "addressTags": [],
          "label": "Amount",
          "value": "22482752736800000000000",
          "format": "amount",
          "unit": {
            "symbol": "DAV",
            "decimals": 18
          }
        }
      ]
    }
  ],
  "actions": [],
  "inputEncoding": "Fusion native RLP",
  "inputText": null,
  "methodSource": null,
  "feeBreakdown": {
    "gasFee": "25832000000000",
    "additionalFee": "0",
    "additionalFeeLabel": "Native operation fee",
    "totalFee": "25832000000000"
  },
  "blobs": null,
  "internalTransactions": {
    "isTraced": true,
    "coveredFrom": 0,
    "coveredThrough": 15130080,
    "error": null,
    "revertReason": null,
    "frames": []
  },
  "fromLabel": null,
  "fromTags": [],
  "toLabel": "Fusion Native Calls",
  "toTags": [],
  "createdContractLabel": null,
  "createdContractTags": [],
  "kind": "ContractInteraction",
  "fee": "25832000000000",
  "blockNumber": 3013245,
  "transactionIndex": 5,
  "hash": "0x82c9c71b35439e3c5b4502eab3ee6b6dec9bdd60c979795da636b2cf592737d2",
  "blockHash": "0x05d446e1617ee29545a7086ae07d259788abee3c88b4cfd9cfbffbf3eb0d5700",
  "timestamp": 1601364788,
  "from": "0x3fe42260e3d4b0c29d2da2a7cdc271a6f5d9ef72",
  "to": "0xffffffffffffffffffffffffffffffffffffffff",
  "createdContract": null,
  "value": "0",
  "gas": 90000,
  "gasUsed": 25832,
  "gasPrice": "1000000000",
  "effectiveGasPrice": "1000000000",
  "maxFeePerGas": null,
  "maxPriorityFeePerGas": null,
  "nonce": 116,
  "input": "0xf84602b843f841a046992f33b0cce4fbee9af575cb4de6c5cb748b535670477f834780db851ed0ac949996c35cdb692de8a76397e83b9a30982f1b06928a04c2cadf2f63f1654000",
  "method": "Fusion asset transfer",
  "methodArguments": null,
  "type": 0,
  "chainId": null,
  "succeeded": true,
  "logCount": 1,
  "tokenTransfers": [],
  "logs": [
    {
      "removed": false,
      "eventSource": null,
      "logIndex": 11,
      "address": "0xffffffffffffffffffffffffffffffffffffffff",
      "addressLabel": "Fusion Native Calls",
      "addressTags": [],
      "topics": [
        "0x0000000000000000000000000000000000000000000000000000000000000002"
      ],
      "data": "0x7b2241737365744944223a223078343639393266333362306363653466626565…",
      "eventSignature": null,
      "arguments": null
    }
  ]
}
```

## List internal transactions

`GET /v1/internal-transactions`

The call frames inside the chain's transactions that carry native coin or create or destroy a contract, newest first. Each item has the shape of an [address's internal transactions](addresses.md#internal-transactions) without `direction`, since the list names no address: `frame.movedValue` says whether the frame actually moved value, and the `frame` labels and tags name each side.

Only blocks from `coveredFrom` to `coveredThrough` have been traced, the same range [`/v1/network`](network.md#network-status) gives as `tracedFromBlock` and `tracedToBlock`; where nothing is traced, as on BNB Smart Chain, both are `null`, and the list is empty whenever `coveredFrom` is `null`.

A page holds as many items as asked for unless the oldest traced frame is reached, when `nextCursor` is `null`; the newest page's `previousCursor` is `null`. While a deployment is still building the index this list reads through, once after the upgrade that added it, each page reads a bounded number of the chain's frames instead, so a page can hold fewer items than asked for, or none, and still give a cursor: frames the chosen kind leaves out were passed over, and the next page goes on from there.

`fromBlock` and `toBlock` give the blocks a page covers, in place of record numbers, both `null` only when the page read no frame. The list has no `count` and no jump, and a `previousCursor` that reaches the newest frames returns the first page.

| Parameter | Default | Description |
| --- | --- | --- |
| `kind` | `all` | `all` for every frame that carries a value or creates or destroys a contract, or `transfers` for frames that carry value. |
| `pageSize` | `25` | Frames per page, at most 100. |
| `cursor` | | The `nextCursor` or `previousCursor` of the page you have. |
| `last` | `false` | `true` returns the oldest page. |

```bash
curl "https://fsnex.net/api/v1/internal-transactions?pageSize=1"
```

Response `200`

```json
{
  "transactions": [
    {
      "blockNumber": 11376850,
      "transactionIndex": 2,
      "transactionHash": "0xf5b5516c3d1de041fb949ab96316612162c96da93ad72e80227b58cf6dfa9782",
      "timestamp": 1710860423,
      "frame": {
        "notes": [],
        "fromLabel": null,
        "fromTags": [],
        "toLabel": null,
        "toTags": [],
        "traceAddress": "1_1",
        "type": "call",
        "from": "0x62bc5327868a998e1b3d9fe6037273e642a137ae",
        "to": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
        "value": "999545651774693173696",
        "gas": 417706,
        "error": null,
        "revertReason": null,
        "undone": false,
        "movedValue": true
      }
    }
  ],
  "nextCursor": "11376850-2-0000000100000001",
  "previousCursor": null,
  "fromBlock": 11376850,
  "toBlock": 11376850,
  "coveredFrom": 0,
  "coveredThrough": 11376863
}
```
