# 🗺️ Extracting User Journey Data Using SQL

<p align="center">
  <img src="images/example_csv_snapshot.png" alt="Example CSV Snapshot - User Journey Data" width="800">
</p>

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [Business Questions](#-business-questions)
- [Dataset Description](#-dataset-description)
- [Methodology](#-methodology)
- [SQL Solution Approach](#-sql-solution-approach)
- [Results](#-results)
- [Key Insights & Interpretation](#-key-insights--interpretation)
- [Technical Skills Demonstrated](#-technical-skills-demonstrated)
- [Project Structure](#-project-structure)
- [How to Run](#-how-to-run)
- [Future Improvements](#-future-improvements)

---

## 🎯 Project Overview

SQL is often just used to fetch raw data from data storage that is later transferred to other software for preprocessing. But this programming language can do more than select data from a table. Sometimes, aggregating or preprocessing data in SQL directly might be easier or more beneficial — precisely the purpose of this project.

This project extracts **user journey data** from the 365 Data Science platform to analyze the sequence of pages visited by paying customers leading up to a purchase. Here, "user journey" refers to the steps each user goes through while exploring the product or platform before purchasing. The context is an online subscription-based company offering monthly, quarterly, and annual subscription plans.

**Key Objective:** Using only SQL, consider all users that purchased a plan and aggregate all visited pages into a big sequence or string — creating a customer journey data extract as the starting point for later analysis.

**Source:** 365 Data Science SQL Course Project — *Extracting User Journey Data Using SQL*
**Instructor:** Nikola Pulev
**Type:** Skill Track Project | ⚡ Advanced | 5 Hours

---

## ❓ Business Questions

This data extraction enables answering the following critical business questions:

| # | Question |
|---|----------|
| 1 | What **sequence of pages** do paying customers visit before making a purchase? |
| 2 | How do user journeys differ between **Monthly, Quarterly, and Annual** subscribers? |
| 3 | What are the most **common navigation patterns** leading to a conversion? |
| 4 | Which **pages appear most frequently** in the journey of paying users? |

---

## 🗄️ Dataset Description

### Database: `user_journey_data`

The analysis uses three interconnected tables:

### Table 1: `front_interactions`
Records all visitor activity on the company's front page — from scrolling to clicking on buttons.

| Column | Type | Description |
|--------|------|-------------|
| `visitor_id` | INT | The ID number of the visitor |
| `session_id` | INT | The session number during which the interaction took place |
| `event_source_url` | STRING | The URL of the page on which the given event took place |
| `event_destination_url` | STRING | The URL of the page when the event was completed/processed (same as source URL for non-navigation interactions like scrolling) |
| `event_date` | DATETIME | The exact timestamp of the event/interaction |
| `event_name` | STRING | An internal name of the event |

> **Key detail:** Source and destination URLs are the **same** for interactions like scrolling or clicking a form field. They **differ** when a visitor clicks a link to navigate to a different page — this is what we focus on for the journey sequence.

### Table 2: `student_purchases`
Contains records of user payments and the type of product they purchased, including recurring payments.

| Column | Type | Description |
|--------|------|-------------|
| `user_id` | INT | The ID of the user (different from visitor_id) |
| `purchase_id` | INT | The ID of the purchase |
| `purchase_type` | INT | Type of subscription (0=monthly, 1=quarterly, 2=annual) |
| `purchase_price` | DECIMAL | The price the user paid in dollars |
| `date_purchased` | DATETIME | The exact datetime of the purchase |

> **Key detail:** Users who purchased at $0 are test users and must be excluded.

### Table 3: `front_visitors`
The **bridge table** linking `front_interactions` and `student_purchases`.

| Column | Type | Description |
|--------|------|-------------|
| `visitor_id` | INT | The ID of the visitor — every record has this field |
| `user_id` | INT | The ID of the user — many NULLs (visitors who never created an account) |

> **Key detail:** Most visitors have never created an account. A single user can have multiple visitor IDs (different devices/browsers).

### Data Relationships

```
┌──────────────────────┐
│  student_purchases   │
│  (user_id)           │──────┐
└──────────────────────┘      │
                              │  user_id
┌──────────────────────┐      │
│  front_visitors      │◄─────┘
│  (visitor_id,        │──────┐
│   user_id [nullable])│      │
└──────────────────────┘      │  visitor_id
                              │
┌──────────────────────┐      │
│  front_interactions  │◄─────┘
│  (visitor_id,        │
│   session_id,        │
│   event_source_url,  │
│   event_dest_url,    │
│   event_date,        │
│   event_name)        │
└──────────────────────┘
```

### Provided Resource Files

| File | Description |
|------|-------------|
| `User_Journey_Database.sql` | SQL file to generate the database (may take 1-10 min to run) |
| `URL_Aliases.xlsx` | Reference list with all URLs and suggested page aliases |

---

## 🔬 Methodology

### Approach Overview

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   CTE 1         │     │   CTE 2         │     │   CTE 3         │     │   CTE 4         │     │   CTE 5         │
│   Identify      │ ──► │   Extract       │ ──► │   Map URLs      │ ──► │   Concatenate   │ ──► │   Group by      │
│   Paid Users    │     │   Interactions  │     │   to Aliases    │     │   Page Pairs    │     │   Session       │
│  (paid_users)   │     │ (table_inter..) │     │ (table_aliases) │     │ (table_concat.) │     │ (table_group..) │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Tasks Summary

| # | Task | Description |
|---|------|-------------|
| 1 | Filter paid users | Consider users who made their first purchase in Q1 2023 |
| 2 | Extract pre-purchase interactions | Only page interactions **before** the purchase date |
| 3 | Remove test users | Filter out users that paid $0 |
| 4 | Create URL aliases | Replace raw URLs with readable nicknames (e.g., `Homepage`, `Pricing`) |
| 5 | Combine pages per session | All pages of each session into a single hyphen-separated string |
| 6 | Export as CSV | Output: `user_id`, `session_id`, `subscription_type`, `user_journey` |

### Expected Output Format

| Column | Type | Description |
|--------|------|-------------|
| `user_id` | INT | Unique user identifier |
| `session_id` | INT | Unique session identifier |
| `subscription_type` | VARCHAR | Monthly, Quarterly, or Annual |
| `user_journey` | TEXT | Complete page navigation string (hyphen-separated) |

### URL Alias Mapping

| URL Pattern | Alias | Match Type |
|-------------|-------|------------|
| `https://365datascience.com/` | Homepage | Exact match (`=`) |
| `.../login/%` | Log in | LIKE (starts with) |
| `.../signup/%` | Sign up | LIKE (starts with) |
| `.../resources-center/%` | Resources center | LIKE (starts with) |
| `.../courses/%` | Courses | LIKE (starts with) |
| `.../career-tracks/%` | Career tracks | LIKE (starts with) |
| `.../upcoming-courses/%` | Upcoming courses | LIKE (starts with) |
| `.../career-track-certificate/%` | Career track certificate | LIKE (starts with) |
| `.../course-certificate/%` | Course certificate | LIKE (starts with) |
| `.../success-stories/%` | Success stories | LIKE (starts with) |
| `.../blog/%` | Blog | LIKE (starts with) |
| `.../pricing/%` | Pricing | LIKE (starts with) |
| `.../about-us/%` | About us | LIKE (starts with) |
| `.../instructors/%` | Instructors | LIKE (starts with) |
| `.../checkout/%` (with `coupon`) | Coupon | LIKE + LIKE |
| `.../checkout/%` (without `coupon`) | Checkout | LIKE + NOT LIKE |
| Everything else | Other | Default (ELSE) |

**Why `LIKE` instead of exact match?** Every blog post has a separate URL, but all start with `https://365datascience.com/blog/`. We want to analyze the blog section as a whole, not individual articles. The `%` wildcard matches any characters after the base pattern. The **homepage is the only exception** — it uses exact match since it's a unique standalone page.

---

## 💻 SQL Solution Approach

The solution uses a `WITH` clause (Common Table Expressions / CTEs) to break the complex query into smaller, easier-to-understand subqueries. CTEs make the code more readable, easier to debug, and allow inspecting each step individually to see how the result evolves with each subsequent subquery.

### Pre-requisite: Increase GROUP_CONCAT Limit

```sql
SET SESSION group_concat_max_len = 100000;
```

**Why is this necessary?** The `GROUP_CONCAT()` function has a default maximum string length of **1,024 symbols**. Many user journeys in our data are longer than that, so without increasing this limit, the string would be truncated — displaying only the first 1,024 symbols. Setting it to 100,000 is a generically large number, bigger than the longest user journey in the data.

> ⚠️ **Important:** You need to run this line **before** running the main query.

---

### CTE 1: `paid_users` — Identify Eligible Paid Users

**Objective:** Extract all users eligible for the user journey analysis while excluding test users. Since all relevant users have purchased a subscription, we only need the `student_purchases` table.

```sql
paid_users as
(
    SELECT
        user_id,
        MIN(date_purchased) as first_purchase_date,
        CASE
            WHEN purchase_type = 0 THEN 'Monthly'
            WHEN purchase_type = 1 THEN 'Quarterly'
            WHEN purchase_type = 2 THEN 'Annual'
            ELSE 'Other'
        END as subscription_type,
        purchase_price as price
    FROM
        student_purchases
    GROUP BY user_id
    HAVING
        price > 0
        AND
        CAST(first_purchase_date as DATE) >= '2023-01-01'
        AND
        CAST(first_purchase_date as DATE) < '2023-03-01'
),
```

| Column | Purpose |
|--------|---------|
| `user_id` | Unique user identifier |
| `first_purchase_date` | First purchase date via `MIN(date_purchased)` |
| `subscription_type` | Readable name mapped from numeric `purchase_type` codes (0→Monthly, 1→Quarterly, 2→Annual) |
| `price` | Purchase price — used to filter out test users |

**Why `HAVING` instead of `WHERE`?**
The filters are in the `HAVING` clause because we want the user's **first** purchase to be inside the date range, not **any** purchase — as would happen if we used the `WHERE` clause. `WHERE` filters rows **before** aggregation, while `HAVING` filters **after** aggregation. Since `first_purchase_date` is calculated by `MIN(date_purchased)`, we need the aggregation to happen first, then filter.

**Example of why this matters:**
- A user bought in December 2022 AND March 2023
- `WHERE date >= '2023-01-01'` → removes the Dec row **first**, then `MIN()` incorrectly sees March as the first purchase ❌
- `HAVING first_purchase_date >= '2023-01-01'` → `MIN()` correctly finds December first, then the filter excludes this user ✅

**Why `GROUP BY user_id`?** We want one row per user with their earliest purchase date.

**Why `CASE` for subscription_type?** The database stores types as numbers (0, 1, 2). We convert them to readable names for the analyst.

**Why `price > 0`?** Filters out test users who "purchased" at the fantastic price of $0.

---

### CTE 2: `table_interactions` — Extract Pre-Purchase Page Interactions

**Objective:** Take the list of all relevant users and obtain a list of all the relevant interactions they had with the front page from the `front_interactions` table.

```sql
table_interactions as
(
    SELECT
        p.user_id,
        i.visitor_id,
        i.session_id,
        i.event_source_url,
        i.event_destination_url,
        p.subscription_type
    FROM
        paid_users as p
        INNER JOIN
        front_visitors as v ON v.user_id = p.user_id
        INNER JOIN
        front_interactions as i ON i.visitor_id = v.visitor_id
    WHERE
        i.event_date < p.first_purchase_date
),
```

**Why two `INNER JOIN`s?**
The central piece is how to connect the tables. You need two inner joins:
- **First JOIN** (`paid_users` → `front_visitors`): Connects `user_id` to `visitor_id` — a single user can have multiple visitor IDs (different devices/browsers)
- **Second JOIN** (`front_visitors` → `front_interactions`): Extracts all the interactions corresponding to the visitor IDs

**Why `INNER JOIN` instead of `LEFT JOIN`?**
We don't want to collect any sparse data (i.e., a user with a missing user journey). We can directly use `INNER JOIN` instead of `LEFT JOIN` to keep only users who have matching interaction records.

**Why `WHERE event_date < first_purchase_date`?**
The analysis is based on pages visited **before** purchasing — there's no need to analyze "thank you" and "purchase confirmation" pages. We must exclude all visited (or interacted) pages after the purchase timestamp.

**Why is `event_name` excluded?** It's an internal identifier not relevant for this query.

---

### CTE 3: `table_aliases` — Map URLs to Readable Page Names

**Objective:** Select all columns from the previous query and change the strings in the two URL columns. Replace all URLs with short nicknames or aliases for clarity and readability using the `CASE` statement.

```sql
table_aliases as
(
    SELECT
        user_id,
        session_id,
        subscription_type,
        CASE
            WHEN event_source_url = 'https://365datascience.com/' THEN 'Homepage'
            WHEN event_source_url LIKE 'https://365datascience.com/login/%' THEN 'Log in'
            WHEN event_source_url LIKE 'https://365datascience.com/signup/%' THEN 'Sign up'
            WHEN event_source_url LIKE 'https://365datascience.com/resources-center/%' THEN 'Resources center'
            WHEN event_source_url LIKE 'https://365datascience.com/courses/%' THEN 'Courses'
            WHEN event_source_url LIKE 'https://365datascience.com/career-tracks/%' THEN 'Career tracks'
            WHEN event_source_url LIKE 'https://365datascience.com/upcoming-courses/%' THEN 'Upcoming courses'
            WHEN event_source_url LIKE 'https://365datascience.com/career-track-certificate/%' THEN 'Career track certificate'
            WHEN event_source_url LIKE 'https://365datascience.com/course-certificate/%' THEN 'Course certificate'
            WHEN event_source_url LIKE 'https://365datascience.com/success-stories/%' THEN 'Success stories'
            WHEN event_source_url LIKE 'https://365datascience.com/blog/%' THEN 'Blog'
            WHEN event_source_url LIKE 'https://365datascience.com/pricing/%' THEN 'Pricing'
            WHEN event_source_url LIKE 'https://365datascience.com/about-us/%' THEN 'About us'
            WHEN event_source_url LIKE 'https://365datascience.com/instructors/%' THEN 'Instructors'
            WHEN event_source_url LIKE 'https://365datascience.com/checkout/%'
                 AND event_source_url LIKE '%coupon%' THEN 'Coupon'
            WHEN event_source_url LIKE 'https://365datascience.com/checkout/%'
                 AND event_source_url NOT LIKE '%coupon%' THEN 'Checkout'
            ELSE 'Other'
        END as source_page_alias,
        CASE
            WHEN event_destination_url = 'https://365datascience.com/' THEN 'Homepage'
            WHEN event_destination_url LIKE 'https://365datascience.com/login/%' THEN 'Log in'
            WHEN event_destination_url LIKE 'https://365datascience.com/signup/%' THEN 'Sign up'
            WHEN event_destination_url LIKE 'https://365datascience.com/resources-center/%' THEN 'Resources center'
            WHEN event_destination_url LIKE 'https://365datascience.com/courses/%' THEN 'Courses'
            WHEN event_destination_url LIKE 'https://365datascience.com/career-tracks/%' THEN 'Career tracks'
            WHEN event_destination_url LIKE 'https://365datascience.com/upcoming-courses/%' THEN 'Upcoming courses'
            WHEN event_destination_url LIKE 'https://365datascience.com/career-track-certificate/%' THEN 'Career track certificate'
            WHEN event_destination_url LIKE 'https://365datascience.com/course-certificate/%' THEN 'Course certificate'
            WHEN event_destination_url LIKE 'https://365datascience.com/success-stories/%' THEN 'Success stories'
            WHEN event_destination_url LIKE 'https://365datascience.com/blog/%' THEN 'Blog'
            WHEN event_destination_url LIKE 'https://365datascience.com/pricing/%' THEN 'Pricing'
            WHEN event_destination_url LIKE 'https://365datascience.com/about-us/%' THEN 'About us'
            WHEN event_destination_url LIKE 'https://365datascience.com/instructors/%' THEN 'Instructors'
            WHEN event_destination_url LIKE 'https://365datascience.com/checkout/%'
                 AND event_destination_url LIKE '%coupon%' THEN 'Coupon'
            WHEN event_destination_url LIKE 'https://365datascience.com/checkout/%'
                 AND event_destination_url NOT LIKE '%coupon%' THEN 'Checkout'
            ELSE 'Other'
        END as destination_page_alias
    FROM
        table_interactions
),
```

**Why `LIKE` with `%` instead of exact match?**
Consider blog posts: every blog post has a separate URL, as it is a different page. But all of them start with `https://365datascience.com/blog/`. All the differences in the URL come after the last backslash and are usually some form of the blog post's title. We want to look at the blog's performances as a whole, not any particular article. That's why we should match every URL **starting with** that string. The `%` sign indicates "match everything here."

**Exception — Homepage:** The homepage is the only URL that uses an **exact match** (`=`) instead of `LIKE`, because `https://365datascience.com/` is a unique standalone page.

**Checkout vs. Coupon:** Both fall under `checkout/` URLs, but are differentiated by checking whether the URL contains the word `coupon`. Uses `LIKE '%coupon%'` and `NOT LIKE '%coupon%'` to split them.

---

### CTE 4: `table_concatenated` — Combine Source & Destination Pages

**Objective:** At this point, we have a list of the relevant interactions of every user with the session ID, subscription type, and the source and destination pages. Now, we must combine these pages into one big string for every session. First, we combine the source and destination page columns into one.

```sql
table_concatenated as
(
    SELECT
        user_id,
        session_id,
        subscription_type,
        CONCAT(source_page_alias,
                    '-',
                    destination_page_alias) as source_destination
    FROM
        table_aliases
),
```

**Why `CONCAT()`?** It combines two strings from two columns into one in MySQL. Uses a dash `-` as the separator.

**Why is this step needed?** `GROUP_CONCAT` in the next step operates on a **single column**. We currently have two columns (source and destination), so we merge them first.

**Example:**
| source_page_alias | destination_page_alias | → source_destination |
|---|---|---|
| Homepage | Courses | Homepage-Courses |
| Pricing | Checkout | Pricing-Checkout |

---

### CTE 5: `table_group_concatenated` — Build Complete Session Journey

**Objective:** This is the final CTE where we take all records for a single session and combine the strings into one big user journey sequence.

```sql
table_group_concatenated as
(
    SELECT
        user_id,
        session_id,
        subscription_type,
        GROUP_CONCAT(source_destination
            SEPARATOR '-') as user_journey
    FROM
        table_concatenated
    GROUP BY session_id
)
```

**Why `GROUP_CONCAT()`?** It's an aggregate function that combines all the `source_destination` strings belonging to the same session into one continuous journey string. Uses the same dash `-` separator as `CONCAT()` for consistency.

**Why `GROUP BY session_id`?** One session = one journey. All the page-pairs from the same session get combined into one string.

**`CONCAT` vs. `GROUP_CONCAT` — what's the difference?**

| Function | Combines | Direction |
|----------|----------|-----------|
| `CONCAT()` | Columns in the **same row** | Horizontal (left-to-right) |
| `GROUP_CONCAT()` | Values from **multiple rows** | Vertical (top-to-bottom) |

**⚠️ GROUP_CONCAT Limit:** This function has a default limit of 1,024 symbols. If a string is longer, it gets **silently truncated** — no error, no warning. That's why we set `group_concat_max_len = 100000` at the beginning.

---

### Final SELECT Statement

```sql
SELECT
    *
FROM
    table_group_concatenated
ORDER BY user_id, session_id;
```

Orders the final result by `user_id` and `session_id` for clean, organized output ready for CSV export.

---

### Complete Solution Query

```sql
-- !!! Run this line before the query !!!
SET SESSION group_concat_max_len = 100000;


WITH
paid_users as
(
    SELECT
        user_id,
        MIN(date_purchased) as first_purchase_date,
        CASE
            WHEN purchase_type = 0 THEN 'Monthly'
            WHEN purchase_type = 1 THEN 'Quarterly'
            WHEN purchase_type = 2 THEN 'Annual'
            ELSE 'Other'
        END as subscription_type,
        purchase_price as price
    FROM
        student_purchases
    GROUP BY user_id
    HAVING
        price > 0
        AND
        CAST(first_purchase_date as DATE) >= '2023-01-01'
        AND
        CAST(first_purchase_date as DATE) < '2023-03-01'
),
table_interactions as
(
    SELECT
        p.user_id,
        i.visitor_id,
        i.session_id,
        i.event_source_url,
        i.event_destination_url,
        p.subscription_type
    FROM
        paid_users as p
        INNER JOIN
        front_visitors as v ON v.user_id = p.user_id
        INNER JOIN
        front_interactions as i ON i.visitor_id = v.visitor_id
    WHERE
        i.event_date < p.first_purchase_date
),
table_aliases as
(
    SELECT
        user_id,
        session_id,
        subscription_type,
        CASE
            WHEN event_source_url = 'https://365datascience.com/' THEN 'Homepage'
            WHEN event_source_url LIKE 'https://365datascience.com/login/%' THEN 'Log in'
            WHEN event_source_url LIKE 'https://365datascience.com/signup/%' THEN 'Sign up'
            WHEN event_source_url LIKE 'https://365datascience.com/resources-center/%' THEN 'Resources center'
            WHEN event_source_url LIKE 'https://365datascience.com/courses/%' THEN 'Courses'
            WHEN event_source_url LIKE 'https://365datascience.com/career-tracks/%' THEN 'Career tracks'
            WHEN event_source_url LIKE 'https://365datascience.com/upcoming-courses/%' THEN 'Upcoming courses'
            WHEN event_source_url LIKE 'https://365datascience.com/career-track-certificate/%' THEN 'Career track certificate'
            WHEN event_source_url LIKE 'https://365datascience.com/course-certificate/%' THEN 'Course certificate'
            WHEN event_source_url LIKE 'https://365datascience.com/success-stories/%' THEN 'Success stories'
            WHEN event_source_url LIKE 'https://365datascience.com/blog/%' THEN 'Blog'
            WHEN event_source_url LIKE 'https://365datascience.com/pricing/%' THEN 'Pricing'
            WHEN event_source_url LIKE 'https://365datascience.com/about-us/%' THEN 'About us'
            WHEN event_source_url LIKE 'https://365datascience.com/instructors/%' THEN 'Instructors'
            WHEN event_source_url LIKE 'https://365datascience.com/checkout/%'
                 AND event_source_url LIKE '%coupon%' THEN 'Coupon'
            WHEN event_source_url LIKE 'https://365datascience.com/checkout/%'
                 AND event_source_url NOT LIKE '%coupon%' THEN 'Checkout'
            ELSE 'Other'
        END as source_page_alias,
        CASE
            WHEN event_destination_url = 'https://365datascience.com/' THEN 'Homepage'
            WHEN event_destination_url LIKE 'https://365datascience.com/login/%' THEN 'Log in'
            WHEN event_destination_url LIKE 'https://365datascience.com/signup/%' THEN 'Sign up'
            WHEN event_destination_url LIKE 'https://365datascience.com/resources-center/%' THEN 'Resources center'
            WHEN event_destination_url LIKE 'https://365datascience.com/courses/%' THEN 'Courses'
            WHEN event_destination_url LIKE 'https://365datascience.com/career-tracks/%' THEN 'Career tracks'
            WHEN event_destination_url LIKE 'https://365datascience.com/upcoming-courses/%' THEN 'Upcoming courses'
            WHEN event_destination_url LIKE 'https://365datascience.com/career-track-certificate/%' THEN 'Career track certificate'
            WHEN event_destination_url LIKE 'https://365datascience.com/course-certificate/%' THEN 'Course certificate'
            WHEN event_destination_url LIKE 'https://365datascience.com/success-stories/%' THEN 'Success stories'
            WHEN event_destination_url LIKE 'https://365datascience.com/blog/%' THEN 'Blog'
            WHEN event_destination_url LIKE 'https://365datascience.com/pricing/%' THEN 'Pricing'
            WHEN event_destination_url LIKE 'https://365datascience.com/about-us/%' THEN 'About us'
            WHEN event_destination_url LIKE 'https://365datascience.com/instructors/%' THEN 'Instructors'
            WHEN event_destination_url LIKE 'https://365datascience.com/checkout/%'
                 AND event_destination_url LIKE '%coupon%' THEN 'Coupon'
            WHEN event_destination_url LIKE 'https://365datascience.com/checkout/%'
                 AND event_destination_url NOT LIKE '%coupon%' THEN 'Checkout'
            ELSE 'Other'
        END as destination_page_alias
    FROM
        table_interactions
),
table_concatenated as
(
    SELECT
        user_id,
        session_id,
        subscription_type,
        CONCAT(source_page_alias,
                    '-',
                    destination_page_alias) as source_destination
    FROM
        table_aliases
),
table_group_concatenated as
(
    SELECT
        user_id,
        session_id,
        subscription_type,
        GROUP_CONCAT(source_destination
            SEPARATOR '-') as user_journey
    FROM
        table_concatenated
    GROUP BY session_id
)
SELECT
    *
FROM
    table_group_concatenated
ORDER BY user_id, session_id;
```

---

## 📈 Results

### Sample Output

<p align="center">
  <img src="images/query_results.png" alt="Query Results - User Journey Data" width="800">
</p>

### Sample Data:

| user_id | session_id | subscription_type | user_journey |
|---------|------------|--------------------|--------------|
| 10107 | 360608 | Annual | Homepage-Homepage |
| 10107 | 360609 | Annual | Homepage-Homepage-Homepage-Homepage-Homepage-Career tracks-Hom... |
| 10107 | 3653049 | Annual | Homepage-Resources center-Resources center-Resources center-Resources... |
| 10107 | 3655508 | Annual | Homepage-Career tracks-Career tracks-Career tracks |
| 10107 | 3655510 | Annual | Career tracks-Courses-Career tracks-Career tracks-Courses-Ca... |
| 10107 | 3655968 | Annual | Career tracks-Log in-Log in-Log in-Log in-Log in |
| 10107 | 3656751 | Annual | Homepage-Log in-Log in-Log in-Log in-Log in-Log in |
| 10107 | 3656761 | Annual | Checkout-Checkout |
| 10107 | 3657001 | Annual | Checkout-Checkout-Checkout-Checkout-Checkout-Checkout-Che... |
| 10107 | 3657036 | Annual | Log in-Log in-Log in-Log in-Log in-Log in-Log in-Log in |
| 10107 | 3657049 | Annual | Checkout-Checkout |
| 10107 | 3657083 | Annual | Log in-Log in-Log in-Log in-Log in-Log in-Log in-Log in |
| 10107 | 3657114 | Annual | Log in-Log in-Log in-Log in-Log in-Log in-Log in-Log in-Lo... |
| 10107 | 3657143 | Annual | Checkout-Checkout |
| 10107 | 3657160 | Annual | Checkout-Checkout |
| 10107 | 3657165 | Annual | Checkout-Checkout |
| 11145 | 501166 | Monthly | Homepage-Log in-Log in-Log in-Log in-Log in |
| 11145 | 802038 | Monthly | Homepage-Log in-Log in-Log in |
| 11145 | 2173786 | Monthly | Homepage-Log in |

---

## 💡 Key Insights & Interpretation

### 1. Repeated Page Visits Are Common

Many journeys show the same page repeated (e.g., `Log in-Log in-Log in`). This happens because the `front_interactions` table records **all events** — scrolling, clicking form fields, etc. — where source and destination URLs are the same. The project explicitly states this is expected and doesn't need cleaning.

### 2. Homepage Is the Primary Entry Point

Most user sessions begin with `Homepage`, confirming it as the critical first impression. Optimizing the homepage for conversion (clear CTAs, engaging previews, visible pricing) could significantly impact purchase rates.

### 3. Multi-Session Purchase Journeys

Users go through distinct phases before purchasing:

```
Exploration ──���─► Engagement ────► Decision ────► Purchase
(Homepage,        (Log in,          (Checkout       (Final
 Courses,          platform          visits x6)      purchase)
 Career tracks)    usage)
```

User 10107 had **17 sessions** before their Annual purchase, with **6 separate Checkout visits** — indicating significant hesitation.

### 4. Subscription Type Affects Journey Length

| Aspect | Annual Subscribers | Monthly Subscribers |
|--------|-------------------|-------------------|
| Sessions before purchase | Many (10+) | Few (1-5) |
| Exploration depth | High — browse many sections | Low — direct to login/checkout |
| Checkout visits | Multiple (hesitation) | Few (decisive) |

Higher financial commitment = more research and deliberation.

### 5. Checkout Hesitation Is a Key Signal

Multiple checkout-only sessions indicate users who are interested but need a push — prime candidates for:
- Exit-intent popups with discount offers
- Cart abandonment emails
- Price comparison pages
- Live chat support on checkout

---

## 🛠️ Technical Skills Demonstrated

| Category | Skills |
|----------|--------|
| **Common Table Expressions** | 5 chained CTEs using `WITH` clause for clean, debuggable code |
| **SQL Joins** | Multiple `INNER JOIN`s across 3 tables via bridge table |
| **Aggregations** | `MIN()`, `GROUP_CONCAT()`, `GROUP BY` |
| **String Functions** | `CONCAT()` for column merging, `GROUP_CONCAT()` with custom `SEPARATOR` |
| **Conditional Logic** | Complex `CASE` with `LIKE` pattern matching for URL aliasing |
| **Date Filtering** | `CAST()` for date conversion, `HAVING` for post-aggregation filtering |
| **URL Pattern Matching** | `LIKE` with `%` wildcard, combined `LIKE` + `NOT LIKE` for checkout/coupon |
| **Session Configuration** | `SET SESSION group_concat_max_len` to prevent string truncation |
| **Data Cleaning** | Removing test users (`price > 0`), excluding post-purchase interactions |
| **Business Logic** | Translating complex requirements into structured SQL queries |

---

## 📁 Project Structure

```
user-journey-sql-analysis/
│
├── README.md                              # Project documentation (this file)
│
├── data/
│   ├── User_Journey_Database.sql          # Database file with all three tables
│   └── URL_Aliases.xlsx                   # URL to alias mapping reference
│
├── sql/
│   └── user_journey_solution.sql          # Complete SQL solution
│
├── docs/
│   ├── requirements.md                    # Problem statement & task details
│   └── interpretation.md                  # Detailed results analysis
│
└── images/
    ├── example_csv_snapshot.png            # Expected output format
    └── query_results.png                  # Actual query results
```

---

## 🚀 How to Run

### Prerequisites
- MySQL Server 8.0+
- MySQL Workbench (recommended) or any SQL client

### Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/TusharVarma077/user-journey-sql-analysis.git
   ```

2. **Import the database** (may take 1-10 minutes — be patient, don't restart MySQL)
   ```sql
   SOURCE data/User_Journey_Database.sql;
   ```

3. **Set session variable** (⚠️ required before running the query)
   ```sql
   SET SESSION group_concat_max_len = 100000;
   ```

4. **Run the solution query**
   ```sql
   SOURCE sql/user_journey_solution.sql;
   ```

5. **Export results as CSV**
   - In MySQL Workbench: Click the export button in the result grid
   - Select CSV format and save

### Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| Journey strings appear truncated | `group_concat_max_len` not set | Run `SET SESSION group_concat_max_len = 100000;` first |
| No results returned | Database not imported | Run `SOURCE data/User_Journey_Database.sql;` first |
| Import takes too long | Large database file | Wait 1-10 minutes, don't restart MySQL |

---

## 🔜 Future Improvements

- [ ] **Deduplication:** Remove consecutive repeated pages for cleaner journey strings (e.g., `Log in-Log in-Log in` → `Log in`)
- [ ] **Journey Length Analysis:** Calculate and compare average journey lengths by subscription type
- [ ] **Funnel Analysis:** Identify drop-off rates at each stage of the navigation funnel
- [ ] **Python Visualization:** Create Sankey diagrams to visualize common user flows
- [ ] **Cohort Comparison:** Compare journeys of Monthly vs. Quarterly vs. Annual subscribers
- [ ] **Page Frequency Analysis:** Rank pages by visit frequency to identify high-traffic touchpoints
- [ ] **Time-Based Analysis:** Incorporate timestamps to measure time spent on each page
- [ ] **Session Count Analysis:** Analyze average number of sessions before purchase by subscription type
- [ ] **SQL Mode Compatibility:** Ensure query works with `only_full_group_by` strict mode

---

## 📚 What I Learned

1. **Complex CTEs** — Building 5 chained Common Table Expressions for clean, debuggable data transformations
2. **GROUP_CONCAT()** — Aggregating multiple rows into a single delimited string per group, including managing the default 1,024 character limit
3. **HAVING vs. WHERE** — Understanding the critical difference: `HAVING` filters on aggregated results (first purchase date), while `WHERE` would filter before aggregation (any purchase date)
4. **INNER JOIN vs. LEFT JOIN** — Using `INNER JOIN` to exclude users without interaction data, avoiding unnecessary NULLs
5. **URL Pattern Matching** — Using `CASE` with `LIKE` and `%` wildcards to categorize URLs into readable page aliases
6. **Multi-Table JOINs** — Connecting three tables through a bridge table (user_id → visitor_id → interactions)
7. **Data Cleaning in SQL** — Removing test records ($0 purchases) and post-purchase interactions directly in the query
8. **Session Configuration** — Setting `group_concat_max_len` to prevent silent data truncation
9. **Business Context** — Understanding how raw clickstream data translates into actionable user journey insights

---

## 👤 Author

**Tushar Varma**
- GitHub: [@TusharVarma077](https://github.com/TusharVarma077)

---

*Project completed: February 2026*