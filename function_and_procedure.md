# PostgreSQL FUNCTION va PROCEDURE (`LANGUAGE plpgsql`)

FUNCTION – qiymat qaytaradigan server tomondagi kod.  
PROCEDURE – qiymat qaytarmaydi, `CALL` bilan chaqiriladi, ko‘proq transaction-ishlar uchun.

- **Oddiy (scalar) FUNCTION – qiymat qaytaradigan funksiya**
- **Scalar FUNCTION – validation va `RAISE EXCEPTION` bilan**
- **Set-returning FUNCTION (`RETURNS TABLE` + `RETURN QUERY`)**
- **IMMUTABLE / STABLE / VOLATILE – plpgsql funksiyalar uchun**
- **PROCEDURE – `CALL` bilan chaqiriladigan plpgsql kod**
- **FUNCTION va PROCEDURE taqqoslanishi (PostgreSQL kontekstida)**


## 1. Oddiy (scalar) FUNCTION – qiymat qaytaradigan

> Scalar function – bitta qiymat (son, text, date va h.k.) qaytaradi.

**Masala:** berilgan IATA uchun eng oxirgi `amount` qiymatini qaytarish.

```sql
create or replace function get_latest_fuelprice(p_iata varchar)
returns double precision
language plpgsql
stable
as
$$
declare
    v_amount double precision;
begin
    select amount
    into v_amount
    from fuelprice
    where iata = p_iata
    order by date desc, id desc
    limit 1;

    return v_amount;
end;
$$;
```

Foydalanish:

```sql
select get_latest_fuelprice('TAS');
```

---

## 2. Scalar FUNCTION – validation va exception bilan

> Exception bilan function – xato holatda meaningful xabar bilan yiqiladi.

```sql
create or replace function get_latest_fuelprice_checked(p_iata varchar)
returns double precision
language plpgsql
stable
as
$$
declare
    v_amount double precision;
begin
    select amount
    into v_amount
    from fuelprice
    where iata = p_iata
    order by date desc, id desc
    limit 1;

    if v_amount is null then
        raise exception 'No fuel price found for IATA: %', p_iata;
    end if;

    return v_amount;
end;
$$;
```

Foydalanish:

```sql
select get_latest_fuelprice_checked('TAS');
```

---

## 3. Set-returning FUNCTION (`RETURNS TABLE` + `RETURN QUERY`)

> Set-returning function – ko‘p qatorlik natija qaytaradi, `SELECT` kabi ishlatiladi.

**Masala:** berilgan IATA bo‘yicha oxirgi N ta yozuvni qaytarish.

```sql
create or replace function get_last_n_fuelprices(
    p_iata  varchar,
    p_limit int
)
returns table (
    date        date,
    supplier    varchar,
    amount      double precision
)
language plpgsql
stable
as
$$
begin
    return query
    select
        f.date,
        f.supplier,
        f.amount
    from fuelprice f
    where f.iata = p_iata
    order by f.date desc, f.id desc
    limit p_limit;
end;
$$;
```

Foydalanish:

```sql
select * from get_last_n_fuelprices('TAS', 5);
```

---

## 4. IMMUTABLE / STABLE / VOLATILE (plpgsql uchun ham ishlaydi)

> Bu belgilashlar optimizer’ga function qanchalik o‘zgaruvchanligini aytadi.

* `IMMUTABLE` – bir xil parametrlar uchun natija doim bir xil (masalan: matematik hisob).
* `STABLE` – bitta query ichida natija o‘zgarmaydi, lekin vaqt o‘tishi bilan o‘zgarishi mumkin (bizning fuelprice funksiyalari).
* `VOLATILE` – har chaqirilganda natija o‘zgarishi mumkin (`now()`, `random()`).

Yuqoridagi funksiyalarda biz `stable` deb belgiladik.

---

## 5. PROCEDURE (`CALL` bilan chaqiriladi)

> PROCEDURE – `CALL` orqali ishlaydi, `RETURNS` yo‘q, ko‘proq transaction ichida DML (insert/update/delete) bajarish uchun.

Log jadvali:

```sql
create table fuelprice_log
(
    id            serial primary key,
    run_at        timestamptz default current_timestamp,
    rows_affected int
);
```

**Masala:** 1 yildan eski narxlarni 5% oshirish va nechta qator o‘zgarganini logga yozish.

```sql
create or replace procedure adjust_old_fuelprices()
language plpgsql
as
$$
declare
    v_count int;
begin
    update fuelprice
    set amount = amount * 1.05
    where date < current_date - interval '1 year';

    get diagnostics v_count = row_count;

    insert into fuelprice_log(rows_affected)
    values (v_count);
end;
$$;
```

Foydalanish:

```sql
call adjust_old_fuelprices();

select *
from fuelprice_log
order by run_at desc;
```

---

## 6. FUNCTION vs PROCEDURE (PostgreSQL, `plpgsql`)

> Qisqa taqqoslash, faqat PostgreSQL kontekstida.

* **FUNCTION (plpgsql)**

  * `select get_latest_fuelprice('TAS');`
  * Har doim **qiymat qaytaradi** (`returns ...`).
  * Query ichida (`select`, `where`, `join`) erkin ishlatiladi.
  * `language plpgsql` + `immutable/stable/volatile`.

* **PROCEDURE (plpgsql)**

  * `call adjust_old_fuelprices();`
  * **Hech narsa qaytarmaydi**, faqat side-effect (insert/update/delete/log).
  * Faqat `CALL` sifatida ishlatiladi, `select` ichida chaqirib bo‘lmaydi.
  * Katta transaction flow’larini (ketma-ket bir nechta DML) joylashtirish uchun qulay.

