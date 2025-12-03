# CTE (WITH – Common Table Expression)
> CTE – bu WITH bilan yoziladigan nomlangan subquery, kodni bo‘lib-bo‘lib o‘qishni yengillashtiradi.
- **Oddiy CTE**
- **Recursive CTE**
- **Data-modifying CTE**
- **Materialized CTE**
## Oddiy CTE
> Oddiy CTE – faqat `SELECT` dan iborat, murakkab query’ni bosqichlarga bo‘lib yozish uchun ishlatiladi.

```sql
with avg_price as (
    select
        iata,
        avg(amount) as avg_amount
    from fuelprice
    group by iata
)
select *
from avg_price
order by avg_amount desc;
```
## Recursive CTE
> Recursive CTE – with recursive bilan yoziladi va o‘zini o‘zi chaqiradi; daraxt/hierarxiya va ketma-ketliklar uchun ishlatiladi.

```sql
with recursive nums as (
    select 1 as n
    union all
    select n + 1
    from nums
    where n < 10
)
select * from nums;
```

## Data-modifying CTE
> Data-modifying CTE – CTE ichida INSERT / UPDATE / DELETE ... RETURNING ishlatiladigan, transaction ichida “o‘zgartir + ko‘rsat” patterni.
```sql
with updated as (
    update fuelprice
    set amount = amount * 1.05
    where date < current_date - interval '1 year'
    returning iata, date, amount
)
select *
from updated
order by date, iata;
```

## Materialized CTE (PostgreSQL optimizatsiyasi)
> Materialized CTE – query planner’ga CTE natijasini vaqtinchalik saqlab, keyin ishlatishni buyuradi (performance nazorati).
```sql
with materialized avg_price as (
    select
        iata,
        avg(amount) as avg_amount
    from fuelprice
    group by iata
)
select *
from avg_price
where avg_amount > 1000;
```
