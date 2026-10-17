# Tokens, NFTs and trades

[Back to overview](../README.md)

- [List tokens](#list-tokens)
- [Get a token](#get-a-token)
- [Token price](#token-price)
- [Token holders](#token-holders)
- [Token DEX trades](#token-dex-trades)
- [List token transfers](#list-token-transfers)
- [List NFT collections](#list-nft-collections)
- [Collection items](#collection-items)
- [Get an NFT item](#get-an-nft-item)
- [NFT item holders](#nft-item-holders)
- [Collection NFT trades](#collection-nft-trades)
- [List DEX trades](#list-dex-trades)
- [List NFT trades](#list-nft-trades)

Tokens and NFT collections are both addressed by their contract address under `/v1/tokens/{address}`. Every endpoint on this page returns `400` when an address is not 20 bytes.

## List tokens

`GET /v1/tokens`

Fungible tokens with known metadata, most recently first seen first.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Tokens per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

```bash
curl "https://bnb.blockgaze.com/api/v1/tokens?pageSize=2"
```

Response `200`

```json
{
  "tokens": [
    {
      "address": "0xe7f5a7e6a828ff0ca6535db39a15efaebfce7777",
      "name": "Trump Fly With Tesla Roadster",
      "symbol": "TRUMPROAD",
      "decimals": 18,
      "firstSeenBlock": 128033602
    },
    {
      "address": "0x01be6ae2adedfccfe5a24429a28ad5945b34ffff",
      "name": "CHAIRMAN",
      "symbol": "CHAIRMAN",
      "decimals": 18,
      "firstSeenBlock": 128033602
    }
  ],
  "nextCursor": "128033602-0x01be6ae2adedfccfe5a24429a28ad5945b34ffff",
  "previousCursor": null,
  "total": null
}
```

## Get a token

`GET /v1/tokens/{address}`

A token's or NFT collection's metadata: name, symbol, decimals, standard, total supply and the block it was read at, and the block the token was first seen in. The `has…` flags say which of the lists below have anything to show. `sliceOf`, `slices` and the `slice…` fields describe time-sliced tokens.

Returns `404` when no token with metadata has been indexed at the address.

```bash
curl "https://fsnex.net/api/v1/tokens/0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe"
```

Response `200`

```json
{
  "address": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
  "name": "Wrapped Fusion",
  "symbol": "WFSN",
  "decimals": 18,
  "totalSupply": "2904372233396468405839489",
  "totalSupplyBlock": 15130080,
  "firstSeenBlock": 10458290,
  "standard": "Erc20",
  "sliceOf": null,
  "sliceStart": null,
  "sliceEnd": null,
  "slices": [],
  "hasMoreSlices": false,
  "hasHoldersAndTrades": true,
  "hasNftTrades": false,
  "hasNftItems": false,
  "listsForEverHoldings": false
}
```

## Token price

`GET /v1/tokens/{address}/price`

A token's price, worked out from its own recent DEX trades: the median of the newest trades against a stablecoin or the native currency. `nativePrice` is in whole units of the native currency and `usdPrice` in US dollars; the dollar price needs the deployment's market quote, so it is `null` on FSNEX. The wrapped native currency is priced at exactly 1. `fromBlock`, `toBlock` and `newestTradeTimestamp` say which trades the price comes from.

| Parameter | Description |
| --- | --- |
| `holder` | Optional. An address whose balance of the token is valued at this price, returned in `holder`. |

Returns `404` with the title `No price` when none of the token's newest trades is against a stablecoin or the native currency.

```bash
curl "https://bnb.blockgaze.com/api/v1/tokens/0x55d398326f99059ff775485246999027b3197955/price"
```

Response `200`

```json
{
  "nativePrice": "0.0013837417",
  "usdPrice": "0.9978564",
  "tradeCount": 5,
  "fromBlock": 128033977,
  "toBlock": 128033984,
  "newestTradeTimestamp": 1792178715,
  "quoteTimestamp": 1792178501,
  "marketCapUsd": "9165302932",
  "marketCapNative": "12709656",
  "supplyBlock": 126088633,
  "holder": null
}
```

With a holder, on FSNEX:

```bash
curl "https://fsnex.net/api/v1/tokens/0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe/price?holder=0x66ff2f0ac3214758d1e61b16b41e3d5e62caecf1"
```

Response `200`

```json
{
  "nativePrice": "1",
  "usdPrice": null,
  "tradeCount": 0,
  "fromBlock": 14613670,
  "toBlock": 14613670,
  "newestTradeTimestamp": 1753101293,
  "quoteTimestamp": null,
  "marketCapUsd": null,
  "marketCapNative": "2904372.2",
  "supplyBlock": 15130080,
  "holder": {
    "address": "0x66ff2f0ac3214758d1e61b16b41e3d5e62caecf1",
    "balance": "2519641532103451207500302",
    "asOfBlock": 15130080,
    "nativeValue": "2519641.5",
    "usdValue": null
  }
}
```

## Token holders

`GET /v1/tokens/{address}/holders`

A token's holders, largest balance first, with each holder's `holderLabel` and `holderTags` and each balance's share of the supply in `shares`. Tags are those [Get an address](addresses.md#get-an-address) lists, in the same order and shape; an untagged address has an empty list.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Holders per page, at most 100. |
| `cursor`, `last`, `count`, `page` | | See [pagination](../README.md#pagination). This list is not in block order, so it does not accept `from`. |

```bash
curl "https://fsnex.net/api/v1/tokens/0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe/holders?pageSize=2"
```

Response `200`

```json
{
  "holders": [
    {
      "holder": "0x66ff2f0ac3214758d1e61b16b41e3d5e62caecf1",
      "holderLabel": null,
      "holderTags": [],
      "balance": "2519641532103451207500302",
      "asOfBlock": 15130080
    },
    {
      "holder": "0xc2e094589db6070f3d7cad1fa1e88a03c20d5e83",
      "holderLabel": null,
      "holderTags": [],
      "balance": "92017674983102992888194",
      "asOfBlock": 15130080
    }
  ],
  "shares": {
    "basis": 2,
    "asOf": 15130080,
    "shares": [
      0.8675339555759696,
      0.031682466153966256
    ]
  },
  "nextCursor": "92017674983102992888194-0xc2e094589db6070f3d7cad1fa1e88a03c20d5e83",
  "previousCursor": null,
  "total": null
}
```

## Token DEX trades

`GET /v1/tokens/{address}/dex-trades`

DEX trades in pools that hold the token, newest first. Same shape as [`GET /v1/dex-trades`](#list-dex-trades).

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Trades per page, at most 100. |
| `cursor` | | The `nextCursor` or `previousCursor` of the page you have. |
| `last` | `false` | `true` returns the oldest page. |

```bash
curl "https://fsnex.net/api/v1/tokens/0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe/dex-trades?pageSize=2"
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
      "traderTags": [],
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
      "logIndex": 16,
      "transactionHash": "0x1d5a3573fef26f15a3a035718e3b21e10c688a0035277086abd52471fdbbea05",
      "timestamp": 1753101293,
      "pool": "0x16ee31ac8020ee5421994b3e5afae7fa49f2bf43",
      "poolAddress": "0x16ee31ac8020ee5421994b3e5afae7fa49f2bf43",
      "protocol": "V2",
      "exchangeName": "Chainge",
      "trader": "0xc2c007352c00fefa1a4752a28658705168abd873",
      "traderLabel": null,
      "traderTags": [],
      "baseToken": {
        "address": "0xab1f7e5bf2587543fe41f268c59d35da95f046e0",
        "isNative": false,
        "name": "Chainge",
        "symbol": "XCHNG",
        "decimals": 18
      },
      "quoteToken": {
        "address": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
        "isNative": false,
        "name": "Wrapped Fusion",
        "symbol": "WFSN",
        "decimals": 18
      },
      "side": "Buy",
      "baseAmount": "7193341152092284389",
      "quoteAmount": "2162060962723682439",
      "price": "0.30056421"
    }
  ],
  "nextCursor": "14613670-2-16",
  "previousCursor": null,
  "fromBlock": 14613670,
  "toBlock": 14616611
}
```

## List token transfers

`GET /v1/token-transfers`

Token and NFT transfers, newest first, with the token's name, symbol and decimals on each. `value` is in the token's base units; `tokenId` is set for NFT transfers. Without filters this lists every indexed transfer. `fromLabel` / `fromTags` and `toLabel` / `toTags` name each side. Tags are those [Get an address](addresses.md#get-an-address) lists, in the same order and shape; an untagged address has an empty list.

| Parameter | Default | Description |
| --- | --- | --- |
| `token` | | Only transfers of this token or collection. |
| `address` | | Only transfers from or to this address. |
| `kind` | | With `address` only: `tokens` for fungible token transfers, or `nfts` for NFT transfers. |
| `pageSize` | `25` | Transfers per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

```bash
curl "https://bnb.blockgaze.com/api/v1/token-transfers?pageSize=2"
```

Response `200`

```json
{
  "transfers": [
    {
      "blockNumber": 128209084,
      "transactionIndex": 75,
      "logIndex": 445,
      "batchIndex": 0,
      "transactionHash": "0x50a3574c25bcd4d582fa8d5de37f3c25211b1cada010796e4e83f86480fdb4b8",
      "timestamp": 1792257537,
      "token": "0x55d398326f99059ff775485246999027b3197955",
      "tokenName": "Tether USD",
      "tokenSymbol": "USDT",
      "tokenDecimals": 18,
      "standard": "Erc20",
      "from": "0x29be31e7ef434a74d69da1cb6b51ad65c83e9a7b",
      "to": "0x9f8f4615ff5143aee365fa34f34196fb85be7650",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": null,
      "toTags": [],
      "value": "62515907277819439041",
      "tokenId": null,
      "sliceStart": null,
      "sliceEnd": null,
      "direction": null
    },
    {
      "blockNumber": 128209084,
      "transactionIndex": 75,
      "logIndex": 444,
      "batchIndex": 0,
      "transactionHash": "0x50a3574c25bcd4d582fa8d5de37f3c25211b1cada010796e4e83f86480fdb4b8",
      "timestamp": 1792257537,
      "token": "0xac23b90a79504865d52b49b327328411a23d4db2",
      "tokenName": "Falcon Finance",
      "tokenSymbol": "FF",
      "tokenDecimals": 18,
      "standard": "Erc20",
      "from": "0x9f8f4615ff5143aee365fa34f34196fb85be7650",
      "to": "0x29be31e7ef434a74d69da1cb6b51ad65c83e9a7b",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": null,
      "toTags": [],
      "value": "600597985935399757804",
      "tokenId": null,
      "sliceStart": null,
      "sliceEnd": null,
      "direction": null
    }
  ],
  "scanCursor": null,
  "nextCursor": "128209084-75-444-0",
  "previousCursor": null,
  "total": null
}
```

Filtered by token:

```bash
curl "https://fsnex.net/api/v1/token-transfers?token=0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe&pageSize=2"
```

Response `200`

```json
{
  "transfers": [
    {
      "blockNumber": 14616611,
      "transactionIndex": 1,
      "logIndex": 2,
      "batchIndex": 0,
      "transactionHash": "0x08637c648d1c67f513c8c1357c1e8a77039a550c2eceff6d2700de3162c64401",
      "timestamp": 1753139660,
      "token": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
      "tokenName": "Wrapped Fusion",
      "tokenSymbol": "WFSN",
      "tokenDecimals": 18,
      "standard": "Erc20",
      "from": "0x3668c219b1fa8fe8175158f6ce91ded36fde9152",
      "to": "0x66ff2f0ac3214758d1e61b16b41e3d5e62caecf1",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": null,
      "toTags": [],
      "value": "500000000000000000000",
      "tokenId": null,
      "sliceStart": null,
      "sliceEnd": null,
      "direction": null
    },
    {
      "blockNumber": 14613670,
      "transactionIndex": 2,
      "logIndex": 22,
      "batchIndex": 0,
      "transactionHash": "0x1d5a3573fef26f15a3a035718e3b21e10c688a0035277086abd52471fdbbea05",
      "timestamp": 1753101293,
      "token": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
      "tokenName": "Wrapped Fusion",
      "tokenSymbol": "WFSN",
      "tokenDecimals": 18,
      "standard": "Erc20",
      "from": "0xc1e9125dca7fc864af4bd731526a5b8c298571ed",
      "to": "0x31c798ac8159568d44c7f0c8678bf6fdc4caf57c",
      "fromLabel": null,
      "fromTags": [],
      "toLabel": null,
      "toTags": [],
      "value": "10355609926761016",
      "tokenId": null,
      "sliceStart": null,
      "sliceEnd": null,
      "direction": null
    }
  ],
  "scanCursor": null,
  "nextCursor": "14613670-2-22-0",
  "previousCursor": null,
  "total": null
}
```

## List NFT collections

`GET /v1/nfts`

ERC-721 and ERC-1155 collections, most recently first seen first.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Collections per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/nfts?pageSize=2"
```

Response `200`

```json
{
  "collections": [
    {
      "address": "0xd9657dfe4c629978b99abf4df1e1d5a42f29c421",
      "name": "XCHNG Name Service",
      "symbol": "XNS",
      "standard": "Erc721",
      "firstSeenBlock": 11948958
    },
    {
      "address": "0x1ced084b3015cd5535e848828e1671848a261376",
      "name": "XCHNG Name Service",
      "symbol": "XNS",
      "standard": "Erc721",
      "firstSeenBlock": 11937602
    }
  ],
  "nextCursor": "11937602-0x1ced084b3015cd5535e848828e1671848a261376",
  "previousCursor": null,
  "total": null
}
```

## Collection items

`GET /v1/tokens/{address}/nft-items`

The items of an NFT collection by token id, with owner, its `ownerLabel` and `ownerTags`, and item name where known. Tags are those [Get an address](addresses.md#get-an-address) lists, in the same order and shape; an untagged address has an empty list.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Items per page, at most 100. |
| `cursor`, `last`, `count`, `page` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/tokens/0xd9657dfe4c629978b99abf4df1e1d5a42f29c421/nft-items?pageSize=2"
```

Response `200`

```json
{
  "items": [
    {
      "tokenId": "0",
      "owner": "0x22886574a42834921aaca6a772e63c234cec9764",
      "ownerLabel": null,
      "ownerTags": [],
      "quantity": null,
      "itemName": "xchng.xchng",
      "asOfBlock": 15130080
    },
    {
      "tokenId": "1",
      "owner": "0x710e89f20f07e638851af76c42c8aa3464c088ff",
      "ownerLabel": null,
      "ownerTags": [],
      "quantity": null,
      "itemName": "chainge.xchng",
      "asOfBlock": 15130080
    }
  ],
  "nextCursor": "1-0x710e89f20f07e638851af76c42c8aa3464c088ff",
  "previousCursor": null,
  "total": null
}
```

## Get an NFT item

`GET /v1/tokens/{address}/nft-items/{tokenId}`

One item: its owner with its `ownerLabel` and `ownerTags`, and the metadata the indexer fetched from the token URI. `metadataState` says whether the metadata has been fetched (`Fetched`) or not yet (`NotQueued` and other states), and `imageHref` is a browser-ready link for IPFS and Arweave images.

| Parameter | Description |
| --- | --- |
| `tokenId` (path) | The token id as a decimal number. |

Returns `400` for a token id that is not a decimal uint256, and `404` when neither the collection nor the item has been indexed.

```bash
curl "https://fsnex.net/api/v1/tokens/0xd9657dfe4c629978b99abf4df1e1d5a42f29c421/nft-items/0"
```

Response `200`

```json
{
  "token": "0xd9657dfe4c629978b99abf4df1e1d5a42f29c421",
  "tokenName": "XCHNG Name Service",
  "tokenSymbol": "XNS",
  "tokenId": "0",
  "standard": "Erc721",
  "owner": "0x22886574a42834921aaca6a772e63c234cec9764",
  "ownerLabel": null,
  "ownerTags": [],
  "ownerAsOfBlock": 15130080,
  "metadataState": "Fetched",
  "metadataFetchedAt": "2026-10-17T01:03:45.127944+00:00",
  "name": "xchng.xchng",
  "description": "Embrace the decentralized future of your XCHNG identity on Fusion Mainnet",
  "image": null,
  "imageHref": null,
  "traits": []
}
```

## NFT item holders

`GET /v1/tokens/{address}/nft-items/{tokenId}/holders`

The holders of an ERC-1155 item, largest quantity first, each with its `holderLabel` and `holderTags`. An ERC-721 item has a single owner, given by [Get an NFT item](#get-an-nft-item), so this list is empty for it, as in this example.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Holders per page, at most 100. |
| `cursor`, `last`, `count`, `page` | | See [pagination](../README.md#pagination). This list does not accept `from`. |

```bash
curl "https://fsnex.net/api/v1/tokens/0xd9657dfe4c629978b99abf4df1e1d5a42f29c421/nft-items/0/holders?pageSize=2"
```

Response `200`

```json
{
  "holders": [],
  "nextCursor": null,
  "previousCursor": null,
  "total": null
}
```

## Collection NFT trades

`GET /v1/tokens/{address}/nft-trades`

Marketplace trades of a collection's items, newest first. Same shape as [`GET /v1/nft-trades`](#list-nft-trades), plus `fromBlock` and `toBlock`, the blocks the page covers.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Trades per page, at most 100. |
| `cursor` | | The `nextCursor` or `previousCursor` of the page you have. |
| `last` | `false` | `true` returns the oldest page. |

```bash
curl "https://bnb.blockgaze.com/api/v1/tokens/0x43bde1047a25274c377800da13eaaa506f29618b/nft-trades?pageSize=1"
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
      "sellerTags": [],
      "buyer": "0x5813c157b33a09685e30a5f2e6e1fc54089d08e9",
      "buyerLabel": null,
      "buyerTags": [],
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

## List DEX trades

`GET /v1/dex-trades`

Swaps on decentralised exchanges, newest first. Each trade names the pool, the protocol and exchange, the trader with its `traderLabel` and `traderTags`, the base and quote tokens, the side (`Buy` or `Sell` of the base token), both amounts in base units, and the price of the base token in quote tokens.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Trades per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/dex-trades?pageSize=2"
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
      "traderTags": [],
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
      "traderTags": [],
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
  "total": null
}
```

## List NFT trades

`GET /v1/nft-trades`

NFT marketplace sales, newest first: marketplace, collection, token id, quantity, seller and buyer with their labels and tags (`sellerTags`, `buyerTags`), and price in base units of the payment currency. Tags are those [Get an address](addresses.md#get-an-address) lists, in the same order and shape; an untagged address has an empty list. `bundleSize` is the number of items sold together in one order, and `position` the item's place in it.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Trades per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

```bash
curl "https://bnb.blockgaze.com/api/v1/nft-trades?pageSize=2"
```

Response `200`

```json
{
  "trades": [
    {
      "blockNumber": 128187128,
      "transactionIndex": 36,
      "logIndex": 210,
      "position": 0,
      "bundleSize": 1,
      "transactionHash": "0x3825d941c593eba3e7dfe2b55c618735b7261b0dd9044fc85a308e57c30135a4",
      "timestamp": 1792247653,
      "marketplace": "0x0000000000000068f116a894984e2db1123eb395",
      "marketplaceName": null,
      "protocol": "Seaport",
      "standard": "Erc721",
      "collection": {
        "address": "0xb1024b89886b9a34aa4ff5f31c411d708b20a14c",
        "isNative": false,
        "name": "TapeOut",
        "symbol": "TapeOut",
        "decimals": null
      },
      "tokenId": "15754",
      "quantity": "1",
      "seller": "0x0622f0364f6687e4e468d54658f5e0b4eb1f522d",
      "sellerLabel": null,
      "sellerTags": [],
      "buyer": "0x3b560ed68e58d31418458c514d9b823ca0b29e93",
      "buyerLabel": null,
      "buyerTags": [],
      "price": "1151400000000000000",
      "currency": {
        "address": "0x0000000000000000000000000000000000000000",
        "isNative": true,
        "name": "BNB",
        "symbol": "BNB",
        "decimals": 18
      }
    },
    {
      "blockNumber": 128184018,
      "transactionIndex": 25,
      "logIndex": 141,
      "position": 0,
      "bundleSize": 1,
      "transactionHash": "0xfdf4c7b3af7754fb71fa4f9321c965bdb63fbecec0087f29bdabb8aa1281d94f",
      "timestamp": 1792246253,
      "marketplace": "0x0000000000000068f116a894984e2db1123eb395",
      "marketplaceName": null,
      "protocol": "Seaport",
      "standard": "Erc721",
      "collection": {
        "address": "0xfe8128d79b88c21cd246c639c3ee608f40a7cd37",
        "isNative": false,
        "name": "TapeOut Miner",
        "symbol": "TMINER",
        "decimals": null
      },
      "tokenId": "17",
      "quantity": "1",
      "seller": "0xba7ec7d0f21b404e896e18b3edc3d9dd8502905f",
      "sellerLabel": null,
      "sellerTags": [],
      "buyer": "0x0903f8167008de5b8e21ea0a5475c689e9fbdf38",
      "buyerLabel": null,
      "buyerTags": [],
      "price": "404000000000000000",
      "currency": {
        "address": "0x0000000000000000000000000000000000000000",
        "isNative": true,
        "name": "BNB",
        "symbol": "BNB",
        "decimals": 18
      }
    }
  ],
  "logCount": 2,
  "nextCursor": "128184018-25-141",
  "previousCursor": null,
  "total": null
}
```
