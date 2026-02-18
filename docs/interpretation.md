# 📊 User Journey Data — Results Interpretation

## Project Context

This project extracts user journey data for paying customers on the 365 Data Science platform. The platform is an online subscription-based company offering monthly, quarterly, and annual subscription plans. The data shows the sequence of pages each customer visited before purchasing a subscription in Q1 2023.

The database consists of three tables:
- **`front_interactions`** — Records all visitor activity (scrolling, clicking, page navigation) with source and destination URLs
- **`student_purchases`** — Contains records of user payments including subscription type and price
- **`front_visitors`** — The bridge table linking `visitor_id` to `user_id` (many visitors have NULL user_ids as they never created an account)

Unlike a metrics-based project (conversion rates, averages), this project produces a **raw data extract** — a CSV of user journey strings ready for further analysis. The interpretation below focuses on **patterns observed** in the extracted data and their business implications.

---

## Understanding the Data

### How Interactions Are Recorded
The `front_interactions` table records **all events** on the front pages — from scrolling to clicking on buttons. For each event:
- **`event_source_url`**: The page where the event happened
- **`event_destination_url`**: The page after the event completed

When a user **scrolls or clicks a form field** → source and destination URLs are the **same**
When a user **clicks a link to navigate** → source and destination URLs **differ**

This is why we see repeated pages in the journey strings — they represent non-navigational interactions.

### User vs. Visitor
- A **visitor** is anyone who browses the site (identified by `visitor_id`)
- A **user** is someone who has created an account and purchased (identified by `user_id`)
- The `front_visitors` table maps between the two — but most visitors never become users
- A single user can have **multiple visitor IDs** (different devices, browsers, cleared cookies)

---

## Sample Results Summary

| user_id | session_id | subscription_type | user_journey |
|---------|------------|--------------------|--------------|
| 10107 | 360608 | Annual | Homepage-Homepage |
| 10107 | 360609 | Annual | Homepage-Homepage-Homepage-Homepage-Homepage-Career tracks-Hom... |
| 10107 | 3655508 | Annual | Homepage-Career tracks-Career tracks-Career tracks |
| 10107 | 3656761 | Annual | Checkout-Checkout |
| 10107 | 3657001 | Annual | Checkout-Checkout-Checkout-Checkout-Checkout-Checkout-Che... |
| 11145 | 501166 | Monthly | Homepage-Log in-Log in-Log in-Log in-Log in |
| 11145 | 802038 | Monthly | Homepage-Log in-Log in-Log in |

---

## 1. Page Repetition Patterns

### What We See
Many user journeys contain the same page repeated multiple times. For example:
- `Log in-Log in-Log in-Log in-Log in`
- `Checkout-Checkout-Checkout-Checkout`
- `Homepage-Homepage-Homepage-Homepage`

### Why This Happens
As the case description explains, the `front_interactions` table records **all events on the front pages — from scrolling to clicking on buttons**. The source and destination URLs are the **same for such interactions as scrolling or clicking on a form field**. So when a user:
- Scrolls up or down on the Login page
- Clicks a form field to type their password
- Expands a dropdown or clicks a non-link element

...each action generates an event where `event_source_url = event_destination_url`, creating a `Login-Login` pair.

### Impact on Analysis
- The raw data is **intentionally not cleaned** of these duplicates — the project states: *"For this task, you're not required to clean this up. You can safely squeeze all the pages from the 'front_interactions' table together without removing duplicates."*
- For downstream analysis, deduplicating consecutive identical pages would reveal cleaner navigation flows
- The repetition count itself carries information — a high count of repeated interactions could indicate "time spent" or "engagement level"

### Business Implication
> **Recommendation:** Future analysis should include a deduplication step to see the true page-to-page navigation flow. However, the *count* of repeated interactions per page could serve as a useful proxy for engagement depth on that page.

---

## 2. Common Entry Points

### What We See
The vast majority of user sessions begin with one of these pages:
- **Homepage** — Most common starting point
- **Career tracks** — Direct entry from bookmarks or external links
- **Log in** — Returning users accessing their accounts
- **Checkout** — Users returning specifically to complete a purchase

