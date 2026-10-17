# Network and statistics

[Back to overview](../README.md)

- [Network status](#network-status)
- [Network activity](#network-activity)
- [Sync status](#sync-status)
- [Gas](#gas)
- [Daily statistics](#daily-statistics)
- [Daily staked amount](#daily-staked-amount)
- [Block producers](#block-producers)
- [Block producer history](#block-producer-history)
- [Stake distribution](#stake-distribution)
- [Labels](#labels)
- [Search](#search)
- [Market data](#market-data) (BNB Smart Chain)
- [Staking total](#staking-total) (BNB Smart Chain)

## Network status

`GET /v1/network`

The chain the deployment indexes, the feature sets it serves, and the range of blocks it holds. Read this first: `indexedFromBlock` and `indexedToBlock` give the retained range, `historyState` says whether the chain is live (`Live`) or halted and complete (`HaltedComplete`), and `refresh` gives how often the explorer polls each kind of data on a live chain.

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
    "marketData": true,
    "stakeHubStaking": true,
    "spaceIdNames": true
  },
  "transactionOperationKinds": [],
  "exportContentKinds": [],
  "chainName": "BNB Smart Chain",
  "chainId": 56,
  "nativeCurrencySymbol": "BNB",
  "nativeCurrencyDecimals": 18,
  "indexedFromBlock": 127630000,
  "indexedFromTimestamp": 1791996802,
  "indexedToBlock": 128035237,
  "latestBlockTimestamp": 1792179279,
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
    "marketData": false,
    "stakeHubStaking": false,
    "spaceIdNames": false
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

`status` is `Fresh` when the newest block is recent and the required projectors are caught up, and `Delayed` otherwise. FSNEX always reports `Delayed`, because its chain has stopped.

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

## Daily staked amount

`GET /v1/statistics/staked`

The native currency staked at the end of each UTC day, in wei. Served where `features.stakedHistory` is `true`, which is both deployments.

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
    "day": "2026-10-15",
    "isPartial": false,
    "staked": "24148045016268836234634939"
  },
  {
    "day": "2026-10-16",
    "isPartial": true,
    "staked": "24148425195650580952256259"
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

Works out what a search string refers to and where the explorer shows it. `queryKind` is how the string was read: `BlockNumber`, `Address`, `TransactionHash` (any 32-byte hash) or `Unrecognised`. `matches` lists the indexed records found, each with its path on the explorer, and is empty when nothing indexed matches.

| Parameter | Description |
| --- | --- |
| `q` | A block number, an address or a 32-byte hash. |

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
      "path": "/address/0x8fdc02dc969a22c02ddffd3d8b547fab3d7702fe"
    }
  ]
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
