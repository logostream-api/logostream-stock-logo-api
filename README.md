# Stock Logo API — Company, Bank & Crypto Logos by ISIN, WKN, Ticker or BIC

**[logostream](https://logostream.dev)** is a REST API that returns a company logo
for a financial identifier. Look up **380,000+ logos** by **ISIN**, **WKN**, **ticker
symbol**, **BIC/SWIFT**, crypto symbol, commodity or futures contract, country code
or currency code — SVG on every plan, raster on demand.

Built for fintech: brokerage apps, portfolio trackers, trading platforms, banking
dashboards and research tools.

- 🌐 **Website:** [logostream.dev](https://logostream.dev)
- 📚 **Full documentation:** [docs.logostream.dev](https://docs.logostream.dev)
- 💰 **Pricing & free key:** [logostream.dev/pricing](https://logostream.dev/pricing)
- 📡 **Status:** [logostream.dev/status](https://logostream.dev/status)

> This repository is the public API reference. There is no package to install —
> logostream is a plain REST API you call with an `<img>` tag or any HTTP client.

---

## Quick start

```bash
curl -H "X-API-Key: YOUR_API_KEY" \
  "https://api.logostream.dev/stocks/isin/US0378331005?format=png&size=256" \
  -o apple.png
```

Or straight in HTML:

```html
<img src="https://api.logostream.dev/stocks/isin/US0378331005?key=YOUR_API_KEY"
     alt="Apple Inc. logo" width="64" height="64">
```

Get a key at [logostream.dev/pricing](https://logostream.dev/pricing).

---

## Authentication

Pass your key either as a header or as a query parameter:

| Method | Example | Use for |
|---|---|---|
| Header | `X-API-Key: YOUR_API_KEY` | Server-to-server calls |
| Query | `?key=YOUR_API_KEY` | `<img>` tags, CSS, anywhere headers are impossible |

`<img>` tags cannot send custom headers, which is why the query form exists. Treat a
key used that way as public to your site's visitors, and keep a separate
server-side key for backend calls.

---

## Endpoints

### Stock logos — by ISIN, WKN or ticker

```
GET /stocks/isin/{isin}
GET /stocks/wkn/{wkn}
GET /stocks/symbol/{symbol}
```

```bash
curl "https://api.logostream.dev/stocks/isin/US0378331005?key=KEY"   # Apple
curl "https://api.logostream.dev/stocks/wkn/865985?key=KEY"          # Apple, German WKN
curl "https://api.logostream.dev/stocks/symbol/AAPL?key=KEY"         # Apple, ticker
```

ISIN and WKN are the precise identifiers. Ticker symbols are ambiguous across
exchanges — prefer ISIN where you have it.

### Bank logos — by BIC/SWIFT or German BLZ

```
GET /banks/bic/{bic}
GET /banks/blz/{blz}
```

```bash
curl "https://api.logostream.dev/banks/bic/DEUTDEFF?key=KEY"         # Deutsche Bank
curl "https://api.logostream.dev/banks/blz/10070000?key=KEY"         # by Bankleitzahl
```

### Crypto logos

```
GET /cryptos/{symbol}
```

```bash
curl "https://api.logostream.dev/cryptos/BTC?key=KEY"
```

### Commodity logos — by name, symbol or futures contract

```
GET /commodities/{identifier}
```

Accepts four kinds of identifier, resolved in that order: the canonical slug, a
CFD symbol as brokers display it, a futures root, and any contract form built
from that root.

```bash
# Canonical name
curl "https://api.logostream.dev/commodities/gold?key=KEY"

# CFD symbol
curl "https://api.logostream.dev/commodities/SOYBEAN?key=KEY"

# Futures root
curl "https://api.logostream.dev/commodities/BGI?key=KEY"

# Contract forms — all resolve to the same root
curl "https://api.logostream.dev/commodities/BGIV26?key=KEY"   # Oct 2026
curl "https://api.logostream.dev/commodities/BGI1%21?key=KEY"  # continuation
```

Contract forms are resolved structurally, not from a lookup table, so new
delivery months work the day they list. `BGIV26`, `BGIV2026`, `BGI1!`, `BGIc1`
and `BGI=F` all resolve to `BGI`.

#### Ambiguous roots — the `mic` parameter

Eighteen root symbols mean different commodities on different exchanges. Add
`mic` to say which exchange you mean:

```bash
curl "https://api.logostream.dev/commodities/ZS?mic=CBOT&key=KEY"   # soybeans
curl "https://api.logostream.dev/commodities/ZS?mic=LME&key=KEY"    # zinc
```

Without `mic`, those eighteen roots split into two groups:

**Thirteen return the placeholder** rather than guess — `BR`, `CB`, `CJ`, `CU`,
`CY`, `EN`, `GF`, `HC`, `MA`, `RT`, `SA`, `SF`, `SR`. A wrong logo is worse
than no logo.

```bash
curl "https://api.logostream.dev/commodities/BR?key=KEY"            # placeholder
curl "https://api.logostream.dev/commodities/BR?mic=RUS&key=KEY"    # Brent crude
curl "https://api.logostream.dev/commodities/BR?mic=SHFE&key=KEY"   # butadiene rubber
```

**Five have a curated default** — the contract the symbol is commonly
understood to mean — and `mic` overrides it: `PL` (platinum), `RB` (gasoline),
`SI` (silver), `ZC` (corn), `ZS` (soybeans).

```bash
curl "https://api.logostream.dev/commodities/SI?key=KEY"            # silver (COMEX)
curl "https://api.logostream.dev/commodities/SI?mic=LME&key=KEY"    # steel
```

`mic` takes the exchange code as TradingView spells it (`LME`, `CBOT`, `NYMEX`,
`ZCE`, `SHFE`), not an ISO 10383 MIC. An unknown exchange returns the
placeholder; it does not fall back to the bare lookup.

### Country flags

```
GET /country/{code}
```

```bash
curl "https://api.logostream.dev/country/DE?key=KEY"
curl "https://api.logostream.dev/country/US?key=KEY&aspect=4x3"
```

### Forex / currency logos

```
GET /forex/{currencyCode}
```

```bash
curl "https://api.logostream.dev/forex/EUR?key=KEY"
```

---

## Query parameters

| Parameter | Values | Default | Applies to | Description |
|---|---|---|---|---|
| `format` | `svg`, `png`, `webp`, `jpg`, `jpeg`, `avif` | `svg` | all | Output format |
| `size` | e.g. `64`, `256`, `512` | native | raster formats | Pixel size; ignored for SVG |
| `variant` | `xs`, `logo`, `transparent` | icon | all; commodities: `transparent` only | Icon variants and full wordmark |
| `mode` | `light`, `dark`, `white`, `black` | — | banks | Light/dark-optimised bank marks |
| `aspect` | e.g. `1x1`, `4x3` | `1x1` | country | Flag aspect ratio |
| `mic` | e.g. `LME`, `CBOT`, `NYMEX` | — | commodities | Disambiguate a root that means different commodities on different exchanges |
| `fallback` | `none` | — | all | Return `404` instead of a placeholder |
| `force` | `true` | — | all | Bypass caches (slower, use sparingly) |

**`format` is a query parameter, not a file extension.**
`?format=png` works; `/stocks/isin/US0378331005.png` does not.

---

## Responses

Every successful request returns image bytes with the matching `Content-Type`.

### Response headers

| Header | Meaning |
|---|---|
| `X-Source` | Which layer served the image: `edge`, `R2`, `TwicPics` or `fallback` |
| `X-Fallback` | `1` when a placeholder was returned instead of a real logo |

### How to tell a real logo from a placeholder

When no logo exists, the API returns **HTTP 200** with a generated
initials placeholder — so **HTTP status alone is not a coverage signal.**

Two reliable ways to check:

```bash
# 1. Ask for a hard 404 instead of a placeholder
curl -o /dev/null -w "%{http_code}\n" \
  "https://api.logostream.dev/stocks/isin/XX0000000000?key=KEY&fallback=none"
# -> 404

# 2. Inspect the X-Fallback header
curl -sI "https://api.logostream.dev/stocks/isin/US0378331005?key=KEY" | grep -i x-fallback
```

### Status codes

| Code | Meaning |
|---|---|
| `200` | Image returned — check `X-Fallback` to see whether it is a real logo |
| `400` | Missing or malformed parameters |
| `401` / `403` | Missing or invalid API key |
| `404` | No logo, and `fallback=none` was requested |
| `429` | Rate limit or plan quota exceeded |
| `503` | A backend lookup failed — availability is **unknown**, retry rather than treating it as "no logo" |

---

## Code examples

### JavaScript (Node.js)

```js
const res = await fetch(
  'https://api.logostream.dev/stocks/isin/US0378331005?format=png&size=256',
  { headers: { 'X-API-Key': process.env.LOGOSTREAM_API_KEY } }
);

if (res.headers.get('X-Fallback') === '1') {
  console.log('placeholder, no real logo for this ISIN');
}

const buffer = Buffer.from(await res.arrayBuffer());
```

### Python

```python
import os, requests

r = requests.get(
    "https://api.logostream.dev/stocks/isin/US0378331005",
    params={"format": "png", "size": 256},
    headers={"X-API-Key": os.environ["LOGOSTREAM_API_KEY"]},
    timeout=10,
)
r.raise_for_status()

if r.headers.get("X-Fallback") == "1":
    print("placeholder, no real logo for this ISIN")

open("apple.png", "wb").write(r.content)
```

### React

```jsx
const KEY = process.env.NEXT_PUBLIC_LOGOSTREAM_KEY;

function StockLogo({ isin, size = 48, alt }) {
  return (
    <img
      src={`https://api.logostream.dev/stocks/isin/${isin}?key=${KEY}&format=png&size=${size * 2}`}
      width={size}
      height={size}
      alt={alt}
      loading="lazy"
    />
  );
}
```

### cURL — batch coverage check

```bash
while read -r isin; do
  code=$(curl -s -o /dev/null -w '%{http_code}' \
    "https://api.logostream.dev/stocks/isin/$isin?key=$KEY&fallback=none")
  echo "$isin $code"
done < isins.txt
```

---

## Coverage

| Category | Coverage |
|---|---|
| Stocks | 380,000+ logos, by ISIN, WKN and ticker |
| Banks | By BIC/SWIFT and German BLZ, light and dark variants |
| Crypto | Major coins and tokens by symbol |
| Commodities | 90 raw materials — metals, energy, agriculture, chemicals — by name, CFD symbol or futures contract |
| Countries | Flags in multiple aspect ratios |
| Forex | Currency marks by ISO currency code |

Missing an instrument? Requests for uncovered identifiers are logged automatically
and reviewed — coverage grows from real lookups.

---

## Frequently asked questions

**Can I get a stock logo by ISIN?**
Yes — `GET /stocks/isin/{isin}` is the primary lookup. ISIN is the most precise
identifier the API accepts.

**Can I get a stock logo by WKN?**
Yes — `GET /stocks/wkn/{wkn}`. WKN support matters for German and Austrian brokers,
where WKN is still the identifier customers recognise.

**Does it return SVG?**
SVG is the default format on every plan. Raster output (PNG, WebP, JPEG, AVIF) is
generated on request via `?format=`.

**What happens when a logo does not exist?**
You get HTTP 200 with an initials placeholder. Add `?fallback=none` to get a 404
instead, or read the `X-Fallback` response header.

**Can I look up a futures contract directly?**
Yes. `GET /commodities/BGIV26` resolves the contract to its root and returns the
commodity logo. The same holds for `BGI1!`, `BGIc1`, `BGI=F` and `BGIV2026` — the
rule is structural, so future delivery months need no update on our side.

**Why does a commodity root sometimes return the placeholder?**
Because it is ambiguous across exchanges. `BR` is Brent at MOEX and butadiene
rubber at Shanghai — rather than guess, the API returns the placeholder and lets
you disambiguate with `?mic=RUS`. A wrong logo is worse than no logo. Five
well-known roots (`PL`, `RB`, `SI`, `ZC`, `ZS`) are the exception: they keep the
contract the symbol is commonly understood to mean, and `mic` overrides that.

**Is there a free tier?**
Yes — see [logostream.dev/pricing](https://logostream.dev/pricing).

**How does this compare to other logo APIs?**
Side-by-side comparisons with Clearbit, Logo.dev, Brandfetch, LogoKit, CompaniesLogo,
EODHD and others are on [logostream.dev](https://logostream.dev).

---

## Related

- **Aviation logos and data** — airline logos, airport logos, aircraft liveries,
  routes and carbon data: [airline.logostream.dev](https://airline.logostream.dev)

## License

Documentation in this repository is MIT licensed. Logo images delivered through the
API are subject to the [logostream terms of service](https://logostream.dev/terms)
and remain the property of their respective trademark holders.