### Interpretation

| Entry Point | What It Suggests | User Intent Level |
|-------------|-----------------|-------------------|
| Homepage | Arrived organically or via marketing link — exploring the platform | Medium |
| Career tracks | Has a specific goal (career path) — bookmarked or searched directly | High |
| Log in | Returning user — already familiar with the platform | Medium-High |
| Checkout | Came back specifically to complete a purchase | Very High |

### Business Implication
> **Recommendation:** The homepage is the critical first impression for most users. Ensure it clearly communicates the value proposition, displays popular courses/career tracks, and has visible CTAs for pricing. Users entering directly at Career tracks or Checkout pages show high intent — these pages should have streamlined paths to purchase.

---

## 3. Multi-Session Purchase Journeys

### What We See
A single user (e.g., user 10107) has **17 different sessions** before purchasing an Annual subscription. The sessions progress through distinct phases:

**Phase 1 — Exploration (Early Sessions):**
```
Session 360608:  Homepage → Homepage
Session 360609:  Homepage → Homepage → Homepage → Career tracks → ...
Session 3653049: Homepage → Resources center → Resources center → ...
Session 3655508: Homepage → Career tracks → Career tracks → Career tracks
```
The user is browsing, exploring different sections — Homepage, Career tracks, Resources center.

**Phase 2 — Engagement (Middle Sessions):**
```
Session 3655968: Career tracks → Log in → Log in → Log in → ...
Session 3656751: Homepage → Log in → Log in → Log in → ...
Session 3657036: Log in → Log in → Log in → Log in → ...
```
The user is logging in repeatedly — they're actively using the platform, probably watching free content.

**Phase 3 — Decision (Final Sessions):**
```
Session 3656761: Checkout → Checkout
Session 3657001: Checkout → Checkout → Checkout → Checkout → ...
Session 3657049: Checkout → Checkout
Session 3657143: Checkout → Checkout
Session 3657160: Checkout → Checkout
Session 3657165: Checkout → Checkout
```
The user visits Checkout **6 separate times** before finally purchasing. This indicates significant hesitation or comparison shopping.

### Interpretation
```
Exploration ────► Engagement ────► Decision ────► Purchase
(Browse pages)   (Log in, use)    (Checkout x6)  (Annual sub)
 Sessions 1-4     Sessions 5-8    Sessions 9-17
```

This three-phase pattern represents the typical customer journey:
1. **Awareness** — "What does this platform offer?"
2. **Consideration** — "Let me try the free content and explore"
3. **Decision** — "Should I pay? How much? Let me check checkout again..."

### Business Implication
> **Recommendation:** The 6 checkout visits before purchase is a red flag for friction. Consider:
> - **Exit-intent popups** on the checkout page with a limited-time discount
> - **Cart abandonment emails** after a user visits checkout but doesn't complete
> - **Price comparison table** so users don't need to navigate back and forth
> - **Live chat / FAQ support** on the checkout page to address last-minute concerns
> - **Trust signals** — money-back guarantee badges, student testimonials

---

## 4. Subscription Type Differences

### What We See

**Annual Subscriber (User 10107):**
- 17 sessions before purchase
- Extensive exploration of Career tracks, Resources center, Courses
- Multiple Checkout visits (hesitation on a larger commitment)
- Journey spans many pages and interactions

**Monthly Subscriber (User 11145):**
- 3 sessions before purchase
- Short, direct journeys (Homepage → Log in)
- Less exploration, faster decision
- Lower financial commitment = quicker decision

### Interpretation

| Aspect | Annual Subscribers | Monthly Subscribers |
|--------|-------------------|-------------------|
| **Sessions before purchase** | Many (10+) | Few (1-5) |
| **Exploration depth** | High — browse many sections | Low — direct to login/checkout |
| **Checkout visits** | Multiple (hesitation) | Few (decisive) |
| **Decision time** | Longer consideration period | Shorter, more impulsive |
| **Financial commitment** | High ($199+/year) | Low ($29/month) |
| **Risk perception** | Higher (locked in for a year) | Lower (can cancel after a month) |

