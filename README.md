# ДЗ 5 · CTE та materialized view: кеш ознак Olist

Матеріали домашньої роботи з курсу **Relational Databases for Data Science**.

[Відкрити notebook у Google Colab](https://colab.research.google.com/github/humanistitel/goit-rdb-hw-05/blob/main/Diuryk_hw5_olist_features.ipynb) · [Завантажити `.ipynb`](https://raw.githubusercontent.com/humanistitel/goit-rdb-hw-05/main/Diuryk_hw5_olist_features.ipynb)

## Google Colab

1. Відкрийте посилання Colab вище. За потреби збережіть копію notebook у потрібній папці Google Drive.
2. Підключіть Python runtime і виконайте клітинку підготовки робочої папки перед основним кодом notebook:

```python
from pathlib import Path
import os
import subprocess

repo_dir = Path("/content/goit-rdb-hw-05")
if not repo_dir.exists():
    subprocess.run([
        "git", "clone", "--depth", "1",
        "https://github.com/humanistitel/goit-rdb-hw-05.git", str(repo_dir)
    ], check=True)
os.chdir(repo_dir)
print(Path.cwd())
```

3. Виконайте **Runtime → Restart session and run all**, залишивши клітинку підготовки перед основним кодом. Перший запуск встановлює залежності та потребує мережі.
4. Збережіть notebook із результатами. Якщо виконання перервано після створення PostgreSQL, запустіть останню клітинку cleanup.

Пряме посилання на raw-файл також дозволяє завантажити `.ipynb` і відкрити його через **File → Upload notebook**; підготовка робочої папки залишається такою самою.

## Запуск і залежності

Усі SQL і runtime-код вбудовано в notebook; зовнішні генератори не потрібні. Перша клітинка перевіряє точні версії `pgserver==0.1.4`, `psycopg2-binary==2.9.12`, `SQLAlchemy==2.0.54`, `pandas==2.3.3`. Notebook запускає власний PostgreSQL 16 через локальний Unix socket; зовнішня база або credentials не потрібні. Остання клітинка виконує cleanup.

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
