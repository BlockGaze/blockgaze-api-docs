# Network and statistics

[Back to overview](../README.md)

- [Network status](#network-status)
- [Network activity](#network-activity)
- [Sync status](#sync-status)
- [Gas](#gas)
- [Daily statistics](#daily-statistics)
- [Daily staked amount and yield](#daily-staked-amount-and-yield)
- [Block producers](#block-producers)
- [Block producer history](#block-producer-history)
- [Staked supply](#staked-supply)
- [Stake distribution](#stake-distribution)
- [Labels](#labels)
- [Search](#search)
- [Market data](#market-data) (BNB Smart Chain)
- [Staking total](#staking-total) (BNB Smart Chain)

## Network status

`GET /v1/network`

The chain the deployment indexes, the feature sets it serves, and the range of blocks it holds. Read this first: `indexedFromBlock` and `indexedToBlock` give the retained range, `historyState` says whether the chain is live (`Live`) or halted and complete (`HaltedComplete`), and `refresh` gives how often the explorer polls each kind of data on a live chain. `tracedFromBlock` and `tracedToBlock` give the blocks whose internal transactions have been traced, the range the [internal transaction list](blocks-and-transactions.md#list-internal-transactions) covers, and are both `null` where no block has been traced. `tracingEnabled` says whether the deployment traces new blocks: where it is `false` the traced range stays readable but no longer grows, as on a deployment that switched tracing off after it ran.

`features` holds one flag for each feature set, `true` where the deployment serves it: `marketData`, `stakeHubStaking` and `spaceIdNames` (BNB Smart Chain), `fusionAssets` and `fusionStaking` (FSNEX), `stakeDistribution`, `stakedHistory` and `stakedSupply` (both), and `sourcify`, `true` where the chain is one [Sourcify](https://sourcify.dev) lists, so the deployment may import the verified sources Sourcify holds into the verified contracts its contract pages read (BNB Smart Chain; a deployment can still leave the import off). A client should read a flag rather than assume one from the chain.

`transactionOperationKinds` lists the values the [transaction list](blocks-and-transactions.md#list-transactions) accepts for `operation`, and `exportContentKinds` lists the chain-specific values the [CSV export](addresses.md#csv-export) accepts for `content`.

```bash
curl "https://bnb.blockgaze.com/api/v1/network"
```

Response `200`

```json
{
  "chainKey": "bsc",
  "features": {
    "fusionAssets": false,
    "fusionStaking": false,
    "stakeDistribution": true,
    "stakedHistory": true,
    "stakedSupply": true,
    "marketData": true,
    "stakeHubStaking": true,
    "spaceIdNames": true,
    "sourcify": true
  },
  "transactionOperationKinds": [],
  "exportContentKinds": [],
  "chainName": "BNB Smart Chain",
  "chainId": 56,
  "nativeCurrencySymbol": "BNB",
  "nativeCurrencyDecimals": 18,
  "indexedFromBlock": 128390000,
  "indexedFromTimestamp": 1792338970,
  "indexedToBlock": 128793732,
  "latestBlockTimestamp": 1792520738,
  "tracedFromBlock": null,
  "tracedToBlock": null,
  "tracingEnabled": false,
  "statisticsFromTimestamp": 1790791966,
  "halt": null,
  "totalSupply": null,
  "historyState": "Live",
  "refresh": {
    "latest": "00:00:05",
    "activity": "00:00:30",
    "daily": "00:05:00"
  }
}
```

On FSNEX:

```bash
curl "https://fsnex.net/api/v1/network"
```

Response `200` (trimmed: `transactionOperationKinds` shows 2 of 10)

```json
{
  "chainKey": "fusion",
  "features": {
    "fusionAssets": true,
    "fusionStaking": true,
    "stakeDistribution": true,
    "stakedHistory": true,
    "stakedSupply": true,
    "marketData": false,
    "stakeHubStaking": false,
    "spaceIdNames": false,
    "sourcify": false
  },
  "transactionOperationKinds": [
    {
      "key": "asset-transfer",
      "name": "Asset transfer"
    },
    {
      "key": "time-lock",
      "name": "Time-lock operation"
    }
  ],
  "exportContentKinds": [
    {
      "key": "fusion-assets",
      "name": "Fusion asset transfers"
    },
    {
      "key": "fusion-swaps",
      "name": "Fusion native swaps"
    }
  ],
  "chainName": "Fusion",
  "chainId": 32659,
  "nativeCurrencySymbol": "FSN",
  "nativeCurrencyDecimals": 18,
  "indexedFromBlock": 0,
  "indexedFromTimestamp": 1561852800,
  "indexedToBlock": 15130080,
  "latestBlockTimestamp": 1759826630,
  "tracedFromBlock": 0,
  "tracedToBlock": 11377063,
  "tracingEnabled": true,
  "statisticsFromTimestamp": 1561852800,
  "halt": {
    "block": 15130080,
    "timestamp": 1759826630
  },
  "totalSupply": "78968147812500000000000000",
  "historyState": "HaltedComplete",
  "refresh": null
}
```

## Network activity

`GET /v1/network/activity`

Averages over the most recent blocks and complete days: block time, validator count, transactions per second, and daily transaction and token transfer counts.

```bash
curl "https://bnb.blockgaze.com/api/v1/network/activity"
```

Response `200`

```json
{
  "sampleBlockCount": 1000,
  "averageBlockTime": "00:00:00.4494494",
  "validatorCount": 26,
  "transactionsPerSecond": 155.74610244988864,
  "completeDayCount": 14,
  "dailyTransactionAverage": 15750138,
  "dailyTokenTransferAverage": 53292002
}
```

## Sync status

`GET /v1/sync-status`

How far the indexer has got. `lastBlock` is the newest indexed block and `latestBlockAge` how long ago it was produced. Each entry in `projectors` is a derived data set (balances, token metadata, DEX pools, internal transactions and so on) with the block it has reached and how far it is behind. Data from a projector that is behind can be missing for the newest blocks.

`status` is `Fresh` when the newest block is recent and the required projectors are caught up, and `Delayed` otherwise. FSNEX always reports `Delayed`, because its chain data ends at a fixed block.

```bash
curl "https://bnb.blockgaze.com/api/v1/sync-status"
```

Response `200` (trimmed: `projectors` shows 2 of 12; `balanceRefresh` shows 2 of 3)

```json
{
  "observedAt": "2026-10-16T19:22:51.9326103+00:00",
  "status": "Delayed",
  "firstBlock": 127630000,
  "lastBlock": 128033661,
  "latestBlockAge": "00:00:01.9326103",
  "retainedBlockSpan": 403662,
  "databaseSizeBytes": 176725595839,
  "projectors": [
    {
      "name": "daily-statistics",
      "required": true,
      "blockNumber": 128033661,
      "blocksBehind": 0,
      "movedAt": "2026-10-16T19:22:51.241617+00:00",
      "stalled": false
    },
    {
      "name": "event-definitions",
      "required": true,
      "blockNumber": 128033661,
      "blocksBehind": 0,
      "movedAt": "2026-10-16T19:22:51.272213+00:00",
      "stalled": false
    }
  ],
  "balanceRefreshStatus": "Pending",
  "balanceRefresh": [
    {
      "kind": "native",
      "hasPending": true,
      "pendingKeys": null,
      "oldestRequiredBlock": null
    },
    {
      "kind": "token",
      "hasPending": true,
      "pendingKeys": null,
      "oldestRequiredBlock": null
    }
  ]
}
```

## Gas

`GET /v1/gas`

Gas prices paid over the most recent 200 blocks: low, median and high price in wei, the current base fee, how full blocks are, and the same figures per block.

```bash
curl "https://bnb.blockgaze.com/api/v1/gas"
```

Response `200` (trimmed: `blocks` shows 2 of 20)

```json
{
  "fromBlock": 128033462,
  "toBlock": 128033661,
  "toTimestamp": 1792178570,
  "blockCount": 200,
  "pricedTransactionCount": 12005,
  "lowPrice": "50000000",
  "medianPrice": "51869123",
  "highPrice": "184391635",
  "baseFeePerGas": "0",
  "gasUsedShare": 0.164361,
  "blocks": [
    {
      "number": 128033661,
      "timestamp": 1792178570,
      "transactionCount": 64,
      "gasUsed": 10053952,
      "gasLimit": 70000000,
      "gasUsedShare": 0.143627,
      "medianPricePaid": "55000000"
    },
    {
      "number": 128033660,
      "timestamp": 1792178570,
      "transactionCount": 62,
      "gasUsed": 13019991,
      "gasLimit": 70000000,
      "gasUsedShare": 0.185999,
      "medianPricePaid": "57500000"
    }
  ]
}
```

## Daily statistics

`GET /v1/statistics/daily`

One entry per UTC day, oldest first: blocks, transactions, failed transactions, token transfers, average gas use and price, fees, value transferred, and active and cumulative addresses. Figures not yet computed for a day are `null`.

| Parameter | Default | Description |
| --- | --- | --- |
| `days` | `30` | Number of days to return, ending today, or ending at the chain's last day on FSNEX. |

```bash
curl "https://bnb.blockgaze.com/api/v1/statistics/daily?days=2"
```

Response `200`

```json
[
  {
    "day": "2026-10-15",
    "isPartial": false,
    "blockCount": 191855,
    "transactionCount": 17101124,
    "failedTransactionCount": 679851,
    "tokenTransferCount": 55870729,
    "averageGasUsedPerBlock": 17152732,
    "averageBlockTime": "00:00:00.4503372",
    "averageGasPrice": "184055879",
    "totalGasFees": "605697976857416852855",
    "totalFees": null,
    "valueTransferred": "648662925063722979889259",
    "activeAddresses": 2473078,
    "cumulativeAddresses": null
  },
  {
    "day": "2026-10-16",
    "isPartial": true,
    "blockCount": 154903,
    "transactionCount": 14708697,
    "failedTransactionCount": 1052251,
    "tokenTransferCount": 49019865,
    "averageGasUsedPerBlock": 18207427,
    "averageBlockTime": "00:00:00.4503299",
    "averageGasPrice": "233298056",
    "totalGasFees": "657990360581609548513",
    "totalFees": null,
    "valueTransferred": "389610987055637604700841",
    "activeAddresses": null,
    "cumulativeAddresses": null
  }
]
```

## Daily staked amount and yield

`GET /v1/statistics/staked`

The native currency staked at the end of each UTC day, in wei. Served where `features.stakedHistory` is `true`, which is both deployments.

Each day also carries `rewards`, the block rewards its blocks paid, issuance and gas and native fees together, in wei (issuance is none on BNB Smart Chain), and `annualYield`, those rewards over the day's closing stake times 365, as a fraction: `0.05` is 5%. It is a gross yield, before any commission, burn or system-reward share the chain takes (on BNB Smart Chain, validators' commission and the part of each block's fees that is burned or sent to the system reward contract), so a staker earns less. `annualYield` is `null` for the newest day and the first day the deployment indexed, whose blocks it holds only in part, for a day whose stake or rewards are unknown, and past a millionfold. `rewards` is a part day's on the newest day, which `isPartial` marks, and `null` on the first indexed day, whose stake is whole but whose blocks are not, and for a day the deployment holds no statistics of.

| Parameter | Default | Description |
| --- | --- | --- |
| `days` | `30` | Number of days to return. |

```bash
curl "https://bnb.blockgaze.com/api/v1/statistics/staked?days=2"
```

Response `200`

```json
[
  {
    "day": "2026-10-21",
    "isPartial": false,
    "staked": "24192830433408873213685977",
    "rewards": "496912957050617256450",
    "annualYield": 0.007496
  },
  {
    "day": "2026-10-22",
    "isPartial": true,
    "staked": "24185683647157966308380240",
    "rewards": "390403110497644658813",
    "annualYield": null
  }
]
```

## Block producers

`GET /v1/block-producers`

The addresses that produced blocks over the most recent days, most blocks first, with each producer's `label`, `tags`, block count, share, fees, issuance and total reward in wei. Tags are those [Get an address](addresses.md#get-an-address) lists, in the same order and shape; an untagged address has an empty list.

| Parameter | Default | Description |
| --- | --- | --- |
| `days` | `7` | Number of days to cover, ending today. |

```bash
curl "https://bnb.blockgaze.com/api/v1/block-producers?days=1"
```

Response `200` (trimmed: `producers` shows 2 of 45)

```json
{
  "days": 1,
  "endsToday": true,
  "fromBlock": 128070605,
  "toBlock": 128209081,
  "blockCount": 138477,
  "producers": [
    {
      "producer": "0xb4647b856cb9c3856d559c885bed8b43e0846a47",
      "label": "CertiK",
      "tags": [
        {
          "title": "Validator",
          "colour": "Accent",
          "description": "A validator's consensus address: it produces blocks and sends each block's system transactions."
        }
      ],
      "blockCount": 6112,
      "sharePercent": 4.413729355777494,
      "totalFees": "18250999103627067904",
      "issuance": "0",
      "blockReward": "18250999103627067904",
      "lastBlock": 128208983
    },
    {
      "producer": "0xf8b99643fafc79d9404de68e48c4d49a3936f787",
      "label": "Avengers",
      "tags": [
        {
          "title": "Validator",
          "colour": "Accent",
          "description": "A validator's consensus address: it produces blocks and sends each block's system transactions."
        }
      ],
      "blockCount": 6072,
      "sharePercent": 4.384843692454343,
      "totalFees": "15856885027197169108",
      "issuance": "0",
      "blockReward": "15856885027197169108",
      "lastBlock": 128209023
    }
  ]
}
```

## Block producer history

`GET /v1/block-producers/{address}/daily`

One producer's blocks and rewards per UTC day.

| Parameter | Default | Description |
| --- | --- | --- |
| `address` (path) | | The producer's address. |
| `days` | `30` | Number of days to return. |

```bash
curl "https://fsnex.net/api/v1/block-producers/0x0aef49b7935a789003ba9c20c1e23a34d9435734/daily?days=2"
```

Response `200`

```json
{
  "addressIsValid": true,
  "hasBlocks": true,
  "days": [
    {
      "day": "2025-10-06",
      "isPartial": false,
      "blockCount": 4884,
      "reward": "1526901001634538185664"
    },
    {
      "day": "2025-10-07",
      "isPartial": true,
      "blockCount": 1754,
      "reward": "548357217199056656512"
    }
  ]
}
```

## Staked supply

`GET /v1/staked-supply`

How much of the native currency is staked, and its share of the total supply: the figure the explorer's home page shows. Served where `features.stakedSupply` is `true`, which is both deployments. On BNB Smart Chain it is StakeHub's newest recorded total, beside the total supply of the newest market quote; on FSNEX it is the FSN held by the tickets valid at the time of the block the ticket projection has reached, beside the total supply after that block (genesis plus issuance through it).

- `available`: `false` while the chain has nothing complete to say, as before a first StakeHub sample or while FSNEX's ticket history from genesis is not indexed; every other field is then `null`.
- `asOfBlock`, `asOfTimestamp`: when the figure was taken. On BNB Smart Chain the sample is read from StakeHub with the node at block `asOfBlock`, which can be newer than the newest indexed block, and `asOfTimestamp` (Unix seconds) is when the reading was recorded; on FSNEX they are the block the ticket projection has reached and that block's time.
- `amount`: the staked amount in the native currency's smallest unit, as a string.
- `share`: `amount` as a fraction of the total supply, truncated to six digits, or `null` where the supply is not known, as on a BNB Smart Chain network without market data.

```bash
curl "https://bnb.blockgaze.com/api/v1/staked-supply"
```

Response `200`

```json
{
  "available": true,
  "asOfBlock": 128429558,
  "asOfTimestamp": 1792356809,
  "amount": "24189608927739788315672255",
  "share": 0.18166
}
```

## Stake distribution

`GET /v1/stake-distribution`

How the staked native currency is spread across stakers: the largest stakers by amount and share, and the rest summed under `other`. Served where `features.stakeDistribution` is `true`, which is both deployments. On BNB Smart Chain the stakers are validators; on FSNEX they are ticket owners. Each slice carries its staker's `label` and `tags`. Tags are those [Get an address](addresses.md#get-an-address) lists, in the same order and shape; an untagged address has an empty list.

```bash
curl "https://bnb.blockgaze.com/api/v1/stake-distribution"
```

Response `200` (trimmed: `slices` shows 2 of 6)

```json
{
  "available": true,
  "asOfBlock": 128207534,
  "evaluatedAt": null,
  "holderCount": 56,
  "total": "24148770834373050147243207",
  "slices": [
    {
      "address": "0xd3b0d838ccceae7ebf1781d11d1bb741db7fe1a7",
      "label": "BNBEve",
      "tags": [
        {
          "title": "Validator",
          "colour": "Accent",
          "description": "A validator's consensus address: it produces blocks and sends each block's system transactions."
        }
      ],
      "amount": "1245617119928016174571939",
      "share": 0.05158
    },
    {
      "address": "0xbdcc079bbb23c1d9a6f36aa31309676c258abac7",
      "label": "Fuji",
      "tags": [
        {
          "title": "Validator",
          "colour": "Accent",
          "description": "A validator's consensus address: it produces blocks and sends each block's system transactions."
        }
      ],
      "amount": "1148893384610266310271527",
      "share": 0.047575
    }
  ],
  "other": {
    "holderCount": 50,
    "amount": "17275358861338521428653398",
    "share": 0.715372
  }
}
```

## Labels

`GET /v1/labels`

The groups of named addresses the deployment uses for labels, each with a description and its addresses.

```bash
curl "https://fsnex.net/api/v1/labels"
```

Response `200` (trimmed: the array shows 2 of 12; `description` cut from 279 characters; `addresses` shows 2 of 31)

```json
[
  {
    "name": "Anyswap",
    "description": "From the original FSNEX explorer's curated labels. Anyswap was a c…",
    "url": null,
    "addresses": [
      {
        "address": "0x2e1f1c7620eecc7b7c571dff36e43ac7ed276779",
        "name": "Anyswap: ANY Staking"
      },
      {
        "address": "0x0c74199d22f732039e843366a236ff4f61986b32",
        "name": "Anyswap: ANY Token"
      }
    ]
  },
  {
    "name": "Burn Addresses",
    "description": "Addresses recognised by their pattern as unspendable. None of them has ever sent a transaction in the indexed history, so whatever reached them stayed there.",
    "url": null,
    "addresses": [
      {
        "address": "0x000000000000000000000000000000000000dead",
        "name": "Burn Address (0x0000…dead)"
      },
      {
        "address": "0x1111111111111111111111111111111111111111",
        "name": "Burn Address (0x1111…1111)"
      }
    ]
  }
]
```

## Search

`GET /v1/search`

Works out what a search string refers to and where the explorer shows it. `queryKind` is how the string was read: `BlockNumber`, `Address`, `TransactionHash` (any 32-byte hash) or `Unrecognised`. `matches` lists the indexed records found, each with its `kind` and its `path` on the explorer, and is empty when nothing indexed matches. An address, a 40- or 64-digit hex string or a string of digits alone is never searched for as a name.

A string that is none of those is searched for as a name, in this order:

- `Token` and `NftCollection`: up to 10 fungible tokens and 10 NFT collections whose name or symbol contains the string, ignoring case. A string shorter than three characters matches a whole symbol instead, and one longer than 100 matches nothing. Copies of a name or symbol are common, so each group is ranked: a token with a curated label first, then one with verified source, then an exact name or symbol, then one that starts with the string, then the one the explorer saw first.
- `Label`: up to 10 labels, curated or, on BNB Smart Chain, a validator's StakeHub name, whose name holds every word of the string, in any order and case; an exact name first, then one that starts with the words.
- `Name`: a name the chain resolves, such as a `.bnb` name on BNB Smart Chain. Until the release that brought the name search, such a match had the kind `Address`.

On Fusion a number can also match a USAN (`Usan`), and a hash an asset (`Asset`), a swap (`Swap`) or a live ticket (`Ticket`).

Each entry of `matches` carries:

| Field | Description |
| --- | --- |
| `address` | The token, collection or labelled address, or the address a name points at; `null` for other kinds. |
| `name` | The token's or collection's name, the label, or the name searched for; `null` for other kinds and for a token that answered no name. |
| `symbol` | The token's or collection's symbol; `null` otherwise. |
| `label` | The curated label of a token or collection, if it has one; `null` otherwise. |
| `isVerified` | Whether a token's or collection's source is verified; `null` for other kinds. |

Beside `matches`, the response carries:

| Field | Description |
| --- | --- |
| `tokensWithSymbol` | For a non-empty `Unrecognised` string, how many fungible tokens have it as their symbol, ignoring case, as a `count` capped at 100,000 (`capped` is then `true`); `null` for an empty string and any other kind. More than one means the symbol is shared, and the address tells them apart. |

Token names and symbols are text their deployers chose: invisible and direction-changing characters in them are sent as escapes such as `\u202E`.

| Parameter | Description |
| --- | --- |
| `q` | A block number, an address, a 32-byte hash, the name or symbol of a token or NFT collection, the words of a curated label, or a name or number the chain resolves. |

```bash
curl "https://fsnex.net/api/v1/search?q=0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe"
```

Response `200`

```json
{
  "query": "0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
  "queryKind": "Address",
  "matches": [
    {
      "kind": "Address",
      "path": "/address/0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe",
      "address": null,
      "name": null,
      "symbol": null,
      "label": null,
      "isVerified": null
    }
  ],
  "tokensWithSymbol": null
}
```

A token's symbol, which 1,343 tokens on BNB Smart Chain share:

```bash
curl "https://bnb.blockgaze.com/api/v1/search?q=usdt"
```

Response `200` (trimmed: `matches` shows the first 2 of 10 tokens and the one collection)

```json
{
  "query": "usdt",
  "queryKind": "Unrecognised",
  "matches": [
    {
      "kind": "Token",
      "path": "/token/0x55d398326f99059ff775485246999027b3197955",
      "address": "0x55d398326f99059ff775485246999027b3197955",
      "name": "Tether USD",
      "symbol": "USDT",
      "label": "Tether USD",
      "isVerified": false
    },
    {
      "kind": "Token",
      "path": "/token/0xa2dca18d394a3564755aef11822d939e81845b9b",
      "address": "0xa2dca18d394a3564755aef11822d939e81845b9b",
      "name": "Tether USD",
      "symbol": "USDT",
      "label": null,
      "isVerified": true
    },
    {
      "kind": "NftCollection",
      "path": "/token/0xb2acb4f32dcb01ff77e9e93eb9195ded78c97c64",
      "address": "0xb2acb4f32dcb01ff77e9e93eb9195ded78c97c64",
      "name": "kNFT: Locked kUSDT",
      "symbol": "kNFT-USDT",
      "label": null,
      "isVerified": false
    }
  ],
  "tokensWithSymbol": {
    "count": 1343,
    "capped": false
  }
}
```

The words of a curated label:

```bash
curl "https://bnb.blockgaze.com/api/v1/search?q=pancakeswap%20router"
```

Response `200`

```json
{
  "query": "pancakeswap router",
  "queryKind": "Unrecognised",
  "matches": [
    {
      "kind": "Label",
      "path": "/address/0x10ed43c718714eb63d5aa57b78b54704e256024e",
      "address": "0x10ed43c718714eb63d5aa57b78b54704e256024e",
      "name": "PancakeSwap: Router v2",
      "symbol": null,
      "label": null,
      "isVerified": null
    }
  ],
  "tokensWithSymbol": {
    "count": 0,
    "capped": false
  }
}
```

## Market data

`GET /v1/market`

BNB Smart Chain only (`features.marketData`). The native currency's latest USD price, 24-hour and 7-day change, supply, market capitalisation, and price history for the last 24 hours and 7 days. Returns `404` until a first quote has been recorded.

```bash
curl "https://bnb.blockgaze.com/api/v1/market"
```

Response `200` (trimmed: `priceHistory24h` shows 2 of 50; `priceHistory7d` shows 2 of 49)

```json
{
  "asOf": 1792178501,
  "priceUsd": 721.1290964659158,
  "percentChange24h": -1.4600000381469727,
  "percentChange7d": -2.440000057220459,
  "totalSupply": 133158414,
  "circulatingSupply": 133158414,
  "marketCapUsd": 96024406775,
  "circulatingMarketCapUsd": 96024408938,
  "priceHistory24h": [
    {
      "timestamp": 1792092393,
      "priceUsd": 733.5098425803986
    },
    {
      "timestamp": 1792094193,
      "priceUsd": 732.7601335437882
    }
  ],
  "priceHistory7d": [
    {
      "timestamp": 1791585469,
      "priceUsd": 741.2843908215021
    },
    {
      "timestamp": 1791598069,
      "priceUsd": 745.4034117889393
    }
  ]
}
```

## Staking total

`GET /v1/staking`

BNB Smart Chain only (`features.stakeHubStaking`). The total BNB staked with validators through the StakeHub contract, in wei, and its share of the total supply as a percentage.

```bash
curl "https://bnb.blockgaze.com/api/v1/staking"
```

Response `200`

```json
{
  "asOfBlock": 128033507,
  "timestamp": 1792178521,
  "totalStaked": "24148425195650580952256259",
  "percentOfTotalSupply": 18.135110110278124
}
```
