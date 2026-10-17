# Fusion

[Back to overview](../README.md)

These endpoints are served by FSNEX (`https://fsnex.net/api`) only. They cover Fusion's native protocol features, which live outside the EVM: native assets, time locks, swaps, staking tickets and USANs (short account numbers). Fusion native calls also appear as ordinary transactions to `0xffffffffffffffffffffffffffffffffffffffff`, and [`GET /v1/transactions?operation=…`](blocks-and-transactions.md#list-transactions) filters them by kind.

The native FSN currency is asset `0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff`. Time-lock bounds (`timeLockStart`, `timeLockEnd`, `start`, `end`) are Unix seconds as decimal strings, and `18446744073709551615` means forever.

- [List assets](#list-assets)
- [Get an asset](#get-an-asset)
- [Asset transfers](#asset-transfers)
- [Asset holders](#asset-holders)
- [Address asset transfers](#address-asset-transfers)
- [Address holdings](#address-holdings)
- [Swap trades](#swap-trades)
- [Swap offers](#swap-offers)
- [Get a swap](#get-a-swap)
- [Address swaps](#address-swaps)
- [Staking summary](#staking-summary)
- [Address tickets](#address-tickets)
- [Get a ticket](#get-a-ticket)
- [Get a USAN](#get-a-usan)
- [Supply](#supply)
- [Daily native operations](#daily-native-operations)
- [Daily FSN transfers](#daily-fsn-transfers)

## List assets

`GET /v1/fusion/assets`

Fusion native assets, most recently created first, with each asset's definition and current supply.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Assets per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/fusion/assets?pageSize=2"
```

Response `200` (trimmed: `assets.definition.description` cut from 305 characters)

```json
{
  "assets": [
    {
      "definition": {
        "assetId": "0x2e6018f85f10bbd1210b79b4835d5739555c3f43834a772b8c960a3c5c2c2536",
        "standing": "Unbacked",
        "name": "XCHNG",
        "symbol": "CHNG",
        "decimals": 18,
        "initialSupply": "355473970000000000000000000",
        "canChange": false,
        "description": "0xab1f7e5bf2587543fe41f268c59d35da95f046e0",
        "creator": "0x5efe6633d8a44412238d22b943760fa36d2428bd",
        "creationBlock": 14140815,
        "creationTransaction": "0x670a27129427220ce934003669aa06c01cef522fef59f62e73dfb72a4a8ca89f",
        "createdAt": "2025-05-11T03:31:04+00:00"
      },
      "currentSupply": "355473970000000000000000000"
    },
    {
      "definition": {
        "assetId": "0xfdf5927a9fb829d43218f6e5a4fc69fb4ad6159dade86d20138977761de64ae8",
        "standing": "Unbacked",
        "name": "Dopamine NFT",
        "symbol": "DPN",
        "decimals": 0,
        "initialSupply": "1",
        "canChange": false,
        "description": "{\"IPFSPath\":\"https://dopamineapp-nft.infura-ipfs.io/ipfs/QmTB2HNYz…",
        "creator": "0xb40938427adbf19c91512dcf8d7f69b75146de6d",
        "creationBlock": 13370265,
        "creationTransaction": "0x3763b8e1a03593b06a9aedae060c84a73ce7a9f211c0bb06c63ae39607115c2e",
        "createdAt": "2025-01-14T13:59:06+00:00"
      },
      "currentSupply": "1"
    }
  ],
  "nextCursor": "13370265-2",
  "previousCursor": null,
  "total": null
}
```

## Get an asset

`GET /v1/fusion/assets/{id}`

One asset's definition and current supply. `idIsValid` is `false` for an id that is not 32 bytes of hex.

```bash
curl "https://fsnex.net/api/v1/fusion/assets/0x2e6018f85f10bbd1210b79b4835d5739555c3f43834a772b8c960a3c5c2c2536"
```

Response `200`

```json
{
  "idIsValid": true,
  "definition": {
    "assetId": "0x2e6018f85f10bbd1210b79b4835d5739555c3f43834a772b8c960a3c5c2c2536",
    "standing": "Unbacked",
    "name": "XCHNG",
    "symbol": "CHNG",
    "decimals": 18,
    "initialSupply": "355473970000000000000000000",
    "canChange": false,
    "description": "0xab1f7e5bf2587543fe41f268c59d35da95f046e0",
    "creator": "0x5efe6633d8a44412238d22b943760fa36d2428bd",
    "creationBlock": 14140815,
    "creationTransaction": "0x670a27129427220ce934003669aa06c01cef522fef59f62e73dfb72a4a8ca89f",
    "createdAt": "2025-05-11T03:31:04+00:00"
  },
  "currentSupply": "355473970000000000000000000"
}
```

## Asset transfers

`GET /v1/fusion/assets/{id}/transfers`

Movements of one asset, newest first: creation, transfers, time locks and supply changes, by `kind`.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Transfers per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/fusion/assets/0x2e6018f85f10bbd1210b79b4835d5739555c3f43834a772b8c960a3c5c2c2536/transfers?pageSize=2"
```

Response `200`

```json
{
  "transfers": [
    {
      "blockNumber": 14140815,
      "transactionIndex": 1,
      "logIndex": 1,
      "transactionHash": "0x670a27129427220ce934003669aa06c01cef522fef59f62e73dfb72a4a8ca89f",
      "timestamp": 1746934264,
      "kind": "Creation",
      "assetId": "0x2e6018f85f10bbd1210b79b4835d5739555c3f43834a772b8c960a3c5c2c2536",
      "from": null,
      "to": "0x5efe6633d8a44412238d22b943760fa36d2428bd",
      "amount": "355473970000000000000000000",
      "timeLockStart": null,
      "timeLockEnd": null
    }
  ],
  "nextCursor": null,
  "previousCursor": null,
  "total": null
}
```

## Asset holders

`GET /v1/fusion/assets/{id}/holders`

One asset's holders, largest balance first, with each holder's `holderLabel` and `holderTags` and shares of the supply. Tags are those [Get an address](addresses.md#get-an-address) lists, in the same order and shape; an untagged address has an empty list.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Holders per page, at most 100. |
| `cursor`, `last`, `count`, `page` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/fusion/assets/0x2e6018f85f10bbd1210b79b4835d5739555c3f43834a772b8c960a3c5c2c2536/holders?pageSize=2"
```

Response `200`

```json
{
  "holders": [
    {
      "holder": "0x5efe6633d8a44412238d22b943760fa36d2428bd",
      "holderLabel": null,
      "balance": "355473970000000000000000000",
      "asOfBlock": 15130080
    }
  ],
  "shares": {
    "basis": 4,
    "asOf": 15130080,
    "shares": [
      1
    ]
  },
  "nextCursor": null,
  "previousCursor": null,
  "total": null
}
```

## Address asset transfers

`GET /v1/fusion/addresses/{address}/asset-transfers`

Asset movements to or from one address, newest first, across every asset, with `direction` relative to the address.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Transfers per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/fusion/addresses/0x0aef49b7935a789003ba9c20c1e23a34d9435734/asset-transfers?pageSize=2"
```

Response `200`

```json
{
  "transfers": [
    {
      "transfer": {
        "blockNumber": 15129933,
        "transactionIndex": 0,
        "logIndex": 1,
        "transactionHash": "0xbd566d7b2c8e4c93182fbeb0049b1c19165a75b62f3d0070eda54d0bfbdc2e1f",
        "timestamp": 1759824717,
        "kind": "TimeLockToAsset",
        "assetId": "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
        "from": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
        "to": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
        "amount": "5000000000000000000000",
        "timeLockStart": "1759824704",
        "timeLockEnd": "18446744073709551615"
      },
      "assetSymbol": "FSN",
      "assetDecimals": 18,
      "direction": "Self"
    },
    {
      "transfer": {
        "blockNumber": 15129039,
        "transactionIndex": 0,
        "logIndex": 1,
        "transactionHash": "0x356b8c5e9e14a72cfc99a21c0c169aae721d48c498ea6b63ea140570c3724898",
        "timestamp": 1759813093,
        "kind": "TimeLockToAsset",
        "assetId": "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
        "from": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
        "to": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
        "amount": "5000000000000000000000",
        "timeLockStart": "1759813080",
        "timeLockEnd": "18446744073709551615"
      },
      "assetSymbol": "FSN",
      "assetDecimals": 18,
      "direction": "Self"
    }
  ],
  "nextCursor": "15129039-0-1",
  "previousCursor": null,
  "total": null
}
```

## Address holdings

`GET /v1/fusion/addresses/{address}/holdings`

Every asset an address holds at the final block, each with its liquid balance and its time locks, and the address's USAN in `notation`.

```bash
curl "https://fsnex.net/api/v1/fusion/addresses/0x0aef49b7935a789003ba9c20c1e23a34d9435734/holdings"
```

Response `200` (trimmed: `assets` shows 2 of 3)

```json
{
  "addressIsValid": true,
  "asOfBlock": 15130080,
  "blockHash": "0xe93ffded087a79097d4309c7831690161db6ed136f4b1a22c4c83a99db80f99f",
  "observedAt": "2026-09-30T20:36:44.985093+00:00",
  "evaluatedAt": "2025-10-07T08:43:50+00:00",
  "notation": "985978",
  "assets": [
    {
      "assetId": "0x991cf749f291755829f74ef756778bb1087396042595ee7ed114450448316921",
      "standing": "Unbacked",
      "name": "3019scammer",
      "symbol": "SCAM",
      "decimals": 18,
      "liquidBalance": "0",
      "timeLocks": []
    },
    {
      "assetId": "0xf4798cc45b0fe0c6a27255745f6977166619bf9615d5d25a0f086e6fb24756aa",
      "standing": "Unbacked",
      "name": "Vote1",
      "symbol": "Vote1",
      "decimals": 18,
      "liquidBalance": "1000000000000000000",
      "timeLocks": []
    }
  ]
}
```

## Swap trades

`GET /v1/fusion/swaps/trades`

Takes of Fusion native swaps, newest first: who made and took the swap, how many parts were taken, and what each side gave.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Trades per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/fusion/swaps/trades?pageSize=2"
```

Response `200`

```json
{
  "trades": [
    {
      "blockNumber": 13630056,
      "transactionIndex": 1,
      "transactionHash": "0xa6705b3154472bce55caac7d3033657b9d7248668312761755f7da061adbbaeb",
      "timestamp": 1740254022,
      "swapId": "0xc3bea168ff2933a9b89bd82a5ce2f3b5efcc5458184b30483a56de261cf31b02",
      "taker": "0xfb5801158426bb143a5ce43dba0eae4eae2f1fc9",
      "parts": "1",
      "maker": "0x8148e2dbec9881d735fe184393bfa89e9d4c11bf",
      "makerGives": [
        {
          "assetId": "0x04084ac2252d4d43a3738836e6de0952316f32e48c739d03f40ef2f55f6a3157",
          "isNotation": false,
          "standing": "Unbacked",
          "symbol": "GEO",
          "decimals": 18,
          "amount": "6076000000000000000000",
          "timeLockStart": null,
          "timeLockEnd": null
        }
      ],
      "takerGives": [
        {
          "assetId": "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
          "isNotation": false,
          "standing": "Native",
          "symbol": "FSN",
          "decimals": 18,
          "amount": "1000000000000000000",
          "timeLockStart": null,
          "timeLockEnd": null
        }
      ]
    },
    {
      "blockNumber": 13630022,
      "transactionIndex": 1,
      "transactionHash": "0xeb4f757a82eab837f41dcd3b658fd0bef4792fca71050a583f5cbc2f4afe5269",
      "timestamp": 1740253578,
      "swapId": "0xa2d286a86ddfd51034dfc9da4988fa02ede6890ccf0232a5bcc613337bae17f5",
      "taker": "0xfb5801158426bb143a5ce43dba0eae4eae2f1fc9",
      "parts": "1",
      "maker": "0xb75c85c4304bab2230a2219c93c23af52eb8137b",
      "makerGives": [
        {
          "assetId": "0x46992f33b0cce4fbee9af575cb4de6c5cb748b535670477f834780db851ed0ac",
          "isNotation": false,
          "standing": "Unbacked",
          "symbol": "DAV",
          "decimals": 18,
          "amount": "20000157063240000000000",
          "timeLockStart": null,
          "timeLockEnd": null
        }
      ],
      "takerGives": [
        {
          "assetId": "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
          "isNotation": false,
          "standing": "Native",
          "symbol": "FSN",
          "decimals": 18,
          "amount": "10000000000000000000",
          "timeLockStart": null,
          "timeLockEnd": null
        }
      ]
    }
  ],
  "nextCursor": "13630022-1",
  "previousCursor": null,
  "total": null
}
```

## Swap offers

`GET /v1/fusion/swaps/offers`

Swap offers, most recently made first, each with its status (such as `Open` or `Filled`), what it offers and requests per part, how many parts remain, and its takes.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Offers per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/fusion/swaps/offers?pageSize=2"
```

Response `200`

```json
{
  "offers": [
    {
      "swapId": "0x2c9369ad203894d5e8bfa193f1ad200b76043d0193ce8b9810915fb9544e4513",
      "status": "Open",
      "maker": "0x748e6ac25025758612507cefeed7987db2dbdd8b",
      "creationBlock": 13590598,
      "creationTransaction": "0x70fa7ac249438db6afb7d219f814b596ee4c9ec0d006c50af9425126993d59e9",
      "createdAt": 1739739593,
      "parts": "1",
      "remainingParts": "1",
      "description": null,
      "targets": [],
      "offeredPerPart": [
        {
          "assetId": "0x86fb1bcb6c623029777904a2356a628c4e513649ec7b00368892fec5489cd8d9",
          "isNotation": false,
          "standing": "Unbacked",
          "symbol": "BTC",
          "decimals": 8,
          "amount": "100000000",
          "timeLockStart": null,
          "timeLockEnd": null
        }
      ],
      "requestedPerPart": [
        {
          "assetId": "0xf4852e7415977a6438e4c397b3c5022c3a3e5cb9e399a8179b01bb41a8d94437",
          "isNotation": false,
          "standing": "Unbacked",
          "symbol": "KAS",
          "decimals": 8,
          "amount": "90000000000000",
          "timeLockStart": null,
          "timeLockEnd": null
        }
      ],
      "takes": [],
      "recall": null
    },
    {
      "swapId": "0x47b27043dcb917368c55413969bd306ac95cd007ffc545d53bcd695883307001",
      "status": "Open",
      "maker": "0x77bb1952d492fa4f90be277159a4bcd77f784918",
      "creationBlock": 12824688,
      "creationTransaction": "0x05444373da367f1717c63eeebde104e4d9a6e90d88e7760e63e8d8d3a5fd1c24",
      "createdAt": 1729728649,
      "parts": "1",
      "remainingParts": "1",
      "description": null,
      "targets": [],
      "offeredPerPart": [
        {
          "assetId": "0xf4798cc45b0fe0c6a27255745f6977166619bf9615d5d25a0f086e6fb24756aa",
          "isNotation": false,
          "standing": "Unbacked",
          "symbol": "Vote1",
          "decimals": 18,
          "amount": "1210694092000000000000",
          "timeLockStart": null,
          "timeLockEnd": null
        }
      ],
      "requestedPerPart": [
        {
          "assetId": "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
          "isNotation": false,
          "standing": "Native",
          "symbol": "FSN",
          "decimals": 18,
          "amount": "100000000000000000000",
          "timeLockStart": null,
          "timeLockEnd": null
        }
      ],
      "takes": [],
      "recall": null
    }
  ],
  "nextCursor": "12824688-0",
  "previousCursor": null,
  "total": null
}
```

## Get a swap

`GET /v1/fusion/swaps/{id}`

One swap: its status, maker, what it offered and requested per part, every take, and its recall if the maker withdrew it.

```bash
curl "https://fsnex.net/api/v1/fusion/swaps/0xc3bea168ff2933a9b89bd82a5ce2f3b5efcc5458184b30483a56de261cf31b02"
```

Response `200`

```json
{
  "idIsValid": true,
  "details": {
    "swapId": "0xc3bea168ff2933a9b89bd82a5ce2f3b5efcc5458184b30483a56de261cf31b02",
    "status": "Filled",
    "maker": "0x8148e2dbec9881d735fe184393bfa89e9d4c11bf",
    "creationBlock": 2364756,
    "creationTransaction": "0xd11e2fe6d9aee9adf7a70831e07c699a73bc204a9f24983e1f669ed46e2ac7ce",
    "createdAt": 1592829081,
    "parts": "1",
    "remainingParts": null,
    "description": null,
    "targets": [],
    "offeredPerPart": [
      {
        "assetId": "0x04084ac2252d4d43a3738836e6de0952316f32e48c739d03f40ef2f55f6a3157",
        "isNotation": false,
        "standing": "Unbacked",
        "symbol": "GEO",
        "decimals": 18,
        "amount": "6076000000000000000000",
        "timeLockStart": null,
        "timeLockEnd": null
      }
    ],
    "requestedPerPart": [
      {
        "assetId": "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
        "isNotation": false,
        "standing": "Native",
        "symbol": "FSN",
        "decimals": 18,
        "amount": "1000000000000000000",
        "timeLockStart": null,
        "timeLockEnd": null
      }
    ],
    "takes": [
      {
        "blockNumber": 13630056,
        "transactionHash": "0xa6705b3154472bce55caac7d3033657b9d7248668312761755f7da061adbbaeb",
        "timestamp": 1740254022,
        "account": "0xfb5801158426bb143a5ce43dba0eae4eae2f1fc9",
        "parts": "1"
      }
    ],
    "recall": null
  }
}
```

## Address swaps

`GET /v1/fusion/addresses/{address}/swaps`

Swap takes in which the address was maker or taker, newest first.

| Parameter | Default | Description |
| --- | --- | --- |
| `pageSize` | `25` | Trades per page, at most 100. |
| `cursor`, `last`, `count`, `page`, `from` | | See [pagination](../README.md#pagination). |

```bash
curl "https://fsnex.net/api/v1/fusion/addresses/0x8148e2dbec9881d735fe184393bfa89e9d4c11bf/swaps?pageSize=2"
```

Response `200`

```json
{
  "trades": [
    {
      "blockNumber": 13630056,
      "transactionIndex": 1,
      "transactionHash": "0xa6705b3154472bce55caac7d3033657b9d7248668312761755f7da061adbbaeb",
      "timestamp": 1740254022,
      "swapId": "0xc3bea168ff2933a9b89bd82a5ce2f3b5efcc5458184b30483a56de261cf31b02",
      "taker": "0xfb5801158426bb143a5ce43dba0eae4eae2f1fc9",
      "parts": "1",
      "maker": "0x8148e2dbec9881d735fe184393bfa89e9d4c11bf",
      "makerGives": [
        {
          "assetId": "0x04084ac2252d4d43a3738836e6de0952316f32e48c739d03f40ef2f55f6a3157",
          "isNotation": false,
          "standing": "Unbacked",
          "symbol": "GEO",
          "decimals": 18,
          "amount": "6076000000000000000000",
          "timeLockStart": null,
          "timeLockEnd": null
        }
      ],
      "takerGives": [
        {
          "assetId": "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
          "isNotation": false,
          "standing": "Native",
          "symbol": "FSN",
          "decimals": 18,
          "amount": "1000000000000000000",
          "timeLockStart": null,
          "timeLockEnd": null
        }
      ]
    },
    {
      "blockNumber": 422674,
      "transactionIndex": 1,
      "transactionHash": "0x095cf8672285ccaef5cde4896614f2106e20632dc51dba9b902c36bc0694fde0",
      "timestamp": 1567415269,
      "swapId": "0x2daf7b24f938b6a44b661f88bd42ce4f31b0f5c1b0d195bf551079133269d7ff",
      "taker": "0x8148e2dbec9881d735fe184393bfa89e9d4c11bf",
      "parts": "1",
      "maker": "0xb2a46485d73b47af2c7a62ed1868c48a557dddc0",
      "makerGives": [
        {
          "assetId": "0x1dbdb34e631f9359b5ccec99d156226e4a30fab56529915c76b3303fa16dc640",
          "isNotation": false,
          "standing": "Unbacked",
          "symbol": "POOL",
          "decimals": 18,
          "amount": "930000000000000000000",
          "timeLockStart": "1567415269",
          "timeLockEnd": "1583280000"
        }
      ],
      "takerGives": [
        {
          "assetId": "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
          "isNotation": false,
          "standing": "Native",
          "symbol": "FSN",
          "decimals": 18,
          "amount": "930000000000000000000",
          "timeLockStart": "1567415269",
          "timeLockEnd": "1583280000"
        }
      ]
    }
  ],
  "nextCursor": null,
  "previousCursor": null,
  "total": null
}
```

## Staking summary

`GET /v1/fusion/staking`

Live and valid staking tickets at the final block: how many, how many owners, the FSN they hold, and that FSN's share of the supply.

```bash
curl "https://fsnex.net/api/v1/fusion/staking"
```

Response `200`

```json
{
  "available": true,
  "projectedThroughBlock": 15130080,
  "evaluatedAt": "2025-10-07T08:43:50+00:00",
  "liveTickets": 491,
  "liveOwners": 8,
  "liveValue": "2455000000000000000000000",
  "validTickets": 491,
  "validOwners": 8,
  "validValue": "2455000000000000000000000",
  "validShare": 0.031088,
  "expiredTickets": 0,
  "futureTickets": 0
}
```

## Address tickets

`GET /v1/fusion/addresses/{address}/tickets`

One owner's staking summary and every one of its tickets, with purchase block, start, expiry and whether it is valid. This list is not paged.

```bash
curl "https://fsnex.net/api/v1/fusion/addresses/0x0aef49b7935a789003ba9c20c1e23a34d9435734/tickets"
```

Response `200` (trimmed: `tickets` shows 2 of 362)

```json
{
  "addressIsValid": true,
  "summary": {
    "available": true,
    "projectedThroughBlock": 15130080,
    "evaluatedAt": "2025-10-07T08:43:50+00:00",
    "liveTickets": 362,
    "liveOwners": 1,
    "liveValue": "1810000000000000000000000",
    "validTickets": 362,
    "validOwners": 1,
    "validValue": "1810000000000000000000000",
    "validShare": 0.02292,
    "expiredTickets": 0,
    "futureTickets": 0
  },
  "tickets": [
    {
      "id": "0x211cdb5c97630d057d3aab2f016ec094601699ddc7be38ba187858fc523cd5de",
      "purchaseBlock": 15127371,
      "start": "1759791382",
      "expiry": "1762383382",
      "valid": true
    },
    {
      "id": "0x06481b0ad1eca64a2595918d19fb63ee8776bf6ee30dafea4b23231aacdaa963",
      "purchaseBlock": 15127576,
      "start": "1759794053",
      "expiry": "1762386053",
      "valid": true
    }
  ]
}
```

## Get a ticket

`GET /v1/fusion/tickets/{id}`

One staking ticket: owner, value, purchase block, start, expiry and status. Returns `400` for an id that is not 32 bytes of hex.

```bash
curl "https://fsnex.net/api/v1/fusion/tickets/0x211cdb5c97630d057d3aab2f016ec094601699ddc7be38ba187858fc523cd5de"
```

Response `200`

```json
{
  "idIsValid": true,
  "details": {
    "id": "0x211cdb5c97630d057d3aab2f016ec094601699ddc7be38ba187858fc523cd5de",
    "owner": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
    "purchaseBlock": 15127371,
    "start": "1759791382",
    "expiry": "1762383382",
    "value": "5000000000000000000000",
    "status": "Live",
    "statusBlock": 15130080,
    "valid": true,
    "projectedThroughBlock": 15130080,
    "evaluatedAt": "2025-10-07T08:43:50+00:00"
  }
}
```

## Get a USAN

`GET /v1/fusion/usans/{number}`

A USAN (Fusion's short account number): the address that holds it and its history. Returns `400` for anything other than a positive whole number.

```bash
curl "https://fsnex.net/api/v1/fusion/usans/985978"
```

Response `200`

```json
{
  "numberIsValid": true,
  "details": {
    "number": "985978",
    "holder": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
    "events": [
      {
        "kind": "Created",
        "blockNumber": 812,
        "transactionHash": "0x540a4b48437dc3e562605ceedbcc63ed098472de21d2cabc3fa28bd289f12bdd",
        "timestamp": 1561905171,
        "from": null,
        "to": "0x0aef49b7935a789003ba9c20c1e23a34d9435734",
        "swapId": null
      }
    ]
  }
}
```

## Supply

`GET /v1/fusion/supply`

FSN supply at the final block: genesis allocation, maximum supply, issued and total supply, block issuance and the block at which it next changes, the staked share, and balances of labelled groups of addresses.

```bash
curl "https://fsnex.net/api/v1/fusion/supply"
```

Response `200` (trimmed: `labelGroups` shows 2 of 12)

```json
{
  "genesisAllocation": "57344000000000000000000000",
  "maximumSupply": "81919997499999999906611200",
  "head": {
    "block": 15130080,
    "issued": "21624147812500000000000000",
    "totalSupply": "78968147812500000000000000",
    "blockIssuance": "312500000000000000",
    "nextIssuanceChangeBlock": 19660800
  },
  "staked": {
    "block": 15130080,
    "tickets": 491,
    "value": "2455000000000000000000000",
    "totalSupply": "78968147812500000000000000",
    "share": 0.031088
  },
  "labelGroups": [
    {
      "name": "Anyswap",
      "addressCount": 31,
      "readAddressCount": 31,
      "liquid": "143855641215714524064806",
      "perpetualTimeLocked": "0",
      "total": "143855641215714524064806",
      "firstReadBlock": 15130080,
      "lastReadBlock": 15130080
    },
    {
      "name": "Burn Addresses",
      "addressCount": 4,
      "readAddressCount": 4,
      "liquid": "530929495738601235641411",
      "perpetualTimeLocked": "0",
      "total": "530929495738601235641411",
      "firstReadBlock": 15130080,
      "lastReadBlock": 15130080
    }
  ]
}
```

## Daily native operations

`GET /v1/fusion/statistics/operations`

The number of Fusion native calls per UTC day, by operation kind. `operations` lists the kinds and their names; each day has a count per kind.

| Parameter | Default | Description |
| --- | --- | --- |
| `days` | `30` | Number of days to return, ending at the chain's last day. |

```bash
curl "https://fsnex.net/api/v1/fusion/statistics/operations?days=1"
```

Response `200` (trimmed: `operations` shows 2 of 13; `days.counts` shows 2 of 13)

```json
{
  "operations": [
    {
      "key": "all",
      "name": "All native calls"
    },
    {
      "key": "all-but-tickets",
      "name": "All but ticket purchases"
    }
  ],
  "days": [
    {
      "day": "2025-10-07",
      "isPartial": true,
      "counts": [
        {
          "key": "all",
          "count": 2417
        },
        {
          "key": "all-but-tickets",
          "count": 0
        }
      ]
    }
  ]
}
```

## Daily FSN transfers

`GET /v1/fusion/statistics/fsn-transfers`

FSN moved per UTC day by native transfers, FSN time-locked forever, and the two combined, in wei.

| Parameter | Default | Description |
| --- | --- | --- |
| `days` | `30` | Number of days to return, ending at the chain's last day. |

```bash
curl "https://fsnex.net/api/v1/fusion/statistics/fsn-transfers?days=2"
```

Response `200`

```json
[
  {
    "day": "2025-10-06",
    "isPartial": false,
    "transferred": "0",
    "lockedForever": "0",
    "combined": "0"
  },
  {
    "day": "2025-10-07",
    "isPartial": true,
    "transferred": "0",
    "lockedForever": "0",
    "combined": "0"
  }
]
```
