# Addresses

[Back to overview](../README.md)

- [Rich list](#rich-list)
- [Get an address](#get-an-address)
- [Token balances](#token-balances)
- [NFTs held](#nfts-held)
- [Time-slice holdings](#time-slice-holdings)
- [Token approvals](#token-approvals)
- [NFT approvals](#nft-approvals)
- [Events](#events)
- [Internal transactions](#internal-transactions)
- [DEX trades](#dex-trades)
- [NFT trades](#nft-trades)
- [Primary name](#primary-name) (BNB Smart Chain)
- [CSV export](#csv-export)

An address's transactions are listed by [`GET /v1/transactions?address=…`](blocks-and-transactions.md#list-transactions), and its token transfers by [`GET /v1/token-transfers?address=…`](tokens-and-nfts.md#list-token-transfers). On FSNEX, its Fusion asset holdings, asset transfers, swaps and tickets are on the [Fusion page](fusion.md).

Every endpoint on this page returns `400` when `address` is not a 20-byte address.

## Rich list

`GET /v1/addresses`

Addresses by native balance, largest first, with each balance's share of the supply in `shares`. `amountColumns` adds chain-specific amounts per address in the same order; on FSNEX these are the FSN time-locked forever and the FSN staked in tickets.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Addresses per page, at most 100. |
| `cursor`, `last`, `count`, `page` | | See [pagination](../README.md#pagination). This list is not in block order, so it does not accept `from`, and `page` reaches only the first 100,000 addresses. |

```bash
curl "https://fsnex.net/api/v1/addresses?pageSize=2"
```

Response `200`

```json
{
  "addresses": [
    {
      "address": "0x8449cc37b8b678a138f0e1e73953302e68a45335",
      "label": null,
      "balance": "4645982057110905995742236",
      "asOfBlock": 15130080
    },
    {
      "address": "0x359767ca87d111fcffd244bde6b47f95a9867a89",
      "label": null,
      "balance": "3921644990597518984060240",
      "asOfBlock": 15130080
    }
  ],
  "amountColumns": [
    {
      "key": "time-locked-forever",
      "asOfBlock": null,
      "amounts": [
        "1500000000000000000000000",
        "0"
      ]
    },
    {
      "key": "staked",
      "asOfBlock": 15130080,
      "amounts": [
        "0",
        "0"
      ]
    }
  ],
  "shares": {
    "basis": 1,
    "asOf": null,
    "shares": [
      0.05883362071682636,
      0.04966109880035397
    ]
  },
  "nextCursor": "3921644990597518984060240-0x359767ca87d111fcffd244bde6b47f95a9867a89",
  "previousCursor": null,
  "total": null
}
```

## Get an address

`GET /v1/addresses/{address}`

An address's native balance, label, whether it is a contract, and, for contracts, who created it and in which transaction. `delegatesTo` is the delegated code of an EIP-7702 account. `holdsTimeSlices` and `holdsNfts` say whether the time-slice and NFT lists below have anything to show. `production` gives the number of blocks the address produced and the range they span, and is `null` for an address that produced none. `token` names the token a contract is, with its `name`, `symbol` and `standard` as [Get a token](tokens-and-nfts.md#get-a-token) gives them, and is `null` for an address with no token there.

```bash
curl "https://bnb.blockgaze.com/api/v1/addresses/0xbb4cdb9cbd36b01bd1cbaebf2de08d9173bc095c"
```

Response `200`

```json
{
  "address": "0xbb4cdb9cbd36b01bd1cbaebf2de08d9173bc095c",
  "isContract": true,
  "delegatesTo": null,
  "creator": null,
  "creationTransactionHash": null,
  "createdInBlock": null,
  "label": {
    "groupName": "Token Contracts",
    "name": "Wrapped BNB",
    "description": "Widely used token contracts, labelled so they read by name in transaction lists.",
    "url": null
  },
  "balance": {
    "balance": "1553425822991030814023074",
    "asOfBlock": 128030415
  },
  "holdsTimeSlices": false,
  "holdsNfts": false,
  "production": null
}
```

## Token balances

`GET /v1/addresses/{address}/tokens`

The fungible tokens the address holds, with balance in the token's base units and the block the balance was read at, ordered by token address.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Tokens per page, at most 100. |
| `cursor`, `last`, `count`, `page` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/addresses/0x66ff2f0ac3214758d1e61b16b41e3d5e62caecf1/tokens?pageSize=2"
```

Response `200`

```json
{
  "tokens": [
    {
      "token": "0x022899ba170fc1676a47375ae34bcfe1b0ba57ec",
      "tokenName": "QITMEER NETWORK",
      "tokenSymbol": "MEER",
      "tokenDecimals": 18,
      "balance": "71355731913946312128550",
      "asOfBlock": 15130080,
      "sliceOf": null,
      "sliceStart": null,
      "sliceEnd": null
    },
    {
      "token": "0x031eef26e3b2b60116a1185cb4bfe9f254ac4ac7",
      "tokenName": "FOMOON",
      "tokenSymbol": "FOMOON",
      "tokenDecimals": 8,
      "balance": "15992772335287335",
      "asOfBlock": 15130080,
      "sliceOf": null,
      "sliceStart": null,
      "sliceEnd": null
    }
  ],
  "nextCursor": "0x031eef26e3b2b60116a1185cb4bfe9f254ac4ac7",
  "previousCursor": null,
  "total": null
}
```

## NFTs held

`GET /v1/addresses/{address}/nfts`

The ERC-721 and ERC-1155 items the address holds.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Items per page, at most 100. |
| `cursor`, `last`, `count`, `page` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/addresses/0x22886574a42834921aaca6a772e63c234cec9764/nfts?pageSize=2"
```

Response `200`

```json
{
  "nfts": [
    {
      "token": "0x9472d17589ca3e8856ed8c1640e10acbb0ee06f5",
      "tokenName": "INSCRIPTION",
      "tokenSymbol": "INSCRIPTION",
      "tokenId": "2",
      "quantity": null,
      "itemName": null,
      "asOfBlock": 15130080
    },
    {
      "token": "0xd9657dfe4c629978b99abf4df1e1d5a42f29c421",
      "tokenName": "XCHNG Name Service",
      "tokenSymbol": "XNS",
      "tokenId": "0",
      "quantity": null,
      "itemName": "xchng.xchng",
      "asOfBlock": 15130080
    }
  ],
  "nextCursor": "0xd9657dfe4c629978b99abf4df1e1d5a42f29c421-0",
  "previousCursor": null,
  "total": null
}
```

## Time-slice holdings

`GET /v1/addresses/{address}/time-slices`

Balances of time-sliced tokens: tokens whose balances are valid for a time range, given by `sliceStart` and `sliceEnd` on each holding. The list is empty for an address whose detail shows `"holdsTimeSlices": false`, as in this example.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Holdings per page, at most 100. |
| `cursor`, `last`, `count`, `page` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/addresses/0xc2c007352c00fefa1a4752a28658705168abd873/time-slices?pageSize=2"
```

Response `200`

```json
{
  "holdings": [],
  "nextCursor": null,
  "previousCursor": null,
  "total": null
}
```

## Token approvals

`GET /v1/addresses/{address}/approvals/tokens`

The token allowances the address has granted: token, spender and amount, with `isUnlimited` for the maximum uint256 allowance. `allowancesReadToBlock` is the block the allowances were read to, and is `null` on a deployment that does not track allowances, such as BNB Smart Chain.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `50` | Approvals per page, at most 100. |
| `cursor` | | The `nextCursor` of the previous page. |

```bash
curl "https://fsnex.net/api/v1/addresses/0x66ff2f0ac3214758d1e61b16b41e3d5e62caecf1/approvals/tokens?pageSize=1"
```

Response `200`

```json
{
  "approvals": [
    {
      "token": "0x00b4c5e53e50f3c5b65e7f303d8b8cb15a8d459b",
      "tokenName": null,
      "tokenSymbol": null,
      "tokenDecimals": null,
      "spender": "0x3668c219b1fa8fe8175158f6ce91ded36fde9152",
      "spenderLabel": null,
      "amount": "115792089237316195423570985008687907853269984665640564039457584007913129639935",
      "isUnlimited": true,
      "asOfBlock": 15130080
    }
  ],
  "nextCursor": "0x00b4c5e53e50f3c5b65e7f303d8b8cb15a8d459b-0x3668c219b1fa8fe8175158f6ce91ded36fde9152",
  "allowancesReadToBlock": 15130080
}
```

## NFT approvals

`GET /v1/addresses/{address}/approvals/nfts`

The NFT operators the address has approved.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `50` | Approvals per page, at most 100. |
| `cursor` | | The `nextCursor` of the previous page. |

```bash
curl "https://fsnex.net/api/v1/addresses/0xc2c007352c00fefa1a4752a28658705168abd873/approvals/nfts?pageSize=2"
```

Response `200`

```json
{
  "approvals": [],
  "nextCursor": null,
  "isIndexed": true
}
```

## Events

`GET /v1/addresses/{address}/events`

The event logs a contract emitted, newest first, decoded into `eventSignature` and `arguments` where the event is known.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Events per page, at most 25. |
| `cursor` | | The `nextCursor` of the previous page. |

```bash
curl "https://fsnex.net/api/v1/addresses/0x049ddc3cd20ac7a2f6c867680f7e21de70aca9c3/events?pageSize=1"
```

Response `200`

```json
{
  "events": [
    {
      "blockNumber": 13299771,
      "transactionIndex": 2,
      "transactionHash": "0x05358c425e7a598303d4ced5a47ad549aa1ad353c47da6883770862bd90204bf",
      "timestamp": 1735939829,
      "log": {
        "removed": false,
        "eventSource": "Catalog",
        "logIndex": 14,
        "address": "0x049ddc3cd20ac7a2f6c867680f7e21de70aca9c3",
        "addressLabel": null,
        "topics": [
          "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
          "0x0000000000000000000000000d9b0fca02c67e13e524c5fd4408b13a26454085",
          "0x0000000000000000000000000000000000000000000000000000000000000000"
        ],
        "data": "0x000000000000000000000000000000000000000000000000000000ce82a38aff",
        "eventSignature": "Transfer(address,address,uint256)",
        "arguments": [
          {
            "name": "from",
            "type": "address",
            "value": "0x0d9b0fca02c67e13e524c5fd4408b13a26454085"
          },
          {
            "name": "to",
            "type": "address",
            "value": "0x0000000000000000000000000000000000000000"
          },
          {
            "name": "value",
            "type": "uint256",
            "value": "886955019007"
          }
        ]
      }
    }
  ],
  "nextCursor": "13299771-2-14",
  "isIndexed": true
}
```

## Internal transactions

`GET /v1/addresses/{address}/internal-transactions`

The call frames inside transactions that the address sent or received, newest first. `frame.movedValue` says whether the frame actually moved value: it is `false` when the frame or a frame above it failed, or the transaction failed. `direction` is the address's side of the frame.

Only blocks from `coveredFrom` to `coveredThrough` have been traced. On FSNEX the trace is still being extended through the chain's history; on BNB Smart Chain internal transactions are not traced, and both fields are `null`.

| Parameter | Default | Description |
| --- | --- | --- |
| `kind` | `all` | `all` for every frame, or `transfers` for frames that carry value. |
| `pageSize` | `25` | Frames per page, at most 50. |
| `cursor` | | The `nextCursor` of the previous page. |

```bash
curl "https://fsnex.net/api/v1/addresses/0x049ddc3cd20ac7a2f6c867680f7e21de70aca9c3/internal-transactions?pageSize=1"
```

Response `200`

```json
{
  "transactions": [
    {
      "blockNumber": 7155774,
      "transactionIndex": 0,
      "transactionHash": "0xfab12f07010c19823c0a6f6a72eac70328fff1b49eff7b45ab9d76fa2a23088a",
      "timestamp": 1655545569,
      "frame": {
        "notes": [],
        "traceAddress": "0_4",
        "type": "call",
        "from": "0x049ddc3cd20ac7a2f6c867680f7e21de70aca9c3",
        "to": "0x78917333bec47cee1022b31a136d31feff90d6fb",
        "value": "82735686965967908390",
        "gas": 158631,
        "error": null,
        "revertReason": null,
        "undone": false,
        "movedValue": true
      },
      "direction": "Out"
    }
  ],
  "nextCursor": "7155774-0-0000000000000004",
  "coveredFrom": 0,
  "coveredThrough": 7155960
}
```

## DEX trades

`GET /v1/addresses/{address}/dex-trades`

The address's swaps on decentralised exchanges, newest first. Same shape as [`GET /v1/dex-trades`](tokens-and-nfts.md#list-dex-trades).

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Trades per page, at most 100. |
| `cursor` | | The `nextCursor` or `previousCursor` of the page you have. |
| `last` | `false` | `true` returns the oldest page. |

```bash
curl "https://fsnex.net/api/v1/addresses/0xc2c007352c00fefa1a4752a28658705168abd873/dex-trades?pageSize=2"
```

Response `200`

```json
{
  "trades": [
    {
      "blockNumber": 14613670,
      "transactionIndex": 2,
      "logIndex": 21,
      "transactionHash": "0x1d5a3573fef26f15a3a035718e3b21e10c688a0035277086abd52471fdbbea05",
      "timestamp": 1753101293,
      "pool": "0x32a201389d96e98c895f5315b943ca89f8f921ab",
      "poolAddress": "0x32a201389d96e98c895f5315b943ca89f8f921ab",
      "protocol": "V2",
      "exchangeName": "Chainge",
      "trader": "0xc2c007352c00fefa1a4752a28658705168abd873",
      "traderLabel": null,
      "baseToken": {
        "address": "0x9b9c34f603723ed9aec108a51edfc1b60cd2d7cc",
        "isNative": false,
        "name": "TRON",
        "symbol": "TRX",
        "decimals": 6
      },
      "quoteToken": {
        "address": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
        "isNative": false,
        "name": "Wrapped Fusion",
        "symbol": "WFSN",
        "decimals": 18
      },
      "side": "Sell",
      "baseAmount": "232345",
      "quoteAmount": "2172416572650443455",
      "price": "9.3499605"
    },
    {
      "blockNumber": 14613670,
      "transactionIndex": 2,
      "logIndex": 19,
      "transactionHash": "0x1d5a3573fef26f15a3a035718e3b21e10c688a0035277086abd52471fdbbea05",
      "timestamp": 1753101293,
      "pool": "0x5c76aff0709927b571fcaccf1cec5b21429b84f4",
      "poolAddress": "0x5c76aff0709927b571fcaccf1cec5b21429b84f4",
      "protocol": "V2",
      "exchangeName": "Chainge",
      "trader": "0xc2c007352c00fefa1a4752a28658705168abd873",
      "traderLabel": null,
      "baseToken": {
        "address": "0x9b9c34f603723ed9aec108a51edfc1b60cd2d7cc",
        "isNative": false,
        "name": "TRON",
        "symbol": "TRX",
        "decimals": 6
      },
      "quoteToken": {
        "address": "0xab1f7e5bf2587543fe41f268c59d35da95f046e0",
        "isNative": false,
        "name": "Chainge",
        "symbol": "XCHNG",
        "decimals": 18
      },
      "side": "Buy",
      "baseAmount": "232345",
      "quoteAmount": "7193341152092284389",
      "price": "30.959741"
    }
  ],
  "nextCursor": "14613670-2-19",
  "previousCursor": null,
  "fromBlock": 14613670,
  "toBlock": 14613670
}
```

## NFT trades

`GET /v1/addresses/{address}/nft-trades`

The address's NFT marketplace purchases and sales, newest first. Same shape as [`GET /v1/nft-trades`](tokens-and-nfts.md#list-nft-trades), plus `fromBlock` and `toBlock`, the blocks the page covers.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Trades per page, at most 100. |
| `cursor` | | The `nextCursor` or `previousCursor` of the page you have. |
| `last` | `false` | `true` returns the oldest page. |

```bash
curl "https://bnb.blockgaze.com/api/v1/addresses/0x5813c157b33a09685e30a5f2e6e1fc54089d08e9/nft-trades?pageSize=1"
```

Response `200`

```json
{
  "trades": [
    {
      "blockNumber": 128016438,
      "transactionIndex": 39,
      "logIndex": 201,
      "position": 0,
      "bundleSize": 1,
      "transactionHash": "0x41b3ec580d871860cf31f72c5e9f0f418306d4069c06eb7d7c6ccd1a545391a0",
      "timestamp": 1792170819,
      "marketplace": "0xb3e3dfcb2d9f3dde16d78b9e6eb3538eb32b5ae1",
      "marketplaceName": "Element",
      "protocol": "Element",
      "standard": "Erc721",
      "collection": {
        "address": "0x43bde1047a25274c377800da13eaaa506f29618b",
        "isNative": false,
        "name": "STONKS BROKER",
        "symbol": "STONK",
        "decimals": null
      },
      "tokenId": "7036",
      "quantity": "1",
      "seller": "0x04e7882988c9be3bb3fa1a7aa96af7e62464e752",
      "sellerLabel": null,
      "buyer": "0x5813c157b33a09685e30a5f2e6e1fc54089d08e9",
      "buyerLabel": null,
      "price": "8590000000000000",
      "currency": {
        "address": "0x0000000000000000000000000000000000000000",
        "isNative": true,
        "name": "BNB",
        "symbol": "BNB",
        "decimals": 18
      }
    }
  ],
  "nextCursor": "128016438-39-201",
  "previousCursor": null,
  "fromBlock": 128016438,
  "toBlock": 128016438
}
```

## Primary name

`GET /v1/addresses/{address}/name`

BNB Smart Chain only (`features.spaceIdNames`). The address's primary SPACE ID name, or `null` when it has none.

```bash
curl "https://bnb.blockgaze.com/api/v1/addresses/0xbb4cdb9cbd36b01bd1cbaebf2de08d9173bc095c/name"
```

Response `200`

```json
{
  "address": "0xbb4cdb9cbd36b01bd1cbaebf2de08d9173bc095c",
  "name": null
}
```

## CSV export

`GET /v1/export`

One calendar year of an address's history as a CSV file, the same file the explorer's export page produces. The response is `text/csv` with a `Content-Disposition` file name. A large year can take tens of seconds to produce.

This endpoint has its own [rate limit](../README.md#rate-limits) of 10 requests per 10 minutes per client.

| Parameter | Default | Description |
| --- | --- | --- |
| `address` | | Required. The address to export. |
| `year` | | Required. The calendar year, from 2015 to the current year. |
| `content` | `native` | `native` (native currency transactions), `tokens` (token transfers), `nfts` (NFT transfers) or `rewards` (a block producer's daily rewards), plus the chain-specific values in `exportContentKinds` from [network status](network.md#network-status). FSNEX adds `fusion-assets` and `fusion-swaps`. |
| `timeZone` | `UTC` | An IANA time zone such as `Europe/Stockholm`. The date column is given in it and named for it, for example `DateTime (Europe/Stockholm)`. `rewards` exports in `UTC` only. |
| `format` | `explorer` | `explorer`, or `koinly` for a file Koinly can import. `rewards` exports in `explorer` format only. |

Returns `400` with a problem-details body when a parameter is not accepted, or when reading the year takes longer than the server allows.

```bash
curl -OJ "https://fsnex.net/api/v1/export?address=0x66ff2f0ac3214758d1e61b16b41e3d5e62caecf1&content=tokens&year=2025"
```

Response `200`, `Content-Disposition: attachment; filename=fsnex-fusion-tokens-2025-0x66ff2f0ac3214758d1e61b16b41e3d5e62caecf1.csv` (first rows of 89,299):

```csv
Txhash,BlockNumber,UnixTimestamp,DateTime (UTC),From,To,TokenValue,TokenSymbol,TokenName,TokenContract,TokenId,Standard,SliceStart (Unix),SliceEnd (Unix)
0x6c2101de1f983f159efa725c68252f14e09d2cdefb3088937d090de95d4182c0,13280662,1735689605,2025-01-01 00:00:05,0x3668c219b1fa8fe8175158f6ce91ded36fde9152,0x66ff2f0ac3214758d1e61b16b41e3d5e62caecf1,4471.45036193,PIRATE,PIRATE,0x5402905e2046ea6c14143075cc94ed60ce27a977,,Erc20,,
0x6c2101de1f983f159efa725c68252f14e09d2cdefb3088937d090de95d4182c0,13280662,1735689605,2025-01-01 00:00:05,0x3668c219b1fa8fe8175158f6ce91ded36fde9152,0x66ff2f0ac3214758d1e61b16b41e3d5e62caecf1,22.72000145,PIRATE,PIRATE,0x5402905e2046ea6c14143075cc94ed60ce27a977,,Erc20,,
0x5ad47fd0594102bfe504f50081166591a8572fb5475a6cf3442bdaf727731e14,13280667,1735689671,2025-01-01 00:01:11,0x3668c219b1fa8fe8175158f6ce91ded36fde9152,0x66ff2f0ac3214758d1e61b16b41e3d5e62caecf1,2329862.22679913,PHONKE,PHONKE,0xafdfc779286d2afaa9837485219ea82aa5f2693f,,Erc20,,
```

Unlike the JSON endpoints, the CSV gives amounts in whole units (`TokenValue`), not base units.

A `native` export, here for `0xc2c007352c00fefa1a4752a28658705168abd873` in 2025, starts:

```csv
Txhash,BlockNumber,UnixTimestamp,DateTime (UTC),From,To,Value (FSN),Fee (FSN),Status
0x48d85568371179613abf946b781bb7a713f25ccd13558ce407c1364f168b8f95,13280679,1735689827,2025-01-01 00:03:47,0xc2c007352c00fefa1a4752a28658705168abd873,0xc1e9125dca7fc864af4bd731526a5b8c298571ed,0,0.013288,Success
```

An unknown `content`:

```bash
curl "https://fsnex.net/api/v1/export?address=0xc2c007352c00fefa1a4752a28658705168abd873&content=nope&year=2025"
```

Response `400`

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "Export not possible",
  "status": 400,
  "detail": "'nope' is not an export content; use one of 'native', 'tokens', 'nfts', 'rewards', 'fusion-assets', 'fusion-swaps'.",
  "traceId": "00-e2e89f14390e12968717c30acb9f1657-058fd25761de5705-00"
}
```