This makes intuitive sense: a higher-priced, longer commitment requires more research and consideration.

### Business Implication
> **Recommendation:** Tailor marketing strategies by subscription type:
> - **Annual prospects:** Provide detailed comparison pages, ROI calculators, student success stories, and money-back guarantees to reduce perceived risk
> - **Monthly prospects:** Quick, frictionless checkout — don't overwhelm with information, make it easy to start
> - **Upsell strategy:** After 2-3 months of monthly usage, present annual upgrade offers showing cost savings

---

## 5. Login Page Observations

### What We See
Many sessions consist almost entirely of `Log in` page interactions:
```
Log in-Log in-Log in-Log in-Log in-Log in-Log in-Log in
```

### Possible Interpretations

| Scenario | Likelihood | What It Means |
|----------|-----------|---------------|
| User scrolling/interacting on login page (form fields, etc.) | Very High | Each click on a form field generates a same-page event |
| User struggling to log in (wrong password, forgot credentials) | Medium | Friction in authentication flow |
| Page loading issues causing multiple interaction events | Low | Technical issue |

### Understanding Why (from the database description)
The `front_interactions` table records all events — *"from scrolling to clicking on buttons."* On a login page, typing in the email field, clicking the password field, toggling "remember me," and clicking "Log in" would each generate a separate event where source = destination = Login page.

### Business Implication
> **Recommendation:** While much of the login repetition is due to normal form interactions, investigate whether there are real friction points:
> - Implement **social login** (Google, GitHub, LinkedIn) to reduce form entry
> - Add **"Stay logged in"** functionality to reduce repeat login sessions
> - Check if **"Forgot Password"** flow is smooth and intuitive
> - Consider **biometric/passkey** authentication for returning users

---

## 6. Checkout-Only Sessions

### What We See
Several sessions contain ONLY checkout interactions:
```
Session 3656761: Checkout-Checkout
Session 3657049: Checkout-Checkout
Session 3657143: Checkout-Checkout
Session 3657160: Checkout-Checkout
```

### Interpretation
These users came to the site **specifically** to check the checkout page — likely:
- Checking the current price or looking for deals
- Comparing pricing plans (Monthly vs. Annual)
- Returning to complete an abandoned purchase
- Waiting for a promotional or exclusive price
- Verifying payment options

The fact that user 10107 had **6 checkout-only sessions** before finally purchasing their Annual subscription is particularly telling — this is a user who was very interested but needed something to push them over the edge.

### Business Implication
> **Recommendation:** These are **high-intent users** — they already know the platform and are specifically checking pricing. This is the most valuable segment for targeted interventions:
> - **Time-limited offers:** "Complete your purchase in the next 24 hours for 10% off"
> - **Personalized emails:** "We noticed you've been checking our plans — here's a special offer"
> - **Price anchoring:** Show annual savings prominently (e.g., "Save 40% vs. Monthly")
> - **Payment flexibility:** Offer installment plans for annual subscriptions
> - **Social proof on checkout:** "X students purchased this plan today"

---

## 7. Data Quality Observations

### Observation 1: Test Users ($0 Purchases)
The case description specifically warns: *"There might be some test records in the database. Filter them out by considering that a real user wouldn't pay $0 for the product, or the company would quickly go bankrupt."*

Our query handles this with `HAVING price > 0` in the `paid_users` CTE.

### Observation 2: Repeated Pages (Expected)
Consecutive duplicate pages are present throughout the data due to non-navigational interactions (scrolling, clicking form fields). This is explicitly mentioned as acceptable: *"For this task, you're not required to clean this up."*

### Observation 3: NULL User IDs in front_visitors
The case description notes: *"many NULL values here because many visitors never made an account and so were never assigned a user_id."* Our `INNER JOIN` approach naturally excludes these NULL records — only visitors with valid user_ids (who are also paying customers) are included.

