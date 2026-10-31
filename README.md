# goit-rdb-hw-05

Домашнє завдання 5 з курсу реляційних баз даних (GoIT Neoversity), тема «CTE і проєктування ознак для ML». На даних Olist будую typed-схему, CTE-pipeline ознак для кожного customer, recursive CTE для referral-ланцюжка і materialized view з `REFRESH CONCURRENTLY`, а потім порівнюю плани через `EXPLAIN (ANALYZE, BUFFERS)`.

## Датасет

[Olist Brazilian E-Commerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle `olistbr/brazilian-ecommerce`): замовлення бразильського маркетплейсу за 2016-2018 роки.

Notebook сам качає дані через `kagglehub.dataset_download('olistbr/brazilian-ecommerce')`. Датасет публічний, тому Kaggle credentials не потрібні: завантаження перевірено на машині без `~/.kaggle/kaggle.json` і без змінних `KAGGLE_*`. CSV не лежать у репозиторії, kagglehub тримає їх у своєму кеші.

З 9 CSV в архіві використано 5:

| CSV | Raw-таблиця | Typed-таблиця | Рядків (raw = typed) |
|---|---|---|---|
| `olist_customers_dataset.csv` | `olist_customers_raw` | `olist_customers` | 99 441 |
| `olist_orders_dataset.csv` | `olist_orders_raw` | `olist_orders` | 99 441 |
| `olist_order_items_dataset.csv` | `olist_order_items_raw` | `olist_order_items` | 112 650 |
| `olist_products_dataset.csv` | `olist_products_raw` | `olist_products` | 32 951 |
| `olist_sellers_dataset.csv` | `olist_sellers_raw` | `olist_sellers` | 3 095 |

Smoke-test показує `diff = 0` для всіх п'яти таблиць.

Інші об'єкти в базі:
- `hw5_referrals`: синтетична таблиця з 9 пар referrer → referred (в Olist рефералів немає);
- `mv_olist_customer_features`: materialized view з ознаками, 99 441 рядків, `UNIQUE INDEX` на `customer_id` і два індекси на `monetary`.

## Головна особливість даних

В Olist `customer_id` видається на кожне замовлення: 99 441 `customer_id`, і в жодного немає більше одного замовлення. Людину позначає `customer_unique_id` (96 096, з них 2 997 замовляли 2+ рази). Pipeline рахує ознаки по `customer_id`, як вимагає умова, тому `frequency` у всіх дорівнює 1. Перевірку і варіант для production описано в розділі 1.6 notebook.

## Де що в notebook

| Завдання | Розділ |
|---|---|
| 1. Setup і typed-схема | pgserver і `SELECT version()`, kagglehub, raw-шар, DDL з PK/FK/CHECK, `INSERT ... SELECT`, smoke-test raw vs typed, перевірка зерна `customer_id` vs `customer_unique_id` |
| 2. CTE-pipeline | таблиця stages, pipeline з Stage 0-7 і `LIMIT 20`, перевірка кількості рядків кожного stage |
| 3. Recursive CTE | `hw5_referrals`, глибина і шлях у referral-tree, демо циклу A → B → A з guard і з `CYCLE` |
| 4. Materialized View | `mv_olist_customer_features`, `UNIQUE INDEX`, top-5, профіль ознак, `REFRESH ... CONCURRENTLY` |
| 5. EXPLAIN і Reflection | плани RFM-частини, повного pipeline, lookup з MV і top-N з MV, таблиця порівняння, Reflection |

## Як запустити

**Google Colab:** відкрити `RDB_HOMEWORK5.ipynb`, далі Runtime → Run all. Перша клітинка ставить `pixeltable-pgserver==0.4.0`, `psycopg2-binary`, `sqlalchemy`, `pandas`, `kagglehub` і `jupysql`. PostgreSQL 16 запускається прямо в runtime (`/tmp/hw5_pg`), окремий сервер не потрібен. Оригінальний пакет `pgserver` на поточній версії Python у Colab не встановлюється, а цей форк має той самий API.

**Локально:** потрібен [uv](https://docs.astral.sh/uv/) і доступ до інтернету для kagglehub.

```bash
uv venv --python 3.12 .venv
uv pip install --python .venv/bin/python jupyter nbconvert ipykernel pip
.venv/bin/jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=600 RDB_HOMEWORK5.ipynb
```

`pip` у venv потрібен, бо перша клітинка ставить пакети через `%pip install`. Повторний запуск безпечний: notebook перестворює всі таблиці і MV.

## Файли

| Файл | Що всередині |
|---|---|
| `RDB_HOMEWORK5.ipynb` | notebook з усіма завданнями, output клітинок і поясненнями |
| `README.md` | цей опис |
| `.gitignore` | службові файли, venv і локальні нотатки |
