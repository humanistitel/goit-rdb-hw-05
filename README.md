# ДЗ 5 · CTE та materialized view: кеш ознак Olist

Навчальна робота Diuryk для курсу **Relational Databases for Data Science**.

[Відкрити notebook у Google Colab](https://colab.research.google.com/github/humanistitel/goit-rdb-hw-05/blob/main/Diuryk_hw5_olist_features.ipynb) · [Завантажити оригінальний `.ipynb`](https://raw.githubusercontent.com/humanistitel/goit-rdb-hw-05/main/Diuryk_hw5_olist_features.ipynb)

## Стан перевірки

Notebook є незміненою копією локально перевіреної роботи зі збереженими outputs. Локальний пакет пройшов три хвилі по п’ять незалежних агентських перевірок. Ці outputs отримані локально, а не в Google Colab. **Запуск у Colab цієї копії ще не підтверджено; роботу в LMS не подано.** Створення репозиторію або файла у Drive саме по собі не підтверджує виконання в хмарі. Авторський перегляд тексту й результатів залишається за власником роботи.

## Google Colab

1. Відкрийте посилання Colab вище. За потреби збережіть копію notebook у потрібній папці Google Drive.
2. Підключіть Python runtime. Для підготовки робочої папки виконайте наведену нижче команду в окремій тимчасовій клітинці перед кодом роботи. Вона отримує публічний репозиторій і встановлює поточний каталог; основні клітинки notebook змінювати не потрібно.

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

3. Виконайте основні клітинки послідовно, потім перевірте повторний запуск через **Runtime → Restart session and run all** з клітинкою підготовки перед ними. Перший запуск встановлює залежності та потребує мережі. Сумісність поточного Python/runtime Colab потрібно підтвердити фактичним успішним виконанням.
4. Після успіху збережіть хмарну копію з outputs. Збережені локальні outputs у цьому репозиторії не замінюють цю перевірку. Якщо запуск перервано після створення PostgreSQL, виконайте останню клітинку cleanup.

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

Мережевий fallback підготовлено в notebook; його успіх у поточному Colab ще потрібно підтвердити. Якщо офіційний endpoint недоступний, використайте відповідні перевірені CSV з власного локального пакета в `data/olist/`; hashes мають збігатися. Kaggle credentials у цьому репозиторії не зберігаються.

Сирий шар зберігає вхідні CSV-значення; SQL формує типізований шар і CTE-конвеєр ознак. Є перевірки покриття стадій, recursive CTE, materialized view, індекс, refresh і фактичні EXPLAIN-плани. Ознаки сформовано за явно вказаними правилами часу й зерна. Reflection обґрунтовує щодобовий refresh для пакетної сегментації та пріоритет вартості перерахунку. Службові звіти notebook зберігають початкові локальні metadata; окремий cloud run підтверджується власними outputs, а не цими історичними прапорцями.

## Незмінність і межі публікації

Оригінальний notebook: `Diuryk_hw5_olist_features.ipynb`. SHA-256 цієї локально перевіреної копії:

```text
4eca5719b18103e915cd063cf4fdfa5610b20b043a63b5ee43238137243c0494
```

Репозиторій містить навчальний код і потрібні невеликі допоміжні файли. Runtime-кластери, credentials, повний архів курсу, вихідні наукові статті, QA інших робіт та великі ZIP/датасети не публікуються. Фінальне тестування курсу не є частиною цієї роботи. Посилання на репозиторій і архів для LMS готуються окремо; цей README не заявляє подання чи оцінювання роботи.
