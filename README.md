# IQ Exchange API

Parallel-market dollar buy and sell quotes for Baghdad, northern Iraq and southern Iraq.

**[Open IQ Exchange API](https://wolfin.app/)** · [Wolfin](https://wolfin.app)

[Read the original guide](https://wolfin.app/blog?post=iq-exchange-api-guide)

---

## IQ Exchange API: Keys, Rates and Regional Quotes

IQ Exchange API gives your backend parallel-market USD/IQD buy and sell quotes for Baghdad, northern Iraq and southern Iraq through one Wolfin endpoint. Region codes are `baghdad`, `north` and `south`. The provider connection stays on our server; clients use only the Wolfin URL and their own service keys.

## 1. Create your service key pair

Open your [Wolfin dashboard](https://wolfin.app/dashboard), choose **New key**, select **IQ Exchange API**, and save the public key and secret key shown at creation. The full secret is shown once. An Account API key or Flight API key cannot access exchange rates.

Activate IQ Exchange API access and keep wallet credit available. The existing service price is **$10 per month**; each successful API request costs **$0.02** separately. A cached response is still a successful billable request. Failed authentication, invalid queries, missing access, insufficient credit and source failures are not billed. Key creation alone does not activate a subscription. Purchases and wallet top-ups depend on the current payment setup.

## 2. Request all regions

From your backend, send `GET https://api.wolfin.app/v1/exchange/rates`. Put the secret in `Authorization: Bearer …` and its matching public identifier in `X-Public-Key`. Both are required on this endpoint. The public key alone never grants access. There is no JWT exchange step for this service.

```bash
curl "https://api.wolfin.app/v1/exchange/rates" \
  -H "Authorization: Bearer $WOLFIN_EXCHANGE_SECRET_KEY" \
  -H "X-Public-Key: $WOLFIN_EXCHANGE_PUBLIC_KEY"
```

## 3. Select a region

Add `?region=baghdad`, `?region=north`, or `?region=south`. Omit the parameter or use `region=all` for all three. Each request costs the same amount, so request all regions together when you need them.

```javascript
const response = await fetch(
  "https://api.wolfin.app/v1/exchange/rates?region=baghdad",
  {
    headers: {
      Authorization: `Bearer ${process.env.WOLFIN_EXCHANGE_SECRET_KEY}`,
      "X-Public-Key": process.env.WOLFIN_EXCHANGE_PUBLIC_KEY
    },
    signal: AbortSignal.timeout(15000)
  }
);
const result = await response.json();
if (!response.ok) throw new Error(`${response.status}: ${result.error}`);
console.log(result.rates.baghdad);
console.log("Fetched:", result.meta.fetched_at);
```

## 4. Read the response correctly

Successful responses contain `market: "parallel"`, `base_currency: "USD"`, `quote_currency: "IQD"` and a `rates` object. Each region has an Arabic `name` and numeric `buy` and `sell` values. Baghdad maps to بغداد, north to الشمال, and south to الجنوب.

**Quote scale:** values are preserved from the source without multiplying or dividing them. `unit: "source_quote"` and `unit_verified: false` mean the quote scale has not been independently verified. Confirm the unit before labeling a value as per $1, per $100, or using it in a conversion. The API does not supply a central-bank official rate or an official/parallel spread.

`meta.fetched_at` is when Wolfin last obtained the snapshot, not when the provider changed a price. `meta.source_updated_at` is `null` because the source does not supply a verified update timestamp. Show the fetch time and avoid describing the result as a guaranteed live execution price.

Snapshots are cached for up to 30 seconds. `meta.cached` indicates cache use; `meta.cache_ttl_seconds` is 30. The API does not serve an expired snapshot when a refresh fails. `meta.request_cost_cents` is 2 and `meta.requests_left` estimates remaining calls at that cost.

## 5. Handle errors

<table><tbody><tr><th>HTTP</th><th>Meaning</th></tr><tr><td>400</td><td>Unsupported region or query field.</td></tr><tr><td>401</td><td>Missing, invalid, mismatched or deleted key pair.</td></tr><tr><td>402</td><td>Insufficient wallet credit.</td></tr><tr><td>403</td><td>Wrong service key, inactive purchase or a browser-origin request.</td></tr><tr><td>429</td><td>Rate limit exceeded; follow Retry-After.</td></tr><tr><td>502</td><td>Source unavailable or invalid response. No usage fee is charged.</td></tr></tbody></table>

The limit is 120 requests per minute per client IP. Keep keys in backend environment variables. Do not place the secret in frontend JavaScript, URLs, mobile bundles, screenshots or logs. Browser-origin calls are rejected and cross-origin browser access is not enabled. Replace a lost key from the dashboard and delete the old one; deletion stops future requests.

[Open the dashboard](https://wolfin.app/dashboard) to create a dedicated IQ Exchange API key pair and review your access, wallet and usage.
