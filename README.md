# Sales Analytics - multi-tenant template repository

One SML repository, deployed once per tenant. Only the connection file is tenant-specific:
its data warehouse connection, database and schema are SML environment variables
(SF_CONNECTION, TENANT_DB, TENANT_SCHEMA) resolved at deploy time from the repository .env
file or the engine-level sml.environment.variables setting. Datasets reference bare table
names, so they inherit the tenant database/schema from the connection.

## Inputs
- context/use_case.md - the SE's description of the customer requirement plus the demo questions.
- context/ddl.sql - Snowflake DDL, identical in every tenant schema.
- context/data_profile.yaml - profile of the ACME tenant sample data (keys and cardinalities).
- NLQs - embedded in use_case.md.
- ERD - (not provided; relationships taken from the declared foreign keys in the DDL).
- build.yaml - (not provided).
- Target warehouse - Snowflake.

## Build parameters
- unrelated_dimension_handling: repeat
- warehouse: Snowflake
- model_unique_name: sales_analytics
- catalog_unique_name: sales_analytics_catalog (overridden per tenant at deploy time)
- currency: USD
- time_window: calendar
- use_cases_covered: sales, margin, orders/customers, prior-year, YoY, YTD
- use_cases_excluded: none
- semi_additive_default: none (no snapshot facts)

## Assumptions and decisions
- Connection database/schema/as_connection are environment variables on purpose. This is the
  documented AtScale mechanism for deploying one model to several environments; the skill's
  placeholder rule (literal DDL tokens) is intentionally overridden here. The validator flags
  these three values on the template; rendered tenant copies validate clean.
- Variables are used ONLY in the connection file. AtScale allows them in connection properties
  (not unique_name/object_type) and in dataset sql/dialects sql, but NOT in a dataset table
  property, so table datasets use unqualified names.
- Catalog unique_name/label cannot be variables; tenant naming is done with the sml-cli
  deploy flags catalog-name/catalog-label, and repository-id pins a distinct identity per tenant.
- is_unique_key set on Day, Customer Name and Product Name: their keys are the table primary
  keys (distinct = row_count in the profile).
- PRIOR_YEAR_NUM calculated column + parallel_periods on Quarter/Month/Day for ParallelPeriod.
- Single date role (order date), so no role-play prefix.
- Customer Segment and Product SKU exposed as secondary attributes.

## Generation summary
- Datasets: 4 (DIM_DATE, DIM_CUSTOMER, DIM_PRODUCT, FACT_SALES)
- Dimensions: 3 (Date - time; Customer - Region > Country > Customer; Product - Category > Product)
- Base metrics: 6; calculated metrics: 6; model relationships: 3
- Calculated columns: PRIOR_YEAR_NUM on DIM_DATE
- Role-play prefixes: none; snowflake bridges: none

## Reproducing this build
The context folder holds verbatim copies of every input used for this build (use case, DDL,
data profile). Re-running the generator with the same inputs and build parameters produces an
equivalent model. Per-tenant values live outside the repo, in deploy/tenants.csv.
