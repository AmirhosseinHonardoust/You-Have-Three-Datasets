<p align="center">
  <h1 align="center">You Don’t Have One Dataset, You Have Three </h1>
    <p align="center">
<div align="center">

![Data Analysis](https://img.shields.io/badge/Discipline-Data%20Analysis%20%26%20Data%20Science-0aa)
![Theme](https://img.shields.io/badge/Theme-Data%20Integrity%20%7C%20Cohorts%20%7C%20Bias-6f42c1)
![Focus](https://img.shields.io/badge/Focus-Observed%20vs%20Missing%20vs%20Excluded-orange)
![Outcome](https://img.shields.io/badge/Outcome-More%20Trustworthy%20Insights-success)
![Style](https://img.shields.io/badge/Article-Longform-blue)
 
</div>
 
Most analysts talk about “the dataset” like it’s a single object; stable, complete, and representative.

It isn’t.

In almost every real project, you are quietly working with **three datasets**:

1. **Observed**: what you can see in the table right now
2. **Missing**: what should exist but doesn’t (nulls, gaps, broken tracking, late arrivals)
3. **Excluded**: what existed but you removed (filters, joins, deduping, eligibility rules)

And here’s the uncomfortable truth:
 
> Your conclusions are usually shaped more by **Missing** and **Excluded** than by what’s Observed.

This article is a practical guide to recognizing the three datasets, measuring them, and preventing them from silently destroying your analysis.

---

## Why this matters (even if your SQL is correct)

You can write perfect SQL and still be wrong because:

* the data you **don’t** have is not random
* the rows you **remove** are not neutral
* the gaps you ignore are often concentrated in specific users, regions, devices, or time ranges
* your metrics are computed on “the survivors,” not the population you think you’re measuring

When this happens, it doesn’t fail loudly. It fails politely:

* your dashboard looks clean
* your model trains “fine”
* your KPIs move
* your conclusions sound reasonable

And then you ship decisions based on a dataset that is *not the dataset you think you have.*

---

## The three datasets: a concrete definition

### 1) Observed dataset

This is the dataset you see and query:

* the rows that made it into storage
* the columns that are populated (at least sometimes)
* the records that survived ingestion and parsing

**Observed data is not reality.** It’s *captured reality.*

If you measure only this dataset, you risk confusing “what we captured” with “what happened.”

---

### 2) Missing dataset

This is everything that should exist but does not.

Missing shows up as:

* null values
* missing rows
* missing time windows
* missing categories
* incomplete event streams
* late-arriving data not yet loaded

There are two key types of missingness:

#### Missing values (within-row)

Example: you have an order, but the `payment_method` is null.

#### Missing rows (entire records absent)

Example: orders from iOS are undercounted because the tracking endpoint failed for iOS users.

Missing rows are usually worse because you can’t even see that they’re missing unless you compare against an external reference.

---

### 3) Excluded dataset

This is everything you removed; sometimes intentionally, sometimes accidentally.

Excluded comes from:

* filters (`WHERE country = 'US'`, `WHERE status='completed'`)
* join behavior (inner join drops unmatched)
* deduping logic (keeping only the “latest” row)
* eligibility constraints (“active users only”)
* data cleaning rules (dropping nulls, trimming outliers)
* sampling procedures

Excluded data is often **systematically different** from the data you keep.

That means exclusions create bias, even when they are “reasonable.”

---

## How the three datasets get created (common patterns)

### Pattern A | Inner join = silent row deletion

You think you’re “enriching” data:

```sql
SELECT *
FROM orders o
JOIN users u ON o.user_id = u.user_id
```

But what you actually did is:

* remove any order whose user record is missing
* or any user_id mismatch
* or any late-loaded user profile

Now your “orders analysis” is actually:

> “orders with joinable users”

That’s the **Excluded dataset** taking over.

---

### Pattern B | dropna() = an unacknowledged population change

In Python, you do:

```python
df = df.dropna(subset=["age", "income"])
```

Now your dataset becomes:

> “people who revealed age and income”

And people who reveal those fields are usually not representative.

The conclusions you draw may be valid for that subgroup, but not for the original population.

---

### Pattern C | “active users” is a moving target

Your definition:

> active = “logged in last 30 days”

That’s not a stable population.
It changes with:

* seasonality
* feature launches
* login friction
* outages
* marketing campaigns

If you compare months, your “active cohort” changes composition—even if the underlying behavior doesn’t.

That’s the **Excluded dataset** shaping the trend.

---

### Pattern D | data freshness windows hide missingness

You build a dashboard on “last 7 days.”

But data arrives late. Some sources backfill.

So your “last 7 days” is actually:

* complete for yesterday
* incomplete for today
* partially complete for the last 2 hours
* missing for a source with delay

Now your chart has a “trend” that is just **missingness by time.**

---

## The most important skill: treating exclusions as hypotheses, not defaults

When you filter or drop rows, you are making a claim:

* “This subset represents the problem I’m measuring.”

That claim should be tested.

### Every filter has two meanings:

1. the data you keep
2. the story of what you removed

If you don’t analyze what you removed, you are blind to bias.

---

## A framework: Dataset Accounting

Think like an accountant. Track flows.

Start with a “raw population” and measure how it transforms.

### Dataset accounting checklist

For every analysis, document:

1. **Input population**

   * “All orders created between dates X and Y”

2. **Exclusions applied**

   * “Removed cancelled orders”
   * “Removed test accounts”
   * “Inner join to users”
   * “Removed null ages”

3. **Resulting population size**

   * How many rows remain?
   * What % of the original?

4. **Who got excluded**

   * Which segments lose the most rows?

If you do this consistently, you stop getting fooled by clean tables.

---

## How to measure Missing and Excluded (practical tools)

### 1) Missingness matrix (column-level)

A simple table like:

| column | missing_count | missing_% |
| ------ | ------------- | --------- |
| age    | 12,500        | 18.2%     |
| gender | 1,200         | 1.7%      |

But don’t stop there.

#### Add segmentation:

Missingness by:

* device
* region
* time
* acquisition channel
* user tier

Because missingness is rarely uniform.

---

### 2) Join loss report

If you join tables, always compute:

* rows before join
* rows after join
* drop rate
* drop rate by segment

Example idea:

* “orders missing user profile = 7.8% overall”
* “orders missing user profile = 21% on Android”

That changes the story.

---

### 3) Exclusion diff table (“kept vs removed”)

Whenever you filter, compare the distributions:

* age distribution
* country distribution
* plan distribution
* revenue distribution
* engagement distribution

If the removed group is different, your metric is biased.

---

### 4) Time completeness checks

Build a “data completeness” metric:

* expected events vs observed events
* expected rows per hour vs observed rows

This prevents “data outage” from becoming “business trend.”

---

## A real mental model: “Observed is a sample”

Treat observed data like a sample drawn by a noisy pipeline.

This unlocks two behaviors:

1. You stop trusting single-number KPIs.
2. You start reporting uncertainty and bias risks.

---

## The 5 most dangerous phrases in analytics

### 1) “We’ll just drop nulls”

You are not cleaning data. You are redefining your population.

### 2) “It’s just a join”

Joins are filters. They remove data. Always measure the removal.

### 3) “This dataset is clean”

Clean for what? Clean compared to what baseline? Under what missingness assumptions?

### 4) “The trend is obvious”

Trends are often missingness + composition changes + freshness artifacts.

### 5) “It’s only a few percent”

A few percent missing overall can be 40% missing in a key segment.

---

## How this breaks Data Science (not just dashboards)

If you build models on Observed-only data, you can get:

### Selection bias

Your model learns behavior of “logged users,” “verified users,” “users with complete profiles.”

Then you deploy and wonder why it fails on:

* new users
* anonymous users
* low-connectivity regions
* minority language groups

### Label bias

Labels often exist only for a subset:

* disputes
* manual reviews
* flagged content
* fraud confirmations

If your labels are generated by a biased process, your model learns the bias.

### Feedback loops

Your model decisions affect what data gets collected next.
Example:

* you only review low-confidence items
* so you only label those
* so the model trains more on uncertain items
* while confident-but-wrong errors go undetected

This is “the missing dataset” growing over time.

---

## What to do in practice

### Step 1 | Write a “population statement”

Before you analyze anything, write:

> “This analysis measures X for population Y over time window Z.”

If you can’t do this, you’re not ready to query.

### Step 2 | Produce a dataset accounting report

Include:

* row counts at each stage
* missingness by key columns
* join loss
* exclusion rates by segment

### Step 3 | Add “coverage metrics” to dashboards

For every KPI, include:

* denominator size
* % of expected data present
* join match rate
* null rate for key fields

This prevents silent failure.

### Step 4 | Treat exclusions as decisions

Document why exclusions exist and what risks they introduce.

---

## A reusable template: “Three Dataset Summary” block

You can paste this into any report:

**Observed dataset**

* N rows, time range X–Y, sources A/B/C

**Missing dataset (known)**

* Columns with high null rates: …
* Time windows with low completeness: …
* Segments with missingness spikes: …

**Excluded dataset**

* Filters applied: …
* Join drop rate: …
* Outlier removal rate: …
* Who is most excluded: …

**Risk statement**

* “Results may under-represent segment S due to missingness in field F.”
* “Trends for last 24h may be biased due to ingestion delay.”

That’s professional analytics.

---

## The closing idea

The one sentence that captures the entire article:

> Analytics isn’t just computing metrics on what you have, it’s proving what you *don’t* have won’t change the story.

If you ignore Missing and Excluded, you’re not doing analysis.
You’re doing storytelling with a selective camera.
