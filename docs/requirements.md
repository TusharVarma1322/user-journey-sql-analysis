# 📋 Project Requirements

## Project Title
**Extracting User Journey Data Using SQL**

## Case Description

SQL is often just used to fetch raw data from data storage that is later transferred to other software for preprocessing. But this programming language can do more than select data from a table. Sometimes, aggregating or preprocessing data in SQL directly might be easier or more beneficial — precisely the purpose of this project.

This Extracting User Journey Data with SQL project centers around creating a customer journey data extract as the starting point for later analysis. Here, "user journey" refers to the steps each user goes through while exploring the product or platform before purchasing. The context is an online subscription-based company offering monthly, quarterly, and annual subscription plans. We'll use our own 365 platform as the basis for the data.

Using only SQL, you must consider all users that purchased a plan and aggregate all visited pages into a big sequence or string. (This user journey will be analyzed as a different project.)

**Type:** Skill Track Project
**Difficulty:** ⚡ Advanced
**Duration:** 5 Hours
**Instructor:** Nikola Pulev
**Source:** 365 Data Science Platform
**Related Courses:** Advanced SQL, SQL
**Tags:** Programming

---

## Project Requirements

For this project, you'll need **MySQL Workbench 8.0 or later**.

---

## Project Files

For this Extracting User Journey Data with SQL project, you're provided with an SQL file `User_Journey_Database.sql` to generate the database you'll use to extract customer journey data. The steps in generating it are straightforward:

1. First, open your local connection in MySQL
2. Then, open the SQL file from within this connection and run *all* commands in that file, creating a new schema containing all the relevant tables
3. This may take a while (1-10 min), so just be patient and don't restart MySQL during that time

All this data has been stripped of personally identifiable information (PII). But you may find some test user records in the database that must be excluded from the queries.

---

## Database Structure

The database consists of three tables: `front_interactions`, `student_purchases`, and `front_visitors`.

### Table 1: `front_interactions`

Records all visitor activity on the company's front page, including visiting specific pages, clicks, and other interactions on said pages.

| Column | Type | Description |
|--------|------|-------------|
| `visitor_id` | INT | The ID number of the visitor |
| `session_id` | INT | The session number during which the interaction took place |
| `event_source_url` | STRING | The URL of the page on which the given event took place |
| `event_destination_url` | STRING | The URL of the page when the event was completed/processed (for interactions during which the user stays on the same page, this is the same as source URL) |
| `event_date` | DATETIME | The exact timestamp of the event/interaction |
| `event_name` | STRING | An internal name of the event |

**Important notes:**
- This table records **all events** on the front pages — from scrolling to clicking on buttons
- It records the source and destination URLs for every event
- Source and destination URLs are the **same** for interactions like scrolling or clicking on a form field (user stays on the same page)
- When a visitor **clicks on a page link**, the two URLs would **differ** since they moved to a different page — this is what we focus on for the user journey sequence

### Table 2: `student_purchases`

Contains records of user payments and the type of product they purchased. This includes all payments — even if they are subsequent recurring payments for the same subscription.

| Column | Type | Description |
|--------|------|-------------|
| `user_id` | INT | The ID of the user (different from the visitor_id) |
| `purchase_id` | INT | The ID of the purchase |
| `purchase_type` | INT | The type of subscription purchased (0=monthly, 1=quarterly, 2=annual) |
| `purchase_price` | DECIMAL | The price the user paid in dollars |
| `date_purchased` | DATETIME | The exact datetime of the purchase |

**Important notes:**
- Since the person needs to have purchased a product to be in this table and has an account, they are no longer considered a *visitor* but a *user*
- The purchase price can be used as an indicator for test users — if a user has purchased a product at $0, they're probably just a test one
- The default sorting for this table is based on `date_purchased`, from old to new

### Table 3: `front_visitors`

The link between `front_interactions` and `student_purchases`.

| Column | Type | Description |
|--------|------|-------------|
| `visitor_id` | INT | The ID of the visitor — each record has this field filled in |
| `user_id` | INT | The ID of the user corresponding to this visitor — many NULL values here because many visitors never made an account and so were never assigned a user_id |

**Important notes:**
- From this table, you can determine which user corresponds to which visitor
- Most visitors have never created an account and are **not** considered users
- This table serves as the **bridge** between the interactions table (which uses `visitor_id`) and the purchases table (which uses `user_id`)

### Data Relationships Diagram

