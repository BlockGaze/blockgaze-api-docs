# Etherscan-compatible module

[Back to overview](../README.md)

The base URL of each deployment answers a subset of Etherscan's API, with Etherscan's parameter names and response format, so existing Etherscan clients and scripts can point at a BlockGaze deployment:

| Deployment | Etherscan-compatible URL |
| --- | --- |
| FSNEX | `https://fsnex.net/api` |
| BNB Smart Chain | `https://bnb.blockgaze.com/api` |

The answers come from the same index as the JSON endpoints, so on BNB Smart Chain they cover the [retained window](../README.md#retention-differs-per-deployment) only. No API key is needed.

- [Response format](#response-format)
- [Errors](#errors)
- [Supported actions](#supported-actions)
- [Normal transactions](#normal-transactions)
- [Internal transactions](#internal-transactions)
- [Token transfers](#token-transfers)
- [Balances](#balances)
- [Block by timestamp](#block-by-timestamp)
- [Event logs](#event-logs)
- [Contract ABI and source](#contract-abi-and-source)
- [Contract verification from Hardhat and Foundry](#contract-verification-from-hardhat-and-foundry)

## Response format

Every answer is HTTP `200` with Etherscan's envelope. `status` is the string `"1"` on success and `"0"` otherwise, `message` is `OK` or a short reason, and `result` holds the answer. Numbers in `result` are decimal strings, except in `getLogs`, which uses hex as Etherscan does.

## Errors

Errors have `"status": "0"` and the reason in `result`, or in `message` for an empty list:

```bash
curl "https://fsnex.net/api?module=account&action=nope"
```

Response `200`

```json
{
  "status": "0",
  "message": "NOTOK",
  "result": "Error! Missing Or invalid Action name"
}
```

```bash
curl "https://fsnex.net/api?module=account&action=tokentx&address=0xc2c007352c00fefa1a4752a28658705168abd873&page=1&offset=2"
```

Response `200`

```json
{
  "status": "0",
  "message": "No transactions found",
  "result": []
}
```

The `proxy` module, which Etherscan passes to a node, is not supported:

```bash
curl "https://fsnex.net/api?module=proxy&action=eth_blockNumber"
```

Response `200`

```json
{
  "status": "0",
  "message": "NOTOK",
  "result": "Error! The proxy module is not supported: this explorer answers from its index and passes no calls to a node."
}
```

A request over the [rate limit](../README.md#rate-limits) gets HTTP `429` with the Etherscan envelope and a `result` that begins `Max rate limit reached`. Calls to this module run at most 4 at a time for the whole deployment, whoever sends them, so send them one at a time rather than in parallel.

## Supported actions

| `module` | `action` | Returns |
| --- | --- | --- |
| `account` | `txlist` | Normal transactions of an address |
| `account` | `txlistinternal` | Internal transactions of an address or of one transaction |
| `account` | `tokentx` | ERC-20 transfers |
| `account` | `tokennfttx` | ERC-721 transfers |
| `account` | `token1155tx` | ERC-1155 transfers |
| `account` | `balance` | Native balance of an address |
| `account` | `balancemulti` | Native balances of up to 20 addresses |
| `block` | `getblocknobytime` | The block closest to a timestamp |
| `logs` | `getLogs` | Event logs |
| `contract` | `getabi` | ABI of a verified contract |
| `contract` | `getsourcecode` | Source and settings of a verified contract |
| `contract` | `verifysourcecode`, `checkverifystatus` | Contract verification, for Hardhat and Foundry |

## Normal transactions

`?module=account&action=txlist`

The list actions share these parameters:

| Parameter | Default | Description |
| --- | --- | --- |
| `address` | | The address whose transactions to list. |
| `startblock` | `0` | First block to include. |
| `endblock` | newest | Last block to include. |
| `page` | `1` | Page number. |
| `offset` | `10000` | Rows per page. `page` × `offset` may be at most 10,000. |
| `sort` | `asc` | `asc` for oldest first, `desc` for newest first. |

```bash
curl "https://fsnex.net/api?module=account&action=txlist&address=0xc2c007352c00fefa1a4752a28658705168abd873&page=1&offset=2&sort=desc"
```

Response `200` (trimmed: `result.input` cut from 320 characters)

```json
{
  "status": "1",
  "message": "OK",
  "result": [
    {
      "blockNumber": "14613670",
      "timeStamp": "1753101293",
      "hash": "0x1d5a3573fef26f15a3a035718e3b21e10c688a0035277086abd52471fdbbea05",
      "nonce": "137488",
      "blockHash": "0x92dfa9abe482dec2c4369eeac3e598a113ee57082f546e6a1f3e5ea1b07003c7",
      "transactionIndex": "2",
      "from": "0xc2c007352c00fefa1a4752a28658705168abd873",
      "to": "0xc1e9125dca7fc864af4bd731526a5b8c298571ed",
      "value": "0",
      "gas": "423318",
      "gasPrice": "14002068809",
      "isError": "0",
      "txreceipt_status": "1",
      "input": "0xff030000000000000000000c5a482ac95f6e16ee31ac8020ee5421994b3e5afa…",
      "contractAddress": "",
      "cumulativeGasUsed": "506263",
      "gasUsed": "237746",
      "confirmations": "516411",
      "methodId": "0xff030000",
      "functionName": ""
    },
    {
      "blockNumber": "14613669",
      "timeStamp": "1753101280",
      "hash": "0x57f418ff092f7014ea880b8f1182868195c198eafa5f9563cf7e3c3469322f74",
      "nonce": "137487",
      "blockHash": "0xcf3bf6a6ea8f88df8040cccc73248931a02731c7456c7480e2563dddd35a98ea",
      "transactionIndex": "2",
      "from": "0xc2c007352c00fefa1a4752a28658705168abd873",
      "to": "0xc1e9125dca7fc864af4bd731526a5b8c298571ed",
      "value": "0",
      "gas": "450504",
      "gasPrice": "10000000000",
      "isError": "0",
      "txreceipt_status": "1",
      "input": "0xff030000000000000000000000000000004f4981884daa6e96800f88a60f0f9b…",
      "contractAddress": "",
      "cumulativeGasUsed": "482737",
      "gasUsed": "264932",
      "confirmations": "516412",
      "methodId": "0xff030000",
      "functionName": ""
    }
  ]
}
```

## Internal transactions

`?module=account&action=txlistinternal`

Takes the list parameters above with `address`, or `txhash` for one transaction's internal transactions. The answer covers only blocks the deployment has traced; see [internal transactions](addresses.md#internal-transactions).

```bash
curl "https://fsnex.net/api?module=account&action=txlistinternal&address=0x049ddc3cd20ac7a2f6c867680f7e21de70aca9c3&page=1&offset=1"
```

Response `200`

```json
{
  "status": "1",
  "message": "OK",
  "result": [
    {
      "blockNumber": "2538520",
      "timeStamp": "1595149540",
      "hash": "0x2db2445851075399e044420ad0566087d6649349ed02084f685b95ca269fda55",
      "from": "0xa12cba22e4c316820bf4883ebb98a3789cf194a3",
      "to": "",
      "value": "0",
      "contractAddress": "0x049ddc3cd20ac7a2f6c867680f7e21de70aca9c3",
      "input": "",
      "type": "create",
      "gas": "211681",
      "gasUsed": "9230",
      "traceId": "0",
      "isError": "0",
      "errCode": ""
    }
  ]
}
```

## Token transfers

`?module=account&action=tokentx` (ERC-20), `tokennfttx` (ERC-721) and `token1155tx` (ERC-1155)

Take the list parameters above, plus `contractaddress` to list one token's transfers. If a page cannot be completed within the transfer logs the server reads in one call, the answer asks you to narrow `startblock` and `endblock`.

```bash
curl "https://bnb.blockgaze.com/api?module=account&action=tokentx&address=0x5813c157b33a09685e30a5f2e6e1fc54089d08e9&page=1&offset=1"
```

Response `200`

```json
{
  "status": "1",
  "message": "OK",
  "result": [
    {
      "blockNumber": "127967012",
      "timeStamp": "1792148569",
      "hash": "0x522ac2dc2bdf2fd85b63a810caf1f15ca1db9c17587be5bea7b9599cc58cc2a1",
      "nonce": "125",
      "blockHash": "0x4141de1b4995b82807780adb8956b1133bcb82a022d5a973c6a262d2d3cf067f",
      "from": "0xa280bf6666a898ded0d8c7dced1dbd15b749430a",
      "contractAddress": "0xa280bf6666a898ded0d8c7dced1dbd15b749430a",
      "to": "0x5813c157b33a09685e30a5f2e6e1fc54089d08e9",
      "value": "7000000000000000000",
      "tokenName": "GM",
      "tokenSymbol": "GM",
      "tokenDecimal": "18",
      "transactionIndex": "75",
      "gas": "7206713",
      "gasPrice": "52500000",
      "gasUsed": "7148809",
      "cumulativeGasUsed": "18918988",
      "input": "deprecated",
      "methodId": "0x729ad39e",
      "functionName": "",
      "confirmations": "67057"
    }
  ]
}
```

```bash
curl "https://bnb.blockgaze.com/api?module=account&action=tokennfttx&address=0x5813c157b33a09685e30a5f2e6e1fc54089d08e9&page=1&offset=1"
```

Response `200`

```json
{
  "status": "1",
  "message": "OK",
  "result": [
    {
      "blockNumber": "127972885",
      "timeStamp": "1792151213",
      "hash": "0x41474755829efa9f2cfe8faabde9ba7b665bc9e34d35ce64a2b24bb113b13500",
      "nonce": "113",
      "blockHash": "0x1985d905eebb8aea194d3f3cf94cd440bfc7667db1723fa5c52e9b7c45df06e0",
      "from": "0x8389a3e684d532a3b0b4cdbb72c9c7797e6fa4f7",
      "contractAddress": "0x43bde1047a25274c377800da13eaaa506f29618b",
      "to": "0x5813c157b33a09685e30a5f2e6e1fc54089d08e9",
      "tokenID": "7301",
      "tokenName": "STONKS BROKER",
      "tokenSymbol": "STONK",
      "tokenDecimal": "0",
      "transactionIndex": "48",
      "gas": "1151152",
      "gasPrice": "66828968",
      "gasUsed": "889833",
      "cumulativeGasUsed": "9547459",
      "input": "deprecated",
      "methodId": "0x149b8ce6",
      "functionName": "",
      "confirmations": "61188"
    }
  ]
}
```

## Balances

`?module=account&action=balance` and `?module=account&action=balancemulti`

The native balance in wei, as last read by the indexer.

| Parameter | Description |
| --- | --- |
| `address` | One address, or for `balancemulti` up to 20 addresses separated by commas. |
| `tag` | Optional. Only `latest` is accepted, because the deployment stores one balance reading per address. |

```bash
curl "https://fsnex.net/api?module=account&action=balance&address=0xc2c007352c00fefa1a4752a28658705168abd873"
```

Response `200`

```json
{
  "status": "1",
  "message": "OK",
  "result": "8397098381426989763"
}
```

```bash
curl "https://fsnex.net/api?module=account&action=balancemulti&address=0xc2c007352c00fefa1a4752a28658705168abd873,0x0aef49b7935a789003ba9c20c1e23a34d9435734"
```

Response `200`

```json
{
  "status": "1",
  "message": "OK",
  "result": [
    {
      "account": "0xc2c007352c00fefa1a4752a28658705168abd873",
      "balance": "8397098381426989763"
    },
    {
      "account": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
      "balance": "1644665884658536059017"
    }
  ]
}
```

## Block by timestamp

`?module=block&action=getblocknobytime`

| Parameter | Description |
| --- | --- |
| `timestamp` | Unix seconds. |
| `closest` | `before` or `after`. |

```bash
curl "https://fsnex.net/api?module=block&action=getblocknobytime&timestamp=1753101293&closest=before"
```

Response `200`

```json
{
  "status": "1",
  "message": "OK",
  "result": "14613670"
}
```

## Event logs

`?module=logs&action=getLogs`

| Parameter | Description |
| --- | --- |
| `fromBlock` | Required. First block. |
| `toBlock` | Last block. |
| `address` | Only logs emitted by this contract. |
| `topic0` … `topic3` | Only logs with these 32-byte topics. |
| `topic0_1_opr` and similar | `and` or `or` between two topics. Three or four topics combine with `and` only. |
| `page`, `offset` | Paging. `offset` is at most 1,000. |

Without `address`, a call reads at most 100 blocks. With `address`, wider ranges are allowed, up to a limit on the logs read per call; past it the answer asks you to narrow the range.

```bash
curl "https://fsnex.net/api?module=logs&action=getLogs&address=0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe&fromBlock=14613670&toBlock=14613670&page=1&offset=2"
```

Response `200`

```json
{
  "status": "1",
  "message": "OK",
  "result": [
    {
      "address": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
      "topics": [
        "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "0x00000000000000000000000032a201389d96e98c895f5315b943ca89f8f921ab",
        "0x0000000000000000000000006037c8fd5a32fa44ed358ef96d94bd323d5b5e90"
      ],
      "data": "0x000000000000000000000000000000000000000000000000540091117bb9090e",
      "blockNumber": "0xdefca6",
      "blockHash": "0x92dfa9abe482dec2c4369eeac3e598a113ee57082f546e6a1f3e5ea1b07003c7",
      "timeStamp": "0x687e33ed",
      "gasPrice": "0x342969d49",
      "gasUsed": "0x3c5fd",
      "logIndex": "0x1",
      "transactionHash": "0xa5eeec0876526c98637882c7b0e68f855fd0e668c9dda78df0df5c951ad3d2a8",
      "transactionIndex": "0x1"
    },
    {
      "address": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
      "topics": [
        "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "0x0000000000000000000000006037c8fd5a32fa44ed358ef96d94bd323d5b5e90",
        "0x000000000000000000000000c2e094589db6070f3d7cad1fa1e88a03c20d5e83"
      ],
      "data": "0x00000000000000000000000000000000000000000000000053a31fefac0cb5c0",
      "blockNumber": "0xdefca6",
      "blockHash": "0x92dfa9abe482dec2c4369eeac3e598a113ee57082f546e6a1f3e5ea1b07003c7",
      "timeStamp": "0x687e33ed",
      "gasPrice": "0x342969d49",
      "gasUsed": "0x3c5fd",
      "logIndex": "0x2",
      "transactionHash": "0xa5eeec0876526c98637882c7b0e68f855fd0e668c9dda78df0df5c951ad3d2a8",
      "transactionIndex": "0x1"
    }
  ]
}
```

## Contract ABI and source

`?module=contract&action=getabi` and `?module=contract&action=getsourcecode`

| Parameter | Description |
| --- | --- |
| `address` | The contract address. |

`getabi` returns the ABI as a JSON string. `getsourcecode` returns the source in Etherscan's layout; a multi-file source is a standard JSON input wrapped in double braces, as Etherscan returns it.

```bash
curl "https://bnb.blockgaze.com/api?module=contract&action=getabi&address=0xca11bde05977b3631167028862be2a173976ca11"
```

Response `200` (trimmed: `result` cut from 5772 characters)

```json
{
  "status": "1",
  "message": "OK",
  "result": "[{\"name\": \"aggregate\", \"type\": \"function\", \"inputs\": [{\"name\": \"ca…"
}
```

```bash
curl "https://bnb.blockgaze.com/api?module=contract&action=getsourcecode&address=0xca11bde05977b3631167028862be2a173976ca11"
```

Response `200` (trimmed: `result.SourceCode` cut from 10180 characters; `result.ABI` cut from 5772 characters)

```json
{
  "status": "1",
  "message": "OK",
  "result": [
    {
      "SourceCode": "{{\"sources\": {\"contracts/Multicall3.sol\": {\"content\": \"// SPDX-Lic…",
      "ABI": "[{\"name\": \"aggregate\", \"type\": \"function\", \"inputs\": [{\"name\": \"ca…",
      "ContractName": "Multicall3",
      "CompilerVersion": "v0.8.12+commit.f00d7308",
      "OptimizationUsed": "1",
      "Runs": "10000000",
      "ConstructorArguments": "",
      "EVMVersion": "Default",
      "Library": "",
      "LicenseType": "Unknown",
      "Proxy": "0",
      "Implementation": "",
      "SwarmSource": ""
    }
  ]
}
```

For an unverified contract:

```bash
curl "https://fsnex.net/api?module=contract&action=getabi&address=0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe"
```

Response `200`

```json
{
  "status": "0",
  "message": "NOTOK",
  "result": "Contract source code not verified"
}
```

## Contract verification from Hardhat and Foundry

The module also accepts the `verifysourcecode` and `checkverifystatus` actions that `hardhat verify` and `forge verify-contract` send, so contracts can be verified on a deployment from those tools by pointing their Etherscan URL at the deployment's base URL. Submissions share the explorer's verification budget, so a client can submit a limited number of verifications in a period.
