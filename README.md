# ДЗ 5 · CTE та materialized view: ознаки Olist

SQL-конвеєр ознак покупців: послідовність CTE, 30-денне вікно, рекурсивний обхід, матеріалізоване представлення та плани EXPLAIN.

[Відкрити у Google Colab](https://colab.research.google.com/github/humanistitel/goit-rdb-hw-05/blob/main/Diuryk_hw5_olist_features.ipynb) · [Завантажити notebook](https://raw.githubusercontent.com/humanistitel/goit-rdb-hw-05/main/Diuryk_hw5_olist_features.ipynb)

## Запуск

У Colab оберіть CPU та натисніть **Runtime → Run all**. Перша клітинка готує залежності й тимчасовий PostgreSQL 16; остання зупиняє сервер. Перший запуск потребує інтернету. Якщо виконання перервано, запустіть останню клітинку для зупинки сервера.

Локально: Jupyter та Python 3.9–3.12.

## Дані

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), Olist, ліцензія [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).

Використано повні таблиці customers (99 441 рядок), orders (99 441), order_items (112 650), products (32 951) та sellers (3 095). Відсутні CSV завантажуються з [офіційного Kaggle ZIP, версія 2](https://www.kaggle.com/api/v1/datasets/download/olistbr/brazilian-ecommerce?datasetVersionNumber=2), storage version `55151/2669146`. SHA-256 ZIP: `967e41e04fc306fe604e2a693f488995a8b41e5047418f8a5c8e4abd6deca784`.

## Конвеєр

Ознаки включають recency, кількість і суму замовлень, когорту, 30-денне середнє, частку скасувань та пріоритетну категорію. Один рядок відповідає `customer_id`; повторного покупця визначає окремий ключ `customer_unique_id`. Суми враховують усі статуси та доставку.

Рекурсивний CTE обходить синтетичний граф запрошень. `mv_olist_customer_features` зберігає весь конвеєр; унікальний індекс дозволяє concurrent refresh. У notebook наведено плани повного конвеєра й пошуку одного клієнта, а в Reflection — щодобове оновлення для пакетної сегментації та вартість перерахунку.
