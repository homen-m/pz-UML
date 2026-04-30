# Practical lesson pz-UML
Побудова повердінкових UML-діаграм для проєктування інформаційних систем

## Тема - Інтерактивний дашборд аналізу ефективності застосування різних типів озброєння

## 1. Діаграма варіантів використання (Use Case Diagram)

Ця діаграма описує функціональність системи з точки зору користувачів (Акторів). 

Основними акторами є **Військовий аналітик (користувач)** та **Системний адміністратор.**

Призначення: Візуалізація меж системи та ключових функцій, доступних різним ролям.

`code`

    useCaseDiagram
    actor "Військовий аналітик" as Analyst
    actor "Адміністратор системи" as Admin
    actor "Зовнішня БД (Джерела даних)" as ExternalDB

    package "Дашборд аналізу озброєння" {
        usecase "Авторизація в системі" as UC_Login
        usecase "Перегляд загальної статистики" as UC_ViewStats
        usecase "Фільтрація даних за типом зброї" as UC_Filter
        usecase "Порівняння ефективності зразків" as UC_Compare
        usecase "Генерація аналітичного звіту" as UC_Report
        usecase "Керування довідниками зброї" as UC_ManageData
        usecase "Оновлення даних з джерел" as UC_Sync
    }

    Analyst --> UC_Login
    Analyst --> UC_ViewStats
    Analyst --> UC_Filter
    Analyst --> UC_Compare
    Analyst --> UC_Report

    Admin --> UC_Login
    Admin --> UC_ManageData
    
    UC_Sync -- ExternalDB
    UC_Sync <.. UC_ManageData : <<include>>
    UC_Filter ..> UC_ViewStats : <<extend>>

`code`

## 2. Діаграма послідовності (Sequence Diagram)

Відображає взаємодію об'єктів у часі для сценарію: «Генерація звіту про порівняльну ефективність».

Призначення: Деталізація логіки обміну повідомленнями між компонентами системи для виконання конкретного завдання.

`code`

    sequenceDiagram
    autonumber
    actor Analyst as Аналітик
    participant UI as Веб-інтерфейс
    participant Controller as Аналітичний модуль
    participant DB as База даних

    Analyst ->> UI: Обирає параметри (тип зброї, період)
    UI ->> Controller: Запит на розрахунок ефективності
    Controller ->> DB: SQL запит даних про застосування
    DB -->> Controller: Набір сирих даних (ураження, витрати)
    
    Note over Controller: Обчислення KPI (коефіцієнт <br/>успішності, вартість цілі)
    
    Controller -->> UI: Результати розрахунків (JSON)
    UI ->> UI: Рендеринг графіків та таблиць
    
    Analyst ->> UI: Натискає "Експорт у PDF"
    UI ->> Controller: Формування файлу звіту
    Controller -->> UI: Посилання на завантаження
    UI -->> Analyst: Видача файлу користувачу
    
`code`

## 3. Діаграма діяльності (Activity Diagram)

Описує бізнес-процес аналізу даних від входу в систему до отримання результату.

Призначення: Моделювання послідовності дій та розгалужень у робочому процесі аналітика.

`code`

    flowchart TD
    %% Використовуємо subgraph для відображення доріжок (swimlanes)
    
    subgraph User [Військовий аналітик]
        Start([Початок]) --> Login[Введення облікових даних]
        SelectParams[Вибір параметрів: тип зброї, період] --> RequestAnalys[Запуск розрахунку]
        ViewCharts[Перегляд візуалізацій та KPI] --> Decision{Потрібен звіт?}
        Decision -- Так --> Export[Натиснути Експорт PDF]
        Decision -- Ні --> NewSearch{Новий запит?}
        NewSearch -- Так --> SelectParams
        NewSearch -- Ні --> End([Кінець])
    end

    subgraph System [Інтерфейс Дашборду]
        Login --> AuthCheck{Перевірка сесії}
        AuthCheck -- Валідно --> ShowDash[Відображення головного екрана]
        AuthCheck -- Помилка --> Login
        
        RequestAnalys --> Loading[Індикатор завантаження]
        Loading --> Render[Рендеринг графіків]
        Render --> ViewCharts
        
        Export --> Download[Підготовка файлу]
        Download --> End
    end

    subgraph Backend [Сервер та БД]
        ShowDash --> LoadMeta[Завантаження списків озброєння]
        LoadMeta --> SelectParams
        
        Loading --> FetchData[Запит до БД уражень]
        FetchData --> Calculate[Розрахунок ефективності та ККД]
        Calculate --> Render
    end

    %% Стилізація для кращого вигляду
    style Start fill:#d4edda,stroke:#28a745
    style End fill:#f8d7da,stroke:#dc3545
    style Decision fill:#fff3cd,stroke:#ffc107
    style AuthCheck fill:#fff3cd,stroke:#ffc107
    style NewSearch fill:#fff3cd,stroke:#ffc107
    
`code`

### Пояснення до виконаної роботи:
**Логічний зв'язок:**

* **Use Case** визначає, що система робить (наприклад, "Генерація звіту").

* **Sequence показує**, як саме технічно реалізується цей Use Case через взаємодію UI, контролера та бази даних.

* **Activity** описує динаміку процесу: як користувач переходить від фільтрації до вибору типу аналізу та фінального експорту.

* **Нотації:** Використано стандартні елементи Mermaid (актори, границі системи, виклики, умови/choices).

* **Предметна область:** Враховано специфіку збройової аналітики (KPI успішності, типи зброї, джерела даних).
