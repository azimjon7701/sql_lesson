# CTE (WITH – Common Table Expression)
CTE – bu WITH bilan yoziladigan nomlangan subquery, kodni bo‘lib-bo‘lib o‘qishni yengillashtiradi.
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
