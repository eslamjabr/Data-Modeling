# Data Modeling Project: End-to-End Guide

A practical guide to transforming a messy Power BI model into a clean,
reliable star schema.

## Project Setup

### Phases

1.  **Prepare & Explore** --- Understand the source data and existing
    model.
2.  **Dimensions** --- Build clean, reusable dimensions.
3.  **Facts** --- Build fact tables, define their grain, and test
    relationships.
4.  **Polish** --- Secure, validate, document, and hand over the model.

### Core Principles

-   Build a star schema.
-   Define the grain before creating relationships.
-   Keep only columns and tables that serve a reporting need.
-   Validate important business numbers after every major change.
-   Protect the core model from unnecessary complexity.

### Naming Standards

  -----------------------------------------------------------------------
  Item                    Standard                Example
  ----------------------- ----------------------- -----------------------
  Language                English                 `customer_name`,
                                                  `order_date`

  Naming style            `snake_case`            `total_revenue`

  Dimension tables        `dim_` prefix           `dim_customer`

  Fact tables             `fact_` prefix          `fact_sales`

  Keys                    `_key` or `_id` suffix  `customer_key`,
                                                  `product_id`

  Business-facing fields  Clear, readable names   `order_status`,
                                                  `total_revenue`
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## Phase 1: Prepare & Explore

**Goal:** Understand the business, source data, and existing model
before making changes.

### Steps

1.  Open and inspect the existing model.
2.  Prepare a workspace and identify the source tables.
3.  Understand the business process being analyzed.
4.  Review table columns, data types, keys, and relationships.
5.  Identify likely dimensions, facts, and supporting tables.

### Questions to Answer

-   What business process does the data represent?
-   What are the main business entities?
-   Which tables describe entities, such as customers, products, or
    dates?
-   Which tables record events or measurable activity?
-   What does one row in each table represent?
-   Which columns identify records, and are those values unique?
-   Are there duplicate records, missing keys, inconsistent data types,
    or unclear column names?

**Deliverable:** A source-table inventory that records each table's
purpose, grain, keys, and likely role in the model.

------------------------------------------------------------------------

## Phase 2: Build Dimensions

**Goal:** Create clean dimensions that describe and categorize business
events.

Use the same three-step process for each dimension.

### The Dimension Loop

1.  **Collect:** Group source tables that describe the same business
    entity.
2.  **Combine:** Merge, append, split, reshape, or clean the data as
    appropriate.
3.  **Standardize:** Apply naming conventions, correct data types,
    resolve duplicates, and remove unnecessary columns.

### Steps

1.  Group source tables by entity, such as customer, product, campaign,
    geography, or date.
2.  Determine the business key and, where necessary, create a surrogate
    key.
3.  Combine related sources into one well-defined dimension.
4.  Ensure the dimension key is unique and nonblank.
5.  Standardize names and data types.
6.  Remove columns that are not required for reporting.
7.  Repeat for each dimension.

### Common Dimensions

-   `dim_customer` --- customer attributes, such as name, segment, and
    region.
-   `dim_product` --- product name, category, brand, and other
    descriptive attributes.
-   `dim_date` --- date, month, quarter, year, and other calendar
    attributes.
-   `dim_campaign` --- campaign name, channel, and campaign dates.
-   `dim_geo` --- geography attributes when geography needs to be
    modeled separately.

**Validation:** Each dimension should have a clear purpose, one row per
dimension key, and descriptive attributes that do not create ambiguous
relationships.

------------------------------------------------------------------------

## Phase 3: Build Fact Tables

**Goal:** Model business events at a clearly defined grain and connect
them to the appropriate dimensions.

### Know the Grain

Before building a fact table, write one sentence that defines what a
single row represents.

Examples:

-   `fact_sales`: one row per sales line.
-   `fact_campaign_spend`: one row per campaign and reporting date, at
    the source's actual level of detail.
-   `fact_inventory`: one row per product, location, and snapshot date.
-   `fact_sales_targets`: one row per target period and target entity,
    such as region or product category.
-   `fact_order_fulfillment`: one row per order, if each order is
    tracked through its fulfillment milestones.

Never connect tables blindly. Confirm that the keys and grain on both
sides support the intended relationship.

### Steps

1.  Choose a business process or event.
2.  Define its grain in plain language.
3.  Build the fact table from the source records at that grain.
4.  Keep the measures that belong to that process.
5.  Add the appropriate dimension keys.
6.  Connect each fact to its dimensions using valid keys.
7.  Check for orphan keys, duplicate rows, and unexpected row-count
    changes.
8.  Reconcile totals with the source system.

### Common Fact Types

#### Transaction Fact

Records individual business events or transactions, such as sales lines,
invoice lines, or payments.

Example: `fact_sales`

#### Periodic Snapshot Fact

Records measurements at regular intervals, such as daily inventory
balances or monthly account balances.

Example: `fact_inventory`

#### Accumulating Snapshot Fact

Tracks a process through milestones in one row per process instance. It
can contain multiple milestone dates, such as order, packing, shipping,
delivery, and return dates.

Example: `fact_order_fulfillment`

#### Factless Fact

Records that an event, relationship, or coverage condition occurred,
even when there is no numeric measure.

