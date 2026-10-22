# Contracts

[Back to overview](../README.md)

- [Contract bytecode](#contract-bytecode)
- [Opcodes](#opcodes)
- [Verified functions](#verified-functions)
- [Read a contract function](#read-a-contract-function)
- [Verified source](#verified-source)
- [Verified contracts](#verified-contracts)
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
