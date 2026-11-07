# Пайплайны данных

## Перечень пайплайнов

Номера потоков F — из [Task2/data-flows.md](../Task2/data-flows.md).

| № | Пайплайн | Состояние | Источник | Приёмник | Тип | Инструменты |
|---|---|---|---|---|---|---|
| P1 | Данные RB в единую управленческую отчётность (F11, F12, F14) | 3 месяца | RB Core, Cards, Loans (CDC); MDM (UCID, справочники) | FU DWH на PostgreSQL (потребители: BI) | CDC + ELT внутри DWH | Debezium, Kafka, Kafka Connect JDBC Sink, Airflow, dbt, PostgreSQL |
| P2 | Клиентские данные, режим coexistence (F01–F06) | 3 месяца | FU CRM, RB CRM, FU/RB Core (CDC) | MDM Customer Hub (затем: FU CRM, RB CRM, FU/RB Core, Cards, Loans) | CDC + сопоставление и слияние в MDM; первичная загрузка — batch | Debezium, Kafka, Schema Registry, ESB, MDM; Airflow — первичная загрузка и сверка |
| P3 | Клиентские данные, режим centralized (F04–F06) | 12 месяцев | API MDM: клиента создают и меняют только через MDM | MDM Customer Hub (затем: FU CRM как реплика, FU Core, Cards, Loans) | Синхронный API + публикация событий об изменении клиента | ESB (REST), MDM, Kafka, Schema Registry |
| P4 | Операционные данные в Lakehouse | 12 месяцев | FU Core, Cards, Loans, CRM, Consent Management (CDC) | Lakehouse: слои Bronze, Silver, Gold | CDC + ELT внутри Lakehouse | Debezium, Kafka Connect Iceberg Sink, Spark, dbt (Trino), Great Expectations, Airflow |
| P5 | Загрузка DWH и регуляторная отчётность | 12 месяцев | Lakehouse Silver | PostgreSQL DWH: слои core и marts (потребители: Банк России, BI) | ETL (Spark готовит данные вне DWH) + ELT (dbt строит core и витрины внутри DWH) | Airflow, Spark, dbt, Great Expectations, PostgreSQL |
| P6 | Наборы данных для ML-песочницы | 12 месяцев | Lakehouse Gold | Облачная ML-песочница | ETL: анонимизация до выгрузки из периметра | Spark, Great Expectations (проверка реидентификации), Airflow |

## Сценарии обработки больших данных

| Поле | S1. История карточных операций | S2. Архивы RetailBank после вывода систем |
|---|---|---|
| Источник данных | FU Card Processing (включая карты, перенесённые из RB): авторизации, клиринг, события жизненного цикла карты; историческая выгрузка RB Cards | RB DWH, RB Core, RB Cards, RB Loans, RB CRM: полные выгрузки на момент заморозки перед выводом |
| Характер данных | Событийные, высокочастотные, многолетняя история; полуструктурированные сообщения процессинга в Avro; только токен или маскированный номер карты | Архивные и исторические, структурированные таблицы и файлы выгрузок; после загрузки не меняются |
| Целевое хранилище | Lakehouse: Bronze (сырые события по дате) → Silver (дедупликация, связь авторизации с клирингом, UCID) → Gold (data-продукт «Карты», агрегаты по клиентам). В DWH — только финансовые операции за 2 года | Lakehouse: Iceberg-таблицы `archive_rb` только для чтения, с кодами UCID и FU. Исходные выгрузки с контрольными суммами — в объектном хранилище в режиме WORM на срок хранения по закону |
| Способ обработки | Streaming: Kafka Connect Iceberg Sink пишет в Bronze каждые несколько минут. Batch: Spark ежечасно строит Silver (MERGE по ID операции) и ежедневно — Gold и агрегаты | Batch ETL, разово на каждую систему: Spark переводит выгрузки в Parquet и Iceberg, сопоставляет с UCID и кодами продуктов FU. Итоги сверяются с отчётностью RB по периодам. После подтверждения сверки система RB выводится |
| Потребители | Антифрод (признаки для моделей), риск-менеджмент (скоринг, поведенческие модели), BI (карточные обороты) | Регуляторная отчётность и запросы Банка России за периоды до присоединения, аудит и комплаенс, BI (сравнение периодов до и после слияния), контакт-центр (история клиента RB) |