Example: `fact_promotion_coverage`, which tracks products covered by
promotions, potentially by store and date.

### Date Relationships

A single `dim_date` can support multiple facts, including:

-   `fact_sales`
-   `fact_invoices`
-   `fact_shipments`
-   `fact_campaign_spend`
-   `fact_sales_targets`

A fact may contain multiple date roles, such as `order_date`,
`ship_date`, and `delivery_date`. In Power BI, use role-playing date
dimensions or carefully managed active and inactive relationships with
explicit measures. Make sure report behavior is understandable and
documented.

------------------------------------------------------------------------

## Phase 4: Polish, Secure & Validate

**Goal:** Keep the model understandable, performant, secure, and
trustworthy.

### Simplify the Model

-   Remove tables and columns that do not support a real reporting need.
-   Avoid keeping fields merely "just in case."
-   Hide technical keys and helper columns from report authors when they
    are not useful for direct analysis.
-   Use explicit DAX measures for business calculations where
    appropriate.
-   Keep special-case or operational requirements from unnecessarily
    complicating the core reporting model.

### Build Core Measures

Examples of useful measures include:

``` dax
Total Sales =
SUM ( fact_sales[line_total] )
```

``` dax
Total Quantity =
SUM ( fact_sales[quantity] )
```

``` dax
Total Orders =
DISTINCTCOUNT ( fact_sales[order_id] )
```

``` dax
Total Customers =
DISTINCTCOUNT ( fact_sales[customer_id] )
```

``` dax
Average Order Value =
DIVIDE ( [Total Sales], [Total Orders] )
```

Adjust table and column names to match the actual model. Confirm the
business definition of each measure---for example, whether sales include
tax, discounts, returns, or canceled orders. Distinct customer and order
counts should be evaluated at the intended business grain.

### Configure Row-Level Security (RLS)

1.  Define which users can access which business areas.
2.  Create roles and filter rules, for example by region.
3.  Use a user-to-region mapping when access rules are more complex than
    a single field filter.
4.  Test each role using Power BI's **View as** functionality.
5.  Verify that users can see only the data they are authorized to
    access.
6.  Test interactions across all related fact tables, not just one
    visual.

For example, a North America user should see only authorized North
America data, while a Europe user should see only authorized Europe
data. Validate the actual mapping and role behavior rather than relying
on the role definition alone.

### Validate the Numbers

Compare the model against trusted source-system or business totals.
Example checkpoints might include:

-   Total sales: €24.3M
-   Total orders: 1,284,402
-   Active customers: 48,217

These figures are illustrative checkpoints, not expected results for
every project. Replace them with verified values from the actual source
data.

Also validate:

-   Row counts before and after transformations.
-   Duplicate dimension keys.
-   Fact rows with missing or unmatched dimension keys.
-   Totals by month, region, product, and other important slices.
-   Date coverage and relationships.
-   Filter direction and relationship cardinality.
-   Measures under different filter contexts.
-   RLS behavior for representative users.

### Handover Documentation

Provide a short reporting guide that explains:

-   The purpose and grain of each fact table.
-   The meaning of each dimension.
-   Approved measures and their business definitions.
-   Which fields report authors should use for slicing and filtering.
-   Known limitations and data refresh timing.
-   Security roles and how access is tested.

------------------------------------------------------------------------

## Recommended Star Schema

The central fact table stores measurable events and foreign keys.
Dimensions surround it and provide descriptive context.

``` text
                 dim_customer
                      |
dim_date ---- fact_sales ---- dim_product
                      |
                 dim_campaign
```

### Example Tables

-   `fact_sales`
    -   Keys: `customer_key`, `product_key`, `date_key`, `campaign_key`
    -   Measures: `quantity`, `amount`, `discount`
-   `dim_customer`
    -   `customer_key`, `customer_name`, `city`, `region`
-   `dim_product`
    -   `product_key`, `product_name`, `category`
-   `dim_date`
    -   `date_key`, `date`, `month`, `quarter`, `year`
-   `dim_campaign`
    -   `campaign_key`, `campaign_name`, `channel`

The exact keys and measures depend on the source data and the declared
grain. Use consistent key types and ensure every fact foreign key maps
to the intended dimension.

------------------------------------------------------------------------

## Final Checklist

-   [ ] Business process and grain are documented for every fact.
-   [ ] Dimensions have unique, nonblank keys.
-   [ ] Fact-to-dimension relationships use the intended cardinality.
-   [ ] Filter directions are deliberate; unnecessary bidirectional
    paths are avoided.
-   [ ] Date dimensions cover the required reporting period.
-   [ ] Measures have documented business definitions.
-   [ ] Source totals reconcile with model totals.
-   [ ] Unmatched keys and unexpected duplicates are investigated.
-   [ ] Row-level security is tested using representative users.
-   [ ] Unnecessary columns and tables are removed or hidden
    appropriately.
-   [ ] Naming standards are applied consistently.
-   [ ] The reporting guide is ready for handover.

## Guiding Principle

**Understand the data, define the grain, model the business process, and
validate the numbers.** A good model is not the one with the most
tables; it is the simplest reliable model that answers the required
business questions.
