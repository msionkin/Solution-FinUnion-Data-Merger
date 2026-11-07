# Task3. Целевая архитектура данных и аналитическая платформа

Целевая архитектура данных объединённого банка через 12 месяцев: операционные хранилища, аналитическая платформа, пайплайны данных, аналитическая модель и доменные data-продукты. Мастер-системы и потоки — по [Task2](../Task2/README.md).

## Основные решения

- **Гибридная архитектура:** DWH на PostgreSQL для регламентированной и управленческой отчётности, Data Lakehouse для детальной истории и событий, элементы Data Mesh (доменное владение и data-продукты) на общей платформе.
- **OLTP и OLAP разделены.** Операционные системы FU и MDM работают только с транзакциями. Аналитика получает данные только через CDC (Debezium) и Kafka.
- **Lakehouse:** Iceberg на S3 (Ceph) с Iceberg REST Catalog, SQL-движок Trino, слои Bronze → Silver → Gold. Данные попадают в аналитику один раз — через Bronze; в Silver очищаются, сопоставляются с UCID и псевдонимизируются.
- **DWH остаётся на PostgreSQL** и загружается из Silver, тяжёлые вычисления — в Spark. Детальные операции — за текущий и прошлый год, остатки и агрегаты — за всю историю, полная история — в Lakehouse через Trino. MPP не нужен.
- **Через 3 месяца:** FU DWH на PostgreSQL — единая управленческая отчётность; данные RB приходят через CDC и Kafka, dbt строит core и витрины внутри DWH (ELT). Lakehouse ещё не внедряется.
- **Инструменты:** Airflow — оркестрация; dbt и Spark — трансформации; Kafka Connect Iceberg Sink — потоковая запись в Bronze; dbt tests и Great Expectations — качество; DataHub — каталог и lineage; FU BI с семантическим слоем (RB BI выводится).
- **Безопасность и размещение:** все данные банка хранятся on-prem (Ceph, Kubernetes), ПДн псевдонимизированы с Silver (ключи — в Vault), в облачную ML-песочницу уходят только анонимизированные наборы.
- **Большие данные:** история карточных операций (streaming + batch) и архивы RB после вывода систем (Iceberg `archive_rb` + WORM-копия).
- **Аналитическая модель:** «звезда» `fact_transactions` с 10 измерениями, одна строка — одна финансовая операция (без авторизаций).
- **Data-продукты:** Customer 360, Карты, Единый продуктовый каталог, Кредитный портфель — с владельцами и привязкой к доменам BIAN.

## Описание файлов

| Файл | Содержание |
|---|---|
| [target-data-architecture.md](target-data-architecture.md) | Компоненты To-Be и архитектурные принципы |
| [architecture-options.md](architecture-options.md) | Сравнение DWH на PostgreSQL, Lakehouse и Data Mesh, выбор гибридного подхода |
| [dwh-lakehouse.md](dwh-lakehouse.md) | DWH и Lakehouse: сравнение, подход на 3 и 12 месяцев, роль Iceberg и Trino, размещение данных, ограничения |
| [data-pipelines.md](data-pipelines.md) | Пайплайны P1–P6, сценарии больших данных S1 и S2 |
| [analytical-model.md](analytical-model.md) | Схема «звезда» для операций и её пригодность для OLAP |
| [data-products.md](data-products.md) | Четыре доменных data-продукта: домен BIAN, данные, владельцы, потребители |
| [diagrams/target-data-architecture-context.puml](diagrams/target-data-architecture-context.puml) | C4 уровень 1 (System Context): [PlantUML](diagrams/target-data-architecture-context.puml), [PNG](diagrams/target-data-architecture-context.png) |
| [diagrams/target-data-architecture-container.puml](diagrams/target-data-architecture-container.puml) | C4 уровень 2 (Container): [PlantUML](diagrams/target-data-architecture-container.puml), [PNG](diagrams/target-data-architecture-container.png) |
| [diagrams/analytical-model.mmd](diagrams/analytical-model.mmd) | Схема «звезда»: [Mermaid](diagrams/analytical-model.mmd), [SVG](diagrams/analytical-model.svg) |
