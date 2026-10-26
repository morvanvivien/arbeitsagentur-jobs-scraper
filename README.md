# How to Scrape the Bundesagentur für Arbeit Jobsuche: German Jobs, Salaries, No Login

> **Run it now, no code: [arbeitsagentur-jobs-scraper on the Apify Store](https://apify.com/bibian/arbeitsagentur-jobs-scraper)** &nbsp;|&nbsp; $1.20 per 1,000 job postings &nbsp;|&nbsp; no login, no browser

The Bundesagentur für Arbeit runs Germany's official job database (Jobsuche), with more than a million active postings including German, Austrian and other EURES-partner listings. This guide shows how to get them as clean structured data: title, company, location, salary when disclosed, contract type and full description, read from the agency's own public JSON API, with a free Apify Actor, from the console or from code.

Searches this page answers: arbeitsagentur scraper, german jobs scraper, jobsuche api, bundesagentur für arbeit jobs.

## What you get from the Arbeitsagentur Jobs scraper

- Every German job posting matching your search, including EURES-partner listings (for example from Austria), newest first.
- Salary range (`salaryMin`, `salaryMax`, `salaryPeriod`) whenever the employer chose to publish one.
- Contract type, full-time/part-time, shift/weekend work and home-office possibility.
- Full description as plain text, with any named recruiter contact removed sentence by sentence.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results, personal data removed, long texts truncated) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url` | Jobsuche reference number and public link (a partner URL for EURES listings) |
| `title`, `profession` | Posting title and the agency's own profession classification |
| `company` | Hiring company as published |
| `offerType`, `contractDuration` | Offer type and contract duration code (for example `UNBEFRISTET`) |
| `fullTime`, `partTime`, `shiftOrWeekendWork`, `homeOfficePossible` | Work-arrangement flags |
| `salaryPeriod`, `salaryMin`, `salaryMax` | Salary period and range, when disclosed |
| `location` | `{ city, postalCode, region, country }` |
| `datePublished`, `dateUpdated`, `startDate` | Publication, last-update and start dates |
| `description` | Full posting text, named recruiter contacts removed |

## Use cases

- **Search Germany's official job database directly.** The same public API the arbeitsagentur.de website itself calls, with a real `page`/`size` pagination over more than a million active postings.
- **Filter by job title and place of work.** `was` and `wo` map straight onto the site's own search fields.
- **Track salary disclosure.** `salaryMin`/`salaryMax`/`salaryPeriod` carry the employer's own published figures, when they choose to publish them.
- **Feed a job alert or aggregator.** Every posting comes with a stable reference number and a public URL for deduplication across runs.

## Input

| Field | What it does |
|---|---|
| `was` | Free-text search on the job title or profession, for example `Softwareentwickler`. Empty means every posting. |
| `wo` | Free-text search on the place of work, for example `Berlin`. Empty means every location. |
| `maxItems` | The run stops once this many postings are saved (default 50). |
| `includeDescriptions` | Fetch each posting's own detail record for the full description and salary range, when disclosed (default on; one extra request per posting). |

Example input:

```json
{
  "was": "Softwareentwickler",
  "wo": "Berlin",
  "maxItems": 50,
  "includeDescriptions": true
}
```

## Four ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/arbeitsagentur-jobs-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in the job title, location and the maximum number of results, then click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor and, optionally, add an integration that sends new results to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/arbeitsagentur-jobs-scraper').call({
    "was": "Softwareentwickler",
    "wo": "Berlin",
    "maxItems": 50,
    "includeDescriptions": true
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.company} | ${item.location?.city ?? '-'} | ${item.salaryMin ?? '-'}-${item.salaryMax ?? '-'} ${item.salaryPeriod ?? ''}`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/arbeitsagentur-jobs-scraper").call(run_input={
    "was": "Softwareentwickler",
    "wo": "Berlin",
    "maxItems": 50,
    "includeDescriptions": True,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    location = (item.get("location") or {}).get("city")
    print(f"{item['title']} | {item.get('company')} | {location} | {item.get('salaryMin')}-{item.get('salaryMax')} {item.get('salaryPeriod') or ''}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

### 4. REST API on RapidAPI, one key for all your APIs

If you already use RapidAPI, the same data is available as a plain REST API with monthly plans and a free tier: [german-jobs-arbeitsagentur on RapidAPI](https://rapidapi.com/VivienMORVAN/api/german-jobs-arbeitsagentur). No Apify account needed, billing goes through your RapidAPI subscription.

```bash
curl -X POST "https://german-jobs-arbeitsagentur.p.rapidapi.com/v2/acts/bibian~arbeitsagentur-jobs-scraper/run-sync-get-dataset-items?clean=true" \
  -H "x-rapidapi-key: <YOUR_RAPIDAPI_KEY>" \
  -H "x-rapidapi-host: german-jobs-arbeitsagentur.p.rapidapi.com" \
  -H "content-type: application/json" \
  -d '{"was": "Softwareentwickler", "wo": "Berlin", "maxItems": 50}'
```

The response is the JSON array of postings, with the same fields as above.

## Pricing

Pay per result: **$1.20 per 1,000 job postings**, plus a tiny start fee of $0.00005 per run. You pay only for the results saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/arbeitsagentur-jobs-scraper) when this guide was written; the Store page is the reference.

## Why this one

- **The official source, read directly.** No scraping of rendered HTML: this Actor calls the same documented JSON API (also catalogued on the community-run bundesAPI/jobsuche-api project) the Bundesagentur's own site uses.
- **Named contacts actually removed.** The raw description is sometimes hard-wrapped across lines with a named recruiter's e-mail or phone mid-sentence; this Actor cuts the whole sentence that carries one, not just the matching word, using sentence punctuation rather than line breaks.
- **No browser, no proxy, no login.** A handful of lightweight JSON requests per run.

## Limits and honest notes

- Some results are postings mirrored from an EURES partner board (for example Austria's AMS); their `url` points at that partner's own page instead of arbeitsagentur.de.
- Most German postings do not disclose a salary; `salaryMin`/`salaryMax`/`salaryPeriod` stay empty or `"KEINE_ANGABEN"` when that is the case.
- A sentence that carries an e-mail address or a German phone number is dropped from the description.
- The dataset contains job postings only. Recruiter and candidate personal data is never exported.
- This is an unofficial tool, not affiliated with the Bundesagentur für Arbeit. Check the terms that apply to your use.

## FAQ

**Does it work without `was` or `wo`?**  
Yes. Leave both empty for a productive default run across the newest postings nationwide.

**Does it need a browser or a proxy?**  
No. It calls the public Jobsuche JSON API directly.

**Why does one posting sometimes point to a different website?**  
A posting published through the EURES partner network (for example from Austria) keeps its own partner URL as the canonical link instead of an arbeitsagentur.de page.

**Can I get the data in Excel or Google Sheets?**  
Yes. Export the dataset as CSV, Excel or JSON from the Apify console, or connect the Apify Google Sheets integration.

## Try it

**[Open arbeitsagentur-jobs-scraper on the Apify Store](https://apify.com/bibian/arbeitsagentur-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
