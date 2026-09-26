# Мастер-системы

Роли систем:
- Источник As-Is — где данные физически хранятся сейчас.
- Точка появления — где данные создаются впервые. Указана в «Обосновании», если отличается от мастер-системы.
- Мастер — система с эталонными данными, которая отвечает за их корректность и распространяет изменения в другие системы.
- MDM — мультидоменная платформа мастер-данных из двух доменов: Customer Hub (золотая запись клиента, UCID) и Reference Data (справочники продуктов, тарифов, подразделений, сегментов). Для счетов, договоров, операций и платежей MDM не мастер: мастерами остаются профильные банковские системы.
- DWH и BI ни для какой сущности не мастер, они только потребители.

## Матрица мастер-систем

В промежуточном состоянии операционные сущности ведутся раздельно: FU-системы — мастер для портфеля FinUnion, RB-системы — для портфеля RetailBank. К 12-му месяцу портфели RB мигрируют в FU-системы, а RB-системы выводятся из эксплуатации.

| Сущность | Источники As-Is | Мастер-система на 3 месяца | Целевая мастер-система | Связь с BIAN | Обоснование |
|---|---|---|---|---|---|
| Клиент | FU/RB CRM, FU/RB Core, FU/RB Loans, FU/RB Cards | MDM Customer Hub (режим coexistence: ввод в CRM, эталон в MDM) | MDM Customer Hub (режим centralized: создание и изменение только через API MDM) | Party Reference Data Directory, Party Lifecycle Management | Точка появления — FU CRM, RB CRM (онбординг в офисе и каналах). Клиент заведён в 4+ системах с разными ID. Нужна золотая запись с UCID. MDM — мастер всех атрибутов клиента, включая контакты |
| Участие в договоре | FU/RB Core, FU/RB Loans, FU/RB Cards | FU/RB Core, FU/RB Loans, FU/RB Cards | FU Core, FU Loans, FU Cards | Customer Agreement, Sales Product Agreement | Роль клиента фиксируется при подписании договора, поэтому её мастер — система, где ведётся договор |
| Договор | FU/RB Core, FU/RB Loans, FU/RB Cards, FU/RB CRM | FU/RB Core (ДКО, счета, вклады), FU/RB Loans (кредиты), FU/RB Cards (карты) | FU Core, FU Loans, FU Cards | Customer Agreement, Sales Product Agreement | Точка появления — CRM (заявка). Юридически значимые условия договора ведутся в учётной системе. CRM хранит только статус заявки и реплику договора |
| Продукт | FU/RB Core, FU/RB Cards, FU/RB Loans, FU/RB CRM | MDM (Reference Data): единый каталог и маппинг кодов RB → FU | MDM (Reference Data) | Product Directory | Одинаковые продукты с разными кодами. Нужен единый каталог, чтобы предлагать клиентам RB продукты FU |
| Тариф | FU/RB Cards, FU/RB Core, FU/RB Loans | MDM (Reference Data): публикует утверждённые версии тарифов в Core/Cards/Loans | MDM (Reference Data) | Product Design | Единые версии тарифов для двух банков. Core/Cards/Loans применяют тариф (реплика). Индивидуальные условия (Special Pricing Conditions) — часть договора в профильной системе |
| Счёт | FU/RB Core | FU Core / RB Core | FU Core | Current Account, Corporate Current Account, Savings Account | Мастер — АБС. Только в ней счёт открывают и закрывают. |
| Вклад | FU/RB Core | FU Core / RB Core | FU Core | Term Deposit | Условия вклада и начисление процентов ведутся в АБС |
| Кредит | FU/RB Loans, FU/RB Core | FU Loans / RB Loans | FU Loans | Consumer Loan, Mortgage Loan, Corporate Loan, Credit Facility | Кредит выдают, рассчитывают по нему график, учитывают погашения, просрочку и реструктуризацию только в кредитной системе |
| Карта | FU/RB Cards, FU/RB Core | FU Cards / RB Cards | FU Cards | Issued Device Administration, Credit Card | Жизненный цикл карты ведёт процессинг. Полный номер карты хранится только в PCI-зоне процессинга, остальным системам передаётся токен |
| Платёж | FU/RB Core, FU ESB | FU Core / RB Core | FU Core | Payment Order Initiation, Payment Orchestration | Точка появления — каналы (ДБО, офис). Каналы только инициируют распоряжение, а исполняет и хранит его статус АБС |
| Транзакция | FU/RB Core, FU/RB Cards, FU/RB Loans, FU Data Lake | FU/RB Core (проводки), FU/RB Cards (карточные операции) | FU Core, FU Cards | Position Keeping, Card Transaction Capture | Неизменяемые факты учёта. DWH и Lakehouse хранят только реплики |
| Обращение клиента | FU/RB CRM | FU CRM / RB CRM | FU CRM | Customer Case Management | Обращения ведёт контакт-центр в CRM |
| Согласие | FU/RB CRM, FU/RB Core | FU CRM / RB CRM, согласие привязано к UCID в MDM | Consent Management | Customer Consent | Нужна неизменяемая история с целями, сроками и отзывами |
| Подразделение | FU/RB Core, FU/RB CRM, DWH | MDM (Reference Data) | MDM (Reference Data) | Branch Location Management | У одного офиса разные коды в FU и RB. Единый справочник нужен для отчётности |
| Клиентский сегмент | FU/RB CRM, FU/RB DWH | MDM (Reference Data) | MDM (Reference Data) | Customer Behavior Models | Без единой сегментации метрики двух банков несопоставимы |
