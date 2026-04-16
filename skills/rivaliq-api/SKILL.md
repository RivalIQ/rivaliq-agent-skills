---
name: rivaliq-api
description: >-
  Competitive owned-social analytics via the Rival IQ API v3. Covers brand
  accounts on Instagram, TikTok, Facebook, Twitter/X, and YouTube. Build
  competitive scorecards, trend analyses, content strategy teardowns, channel
  gap detection, and brand positioning audits. Pairs with ConvTPC earned data
  for owned-vs-earned comparisons. Trigger when the user mentions Rival IQ,
  RIQ, competitive social benchmarking, owned social performance, brand social
  metrics, landscapes, engagement rates, follower growth, or posting cadence.
---

# Rival IQ API v3 - Competitive Social Analytics

This skill teaches you how to pull owned-social data from Rival IQ's REST API
and turn it into competitive intelligence. RIQ tracks brand-owned social
accounts on Instagram, Facebook, Twitter/X, TikTok, and YouTube across a
"landscape" of competing brands, providing follower counts, posting activity,
engagement metrics, and individual post performance.

## Authentication

### Reading the API key

Use the `RIVALIQ_API_KEY` environment variable. If your workspace keeps local
secrets in a `.env` file, load that file before making API calls:

```bash
set -a
source .env
set +a
```

If you do not use a `.env` file, export the variable directly:

```bash
export RIVALIQ_API_KEY=<your-api-key>
```

### When the key is missing

If `RIVALIQ_API_KEY` is not set or still has a placeholder value, guide the
user:

1. Log in to [app.rivaliq.com](https://app.rivaliq.com).
2. Go to **Account -> Profile**.
3. Copy the API key shown on that page.
4. Export it in the shell or add it to a local `.env` file.

Do not commit API keys into source control.

### Using the key

- Pass the key as a query parameter named `apiKey` (camelCase). No other auth
  method works.
  ```text
  https://api.rivaliq.com/v3/landscapes?apiKey=$RIVALIQ_API_KEY
  ```
- Always use `api.rivaliq.com`, never `app.rivaliq.com`. The latter is the web
  UI and returns HTML 404s for API calls.

## Core Endpoints

All endpoints are under `https://api.rivaliq.com/v3`. Every call requires the
`apiKey` query parameter.

### 1. List Landscapes

```text
GET /landscapes
```

Returns all landscapes the account has access to. Each landscape contains a
`companies` array with channel presence details such as handles, native IDs,
and history start dates. Use this first to discover landscape IDs and company
IDs.

**Response shape:**

```json
{
  "landscapes": [{
    "id": 604606,
    "name": "Anthropologie",
    "focusCompanyId": 1942811,
    "companies": [{
      "id": 1942811,
      "name": "Anthropologie",
      "url": "https://www.anthropologie.com",
      "instagram": { "url": "...", "handle": "anthropologie" }
    }]
  }]
}
```

### 2. List Companies in a Landscape

```text
GET /landscapes/:landscapeId/companies
```

Returns the company roster with IDs and names. Useful for mapping company IDs
to brand names before pulling metrics.

### 3. Metrics Summary

```text
GET /landscapes/:landscapeId/metrics/summary?timePeriod=last90Days&channel=all
```

Returns one object per company with aggregated metrics for the period. Metrics
use a flat, channel-prefixed namespace such as `instagramFollowedBy`,
`tikTokPosts`, and `facebookPostLikes`. Note the mixed-case prefix for TikTok:
`tikTok`, not `tiktok`.

**Key query parameters:**

- `timePeriod`: `last30Days`, `last90Days`, `last6Months`, `last12Months`
- `channel`: `all`, `instagram`, `facebook`, `twitter`, `tiktok`, `youtube`

**Important fields per company:**

| Prefix | Key metrics |
|---|---|
| `instagram*` | `followedBy`, `posts`, `postLikes`, `postComments`, `postsEngagementTotal`, `averageEngagementRatePerPost` |
| `tikTok*` | `tikTokFollowers`, `tikTokPosts`, `tikTokPostLikes`, `tikTokPostsEngagementTotal`, `tikTokViews` |
| `facebook*` | `facebookPageLikes`, `facebookPosts`, `facebookPostLikes`, `facebookPostComments` |
| `twitter*` | `twitterAllTweets`, `twitterLikesOfCompanyTweets`, `twitterRetweets` |
| `youTube*` | `youTubeSubscribers`, `youTubeVideos`, `youTubePostLikes`, `youTubePostViews` |

Each object also includes `mainPeriodStart/End` and `comparePeriodStart/End`
for calculating period-over-period changes.

### 4. Time Series

```text
GET /landscapes/:landscapeId/metrics/timeSeries?timePeriod=last90Days&channel=instagram
```

Returns one object per company per date. Same flat field namespace as the
summary. Use this for trend charts such as engagement over time, follower
growth curves, or posting cadence.

Tip: filter by a single channel to keep response sizes manageable. The `all`
channel returns every metric for every date, which can get large.

### 5. Social Posts

```text
GET /landscapes/:landscapeId/socialPosts?timePeriod=last30Days&channel=instagram
```

Returns up to 100 individual posts with `companyName`, `postText`,
`engagementTotal`, `engagementRate`, `publishedAt`, `postLink`, `views`, and
`type` such as photo, video, reel, or carousel.

Use this for:

- finding each brand's top-performing posts
- analyzing content themes that drive engagement
- comparing posting strategies across brands

### 6. Brand Positioning

```text
GET /landscapes/:landscapeId/positioning/current
```

Returns current bios, descriptions, and metadata from each brand's social
profiles. Useful for competitive positioning analysis.

### 7. Landscape Status

```text
GET /landscapes/:landscapeId/status
```

Returns `{"status": 1}` for a ready landscape and `{"status": 2}` while an
update is in progress. Check this if metrics return empty; the landscape may be
paused or still initializing.

## Endpoints That Exist But Were Not Tested

These are documented in the API reference but were not needed for reporting:

- `GET /landscapes/:id/postTags`
- `POST /landscapes`
- `PUT /landscapes/:id`
- `GET /landscapes/:id/bulkSocialPosts`
- `GET /landscapes/:id/privateData/*`

## Common Pitfalls

1. Wrong domain: `app.rivaliq.com` is the web UI. API calls go to
   `api.rivaliq.com`.
2. Wrong auth: the API only accepts `apiKey` as a query parameter. Header auth
   such as `x-api-key` or `Authorization: Bearer` fails.
3. `curl` not on `PATH` in some worktrees: use `/usr/bin/curl` explicitly.
4. "user has invalid plan": the token is recognized but the account does not
   have API access enabled.
5. Endpoint guessing: do not guess undocumented endpoint paths.

## What You Can Build With These Endpoints

These endpoints are composable building blocks.

**Competitive scorecards** compare follower bases, posting cadence, and
engagement rates side by side for a given period.

**Trend analysis** uses time series data to spot momentum shifts, seasonal
patterns, or campaign effects.

**Content strategy teardown** uses social posts sorted by engagement to find
what themes and content types win for each brand.

**Channel gap detection** compares where brands invest against where audiences
engage, surfacing under-invested channels.

**Brand positioning audit** compares current profile bios and descriptions
across a landscape.

### Combining with ConvTPC earned data

RIQ covers owned social. ConvTPC covers earned social. Combining them unlocks
insights neither source has alone:

| RIQ (Owned) | ConvTPC (Earned) | Unlocked Insight |
|---|---|---|
| Posting frequency by channel | Conversation volume by channel | Where the brand invests vs. where the audience talks |
| Engagement rate per post | Earned engagement per mention | Owned content resonance vs. organic buzz |
| Zero TikTok posts | Millions of earned TikTok engagements | Channel gap: audience is there, brand is not |
| Top-performing post themes | Subtopic discovery themes | What the brand pushes vs. what consumers discuss |
| Follower counts | Unique author counts | Owned audience size vs. earned conversation reach |
| Brand positioning (bios) | Consumer language (subtopics) | How the brand describes itself vs. how consumers describe it |

## Response Parsing Tips

- Metric fields use camelCase with a channel prefix such as
  `instagramPostsEngagementTotal` or `tikTokFollowers`.
- Engagement rate fields are decimals, not percentages.
- Time series dates are ISO strings.
- Social post `engagementTotal` sums likes, comments, and shares where the
  platform exposes them.
- The `comparePeriod` fields in summary responses support period-over-period
  comparisons without a second API call.
