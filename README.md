# BlockGaze blockchain data API

BlockGaze indexes EVM chains into a database and serves the indexed records through the explorer and through a read-only HTTP API. The API returns the same blocks, transactions, addresses, tokens, trades and statistics that the explorer pages show, as JSON.

This repository documents the API as it runs on the two public BlockGaze deployments. Every example response in these pages was captured from a live deployment on 16 October 2026, except those that show address tags, which were captured on 17 October 2026.

## Deployments and base URLs

| Deployment | Chain | Chain ID | Explorer | API base URL | Indexed history |
| --- | --- | --- | --- | --- | --- |
| FSNEX | Fusion | 32659 | [fsnex.net](https://fsnex.net) | `https://fsnex.net/api` | Complete, from genesis (block 0) to the chain's final block, 15,130,080 |
| BNB Smart Chain | BNB Smart Chain | 56 | [bnb.blockgaze.com](https://bnb.blockgaze.com) | `https://bnb.blockgaze.com/api` | Rolling window of the most recent 400,000 blocks |

Both deployments run the same API. The JSON endpoints live under `/v1` on the base URL, for example `https://fsnex.net/api/v1/blocks`. The base URL itself answers [Etherscan-style queries](docs/etherscan-compatible.md), for example `https://fsnex.net/api?module=account&action=balance&address=…`.

### Retention differs per deployment

- **FSNEX is fully indexed.** The Fusion chain stopped producing blocks at block 15,130,080, and every block from genesis to that block is indexed. `GET /v1/network` reports `"historyState": "HaltedComplete"`. Internal transactions are traced separately, and that trace is still being extended through the chain's history. Each internal-transaction response states the block range it covers in `coveredFrom` and `coveredThrough`.
- **BNB Smart Chain keeps a rolling window.** The deployment keeps the most recent 400,000 blocks, about two days at the chain's current block time. Older blocks, transactions and transfers are removed as new blocks arrive, and lookups for them return `404`. Read `indexedFromBlock` from [`GET /v1/network`](docs/network.md#network-status) to see where the window starts. Daily statistics are kept for longer than the raw blocks, from `statisticsFromTimestamp`. Internal transactions are not traced on this deployment.

### Chain-specific endpoints

Some endpoints exist only on the chain they describe. `GET /v1/network` lists which feature sets a deployment serves under `features`.

| Feature | FSNEX | BNB Smart Chain | Endpoints |
| --- | --- | --- | --- |
| Fusion assets, swaps, tickets, USANs, supply | Yes | No | [Fusion](docs/fusion.md) |
| Stake distribution and staked history | Yes | Yes | [`/v1/stake-distribution`](docs/network.md#stake-distribution), [`/v1/statistics/staked`](docs/network.md#daily-staked-amount) |
| Market data (native currency price) | No | Yes | [`/v1/market`](docs/network.md#market-data) |
| StakeHub staking total | No | Yes | [`/v1/staking`](docs/network.md#staking-total) |
| SPACE ID names | No | Yes | [`/v1/addresses/{address}/name`](docs/addresses.md#primary-name) |

## Quick start

No API key or account is needed. Send a `GET` request and read the JSON:

```bash
curl "https://fsnex.net/api/v1/blocks/15130080"
```

```bash
curl "https://bnb.blockgaze.com/api/v1/transactions?pageSize=10"
```

The API does not accept cross-origin requests from other websites, so a browser page on your own domain cannot call it directly. Call it from a server, a script or a command-line tool. The one exception is the OpenAPI document below, which any website may read.

## OpenAPI documents

Each deployment generates an OpenAPI 3.1 document from its own endpoints, so it always matches what that deployment serves, including its chain-specific endpoints, and serves it as a Swagger UI page where you can send each request from your browser with "Try it out":

| Deployment | Try the API | OpenAPI document |
| --- | --- | --- |
| FSNEX | [`https://fsnex.net/api/swagger/`](https://fsnex.net/api/swagger/) | [`https://fsnex.net/api/openapi/v1.json`](https://fsnex.net/api/openapi/v1.json) |
| BNB Smart Chain | [`https://bnb.blockgaze.com/api/swagger/`](https://bnb.blockgaze.com/api/swagger/) | [`https://bnb.blockgaze.com/api/openapi/v1.json`](https://bnb.blockgaze.com/api/openapi/v1.json) |

Use the document to generate a client in your language. The document states every route's parameters, defaults, page-size limits, response shapes and the problem-details responses the route itself gives; any route can also answer `429` under the [rate limits](#rate-limits), or `400` when a parameter does not parse. These pages explain what the fields mean. The [Etherscan-compatible module](docs/etherscan-compatible.md) is not in the document, since it takes its action as a parameter rather than as a route.

## Endpoints

| Area | Endpoints |
| --- | --- |
| [Network and statistics](docs/network.md) | Network status, activity, sync status, gas, daily statistics, block producers, stake distribution, labels, search, market data, staking |
| [Blocks and transactions](docs/blocks-and-transactions.md) | Block list and detail, transaction list (by block, address or operation) and detail |
| [Addresses](docs/addresses.md) | Rich list, address detail, token and NFT holdings, approvals, events, internal transactions, trades, CSV export |
| [Tokens, NFTs and trades](docs/tokens-and-nfts.md) | Token list, detail, price and holders; token transfers; NFT collections and items; DEX and NFT trades |
| [Contracts](docs/contracts.md) | Bytecode, opcodes, verified functions, contract reads, verified source |
| [Fusion](docs/fusion.md) | FSNEX only: Fusion assets, native swaps, staking tickets, USANs, supply, operation statistics |
| [Etherscan-compatible module](docs/etherscan-compatible.md) | `account`, `block`, `logs` and `contract` modules in Etherscan's request and response format |

## Conventions

- **Amounts** are decimal strings in the smallest unit, so they never lose precision: wei for the native currency, and the token's base unit for tokens. Divide by `10^decimals` to get whole units. Computed values such as prices and shares are in whole units, and are named for what they hold (`usdPrice`, `share`).
- **Timestamps** are mostly Unix seconds, as a number (`"timestamp": 1759826630`) or a decimal string for Fusion time-lock and ticket bounds. A few fields are ISO 8601 strings in UTC, such as `createdAt` on Fusion assets and the indexer's own `evaluatedAt`, `observedAt` and `metadataFetchedAt`. The examples show which form each field uses.
- **Durations** are .NET time spans: `hh:mm:ss.fffffff`, with a leading `d.` for whole days (`"374.10:38:58.3176093"`).
- **Addresses and hashes** are lowercase `0x` hex in responses.
- **Labels** (`fromLabel`, `minerLabel`, `label`) are the explorer's names for known addresses, and are `null` for addresses without one. **Tags** (`fromTags`, `minerTags`, `tags`) list the curated tags that say what kind of address it is, such as `Validator`, and are empty for an address without one.
- **Days** in daily statistics are UTC calendar days. The current day is included with `"isPartial": true`.

## Pagination

List endpoints return one page at a time, newest first unless the endpoint says otherwise, with cursors for the next and previous pages:

```json
{
  "transactions": [ … ],
  "nextCursor": "15130077-0",
  "previousCursor": "15130078-0-back",
  "total": null
}
```

| Parameter | Meaning |
| --- | --- |
| `pageSize` | Items per page. The default is 25 on most lists. Larger values are reduced to the list's maximum, which is 100 on most lists. |
| `cursor` | Continue from a page: pass the `nextCursor` or `previousCursor` of the page you have. A cursor is `null` when there is no page in that direction. Treat cursors as opaque strings. |
| `last` | `true` returns the oldest page. |
| `count` | `true` adds `total`: `{ "count": …, "capped": … }`. Counting stops at 100,000, and `capped` is `true` when there are more. |
| `page` | Jump to a page number, counting from 1, within the counted records. |
| `from` | Start the page at a block number or a `yyyy-MM-dd` date. Only lists in block order accept it. |

Name at most one of `cursor`, `last`, `page` and `from`. Each endpoint lists which of these it accepts. Lists that are not in block order, such as the rich list and holder lists, do not accept `from`.

Lists that cover a whole deployment, such as `GET /v1/blocks` and `GET /v1/transactions`, also return `indexedFromBlock` and `indexedToBlock`, the range of blocks the deployment holds.

## Errors

The JSON endpoints report errors as [RFC 9457 problem details](https://www.rfc-editor.org/rfc/rfc9457) with the content type `application/problem+json`:

```bash
curl "https://fsnex.net/api/v1/blocks/99999999"
```

Response `404`

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
  "title": "Block not found",
  "status": 404,
  "detail": "No indexed block matches '99999999'.",
  "traceId": "00-8d7fd470cdbf40a589b8af05a3472e58-a090b4a19616d773-00"
}
```

| Status | When |
| --- | --- |
| `400` | A parameter is malformed: an address that is not 20 bytes, a hash that is not 32 bytes, an unknown cursor, or a page request that names more than one of `cursor`, `last`, `page` and `from`. `detail` says which. |
| `404` | Nothing indexed matches the block, transaction, token or contract. On BNB Smart Chain this includes records older than the retention window. |
| `429` | The client has made more requests than the rate limit allows. See below. |
| `503` | `GET /v1/transactions` with `page` asked for a page too deep in a long list to reach quickly. Jump by block or date with `from` instead. |

A malformed page request:

```bash
curl "https://fsnex.net/api/v1/blocks?last=true&page=2"
```

Response `400`

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "Invalid page request",
  "status": 400,
  "detail": "Name at most one of cursor, last, page and from. A page number counts from 1 and must start within the counted records; from is a block number or a yyyy-MM-dd date, and only lists in block order take it.",
  "traceId": "00-075075ccd0e2765c5bbc0948eb016724-c24140c880e7bb98-00"
}
```

The Etherscan-compatible module answers in Etherscan's own envelope instead, with HTTP `200` and `"status": "0"` on errors. See [its page](docs/etherscan-compatible.md#errors).

## Rate limits

Limits apply per client IP address, and per `/64` prefix for IPv6:

- Each client has a bucket of 120 requests, refilled at 10 requests per second. A client can burst to 120 requests and then sustain 10 per second.
- At most 4 of a client's requests run at once. Further requests wait in a queue rather than fail.
- Calls to the [Etherscan-compatible module](docs/etherscan-compatible.md) at `/api` run at most 4 at a time for the whole deployment, whoever sends them, with a short queue behind them. Send them one at a time rather than in parallel.
- [`GET /v1/export`](docs/addresses.md#csv-export) is limited to 10 requests per 10 minutes per client.
- [Contract reads](docs/contracts.md#read-a-contract-function) are limited to 30 per minute per client, and 300 per minute for the whole deployment.

A request over a limit gets `429 Too Many Requests` with a problem-details body whose `title` is `Too many requests`. When the limit knows how long to wait, the response carries a `Retry-After` header in seconds and the `detail` names the same wait. Wait at least that long before retrying. A request refused because the deployment is too busy to queue it carries no `Retry-After`; wait a few seconds and retry.

## Caching

Responses for recent data are cached on the server for a few seconds, so repeated requests within that time return the same data: about 2 seconds for the newest lists and network status, and about 15 seconds for activity, gas and daily statistics. Block and transaction detail for blocks at least 256 blocks below the newest indexed block is final, and is served with `Cache-Control: public, max-age=300`.

## Support

Questions about the API, or about running BlockGaze for your own chain: [info@blockgaze.com](mailto:info@blockgaze.com).
