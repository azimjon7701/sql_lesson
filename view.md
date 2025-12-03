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
## Updatable VIEW
**Misol: faqat bitta supplier bo‘yicha ko‘rinish**
> Updatable VIEW – shunday oddiy viewki, unga INSERT/UPDATE/DELETE qilinsa, o‘zidagi bazaviy jadvalni o‘zgartiradi.
```sql
create or replace view v_fuelprice_shell as
select
    id,
    date,
    iata,
    amount
from fuelprice
where supplier = 'SHELL';
```
Endi (shartlar bajarilgani uchun) quyidagilar bazadagi fuelprice jadvalini o‘zgartiradi:
```sql
update v_fuelprice_shell
set amount = amount * 1.02
where iata = 'TAS';
```
>Updatable bo‘lishi uchun view ichida murakkab JOIN/agg bo‘lmasligi kerak (PostgreSQL cheklovlari mavjud):
* Faqat **bitta jadval**dan `FROM` bo‘lishi kerak (JOIN, subquery yo‘q).
* **`GROUP BY`, `DISTINCT`, `HAVING`, agg funksiyalar (`sum`, `avg`, ...)** bo‘lmasin.
* **`UNION / INTERSECT / EXCEPT`** kabi set operatorlar ishlatilmasin.
* SELECT’dagi ustunlar **jadval ustunlariga to‘g‘ridan-to‘g‘ri mapping** bo‘lsin (expression, `amount * 1.05` yo‘q).
* `FROM (subquery)` yoki self-join (`fuelprice f, fuelprice g`) bo‘lmasin.
