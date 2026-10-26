<p align="center"><a href="https://apify.com/DevilScrapes/google-ads-transparency"><img src="assets/banner.svg" alt="Google Ads Transparency Center Scraper &amp; API — Python example" width="100%"></a></p>

# Google Ads Transparency Center Scraper in Python: competitor ads to JSON and CSV

<p align="center">
<a href="https://apify.com/DevilScrapes/google-ads-transparency"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/google-ads-transparency-scraper-python/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/google-ads-transparency-scraper-python/actions/workflows/ci.yml/badge.svg"></a>
</p>

Google publishes every ad it serves in a public registry, the Transparency Center, and offers no way to download it. This repository is a compact, tested Python client for the **[Google Ads Transparency Center Scraper](https://apify.com/DevilScrapes/google-ads-transparency)** on Apify, which does the downloading for you. Hand it a competitor's domain such as `nike.com`, or a Google advertiser ID, and it returns a table of that advertiser's ads: which creative, in which format, where the click lands, when Google first and last showed it, and how many impressions it reports. The collection happens on Apify's servers, so your script only starts a run and reads the answer. Output lands in `results.json` and `results.csv`, ready for pandas, a spreadsheet or a warehouse.

> **Actor:** [Google Ads Transparency Center Scraper & API](https://apify.com/DevilScrapes/google-ads-transparency) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in JavaScript:** [google-ads-transparency-scraper-javascript](https://github.com/devilscrapes/google-ads-transparency-scraper-javascript)

## 🔥 What you get

- **One row per ad creative**, naming the advertiser, the creative ID, a link back to the ad, the domain it sends people to, and a format code for text, image or video.
- **Lifetime of every ad**: Unix timestamps for the first and the last impression Google logged, plus the impression count Google reports where it publishes one.
- **Domains or advertiser IDs**: look up a brand by the site its ads point to, or by the exact advertiser ID when you already have it, and mix both in one call.
- **A shared ad is saved once**: when two of your targets surface the same creative, the dataset holds one copy and the bill counts one.
- **Screening switch**: flip one boolean to get a single summary row per domain (any ads, how many, since when) for checking a long prospect list cheaply.
- **Limits you control**: a date window, a cap on total rows and a cap on result pages keep a test run small and a large pull predictable.

## 🎯 Use cases

### Rank a rival's ads by how long they survive

An ad that keeps running for months has usually earned its budget, and one that vanishes after a week usually has not. Load `results.json` into a DataFrame, subtract the first date from the last and sort. The top of the list is a reading list of what your competitor trusts.

### Prepare a client review with evidence

Before a quarterly meeting, pull the three biggest rivals in one run and save a CSV per brand. The creative links open the original ads in Google's own registry, so every slide can point at a source rather than a screenshot.

### Watch for ads that use your brand

Run your own domain on a schedule and compare advertiser names and landing domains with the list you expect. An advertiser you do not recognise sending traffic to a domain that is not yours is worth a closer look.

### Screen a prospect list for paid search activity

Turn on `screeningMode`, paste the domains you plan to contact and keep the ones with `has_ads_detected` set to true. Companies already paying for Google ads are a sharper target for an agency pitch than a cold list.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/google-ads-transparency-scraper-python.git
cd google-ads-transparency-scraper-python
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export APIFY_TOKEN=your_token_here
python main.py                      # uses input.json
python main.py my-input.json --out data --max-charge 2
```

The script starts the [Google Ads Transparency Center Scraper & API](https://apify.com/DevilScrapes/google-ads-transparency), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

This is all the integration needs. The official `apify-client` package starts the Actor, waits for the run to end and then reads the rows from the dataset the run produced.

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("DevilScrapes/google-ads-transparency").call(run_input={'searchDomains': ['nike.com'], 'region': 'anywhere', 'maxResults': 50, 'maxPages': 2, 'proxyConfiguration': {'useApifyProxy': True}})
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item)
```

## 📥 Input

`main.py` reads `input.json` and forwards it unchanged. The bundled file looks up one brand and stops at 50 ads over at most 2 result pages, so a first run costs about $0.35 (the $0.20 run fee plus 50 ads at $0.003). Add domains or `advertiserIds`, or set `dateFrom` and `dateTo`, when you want a wider pull.

```json
{
  "searchDomains": [
    "nike.com"
  ],
  "region": "anywhere",
  "maxResults": 50,
  "maxPages": 2,
  "proxyConfiguration": {
    "useApifyProxy": true
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `searchDomains` | array | no | `["nike.com"]` | — |
| `advertiserIds` | array | no | `[]` | — |
| `region` | string | no | `"anywhere"` | `anywhere`, `AE`, `AR`, `AT`, `AU`, `BE`, `BG`, `BR`, `CA`, `CH`, `CL`, `CO`, `CZ`, `DE`, `DK`, `EE`, `EG`, `ES`, `FI`, `FR`, `GB`, `GR`, `HU`, `ID`, `IE`, `IL`, `IN`, `IT`, `JP`, `KR`, `LT`, `LV`, `MX`, `MY`, `NL`, `NO`, `NZ`, `PH`, `PL`, `PT`, `RO`, `SA`, `SE`, `SG`, `SK`, `TH`, `TR`, `UA`, `US`, `VN`, `ZA` |
| `dateFrom` | string | no | — | — |
| `dateTo` | string | no | — | — |
| `maxResults` | integer | no | `1000` | — |
| `maxPages` | integer | no | `25` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true}` | — |
| `screeningMode` | boolean | no | `false` | — |

## 📤 Output

Each row is one ad creative. Dates arrive as Unix seconds exactly as Google reports them, and `region` simply echoes the label you sent: Google returns the same ads whatever region you pick. With `screeningMode` on you receive one summary row per domain instead.

| Field | Type | Always present |
|---|---|---|
| `advertiser_id` | string | no |
| `advertiser_name` | string | no |
| `creative_id` | string | no |
| `creative_url` | string | no |
| `landing_domain` | string | no |
| `format_type` | integer or null | no |
| `first_shown_ts` | integer or null | no |
| `last_shown_ts` | integer or null | no |
| `impressions` | integer or null | no |
| `preview_image_url` | string or null | no |
| `preview_content_js_url` | string or null | no |
| `region` | string | no |
| `scraped_at` | string | no |
| `domain` | string or null | no |
| `has_ads_detected` | boolean or null | no |
| `ad_count` | integer or null | no |
| `first_seen_ts` | integer or null | no |
| `last_seen_ts` | integer or null | no |
| `sample_creative_url` | string or null | no |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Ad creative scraped | $0.003 |

Prices as configured on 2026-10-16. The [Actor page](https://apify.com/DevilScrapes/google-ads-transparency) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in Python

Here is a short pandas analysis (`pip install pandas`). It turns the timestamps into dates, computes how many days each ad was live, labels the format and writes the 25 longest-running ads to a CSV for review.

```python
import json

import pandas as pd

ads = pd.DataFrame(json.load(open("results.json", encoding="utf-8")))
for column in ("first_shown_ts", "last_shown_ts"):
    ads[column] = pd.to_datetime(ads[column], unit="s", utc=True)

ads["days_live"] = (ads["last_shown_ts"] - ads["first_shown_ts"]).dt.days
ads["format"] = ads["format_type"].map({1: "text", 2: "image", 3: "video"})

longest = ads.sort_values("days_live", ascending=False).head(25)
columns = ["advertiser_name", "format", "days_live", "impressions", "landing_domain", "creative_url"]
longest[columns].to_csv("longest_running.csv", index=False)
print(longest[columns[:4]].to_string(index=False))
```

## ❓ FAQ

### Does Google offer an official API for this registry?

Google does not publish one for the Transparency Center. The Actor fills that gap: you call it with an Apify token and receive structured rows, with no Google account, developer key or approval step.

### Why does a big advertiser stop at roughly 1,000 ads?

Google stops answering a single query after about 1,000 ads. To reach further, split the pull by date window with `dateFrom` and `dateTo`, then merge the files on `creative_id`.

### Can I restrict the results to one country?

Not today. The `region` input is stored on each row as a label so you can organise exports by market, but Google returns the same set of ads for every value. Treat it as bookkeeping, not as a filter.

### What does a video ad look like in the output?

Video and animated ads have no still image. They carry `preview_content_js_url`, a link to the script bundle Google uses to draw the ad, while static image ads carry `preview_image_url`. Showing the video frame itself needs a browser on your side.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/google-ads-transparency) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [Google Ads Transparency Center Scraper & API on Apify Store](https://apify.com/DevilScrapes/google-ads-transparency)
- [JavaScript version of this example](https://github.com/devilscrapes/google-ads-transparency-scraper-javascript)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Meta Ad Library Scraper — Ad Search & Export](https://apify.com/DevilScrapes/meta-ad-library-scraper) — Facebook and Instagram ads, found by advertiser or keyword.
- [TikTok Ad Library Scraper](https://apify.com/DevilScrapes/tiktok-ad-library-scraper) — commercial content from TikTok's ad library.
- [Pinterest Ad Library Scraper — Ad Search & Export](https://apify.com/DevilScrapes/pinterest-ad-library-scraper) — Pinterest ads grouped by advertiser.
- [Bing Ad Library Scraper — Ad Search & Export](https://apify.com/DevilScrapes/bing-ad-library-scraper) — the Microsoft Advertising ad library.
- [Apple Ad Repository Scraper](https://apify.com/DevilScrapes/apple-ad-repository-scraper) — App Store Search Ads.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/google-ads-transparency) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with Google or the Google Ads Transparency Center.