```
┌──────────────────────────┐
│  student_purchases       │
│  ┌────────────────────┐  │
│  │ user_id        INT │  │
│  │ purchase_id    INT │  │
│  │ purchase_type  INT │  │
│  │ purchase_price DEC │  │
│  │ date_purchased DT  │  │
│  └─────────────��──────┘  │
└────────────┬─────────────┘
             │ user_id
             ▼
┌──────────────────────────┐
│  front_visitors          │
│  ┌────────────────────┐  │
│  │ visitor_id     INT │  │
│  │ user_id        INT │◄─── Many NULLs (visitors without accounts)
│  └────────────────────┘  │
└────────────┬─────────────┘
             │ visitor_id
             ▼
┌──────────────────────────┐
│  front_interactions      │
│  ┌────────────────────┐  │
│  │ visitor_id     INT │  │
│  │ session_id     INT │  │
│  │ event_source   STR │  │
│  │ event_dest     STR │  │
│  │ event_date     DT  │  │
│  │ event_name     STR │  │
│  └────────────────────┘  │
└──────────────────────────┘
```

**Key relationship insight:**
- `student_purchases` uses `user_id` to identify paying customers
- `front_interactions` uses `visitor_id` to identify page visitors
- `front_visitors` is the **bridge table** that maps `visitor_id` ↔ `user_id`
- A single user can have **multiple** visitor IDs (different devices/browsers)
- Many visitors have **NULL** user IDs (never created an account)

---

## Task: Extracting the Data

### Objective
Write up a query to extract data for a subsequent user journey analysis. The idea is that someone in your company wants to analyze the sequence of pages visited leading to a purchase. For that purpose, you need to group all the pages visited in a session into one string. This shouldn't be done for every visitor but only for paying customers. In other words, you should extract the user journey of only those that have purchased a subscription plan.

### Date Range
You should only consider users that made their first purchase between **January 1, 2023, and March 31, 2023, inclusive** (i.e., Q1 2023).

### Expected Output Format

The expected format for the data extract includes four columns:

| Column | Type | Description |
|--------|------|-------------|
| `user_id` | INT | Unique identification of a user |
| `session_id` | INT | Unique identification of a session |
| `subscription_type` | VARCHAR | The type of subscription purchased (Monthly, Quarterly, Annual) |
| `user_journey` | TEXT | All pages combined into one string, separated by hyphens |

Each session should have a separate user journey string. And all the pages are combined into one string separated with a hyphen `-`. You can use a different separator but don't go for a comma since that will clash with the CSV file.

---

## Detailed Requirements

### Requirement 1: Filter Paying Customers
- Consider all users that purchased a subscription plan for the first time between January 1 and March 31, 2023 (inclusive)
- To give more options for the analyst, later on, you also need to include what subscription plan the user purchased

### Requirement 2: Pre-Purchase Interactions Only
- Consider all their page interactions **before** their purchase date
- The analysis is based on the pages a user visited before purchasing because there's no need to analyze the "thank you" and "purchase confirmation" pages
- You must exclude all visited (or interacted) pages after the purchase timestamp

### Requirement 3: Remove Test Users
- There might be some test records in the database
- Filter them out by considering that a real user wouldn't pay $0 for the product, or the company would quickly go bankrupt

### Requirement 4: Create URL Aliases
- All pages are listed through their respective URLs in the database
- Assign an alias to the most essential URLs
- For example, instead of including `https://365datascience.com/` in the user journey string, only `Homepage` should appear
- The list of URLs and aliases is provided as an Excel file (`URL_Aliases.xlsx`) in the project resources

### Requirement 5: Combine Pages Per Session
- Combine all the pages of each session into a single user journey string
- Use a hyphen `-` as separator (not comma, as that clashes with CSV)

### Requirement 6: Handle Repeated Pages
- There are many repeating pages in the strings because the visitor scrolled or took other measures on the page that didn't redirect them to a different page
- For this task, you're **not required** to clean this up
- You can safely squeeze all the pages from the `front_interactions` table together without removing duplicates

### Requirement 7: Export as CSV
- Export all this data as a CSV with the `user_id`, `session_id`, `subscription_type`, and `user_journey` columns
- Order the result by `user_id` and `session_id`

---

## Implementation Tips

### Tip 1: Use CTEs (`WITH` clause)
The easiest way to achieve all this is to use a statement with a `WITH` clause (CTE). This allows us to create several small queries that are easier to work with, and we can test them out independently.

