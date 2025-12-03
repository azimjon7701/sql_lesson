# VIEW (PostgreSQL Viewlar)

VIEW – bu `SELECT` natijasini nom bilan saqlab qo‘yadigan **virtual jadval**, murakkab query’ni qayta-qayta ishlatishni osonlashtiradi.

- **Oddiy VIEW**
- **Updatable VIEW**
- **Materialized VIEW**
- **Security VIEW**

---

## Oddiy VIEW

> Oddiy VIEW – faqat o‘qish uchun mo‘ljallangan, har `SELECT`da jonli hisoblanadigan virtual jadval.

**Misol: har bir IATA bo‘yicha eng oxirgi narx**

```sql
create or replace view v_fuelprice_latest as
select
    f.id,
    f.date,
    f.iata,
    f.supplier,
    f.amount,
    f.description,
    f.created_at
from fuelprice f
join (
    select iata, max(date) as max_date
    from fuelprice
    group by iata
) last_price
    on last_price.iata = f.iata
   and last_price.max_date = f.date;
```