### Observation 4: "Other" Category
Pages that don't match any known URL pattern are categorized as "Other." A high volume of "Other" pages could indicate:
- New website sections not covered in the `URL_Aliases.xlsx` mapping
- External redirect pages or landing pages
- Error pages (404, 500)

### Observation 5: Coupon vs. Regular Checkout
The separation of Coupon and Checkout pages allows future analysis to determine:
- What percentage of purchasers use coupon codes?
- Does using a coupon correlate with subscription type?
- Are coupon users more or less likely to renew?

---

## 8. Recommendations for Further Analysis

Based on the patterns observed in the extracted data:

### Immediate Analysis Opportunities

| Analysis | Question It Answers | Method |
|----------|-------------------|--------|
| **Page Frequency** | Which pages appear most in user journeys? | Count page occurrences in journey strings (Python) |
| **Journey Length** | How many pages do users visit before purchasing? | Count hyphens in journey strings |
| **Session Count** | How many sessions before purchase by subscription type? | Count sessions per user |
| **Entry Page Analysis** | Where do users start their journey? | Extract first page from each journey |
| **Exit Page Analysis** | What's the last page before purchase? | Extract last page from each journey |
| **Funnel Drop-off** | Where do users abandon the journey? | Track page transition frequencies |

### Advanced Analysis Opportunities

| Analysis | Question It Answers | Tool |
|----------|-------------------|------|
| **Sankey Diagram** | How does traffic flow between pages? | Python (Plotly) |
| **Sequence Mining** | What are the most common page sequences? | Python (mlxtend) |
| **User Clustering** | Are there distinct user journey archetypes? | Python (scikit-learn) |
| **Cohort Analysis** | Do journeys differ by registration month? | SQL + Python |
| **Subscription Comparison** | Do Monthly vs. Annual users follow different paths? | Python (pandas) |
| **Deduplication Analysis** | What do journeys look like without consecutive duplicates? | Python (itertools.groupby) |

---

## 9. Key Takeaways

### For the Business
1. **Homepage is king** — Most journeys start here; optimize it for conversion
2. **Multi-session journeys are normal** — Don't expect one-visit purchases, especially for Annual plans
3. **Checkout hesitation is real** — Users visit checkout multiple times before committing; reduce friction and add trust signals
4. **Login friction may exist** — Repeated login page interactions suggest potential authentication pain points
5. **Subscription type predicts behavior** — Annual buyers research extensively; Monthly buyers decide quickly

### For the Data Team
1. **Deduplication is the critical next step** — Remove consecutive duplicate pages for cleaner analysis
2. **Time dimension should be added** — Including `event_date` timestamps would enable time-on-page and session duration calculations
3. **The data is ready for visualization** — Sankey diagrams and funnel charts are the natural next step
4. **Journey clustering could segment users** — Different journey patterns may predict subscription type or conversion likelihood

### For the Product Team
1. **Simplify the checkout flow** — Multiple checkout visits suggest friction, confusion, or indecision
2. **Improve the login experience** — Consider SSO, social login, biometric authentication, or persistent sessions
3. **Add decision-support content** — Comparison pages, testimonials, ROI calculators, free trial extensions
4. **Implement retargeting** — Users who visit checkout and leave are prime candidates for retargeting ads and emails

---

## Conclusion

The extracted user journey data reveals a clear pattern: **users go through exploration → engagement → decision phases before purchasing**. The journey length and complexity vary significantly by subscription type, with Annual subscribers showing substantially more deliberation and checkout hesitation than Monthly subscribers.

The most impactful business actions based on this data would be:
1. **Reducing checkout friction** for hesitant Annual subscription prospects
2. **Implementing cart abandonment strategies** for users with multiple checkout visits
3. **Streamlining the login experience** to reduce authentication-related friction
4. **Creating tailored conversion paths** for different subscription types

This data extract serves as the foundation for deeper analysis — including funnel visualization, sequence mining, and user segmentation — that can directly inform product and marketing decisions.

---

*Analysis by Tushar Varma | February 2026*