### Tip 2: First CTE — `paid_users`
The first query can list all the users relevant to this task; let's call it `paid_users`. This will allow us to cross-reference them with their visitor ID and extract their sequence of visited pages.
- You can even filter out the test users (those with `purchase_price` of 0) directly here, considering only the relevant date ranges
- This query should have four columns: `user_id`, `first_purchase_date`, `subscription_type` (you can apply the actual names "Monthly", "Quarterly" and "Annual" to the numerical codes here, for clarity) and `purchase_price`
- Consider how your filtering interacts with grouping

### Tip 3: Second CTE — `table_interactions`
The next common table expression in the CTE structure can take all the users from the first query and extract all the relevant entries from the `front_interactions` table.
- Recall that in a statement with a `WITH` clause, you can use all the above subqueries directly in the current one
- Consider all columns from the interactions table except the `event_name` which is not relevant for this query
- Be sure to include only the events before the user subscribes for the first time

### Tip 4: Third CTE — `table_aliases`
At this point, you should have a table considering all relevant users, their subscription type and listing all the relevant sessions and the URLs of the source and destination pages. So, for the overall objective, the only thing left to do is concatenate those pages, one after another, for every session. However, at this point, all those pages are represented through URLs, which are long strings and unnecessarily cluttered. So, for clarity and readability, replace all those URLs with short nicknames or aliases. For that purpose, you can use the aliases file provided.

Therefore, the next query in the `WITH` clause needs to select all the columns from the previous one and change the strings in the two URL columns. This change can be achieved using `CASE`.
- One detail to consider here is that URLs like `https://365datascience.com/resources-center/` reference that section's main page which contains many different resources. But every specific resource resides on a page with a URL that starts like the example one but adds more elements after the last backslash.
- So, when assigning the nicknames, consider all URLs **starting with** the provided strings, not the exact strings — except for the homepage; you can look for an exact match there.

### Tip 5: Fourth CTE — `table_concatenated`
You can use the `CONCAT()` function to combine two strings from two columns into one in MySQL. And that's precisely what you should do in the following subquery — just be sure to pick a sensible separator. Use a dash `-`.

### Tip 6: Fifth CTE — `table_group_concatenated`
So, at this point, you have only one column containing the pages instead of two. Thus, you can move on to the last query on this enormous `WITH` statement that will combine all the strings of this one column for each session. This is achieved with the `GROUP_CONCAT()` function. Since that's an aggregate function, you actually need to group by `session_id`. Be sure to use the same separator in this function as in the previous `CONCAT()`.

### Tip 7: GROUP_CONCAT Limit
Unfortunately, there is one small but essential detail when working with `GROUP_CONCAT()`. By default, this function's maximum string length is 1,024 symbols. You may notice that many user journeys in our data are longer than that. In that case, the string is truncated and stops in the middle of a page. So, you must increase this limit to include all pages — 100,000 symbols should work fine. (Check the official documentation to see how to do this.)

### Tip 8: Final Output
And that's it. You can now order the result by `user_id` and `session_id` and export it in a CSV.

---

## Provided Resource Files

| File | Description |
|------|-------------|
| `User_Journey_Database.sql` | SQL file to generate the database with all three tables (may take 1-10 min to run) |
| `URL_Aliases.xlsx` | List of URLs and their corresponding human-readable aliases |

---

## Sanity Check

Your final output should be a CSV-ready result set with four columns (`user_id`, `session_id`, `subscription_type`, `user_journey`) ordered by `user_id` and `session_id`, containing only pre-purchase interactions for paying customers from Q1 2023.

### Sample Output Verification

| user_id | session_id | subscription_type | user_journey |
|---------|------------|--------------------|--------------|
| 10107 | 360608 | Annual | Homepage-Homepage |
| 10107 | 360609 | Annual | Homepage-Homepage-Homepage-Homepage-Homepage-Career tracks-Hom... |
| 10107 | 3653049 | Annual | Homepage-Resources center-Resources center-Resources center-Resources... |
| 10107 | 3655508 | Annual | Homepage-Career tracks-Career tracks-Career tracks |
| 11145 | 501166 | Monthly | Homepage-Log in-Log in-Log in-Log in-Log in |
| 11145 | 802038 | Monthly | Homepage-Log in-Log in-Log in |
| 11145 | 2173786 | Monthly | Homepage-Log in |