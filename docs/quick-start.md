# API quick-start

[Back to overview](../README.md)

Read a Fusion archive block, fetch the newest BNB blocks and follow a pagination cursor. These public APIs require no account or API key. The examples use read-only requests.

## Choose a deployment

| Deployment | API base URL | Coverage |
| --- | --- | --- |
| FSNEX (Fusion) | `https://fsnex.net/api` | Recovered history from block 0 through 15,130,080 |
| BNB Smart Chain | `https://bnb.blockgaze.com/api` | A rolling window of recent blocks |

Use FSNEX for a repeatable historical example. On BNB, request recent records instead of copying an old block number: records leave the retention window over time. Check the current indexed range before querying history:

```bash
curl --fail-with-body --silent --show-error "https://fsnex.net/api/v1/network"
curl --fail-with-body --silent --show-error "https://bnb.blockgaze.com/api/v1/network"
```

The commands use curl. In Windows PowerShell, use `curl.exe` to avoid its `curl` alias, or use the PowerShell example below.

Look for `chainId`, `historyState`, `indexedFromBlock` and `indexedToBlock`. FSNEX reports chain ID `32659` and `HaltedComplete`; BNB reports `56` and `Live`. The [network reference](network.md#network-status) explains the other fields.

### Mainnet and testnet are separate networks

Choose the base URL for the intended deployment, then verify its `chainId` before reading records. The explorer's network picker changes the site address; it does not switch an API client's existing base URL. Do not add a `chainId` query parameter to these requests to select another network.

BNB mainnet has chain ID `56`; BNB testnet has `97`. For a testnet deployment, obtain its published API base URL and check its own `/v1/network` and OpenAPI document. This guide lists only the two public endpoints verified here, not a guaranteed public testnet service.

Features, retention, native currency and tracing coverage belong to the selected network. Do not reuse mainnet cursors on testnet, or assume a matching address or block number refers to the same records. Keep pagination state and cached records separate by deployment.

## Read a Fusion block

```bash
curl --fail-with-body --silent --show-error "https://fsnex.net/api/v1/blocks/15130080"
```

Response `200`. Selected fields from a live response verified on 18 October 2026; the full response includes additional header fields and annotations:

```json
{
  "number": 15130080,
  "hash": "0xe93ffded087a79097d4309c7831690161db6ed136f4b1a22c4c83a99db80f99f",
  "timestamp": 1759826630,
  "transactionCount": 1,
  "totalFees": "132318515417984"
}
```

The timestamp is Unix seconds. Monetary amounts such as `totalFees` are strings in the smallest currency unit. Keep them as strings or use integer/decimal arithmetic; converting them to floating-point numbers can lose precision. See [conventions](../README.md#conventions).

## Read the newest BNB block

```bash
curl --fail-with-body --silent --show-error "https://bnb.blockgaze.com/api/v1/blocks?pageSize=1"
```

Response `200`. Selected fields from a live response verified on 18 October 2026. Your block number, hash, cursor and indexed range will differ:

```json
{
  "blocks": [
    {
      "number": 128322159,
      "hash": "0x2e8709cfee56713d82095cea2398d5f14310f521136abaf08a8e9deeeac0010e",
      "timestamp": 1792308432,
      "transactionCount": 88
    }
  ],
  "nextCursor": "128322159",
  "previousCursor": null,
  "total": null,
  "indexedFromBlock": 127920000,
  "indexedToBlock": 128322159
}
```

The list is in newest-first order. Use a returned block's `number` or `hash` with `/v1/blocks/{numberOrHash}` to get its details while it remains indexed. A null `total` means no count was requested; it does not mean the list is empty.

## Follow the next page with Python

Requires Python 3; no extra packages. Save this as `quick_start.py` and run `python quick_start.py` (or `py quick_start.py` on Windows).

The script checks the network, then makes two sequential requests of two blocks each, prints their numbers and follows the server's cursor. Change `base_url` to `https://fsnex.net/api` and `expected_chain_id` to `32659` to run the same example against Fusion. For testnet, use its published base URL and expected chain ID `97`.

```python
import json
from urllib.parse import urlencode
from urllib.request import urlopen

base_url = "https://bnb.blockgaze.com/api"
expected_chain_id = 56

with urlopen(f"{base_url}/v1/network", timeout=30) as response:
    network = json.load(response)
if network["chainId"] != expected_chain_id:
    raise SystemExit(f"Wrong network: expected {expected_chain_id}, got {network['chainId']}")

print(network["chainName"], network["indexedFromBlock"], network["indexedToBlock"])
cursor = None

for _ in range(2):
    parameters = {"pageSize": 2}
    if cursor is not None:
        parameters["cursor"] = cursor

    url = f"{base_url}/v1/blocks?{urlencode(parameters)}"
    with urlopen(url, timeout=30) as response:
        page = json.load(response)

    for block in page["blocks"]:
        print(block["number"], block["hash"])

    cursor = page["nextCursor"]
    if cursor is None:
        break
```

Treat cursors as opaque: pass the returned value unchanged and URL-encode it. Do not construct a cursor from a block number just because this example's cursor looks like one. Stop when `nextCursor` is null. See [pagination](../README.md#pagination) for other navigation options.

This short script stops on HTTP or network errors. In a longer-running client, handle `429` with a bounded retry policy that honours `Retry-After`, and avoid tight retry loops. It does not attempt to download the entire chain.

## PowerShell alternative

```powershell
$block = Invoke-RestMethod -Uri "https://fsnex.net/api/v1/blocks/15130080"
$block | Select-Object number, hash, timestamp, transactionCount
```

## Coverage and common errors

- **Browser access:** ordinary API endpoints do not allow cross-origin calls from another website. Run these examples in a terminal or on your server. Use the deployment's Swagger UI for browser-based exploration.
- **Missing records:** `404` means the record is not indexed here. On BNB, it may be outside the retained window; it does not prove the transaction never happened on-chain.
- **Invalid requests:** `400` includes problem details explaining malformed parameters. Do not combine cursor navigation with `page`, `last` or `from`.
- **Rate limits:** on `429`, honour `Retry-After` when present. If absent, wait before retrying. See the current [rate limits](../README.md#rate-limits).
- **Internal transactions:** Fusion block coverage does not imply complete tracing coverage. The network response reports `tracedFromBlock` and `tracedToBlock`; internal-transaction responses report their own `coveredFrom` and `coveredThrough`. BNB does not trace internal transactions. See [internal transactions](addresses.md#internal-transactions).
- **Chain-specific features:** check `features` in the network response before using Fusion or other deployment-specific endpoints.

## Continue exploring

- [FSNEX Swagger UI](https://fsnex.net/api/swagger/) and [BNB Swagger UI](https://bnb.blockgaze.com/api/swagger/): inspect schemas and try requests.
- [Blocks and transactions](blocks-and-transactions.md): look up a transaction or filter transaction lists.
- [Addresses](addresses.md): balances, transfers and CSV export.
- [Etherscan-compatible queries](etherscan-compatible.md): a separate request format served at the API base URL.
