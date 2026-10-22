# Contracts

[Back to overview](../README.md)

- [Contract bytecode](#contract-bytecode)
- [Opcodes](#opcodes)
- [Verified functions](#verified-functions)
- [Read a contract function](#read-a-contract-function)
- [Verified source](#verified-source)
- [Verified contracts](#verified-contracts)
- [Compare verified sources](#compare-verified-sources)
- [Decode calldata](#decode-calldata)
- [Decode an event log](#decode-an-event-log)
- [Compiler versions](#compiler-versions)

Verified source comes from contracts verified on the explorer and, on BNB Smart Chain, from verifications imported from Sourcify. The same source is available in Etherscan's format through [`module=contract`](etherscan-compatible.md#contract-abi-and-source).

Every endpoint on this page returns `400` when `address` is not a 20-byte address.

## Contract bytecode

`GET /v1/contracts/{address}/code`

The contract's deployed bytecode as the indexer read it, with its hash, size and the block it was read at, and `creationInput`, the input of the transaction that created it. `sharing` lists other addresses with identical code, and `delegators` lists EIP-7702 accounts that delegate to this contract, each with a capped count.

Returns `404` when no code reading of the address is stored.

```bash
curl "https://fsnex.net/api/v1/contracts/0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe/code"
```

Response `200` (trimmed: `code` cut from 3966 characters; `creationInput` cut from 4932 characters)

```json
{
  "code": "0x6080604052600436106100a05760003560e01c8063313ce56711610064578063…",
  "codeHash": "0xaf50f63bd6c6d0e0c01338af13598a6d6cc91b7114838bf0337921ee88260f73",
  "sizeBytes": 1982,
  "asOfBlock": 15130080,
  "creationInput": "0x60c0604052600e60809081526d2bb930b83832b210233ab9b4b7b760911b60a0…",
  "sharing": {
    "total": {
      "count": 0,
      "capped": false
    },
    "addresses": []
  },
  "delegators": {
    "total": {
      "count": 0,
      "capped": false
    },
    "addresses": []
  }
}
```

## Opcodes

`GET /v1/contracts/{address}/code/opcodes`

The deployed bytecode disassembled into opcodes, as an array of strings.

```bash
curl "https://fsnex.net/api/v1/contracts/0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe/code/opcodes"
```

Response `200` (trimmed: the array shows 2 of 1320)

```json
[
  "PUSH1 0x80",
  "PUSH1 0x40"
]
```

## Verified functions

`GET /v1/contracts/{address}/functions`

The functions in a verified contract's ABI, each with its selector, signature, mutability, inputs and outputs. `isReadOnly` marks `view` and `pure` functions, which [Read a contract function](#read-a-contract-function) can call.

Returns `404` with the title `Not verified` when the contract has no verified source.

```bash
curl "https://bnb.blockgaze.com/api/v1/contracts/0xca11bde05977b3631167028862be2a173976ca11/functions"
```

Response `200` (trimmed: `functions` shows 2 of 16)

```json
{
  "address": "0xca11bde05977b3631167028862be2a173976ca11",
  "functions": [
    {
      "selector": "0x252dba42",
      "name": "aggregate",
      "signature": "aggregate((address,bytes)[])",
      "mutability": "payable",
      "inputs": [
        {
          "name": "calls",
          "type": "(address,bytes)[]",
          "label": "calls ((address,bytes)[])",
          "components": [
            {
              "name": "target",
              "type": "address",
              "label": "target (address)",
              "components": []
            },
            {
              "name": "callData",
              "type": "bytes",
              "label": "callData (bytes)",
              "components": []
            }
          ]
        }
      ],
      "outputs": [
        {
          "name": "blockNumber",
          "type": "uint256",
          "label": "blockNumber (uint256)",
          "components": []
        },
        {
          "name": "returnData",
          "type": "bytes[]",
          "label": "returnData (bytes[])",
          "components": []
        }
      ],
      "isReadOnly": false,
      "isPayable": true,
      "isCallable": true
    },
    {
      "selector": "0x82ad56cb",
      "name": "aggregate3",
      "signature": "aggregate3((address,bool,bytes)[])",
      "mutability": "payable",
      "inputs": [
        {
          "name": "calls",
          "type": "(address,bool,bytes)[]",
          "label": "calls ((address,bool,bytes)[])",
          "components": [
            {
              "name": "target",
              "type": "address",
              "label": "target (address)",
              "components": []
            },
            {
              "name": "allowFailure",
              "type": "bool",
              "label": "allowFailure (bool)",
              "components": []
            },
            {
              "name": "callData",
              "type": "bytes",
              "label": "callData (bytes)",
              "components": []
            }
          ]
        }
      ],
      "outputs": [
        {
          "name": "returnData",
          "type": "(bool,bytes)[]",
          "label": "returnData ((bool,bytes)[])",
          "components": [
            {
              "name": "success",
              "type": "bool",
              "label": "success (bool)",
              "components": []
            },
            {
              "name": "returnData",
              "type": "bytes",
              "label": "returnData (bytes)",
              "components": []
            }
          ]
        }
      ],
      "isReadOnly": false,
      "isPayable": true,
      "isCallable": true
    }
  ],
  "implementation": null,
  "implementationFunctions": null
}
```

## Read a contract function

`POST /v1/contracts/{address}/read`

Calls a read-only function of a verified contract at the latest block and returns the decoded outputs. The call runs against the deployment's node, so it has its own [rate limit](../README.md#rate-limits): 30 reads per minute per client, and 300 per minute for the whole deployment.

The body is JSON:

| Field | Description |
| --- | --- |
| `selector` | The function's four-byte selector as `0x` hex, from [Verified functions](#verified-functions). |
| `arguments` | The arguments in order, each as a string. Leave out or send `[]` for none. |
| `asProxy` | Optional. `true` calls the function from a proxy's verified implementation, at the proxy's address. |

`status` is `Read` when the call succeeded, and `problems` explains anything that stopped it. Returns `400` when the address or selector is malformed.

```bash
curl -X POST "https://bnb.blockgaze.com/api/v1/contracts/0xca11bde05977b3631167028862be2a173976ca11/read" \
  -H "Content-Type: application/json" \
  -d '{"selector":"0x4d2301cc","arguments":["0xbb4cdb9cbd36b01bd1cbaebf2de08d9173bc095c"]}'
```

Response `200`

```json
{
  "status": "Read",
  "problems": [],
  "outputs": [
    {
      "name": "balance",
      "type": "uint256",
      "value": "1553423433427128480030045"
    }
  ]
}
```

## Verified source

`GET /v1/verification/{address}`

A verified contract's source files, ABI, compiler version and settings, constructor arguments, and when it was verified. `language`, `match`, `matchedCode` and `constructorArgumentsAbsence` are numeric codes.

Returns `404` with the title `Not verified` when the contract has no verified source.

```bash
curl "https://bnb.blockgaze.com/api/v1/verification/0xca11bde05977b3631167028862be2a173976ca11"
```

Response `200` (trimmed: `sources.content` cut from 9610 characters; `abi` cut from 5772 characters)

```json
{
  "address": "0xca11bde05977b3631167028862be2a173976ca11",
  "language": 0,
  "compilerVersion": "0.8.12+commit.f00d7308",
  "contractPath": "contracts/Multicall3.sol",
  "contractName": "Multicall3",
  "match": 1,
  "matchedCode": 2,
  "verifiedAt": 1791720374,
  "optimizerEnabled": true,
  "optimizerRuns": 10000000,
  "vyperOptimize": null,
  "evmVersion": null,
  "libraries": [],
  "sources": [
    {
      "name": "contracts/Multicall3.sol",
      "content": "// SPDX-License-Identifier: MIT\npragma solidity 0.8.12;\n\n/// @titl…",
      "invisibleCharacterPositions": [],
      "invisibleCharacterLines": []
    }
  ],
  "abi": "[{\"name\": \"aggregate\", \"type\": \"function\", \"inputs\": [{\"name\": \"ca…",
  "constructorArguments": null,
  "constructorArgumentsAbsence": 2,
  "decodedConstructorArguments": null
}
```

## Verified contracts

`GET /v1/verification/contracts`

Every verified contract, most recently verified first: its `address`, `contractName`, `language`, `compilerVersion`, `match` and `verifiedAt`, the codes as in [Verified source](#verified-source): `language` 0 for Solidity and 1 for Vyper, `match` 1 for a partial match and 2 for a full one. `contractName` is the name the submitter's source gave it, with invisible and direction-changing characters sent as escapes such as `\u202E`.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Contracts per page, at most 100. |
| `cursor`, `last`, `count`, `page` | | See [pagination](../README.md#pagination). This list is not in block order, so it does not accept `from`, and `page` reaches only the first 100,000 contracts. |

```bash
curl "https://fsnex.net/api/v1/verification/contracts?pageSize=2"
```

Response `200`

```json
{
  "contracts": [
    {
      "address": "0xa6b71e26c5e0845f74c812102ca7114b6a896ab2",
      "contractName": "GnosisSafeProxyFactory",
      "language": 0,
      "compilerVersion": "0.7.6+commit.7338295f",
      "match": 2,
      "verifiedAt": 1791725741
    },
    {
      "address": "0x445166c4854836292a5af7e3f165a3b8b4eedf97",
      "contractName": "BtcSwapAssetV2",
      "language": 0,
      "compilerVersion": "0.5.4+commit.9549d8ff",
      "match": 1,
      "verifiedAt": 1791716552
    }
  ],
  "nextCursor": "1791716552-0x445166c4854836292a5af7e3f165a3b8b4eedf97",
  "previousCursor": null,
  "total": null
}
```

## Compare verified sources

`GET /v1/verification/comparison`

Compares the verified sources of two contracts line by line. `maximumLineEdits` is the most removed and added lines a file pair is compared through, 2,000. `left` and `right` each give the contract's `address`, `contractName` and `compilerVersion`. `files` lists every source file of either: the left contract's in the order its input lists them, each beside the right file it pairs with, then the right contract's files that pair with none. Files pair when they have the same path, then when their file name is the only one of its kind on either side, and last when a single file is left on each side.

Each file gives its `leftName` and `rightName` (`null` on the side that lacks it), its line counts `leftLineCount` and `rightLineCount`, its `change` and its `hunks`. `change` is a numeric code: `1` unchanged, `2` changed, `3` changed in more than `maximumLineEdits` removed and added lines, which are not listed, `4` only in the left contract and `5` only in the right. Line endings never count as a change. `hunks` is empty unless `change` is `2`; each hunk's `lines` are a run of changed lines with up to three unchanged lines either side, each line giving its `change` (`1` unchanged, `2` only in the left file, `3` only in the right), its `leftNumber` and `rightNumber` from 1 (`null` on the side that lacks the line) and its `text` without its line ending. Paths, names and line texts carry invisible and direction-changing characters as escapes such as `\u202E`, a tab in line text aside.

| Parameter | Default | Description |
| --- | --- | --- |
| `left` | | Required. The verified contract whose source the comparison starts from, a 20-byte address. |
| `right` | | Required. The verified contract whose source it is compared with, a 20-byte address. |

Returns `400` with the title `Invalid address` when either is not a 20-byte address, and `404` with the title `Not verified` when either has no verified source. Both problems name the parameter they are with in a `parameter` member, `left` or `right`. Both are checked for form before either is looked up, `left` first each time, so only the first that fails is named: a malformed `right` is reported before an unverified `left`.

Two verified contracts of the same name, whose only source files pair as the last file left on each side:

```bash
curl "https://bnb.blockgaze.com/api/v1/verification/comparison?left=0x5313e938b46fc20f47f8fb998fd08932345a860e&right=0x866c317f6ae76540ad00053b171117d811d542f0"
```

Response `200`

```json
{
  "left": {
    "address": "0x5313e938b46fc20f47f8fb998fd08932345a860e",
    "contractName": "MevSandwichExecutor",
    "compilerVersion": "0.8.34+commit.80d5c536"
  },
  "right": {
    "address": "0x866c317f6ae76540ad00053b171117d811d542f0",
    "contractName": "MevSandwichExecutor",
    "compilerVersion": "0.8.34+commit.80d5c536"
  },
  "maximumLineEdits": 2000,
  "files": [
    {
      "leftName": "contracts/sandwich_ait3.sol",
      "rightName": "contracts/sandwich_ait2.sol",
      "change": 2,
      "leftLineCount": 129,
      "rightLineCount": 129,
      "hunks": [
        {
          "lines": [
            {
              "change": 1,
              "leftNumber": 7,
              "rightNumber": 7,
              "text": "contract MevSandwichExecutor {"
            },
            {
              "change": 1,
              "leftNumber": 8,
              "rightNumber": 8,
              "text": "    address public immutable owner;"
            },
            {
              "change": 1,
              "leftNumber": 9,
              "rightNumber": 9,
              "text": "    "
            },
            {
              "change": 2,
              "leftNumber": 10,
              "rightNumber": null,
              "text": "    ISwapRouter public constant ROUTER = ISwapRouter(0x7139dD70593b6C08138CF35D739B2D64cF55C770);"
            },
            {
              "change": 3,
              "leftNumber": null,
              "rightNumber": 10,
              "text": "    ISwapRouter public constant ROUTER = ISwapRouter(0xb300000b72DEAEb607a12d5f54773D1C19c7028d);"
            },
            {
              "change": 1,
              "leftNumber": 11,
              "rightNumber": 11,
              "text": ""
            },
            {
              "change": 1,
              "leftNumber": 12,
              "rightNumber": 12,
              "text": "    modifier onlyOwner() {"
            },
            {
              "change": 1,
              "leftNumber": 13,
              "rightNumber": 13,
              "text": "        require(msg.sender == owner, \"Not Owner: Only bot can execute\");"
            }
          ]
        }
      ]
    }
  ]
}
```

## Decode calldata

`POST /v1/decoder/calldata`

Decodes a call's input data: against the verified ABI of `address` when it is given, then, for a verified proxy, the verified ABI of the implementation it runs now, then the explorer's signature catalog. The answer gives the `selector`, the `method` it names, the `source` of that declaration (`VerifiedAbi`, `ImplementationAbi` or `Catalog`) and the decoded `arguments`, each with its `name`, `type` and `value` or, for a tuple or array, its `items`. `method` and `source` are `null` when nothing declares the selector; `arguments` is `null` then too, and when the data does not fit the declaration.

| Body field | Description |
| --- | --- |
| `input` | The input data as hex, its `0x` optional: the four-byte selector and at most 128 KiB in all. |
| `address` | Optional. The contract the call was sent to. |

Returns `400` with the title `Invalid input` or `Invalid address` when either is malformed, and `413` for a body past 512 KiB.

The input of a PancakeSwap swap, whose `path` argument is an array:

```bash
curl -X POST "https://bnb.blockgaze.com/api/v1/decoder/calldata" \
  -H "Content-Type: application/json" \
  -d '{"input":"0x18cbafe5000000000000000000000000000000000000000000055ed64b7e1e73284c5282000000000000000000000000000000000000000000000000000000e8d4a5100000000000000000000000000000000000000000000000000000000000000000a000000000000000000000000095ed2e56cd3d45d2e3bfd4f27d58f352f9fd158e000000000000000000000000000000000000000000000000000000006ada58900000000000000000000000000000000000000000000000000000000000000002000000000000000000000000d94a72950ead1b78331d91924a309ea322937777000000000000000000000000bb4cdb9cbd36b01bd1cbaebf2de08d9173bc095c","address":"0x10ed43c718714eb63d5aa57b78b54704e256024e"}'
```

Response `200`

```json
{
  "selector": "0x18cbafe5",
  "method": "swapExactTokensForETH(uint256,uint256,address[],address,uint256)",
  "source": "Catalog",
  "arguments": [
    {
      "name": "amountIn",
      "type": "uint256",
      "value": "6492484590513546223964802"
    },
    {
      "name": "amountOutMin",
      "type": "uint256",
      "value": "1000000000000"
    },
    {
      "name": "path",
      "type": "address[]",
      "items": [
        {
          "name": "",
          "type": "address",
          "value": "0xd94a72950ead1b78331d91924a309ea322937777"
        },
        {
          "name": "",
          "type": "address",
          "value": "0xbb4cdb9cbd36b01bd1cbaebf2de08d9173bc095c"
        }
      ]
    },
    {
      "name": "to",
      "type": "address",
      "value": "0x95ed2e56cd3d45d2e3bfd4f27d58f352f9fd158e"
    },
    {
      "name": "deadline",
      "type": "uint256",
      "value": "1792694416"
    }
  ]
}
```

## Decode an event log

`POST /v1/decoder/log`

Decodes a log the same way, its event named by its first topic and the number of topics after it, so that ERC-20's and ERC-721's `Transfer`, which share a first topic, are told apart. The answer gives the `eventSignature`, its `source` and the decoded `arguments`, with the same `null` rules as for calldata.

| Body field | Description |
| --- | --- |
| `topics` | The log's one to four topics, each 32 bytes of hex. |
| `data` | Optional. The log's data as hex, at most 128 KiB; empty or left out when every argument is indexed. |
| `address` | Optional. The contract that emitted the log. |

Returns `400` with the title `Invalid topics`, `Invalid data` or `Invalid address` when one is malformed, and `413` for a body past 512 KiB.

The `Swap` log the same transaction's pool emitted:

```bash
curl -X POST "https://bnb.blockgaze.com/api/v1/decoder/log" \
  -H "Content-Type: application/json" \
  -d '{"topics":["0xd78ad95fa46c994b6551d0da85fc275fe613ce37657fb8d5e3d130840159d822","0x00000000000000000000000010ed43c718714eb63d5aa57b78b54704e256024e","0x00000000000000000000000010ed43c718714eb63d5aa57b78b54704e256024e"],"data":"0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000055ed64b7e1e73284c528200000000000000000000000000000000000000000000000005492fc6625b5c2c0000000000000000000000000000000000000000000000000000000000000000","address":"0x382d2632594024a6e958020bd55142416d0711fe"}'
```

Response `200`

```json
{
  "eventSignature": "Swap(address,uint256,uint256,uint256,uint256,address)",
  "source": "Catalog",
  "arguments": [
    {
      "name": "sender",
      "type": "address",
      "value": "0x10ed43c718714eb63d5aa57b78b54704e256024e"
    },
    {
      "name": "amount0In",
      "type": "uint256",
      "value": "0"
    },
    {
      "name": "amount1In",
      "type": "uint256",
      "value": "6492484590513546223964802"
    },
    {
      "name": "amount0Out",
      "type": "uint256",
      "value": "380888172589702188"
    },
    {
      "name": "amount1Out",
      "type": "uint256",
      "value": "0"
    },
    {
      "name": "to",
      "type": "address",
      "value": "0x10ed43c718714eb63d5aa57b78b54704e256024e"
    }
  ]
}
```

## Compiler versions

`GET /v1/verification/compilers`

The Solidity and Vyper compiler versions the deployment can verify contracts with, newest first.

```bash
curl "https://fsnex.net/api/v1/verification/compilers"
```

Response `200` (trimmed: `solidityVersions` shows 2 of 104; `vyperVersions` shows 2 of 27)

```json
{
  "solidityVersions": [
    "0.8.37+commit.f401782d",
    "0.8.36+commit.8a079791"
  ],
  "vyperVersions": [
    "0.4.3+commit.bff19ea2",
    "0.4.2+commit.c216787f"
  ]
}
```
