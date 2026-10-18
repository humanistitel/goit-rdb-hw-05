# ДЗ 5 · CTE та materialized view: кеш ознак Olist

Матеріали домашньої роботи з курсу **Relational Databases for Data Science**.

[Відкрити notebook у Google Colab](https://colab.research.google.com/github/humanistitel/goit-rdb-hw-05/blob/main/Diuryk_hw5_olist_features.ipynb) · [Завантажити `.ipynb`](https://raw.githubusercontent.com/humanistitel/goit-rdb-hw-05/main/Diuryk_hw5_olist_features.ipynb)

## Google Colab

1. Відкрийте посилання Colab вище у стандартному **Python 3 / CPU** runtime.
2. Виконайте **Runtime → Run all**. Notebook сам підготує залежності, власний PostgreSQL 16 і потрібні дані. Додаткові клітинки або ручне копіювання файлів не потрібні.

Перший запуск потребує інтернету й часу на встановлення PostgreSQL та отримання даних. Сумісні встановлені бібліотеки зберігаються; фактичні версії відображаються під час виконання. Після завершення збережіть notebook із результатами. Якщо виконання перервано після запуску PostgreSQL, виконайте останню клітинку cleanup.

Файл `.ipynb` також можна завантажити та відкрити через **File → Upload notebook**.

## Запуск і залежності

Усі SQL і runtime-код вбудовано в notebook; зовнішні генератори не потрібні.

У стандартному Colab з Python 3.13 PostgreSQL 16 встановлюється із системних пакетів Ubuntu й працює у власному кластері через локальний Unix socket. Зовнішня база або credentials не потрібні. Для локального запуску підтримується Python 3.9–3.12 з `pgserver==0.1.4`. Остання клітинка зупиняє власний PostgreSQL.

## Джерело й отримання даних

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), автор Olist, ліцензія [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Дані не додаються до Git і не отримують іншу ліцензію від цього репозиторію.

За відсутності CSV notebook сам отримує [закріплений офіційний Kaggle ZIP, datasetVersionNumber=2](https://www.kaggle.com/api/v1/datasets/download/olistbr/brazilian-ecommerce?datasetVersionNumber=2), перевіряє SHA-256, витягує тільки потрібні таблиці в `data/olist/`, перевіряє їхні hashes, заголовки й кількість логічних записів. Оригінальний ZIP має 44 717 580 байтів; SHA-256 `967e41e04fc306fe604e2a693f488995a8b41e5047418f8a5c8e4abd6deca784` (зафіксована storage version `55151/2669146`). Дані не обрізаються до довільного sample.

| CSV | Рядків без заголовка |
| --- | ---: |
| `olist_customers_dataset.csv` | 99 441 |
| `olist_orders_dataset.csv` | 99 441 |
| `olist_order_items_dataset.csv` | 112 650 |
| `olist_products_dataset.csv` | 32 951 |
| `olist_sellers_dataset.csv` | 3 095 |

Якщо офіційний endpoint недоступний, використайте відповідні перевірені CSV з власного локального пакета в `data/olist/`; hashes мають збігатися. Kaggle credentials у цьому репозиторії не зберігаються.

Сирий шар зберігає вхідні CSV-значення; SQL формує типізований шар і CTE-конвеєр ознак. Є перевірки покриття стадій, recursive CTE, materialized view, індекс, refresh і фактичні EXPLAIN-плани. Ознаки сформовано за явно вказаними правилами часу й зерна. Reflection обґрунтовує щодобовий refresh для пакетної сегментації та пріоритет вартості перерахунку.
