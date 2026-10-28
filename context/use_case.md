# Use case (captured from the SE request)

The customer sells solutions to external customers and has organized Snowflake with one
database and one schema per end customer (tenant). Every tenant has the same table structures.
They want to maintain ONE AtScale semantic model and deploy it many times, once per tenant
database/schema. Proposed approach: SML environment variables.

Example domain chosen for the demo: tenant sales analytics (orders, customers, products, time).
Questions the model should answer for each tenant:
- Sales Amount, Cost, Gross Margin and Gross Margin % by year, quarter, month
- Sales by region / country / customer, and by product category / product
- Order Count, Customer Count, Average Order Value
- Sales vs prior year, YoY growth %, Year-to-date sales
