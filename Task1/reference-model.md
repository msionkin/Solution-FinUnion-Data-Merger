# Референсная модель BIAN

Использована версия [BIAN Service Landscape 14](https://bian.org/servicelandscape-14-0-0/views/view_54081.html); названия сервисных доменов приведены в соответствии с ней.

## Сопоставление сущностей с сервисными доменами BIAN

| Сущность | Область BIAN | Зачем это нужно |
|---|---|---|
| Клиент | Party Reference Data Directory, Party Lifecycle Management | Единый подход к клиентским данным и онбордингу (KYC) для FU и RB |
| Договор | Customer Agreement, Sales Product Agreement | Разделение рамочного договора обслуживания и продуктовых договоров |
| Продукт | Product Directory, Product Design | Единый каталог продуктов FU и RB и маппинг продуктов RetailBank на линейку FinUnion |
| Тариф | Product Design, Special Pricing Conditions | Отделение стандартных тарифов продукта от индивидуальных условий клиента |
| Счёт | Current Account, Corporate Current Account, Savings Account | Унификация модели счёта двух АБС для розницы и МСБ |
| Вклад | Term Deposit | Единые правила срочных вкладов (капитализация, пролонгация) |
| Кредит | Consumer Loan, Mortgage Loan, Corporate Loan, Credit Facility | Единые правила кредитных продуктов для розницы и МСБ |
| Карта | Issued Device Administration, Credit Card, Card Authorization | Унификация жизненного цикла и статусов карт двух процессингов |
| Платёж | Payment Order Initiation, Payment Orchestration | Разделение инициации распоряжения клиента и его исполнения |

## Как BIAN помогает объединению FinUnion и RetailBank

**Унификация терминологии.** Одни и те же объекты в системах называются по-разному: «клиент», «контрагент», «заёмщик», «держатель». В BIAN это одно лицо (Party) в разных ролях, поэтому в модели есть Клиент (Party) и Участие в договоре. Названия доменов BIAN используются как общий глоссарий для команд FU и RB.

**Границы владения данными.** Сервисный домен BIAN — это граница ответственности: у каждого домена один бизнес-владелец и одна мастер-система. Например, Current Account отвечает за счёт, Payment Order Initiation — за платёж. Поэтому CRM не может быть мастером для счёта, а DWH — мастером для операций.

**Пересечения систем FU и RB.** При наложении систем на домены BIAN видны дубли функций, которые нужно консолидировать, например:

| Область BIAN | FinUnion | RetailBank | Вывод |
|---|---|---|---|
| Party Reference Data Directory | FU CRM, FU Core | RB CRM, RB Core | Клиент ведётся в 4 системах — нужен MDM |
| Current Account, Term Deposit | FU Core | RB Core | Две модели счёта и вклада — миграция в одну АБС |
| Issued Device Administration | FU Cards | RB Cards | Два процессинга — единый справочник статусов, затем консолидация |
| Consumer Loan, Corporate Loan, Credit Facility | FU Loans | RB Loans | Разные правила продуктов — единые правила и затем одна система |

**Целевые домены данных и data products.** Сервисные домены BIAN группируются в домены данных, у каждого из которых есть владелец и data product:

| Домен данных | Области BIAN | Сущности | Data product |
|---|---|---|---|
| Клиент | Party Reference Data Directory, Customer Consent, Customer Agreement | Клиент, Участие в договоре, Сегмент | Customer 360 |
| Продукты | Product Directory, Product Design | Продукт, Тариф | Единый продуктовый каталог |
| Кредитование | Consumer Loan, Mortgage Loan, Corporate Loan, Credit Facility | Кредит | Кредитный портфель |
| Карты | Issued Device Administration, Card Authorization | Карта | Карты |
