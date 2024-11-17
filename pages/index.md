---
title: Q1/Q2 2019 Chargeback Report - Globepay
---

## Scope
- This is an overview of transactions processed using the Globepay API
- The data has been processed in [dbt](https://github.com/alexandredantec/dbt_payment_data)
- For analysis, I chose to test [evidence.dev](https://evidence.dev/)


## Question 1: What is the acceptance rate over time?
- In order to answer this question, time granularity matters: 
- Weekly data offers the most balanced overview to observe variation

- The acceptance rate is calculated by dividing the total number of accepted transactions by the overall number of transactions 
- SQL code

```sql acceptance_rate
with base as (
  select
  transaction_week
  ,count(*) as total
  ,sum(case when transaction_state = 'ACCEPTED' then 1 else 0 end) as total_accepted
  from datamodelingchallenge.transactions
  group by 1 
)

select
transaction_week
,round(total_accepted/total, 3) as acceptance_rate 
from base 
order by 1 
```
<LineChart
    data={acceptance_rate}
    title="Acceptance Rate by Week"
    x=transaction_week
    y=acceptance_rate
/>

- The acceptance rate remained somewhat stable, fluctuating between 0.8 and 0.6. It is difficult to identify any sort of seasonality. 
- The variation by country is also remarkably stable (displayed by month to keep chart organized). 
- SQL Code

```sql acceptance_rate_by_country
with base as (
  select
  transaction_month
  ,transaction_country
  ,count(*) as total
  ,sum(case when transaction_state = 'ACCEPTED' then 1 else 0 end) as total_accepted
  from datamodelingchallenge.transactions
  group by 1,2 
)

select
transaction_month
,transaction_country
,round(total_accepted/total, 3) as acceptance_rate 
from base 
order by 1 
```
<LineChart
    data={acceptance_rate_by_country}
    title="Acceptance Rate by Country by Month"
    x=transaction_month
    y=acceptance_rate
    series = transaction_country
/>

## Question 2: What are the countries where the amount of declined transactions went over $25M?
- The chargeback rate is calculated by computing the total amount in USD for transactions where `is_chargeback is true` 
- SQL code

```sql chargebacks_by_country_usd
  select 
    transaction_country
    ,sum(case when is_chargeback then usd_amount else 0 end) as total_chargebacks
  from datamodelingchallenge.transactions
  group by 1 
```

<BarChart
    data={chargebacks_by_country_usd}
    title="Chargebacks by Country"
    x=transaction_country
    y=total_chargebacks
/>

- There was no country where the total chargeback value exceeded 350K during the period of observation
- Obtaining more data would be useful in order to see whether any country reached 25 Million USD in chargeback losses

## Question 3. Which transactions are missing chargeback data?

- The missing chargeback data can be identified by checking transactions where the column `is_chargeback` is null, i.e. it is neither `true` nor `false`
- In the preliminary exploration, it was already shown that this will not be the case as there is a 1-1 relationship between chargebacks and transactions
- SQL code

```sql missing_chargebacks
  select 
      count(transaction_id) as missing_chargeback_total
  from datamodelingchallenge.transactions
  where is_chargeback is null
```
<BigValue 
  data={missing_chargebacks} 
  value=missing_chargeback_total
/>

- As expected, the total is 0 
- To go further, we can look at the total number of chargebacks
- SQL Code

```sql total_chargebacks
  select 
      count(transaction_id) as total_chargebacks
  from datamodelingchallenge.transactions
  where is_chargeback is true
```

<BigValue 
  data={total_chargebacks} 
  value=total_chargebacks
/>

- Let's now look at the total number of chargebacks by country
- SQL Code

```sql chargebacks_by_country_total
  select 
      transaction_country
      ,sum(case when is_chargeback then 1 else 0 end) as total_chargebacks
  from datamodelingchallenge.transactions
  group by 1 
```

<BarChart
    data={chargebacks_by_country_total}
    title="Chargebacks by Country"
    x=transaction_country
    y=total_chargebacks
/>

## What's Next?
- With more data, it would be interesting to look at the seasonality of chargebacks 
- It would also be great to be able to look at chargebacks by payment type, or according to whether a CVV was required 

