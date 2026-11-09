# Архитектура системы JDX и план внедрения AI
> Система учёта лабораторных испытаний на платформе Data Express с интеграцией искусственного интеллекта

**Версия документа:** 2.0  
**Дата:** 28 октября 2026  
**Платформа:** Data Express (mydataexpress.ru)  
**СУБД:** Firebird 5.0  
**Язык скриптов:** RemObjects Pascal Script  
**Автор:** Cordek

---

## 📋 Содержание

### Часть I. Архитектура системы JDX

1. [Введение](#1-введение)
2. [Обзор системы](#2-обзор-системы)
3. [C4-модель архитектуры](#3-c4-модель-архитектуры)
4. [Модель данных](#4-модель-данных)
5. [Инфраструктура и развёртывание](#5-инфраструктура-и-развёртывание)
6. [Сетевые правила и безопасность](#6-сетевые-правила-и-безопасность)
7. [Структура хранения данных](#7-структура-хранения-данных)

### Часть II. AI-интеграция

8. [Архитектура хранения данных и метаданных для AI](#8-архитектура-хранения-данных-и-метаданных-для-ai)
9. [Ручное редактирование метаданных](#9-ручное-редактирование-метаданных)
10. [Редактирование метаданных с помощью AI агента](#10-редактирование-метаданных-с-помощью-ai-агента)
11. [AI агент как исполнитель процессов](#11-ai-агент-как-исполнитель-процессов)
12. [Использование AI агентом данных из базы](#12-использование-ai-агентом-данных-из-базы)
13. [Интеграционная архитектура AI](#13-интеграционная-архитектура-ai)
14. [Безопасность и контроль AI-действий](#14-безопасность-и-контроль-ai-действий)

### Часть III. План внедрения AI

15. [План внедрения AI в систему JDX](#15-план-внедрения-ai-в-систему-jdx)
16. [Приложения](#16-приложения)

---

# ЧАСТЬ I. АРХИТЕКТУРА СИСТЕМЫ JDX

---

## 1. Введение

### 1.1 Назначение документа
Документ описывает архитектуру системы **JDX** — конфигурации на платформе Data Express для автоматизации учёта лабораторных испытаний, управления документами системы менеджмента качества (СМК) и сопутствующих процессов лаборатории, а также план внедрения AI-агента для расширения функциональности системы.

### 1.2 Область применения
- Лабораторный учёт образцов и результатов испытаний
- Управление методиками, оборудованием, персоналом
- Документооборот СМК: аудиты, оценки рисков, корректирующие действия
- Формирование протоколов и отчётности
- AI-ассистированная разработка и выполнение процессов

### 1.3 Технологический стек

| Компонент | Технология | Версия |
|-----------|-----------|--------|
| Платформа | Data Express | - |
| СУБД | Firebird | 5.0 |
| Язык скриптов | RemObjects Pascal Script | - |
| Толстый клиент | Windows App | - |
| Веб-клиент | Browser (HTTP/HTTPS) | - |
| Интеграции | REST API (модуль в разработке) | - |

---

## 2. Обзор системы

**JDX** — модульная система учёта, построенная на low-code платформе Data Express. Система поддерживает два типа клиентов:
- **Толстый клиент** — основной интерфейс для работы внутри защищённого контура лаборатории
- **Веб-клиент** — упрощённый доступ для внешних пользователей через защищённый шлюз

**Ключевые возможности:**
- Регистрация и отслеживание образцов
- Ввод и валидация результатов испытаний
- Управление нормативной базой (методики, ГОСТы)
- Учёт оборудования и графиков поверки
- Документооборот СМК (аудиты, риски, корректирующие действия)
- Формирование печатных форм протоколов

---

## 3. C4-модель архитектуры

### 3.1 System Context (Уровень 1)

```mermaid
C4Context
    title System Context Diagram - Система JDX

    Person(lab_manager, "Руководитель лаборатории", "Управляет процессами, утверждает документы")
    Person(lab_technician, "Лаборант", "Проводит испытания, вносит результаты")
    Person(auditor, "Аудитор", "Проверяет документы СМК, проводит аудиты")
    Person(admin, "Администратор", "Настраивает систему, управляет доступами")

    System_Boundary(jdx_boundary, "JDX Laboratory System") {
        System(jdx, "JDX", "Data Express", "Система учёта лабораторных испытаний и документов СМК")
    }

    System_Ext(external_users, "Внешние пользователи", "Заказчики, партнёры (через веб-клиент)")

    Rel(lab_manager, jdx, "Утверждает протоколы, контролирует процессы")
    Rel(lab_technician, jdx, "Вносит результаты испытаний, регистрирует образцы")
    Rel(auditor, jdx, "Просматривает документы СМК, аудиты")
    Rel(admin, jdx, "Администрирование, настройка")
    Rel(external_users, jdx, "Получают результаты (веб-клиент)", "HTTPS")
```

### 3.2 Container Diagram (Уровень 2)

```mermaid
C4Container
    title Container Diagram - Система JDX

    Person(lab_user, "Пользователь лаборатории", "Лаборант, руководитель, аудитор")

    System_Boundary(jdx_boundary, "JDX System") {
        Container(thick_client, "Толстый клиент", "Windows App", "Основной интерфейс для работы с данными")
        Container(web_server, "Веб-сервер", "HTTP Server", "Обеспечивает доступ через браузер, включает модуль авторизации")
        Container(web_client, "Веб-клиент", "Browser", "Упрощённый интерфейс для внешнего доступа")
        Container(api_module, "REST API Модуль", "API Gateway", "Интеграционный слой для внешних систем")
        ContainerDb(database, "Firebird 5.0 DB", "Database", "Хранение всех данных системы")
    }

    System_Ext(future_erp, "Внешние системы (ERP, 1С)", "Будущие интеграции")
    System_Ext(future_equipment, "Лабораторное оборудование", "Хроматографы, весы и т.д.")

    Rel(lab_user, thick_client, "Основная работа", "TCP/IP")
    Rel(lab_user, web_client, "Удалённый доступ", "HTTPS")
    Rel(web_client, web_server, "Запросы", "HTTP/HTTPS")
    Rel(thick_client, database, "Прямое подключение", "Firebird Protocol")
    Rel(web_server, database, "Доступ к данным", "Firebird Protocol")
    Rel(api_module, database, "Чтение/запись данных", "Firebird Protocol")
    Rel(future_erp, api_module, "Интеграция (будущее)", "REST API")
    Rel(future_equipment, api_module, "Сбор данных (будущее)", "REST API")
```

### 3.3 Component Diagram — Толстый клиент (Уровень 3)

```mermaid
C4Component
    title Component Diagram - Толстый клиент JDX

    Container_Boundary(thick_client, "Толстый клиент") {
        Component(ui_forms, "Формы ввода", "UI", "Формы регистрации образцов, результатов, документов")
        Component(reports, "Отчёты и протоколы", "Reporting", "Генерация печатных форм протоколов")
        Component(dashboards, "Дашборды", "Analytics", "Визуализация статистики и KPI")
        Component(validation, "Валидаторы", "Business Logic", "Проверка данных, бизнес-правила")
        Component(workflow, "Движок процессов", "Workflow", "Маршруты согласования, статусы")
        Component(auth, "Модуль авторизации", "Security", "Аутентификация, роли, права")
    }

    ContainerDb(database, "Firebird 5.0 DB", "Database", "Данные системы")

    Rel(ui_forms, validation, "Вызывает")
    Rel(ui_forms, database, "Чтение/запись")
    Rel(reports, database, "Генерация из данных")
    Rel(dashboards, database, "Агрегация данных")
    Rel(workflow, database, "Обновление статусов")
    Rel(auth, database, "Проверка учётных данных")
```

### 3.4 Component Diagram — REST API Модуль (Уровень 3)

```mermaid
C4Component
    title Component Diagram - REST API Модуль

    Container_Boundary(api_module, "REST API Модуль") {
        Component(api_gateway, "API Gateway", "Router", "Маршрутизация запросов")
        Component(samples_api, "Samples API", "Endpoint", "Работа с образцами")
        Component(results_api, "Results API", "Endpoint", "Работа с результатами")
        Component(docs_api, "Documents API", "Endpoint", "Документы СМК, аудиты")
        Component(data_mapper, "Data Mapper", "Transformation", "Преобразование форматов")
    }

    ContainerDb(database, "Firebird 5.0 DB", "Database", "Данные системы")
    System_Ext(external_system, "Внешняя система", "1С, ERP, LIMS")

    Rel(external_system, api_gateway, "HTTPS запросы", "REST/JSON")
    Rel(api_gateway, samples_api, "Маршрутизация")
    Rel(api_gateway, results_api, "Маршрутизация")
    Rel(api_gateway, docs_api, "Маршрутизация")
    Rel(samples_api, data_mapper, "Преобразование")
    Rel(results_api, data_mapper, "Преобразование")
    Rel(docs_api, data_mapper, "Преобразование")
    Rel(data_mapper, database, "SQL запросы", "Firebird")
```

---

## 4. Модель данных

### 4.1 ER-диаграмма основных сущностей

```mermaid
erDiagram
    SAMPLE ||--o{ TEST_RESULT : "имеет результаты"
    SAMPLE ||--o{ SAMPLE_STATUS : "имеет статусы"
    METHOD ||--o{ TEST_RESULT : "используется в"
    EQUIPMENT ||--o{ TEST_RESULT : "используется для"
    PERSONNEL ||--o{ TEST_RESULT : "выполняет"
    PERSONNEL ||--o{ AUDIT : "проходит"
    DOCUMENT ||--o{ AUDIT : "проверяется в"
    REQUEST ||--o{ SAMPLE : "содержит"
    
    REQUEST {
        string id "ID заявки"
        date created_date "Дата создания"
        string customer "Заказчик"
        string status "Статус"
    }
    
    SAMPLE {
        string id "ID образца"
        string barcode "Штрихкод"
        date collection_date "Дата отбора"
        string matrix_type "Тип матрицы"
        string status "Статус"
    }
    
    TEST_RESULT {
        string id "ID результата"
        string parameter "Параметр"
        decimal value "Значение"
        string unit "Ед. измерения"
        string method_id "Методика"
    }
    
    METHOD {
        string id "ID методики"
        string name "Название"
        string gost "ГОСТ/ISO"
        string parameters "Параметры"
    }
    
    EQUIPMENT {
        string id "ID оборудования"
        string name "Название"
        string serial_number "Серийный номер"
        date calibration_date "Дата поверки"
    }
    
    PERSONNEL {
        string id "ID сотрудника"
        string name "ФИО"
        string position "Должность"
        string qualifications "Квалификация"
    }
    
    DOCUMENT {
        string id "ID документа"
        string type "Тип (СМК, аудит, риск)"
        string status "Статус"
        date review_date "Дата пересмотра"
    }
    
    AUDIT {
        string id "ID аудита"
        date audit_date "Дата аудита"
        string type "Тип аудита"
        string findings "Замечания"
    }
```

---

## 5. Инфраструктура и развёртывание

### 5.1 Сценарий А: Закрытый контур

```mermaid
C4Deployment
    title "Deployment Diagram - Закрытый контур"

    System_Boundary(closed_zone, "Защищённый контур лаборатории") {
        
        Deployment_Node(db_server, "Сервер БД", "Windows/Linux Server") {
            ContainerDb(firebird_db, "Firebird 5.0 DB", "Database", "Основная база данных JDX")
            ContainerDb(backup_db, "Резервная копия", "Backup", "Ежедневные бэкапы")
        }

        Deployment_Node(app_server, "Сервер приложений", "Windows Server") {
            Container(web_server, "Веб-сервер", "HTTP Server", "Обслуживание веб-клиентов")
            Container(api_module, "REST API Module", "API Gateway", "Интеграционный модуль")
        }

        Deployment_Node(user_zone, "Рабочие места пользователей", "Windows Workstations") {
            Node(workstation1, "ПК Лаборанта 1", "Windows 10/11") {
                Container(thick_client1, "Толстый клиент JDX", "Data Express Client", "Основной интерфейс")
            }
            Node(workstation2, "ПК Лаборанта 2", "Windows 10/11") {
                Container(thick_client2, "Толстый клиент JDX", "Data Express Client", "Основной интерфейс")
            }
            Node(workstation_admin, "ПК Администратора", "Windows 10/11") {
                Container(admin_client, "Толстый клиент + Админ-консоль", "Data Express Client", "Настройка системы")
            }
        }

        Deployment_Node(backup_zone, "Сервер резервного копирования", "Backup Server") {
            Container(backup_service, "Служба бэкапа", "Backup Agent", "Автоматическое резервное копирование")
        }

        Rel(thick_client1, firebird_db, "Прямое подключение", "Firebird Protocol (TCP 3050)")
        Rel(thick_client2, firebird_db, "Прямое подключение", "Firebird Protocol (TCP 3050)")
        Rel(admin_client, firebird_db, "Администрирование + данные", "Firebird Protocol")
        Rel(web_server, firebird_db, "Доступ к данным", "Firebird Protocol")
        Rel(api_module, firebird_db, "Чтение/запись", "Firebird Protocol")
        Rel(backup_service, firebird_db, "Резервное копирование", "Native Backup API")
        Rel(backup_service, backup_db, "Хранение бэкапов", "File System")
    }
```

### 5.2 Сценарий Б: Гибридный доступ с DMZ

```mermaid
C4Deployment
    title "Deployment Diagram - Гибридный доступ"

    System_Boundary(internet, "Интернет / Внешняя сеть") {
        Deployment_Node(external_users, "Внешние пользователи", "Remote Devices") {
            Node(external_pc, "ПК / Ноутбук", "Browser") {
                Container(external_browser, "Веб-браузер", "Chrome/Firefox/Edge", "Доступ к системе через HTTPS")
            }
        }
    }

    System_Boundary(dmz_zone, "DMZ / Шлюзовая зона") {
        Deployment_Node(gateway_server, "Шлюз доступа", "Reverse Proxy") {
            Container(reverse_proxy, "Reverse Proxy", "HTTPS Termination", "Терминация SSL, фильтрация запросов")
        }
    }

    System_Boundary(closed_zone, "Закрытый контур лаборатории") {
        
        Deployment_Node(app_server, "Сервер приложений", "Windows/Linux Server") {
            Container(web_server, "Data Express Web Server", "HTTP Server", "Обработка веб-запросов, авторизация")
            Container(api_module, "REST API Module", "Integration", "Модуль для интеграций")
        }

        Deployment_Node(db_server, "Сервер баз данных", "Secure DB Server") {
            ContainerDb(firebird_db, "Firebird 5.0 Database", "Database", "Основная база данных JDX")
        }

        Deployment_Node(internal_workstations, "Рабочие места внутри контура", "Windows Workstations") {
            Node(internal_pc1, "ПК Лаборанта", "Windows 10/11") {
                Container(thick_client, "Толстый клиент JDX", "Data Express Client", "Прямое подключение к БД")
            }
        }

        Rel(thick_client, firebird_db, "Прямое подключение", "Firebird Protocol (TCP 3050)")
        Rel(web_server, firebird_db, "Доступ к данным", "Firebird Protocol (TCP 3050)")
        Rel(api_module, firebird_db, "Чтение/запись", "Firebird Protocol")
    }

    Rel(external_browser, reverse_proxy, "HTTPS запросы", "TLS 1.2/1.3 (443)")
    Rel(reverse_proxy, web_server, "Проксирование запросов", "HTTP/HTTPS (внутренний порт)")
```

### 5.3 Поток запроса внешнего пользователя

```mermaid
sequenceDiagram
    autonumber
    participant User as "Внешний пользователь"
    participant Browser as "Браузер"
    participant Proxy as "Reverse Proxy (DMZ)"
    participant WebSrv as "Web Server (Closed)"
    participant DB as "Firebird 5.0 DB"

    User->>Browser: Вводит URL системы (https://...)
    Browser->>Proxy: HTTPS запрос (443)
    Proxy->>WebSrv: Проксирование (внутренний порт)
    WebSrv->>DB: SQL запрос (Firebird Protocol)
    DB-->>WebSrv: Данные / Результат
    WebSrv-->>Proxy: HTTP ответ
    Proxy-->>Browser: HTTPS ответ
    Browser-->>User: Отображение интерфейса

    Note right of DB: Прямой доступ к БД извне заблокирован
```

---

## 6. Сетевые правила и безопасность

### 6.1 Матрица сетевых правил (Firewall)

| Источник | Назначение | Порт | Протокол | Действие |
|----------|-----------|------|----------|----------|
| Интернет | DMZ: Reverse Proxy | 443/TCP | HTTPS | ALLOW |
| Интернет | DMZ: Reverse Proxy | 80/TCP | HTTP | REDIRECT→443 |
| DMZ: Proxy | Closed: Web Server | 8080/TCP | HTTP | ALLOW |
| DMZ: Proxy | Closed: Firebird DB | 3050/TCP | Firebird | DENY |
| Closed: Thick Client | Closed: Firebird DB | 3050/TCP | Firebird | ALLOW |
| Closed: Web Server | Closed: Firebird DB | 3050/TCP | Firebird | ALLOW |
| Интернет | Closed: Любые ресурсы | Любой | Любой | DENY |
| Management | Все серверы | 22/3389 | SSH/RDP | ALLOW* |

> *Доступ для администрирования только с выделенных управляющих станций через VPN.

### 6.2 Рекомендации по безопасности

**Обязательные меры:**
1. Запретить прямой доступ к порту 3050 (Firebird) из любой сети, кроме доверенных внутренних сегментов
2. Использовать Reverse Proxy как единственную точку входа извне
3. Включить аудит подключений и изменений в базе данных
4. Регулярно обновлять платформу Data Express и СУБД Firebird 5.0

**Рекомендуемые меры:**
1. Выделить отдельный домен для внешнего доступа (например, `ext.jdx.lab.ru`)
2. Настроить принудительное использование HTTPS с TLS 1.2+
3. Ограничить время сессии и реализовать автоматический выход при неактивности
4. Вести централизованный журнал событий безопасности

---

## 7. Структура хранения данных

### 7.1 Три зоны данных в СУБД Firebird 5.0

```mermaid
C4Container
    title "Data Storage Zones - Три зоны данных"

    System_Boundary(database, "Firebird 5.0 Database") {
        ContainerDb(metadata_zone, "Зона 1: Метаданные", "Platform Schema", "Формы, таблицы, запросы, скрипты, бизнес-логика")
        ContainerDb(security_zone, "Зона 2: Пользователи и безопасность", "Security Schema", "Учётные записи, пароли, роли, интерфейсы")
        ContainerDb(app_data_zone, "Зона 3: Данные приложения", "Application Schema", "Образцы, результаты, справочники, документы СМК")
    }

    Container(thick_client, "Толстый клиент", "Data Express Client", "Прямое подключение к БД")
    Container(web_server, "Веб-сервер", "Data Express Web", "Подключение к БД для веб-доступа")

    Rel(thick_client, metadata_zone, "Чтение метаданных для UI")
    Rel(thick_client, security_zone, "Аутентификация")
    Rel(thick_client, app_data_zone, "Чтение/запись данных")
    
    Rel(web_server, metadata_zone, "Чтение метаданных для UI")
    Rel(web_server, security_zone, "Аутентификация")
    Rel(web_server, app_data_zone, "Чтение/запись данных")

    Rel(metadata_zone, app_data_zone, "Определяет структуру форм")
    Rel(security_zone, metadata_zone, "Определяет доступ к формам")
```

### 7.2 Детальное описание зон

#### Зона 1: Метаданные (Platform Metadata)

| Компонент | Описание |
|-----------|----------|
| Формы ввода | Описание форм для ввода информации |
| Таблицы просмотра | Конфигурация таблиц для отображения данных |
| Запросы | SQL-запросы для выборки данных |
| Бизнес-логика | Скрипты, модули, валидаторы на RemObjects Pascal Script |

**Важно:** Справочники **НЕ входят** в метаданные — они относятся к данным приложения (Зона 3).

#### Зона 2: Пользователи и безопасность (Security)

```mermaid
erDiagram
    USERS ||--o{ USER_ROLES : "назначены роли"
    ROLES ||--o{ ROLE_INTERFACES : "настроены интерфейсы"
    ROLES ||--o{ ROLE_PERMISSIONS : "настроены права"
    INTERFACES ||--o{ ROLE_INTERFACES : "привязаны к ролям"
    
    USERS {
        string id "ID пользователя"
        string login "Логин"
        string full_name "ФИО"
        string password_hash "Хэш пароля"
        boolean is_active "Активен"
    }
    
    ROLES {
        string id "ID роли"
        string name "Название роли"
        string description "Описание"
    }
    
    INTERFACES {
        string id "ID интерфейса"
        string name "Название"
        string form_list "Список доступных форм"
    }
    
    ROLE_PERMISSIONS {
        string role_id "ID роли"
        string permission "Разрешение"
        string object "Объект доступа"
        string access_level "Уровень доступа"
    }
```

**Схема доступа:** `Пользователь → Роли → Интерфейсы → Формы/Данные`

#### Зона 3: Данные приложения (Application Data)

| Компонент | Описание |
|-----------|----------|
| Лабораторные данные | Образцы, результаты, заявки |
| Справочники (НСИ) | Основные справочники системы |
| Нормативная база | Методики, ГОСТы, параметры |
| Ресурсы | Оборудование, персонал, реактивы |
| Документы СМК | Аудиты, риски, корректирующие действия |

### 7.3 Процесс авторизации и загрузки интерфейса

```mermaid
sequenceDiagram
    autonumber
    participant User as "Пользователь"
    participant TC as "Толстый клиент"
    participant Z2 as "Зона 2: Безопасность"
    participant Z1 as "Зона 1: Метаданные"
    participant Z3 as "Зона 3: Данные приложения"

    Note over User, TC: Этап 1: Авторизация
    
    User->>TC: Запуск приложения
    TC->>Z2: Подключение к БД (порт 3050)
    User->>TC: Ввод логина/пароля
    TC->>Z2: Проверка учётных данных
    Z2-->>TC: Пользователь найден + роли
    
    Note over User, TC: Этап 2: Загрузка метаданных
    
    TC->>Z2: Получение интерфейсов по ролям
    Z2-->>TC: Список доступных интерфейсов
    TC->>Z1: Загрузка метаданных форм
    Z1-->>TC: Конфигурация форм, таблиц, запросов
    TC->>TC: Построение интерфейса
    
    Note over User, TC: Этап 3: Работа с данными
    
    User->>TC: Запрос данных (образцы, справочники)
    TC->>Z2: Проверка прав доступа
    Z2-->>TC: Доступ разрешён
    TC->>Z3: Чтение данных
    Z3-->>TC: Данные образцов/справочников
    TC-->>User: Отображение данных
```

### 7.4 Варианты развёртывания зон данных

| Критерий | Единая БД | Разнесённые БД |
|----------|-----------|----------------|
| Сложность развёртывания | Низкая | Высокая |
| Производительность | Выше (одно подключение) | Ниже (множественные подключения) |
| Масштабируемость | Ограничена | Высокая |
| Резервное копирование | Единый бэкап | Раздельные бэкапы |
| Целостность данных | Гарантирована транзакциями | Требует синхронизации |
| Подключений к БД | Одно на клиент | Три на клиент |
| Рекомендуемый размер | До 100 пользователей | 100+ пользователей |
| **Для JDX** | **✅ Рекомендуется** | ⚠️ При росте |

---

# ЧАСТЬ II. AI-ИНТЕГРАЦИЯ

---

## 8. Архитектура хранения данных и метаданных для AI

### 8.1 Четыре логических слоя данных

**Метаданные** — конфигурация системы в текстовом формате:
- Формы ввода (описание полей, типов данных, валидации)
- Таблицы просмотра (конфигурация списков, колонок, фильтров)
- Запросы (SQL для выборки данных)
- Скрипты и модули (код на RemObjects Pascal Script)
- Валидаторы (правила проверки данных)

**Рабочие данные** — операционная информация:
- Заявки на испытания, образцы, результаты анализов, протоколы

**НСИ** — нормативно-справочная информация:
- Методики испытаний, ГОСТы, типы образцов, единицы измерения, классификаторы

**Знания** — экспертная информация:
- Описание методик, правила интерпретации, критерии приёмки, исторические прецеденты

```mermaid
C4Container
    title "Data Storage Architecture for AI"

    System_Boundary(firebird_db, "Firebird 5.0 Database") {
        ContainerDb(metadata, "Метаданные", "Platform Schema", "Формы, скрипты, запросы, валидаторы")
        ContainerDb(operational, "Рабочие данные", "Application Data", "Заявки, образцы, результаты")
        ContainerDb(nsi, "НСИ", "Reference Data", "Справочники, классификаторы")
        ContainerDb(knowledge, "Знания", "Knowledge Base", "Методики, правила, прецеденты")
    }

    Container(thick_client, "Толстый клиент", "Data Express Client", "Основной интерфейс")
    Container(web_server, "Веб-сервер", "Data Express Web", "Веб-доступ")
    Container(ai_agent, "AI Агент", "LLM Integration", "Обработка запросов, генерация кода")

    Rel(thick_client, metadata, "Чтение/запись метаданных")
    Rel(thick_client, operational, "Работа с данными")
    Rel(thick_client, nsi, "Использование справочников")
    
    Rel(web_server, metadata, "Чтение метаданных")
    Rel(web_server, operational, "Работа с данными")
    
    Rel(ai_agent, metadata, "Прямой доступ для редактирования")
    Rel(ai_agent, operational, "Чтение для анализа")
    Rel(ai_agent, nsi, "Использование справочников")
    Rel(ai_agent, knowledge, "Извлечение знаний")

    Rel(metadata, operational, "Определяет структуру")
    Rel(nsi, operational, "Справочные значения")
    Rel(knowledge, operational, "Правила обработки")
```

### 8.2 Обращение к данным

```mermaid
sequenceDiagram
    autonumber
    participant User as "Пользователь"
    participant Client as "Толстый клиент"
    participant Meta as "Метаданные"
    participant Op as "Рабочие данные"
    participant NSI as "НСИ"
    participant Know as "Знания"

    Note over User, Client: Загрузка интерфейса
    User->>Client: Открытие формы
    Client->>Meta: Чтение конфигурации формы
    Meta-->>Client: Описание полей, валидация
    Client->>NSI: Загрузка справочных значений
    NSI-->>Client: Данные справочников
    Client-->>User: Отображение формы

    Note over User, Client: Работа с данными
    User->>Client: Ввод данных образца
    Client->>Meta: Проверка бизнес-правил
    Meta-->>Client: Валидация пройдена
    Client->>Op: Сохранение образца
    Op-->>Client: Подтверждение
    
    Note over User, Client: Получение рекомендаций
    User->>Client: Запрос методики
    Client->>Know: Извлечение знаний
    Know-->>Client: Описание методики
    Client-->>User: Отображение инструкции
```

---

## 9. Ручное редактирование метаданных

### 9.1 Два метода редактирования

**Визуальный конструктор (UI/UX)** — для настройки:
- Структуры форм (добавление полей, вкладок, групп)
- Таблиц просмотра (настройка колонок, фильтров, сортировки)
- Связей между объектами
- Маршрутов бизнес-процессов

**Текстовый редактор** — для:
- Написания скриптов на RemObjects Pascal Script
- Создания модулей и расширений
- Настройки валидаторов
- Модификации SQL-запросов

Прямой ввод текста доступен **только для кода**. Конфигурация интерфейсов настраивается исключительно через конструктор.

```mermaid
flowchart TD
    subgraph Manual_Editing ["Ручное редактирование метаданных"]
        direction TB
        
        User[Разработчик/Аналитик]
        
        subgraph Methods ["Методы редактирования"]
            Constructor[Конструктор UI/UX<br/>Визуальная настройка]
            Editor[Текстовый редактор<br/>Прямой ввод кода]
        end
        
        subgraph Metadata_Types ["Типы метаданных"]
            Forms[Формы ввода]
            Tables[Таблицы просмотра]
            Scripts[Скрипты/Модули<br/>Pascal Script]
            Validators[Валидаторы<br/>Pascal Script]
            Queries[SQL запросы]
        end
        
        subgraph Storage ["Хранение"]
            MetaDB[(Метаданные<br/>Firebird 5.0)]
        end
    end

    User --> Constructor
    User --> Editor
    
    Constructor --> Forms
    Constructor --> Tables
    
    Editor --> Scripts
    Editor --> Validators
    Editor --> Queries
    
    Forms --> MetaDB
    Tables --> MetaDB
    Scripts --> MetaDB
    Validators --> MetaDB
    Queries --> MetaDB
```

### 9.2 Процесс ручного редактирования

```mermaid
sequenceDiagram
    autonumber
    participant Dev as "Разработчик"
    participant TC as "Толстый клиент"
    participant UI_Constr as "Конструктор UI"
    participant Code_Edit as "Редактор кода"
    participant MetaDB as "Метаданные"

    Note over Dev, MetaDB: Сценарий 1: Настройка формы через конструктор
    
    Dev->>TC: Открытие конструктора форм
    TC->>UI_Constr: Загрузка текущей конфигурации
    UI_Constr-->>TC: Отображение формы
    Dev->>UI_Constr: Добавление поля "Дата отбора"
    UI_Constr->>MetaDB: Сохранение изменений
    MetaDB-->>UI_Constr: Подтверждение
    UI_Constr-->>TC: Обновление интерфейса
    TC-->>Dev: Отображение обновлённой формы

    Note over Dev, MetaDB: Сценарий 2: Написание скрипта
    
    Dev->>TC: Открытие редактора скриптов
    TC->>Code_Edit: Загрузка текущего кода
    Code_Edit-->>TC: Отображение кода
    Dev->>Code_Edit: Ввод Pascal Script кода
    Dev->>Code_Edit: Сохранение скрипта
    Code_Edit->>MetaDB: Запись скрипта в БД
    MetaDB-->>Code_Edit: Подтверждение
    Code_Edit-->>TC: Обновление интерфейса
```

---

## 10. Редактирование метаданных с помощью AI агента

### 10.1 Возможности AI агента

**Генерация кода:**
- Новые скрипты на RemObjects Pascal Script на основе описания задачи
- Модификации существующих модулей
- Валидаторы с заданными правилами
- SQL-запросы для выборки данных

**Контекстная генерация:**
- Чтение текущих метаданных для понимания структуры
- Анализ связанных объектов и зависимостей
- Учёт существующих паттернов кода Pascal Script
- Предложение изменений с учётом контекста

**Безопасность изменений:**
- Валидация перед сохранением
- Ручное подтверждение разработчиком
- Версионирование для возможности отката
- Логирование в аудите изменений

```mermaid
flowchart TD
    subgraph AI_Editing ["AI-редактирование метаданных"]
        direction TB
        
        User[Разработчик]
        
        subgraph Input ["Ввод"]
            Desc[Описание задачи<br/>естественным языком]
            Context[Контекст из метаданных]
        end
        
        subgraph AI_Processing ["Обработка AI"]
            Gateway[AI Gateway]
            Prompt[Prompt Engine]
            LLM[Нейросеть LLM]
            Validator[Валидация Pascal Script]
        end
        
        subgraph Output ["Результат"]
            Code[Сгенерированный код<br/>Pascal Script]
            Diff[Diff изменений]
            Recommend[Рекомендации]
        end
        
        subgraph Approval ["Подтверждение"]
            Review[Ручная проверка]
            Confirm[Подтверждение]
        end
        
        subgraph Storage ["Хранение"]
            MetaDB[(Метаданные<br/>Firebird 5.0)]
            Version[История версий]
            Audit[Аудит изменений]
        end
    end

    User --> Desc
    Context --> Gateway
    Desc --> Gateway
    Gateway --> Prompt
    Prompt --> LLM
    LLM --> Validator
    Validator --> Code
    Validator --> Diff
    Validator --> Recommend
    Code --> Review
    Diff --> Review
    Review --> Confirm
    Confirm --> MetaDB
    Confirm --> Version
    Confirm --> Audit
```

### 10.2 Процесс AI-редактирования

```mermaid
sequenceDiagram
    autonumber
    participant Dev as "Разработчик"
    participant TC as "Толстый клиент"
    participant AIG as "AI Gateway"
    participant LLM as "Нейросеть"
    participant MetaDB as "Метаданные"

    Note over Dev, MetaDB: Запрос на генерацию скрипта
    
    Dev->>TC: "Создай валидатор для проверки диапазона значений pH"
    TC->>AIG: Отправка запроса
    AIG->>MetaDB: Чтение контекста (структура формы, типы данных)
    MetaDB-->>AIG: Контекст метаданных
    AIG->>LLM: Prompt с контекстом и задачей
    LLM-->>AIG: Сгенерированный Pascal Script код
    AIG->>AIG: Валидация синтаксиса Pascal Script
    AIG-->>TC: Возврат кода + diff
    TC-->>Dev: Отображение предложенного кода
    
    Note over Dev, MetaDB: Проверка и сохранение
    
    Dev->>TC: Редактирование/принятие кода
    Dev->>TC: Подтверждение сохранения
    TC->>MetaDB: Запись скрипта в метаданные
    MetaDB-->>TC: Подтверждение
    TC->>MetaDB: Сохранение версии
    TC->>MetaDB: Запись в аудит
```

### 10.3 Пример генерации кода

**Запрос:** "Создай валидатор, который проверяет, что значение pH находится в диапазоне от 0 до 14"

**Сгенерированный код на Pascal Script:**
```pascal
function ValidatePH(Value: Variant): Boolean;
var
  PHValue: Double;
begin
  Result := False;
  
  if VarIsNull(Value) or VarIsEmpty(Value) then
  begin
    ShowMessage('Значение pH не может быть пустым');
    Exit;
  end;
  
  try
    PHValue := StrToFloat(VarToStr(Value));
  except
    on E: Exception do
    begin
      ShowMessage('Значение pH должно быть числом');
      Exit;
    end;
  end;
  
  if (PHValue < 0) or (PHValue > 14) then
  begin
    ShowMessage('Значение pH должно быть в диапазоне от 0 до 14');
    Exit;
  end;
  
  Result := True;
end;
```

---

## 11. AI агент как исполнитель процессов

### 11.1 Функции AI агента

**Автоматическое заполнение форм:**
- Анализ входных данных (описание образца, результаты измерений)
- Извлечение структурированной информации из неструктурированного текста
- Заполнение полей формы автоматически
- Предложение значений на основе контекста

**Обработка заявок:**
- Классификация входящих заявок по типу
- Определение необходимых методик испытаний
- Распределение задач между лаборантами
- Отслеживание статусов и сроков

**Валидация данных:**
- Проверка результатов на соответствие методикам
- Выявление аномалий и потенциальных ошибок
- Предложение корректировок или повторных измерений

**Консультирование:**
- Ответы на вопросы о методиках
- Подсказки по последовательности действий
- Объяснение причин ошибок валидации

```mermaid
flowchart TD
    subgraph AI_Process_Execution ["AI агент как исполнитель процессов"]
        direction TB
        
        subgraph Triggers ["Триггеры"]
            FormOpen[Открытие формы]
            NewRequest[Новая заявка]
            DataEntry[Ввод данных]
            UserQuestion[Вопрос пользователя]
        end
        
        subgraph AI_Agent ["AI Агент"]
            Context[Анализ контекста]
            Decision[Принятие решения]
            Action[Выполнение действия]
            Response[Формирование ответа]
        end
        
        subgraph Actions ["Действия"]
            FillForm[Заполнение формы]
            Classify[Классификация заявки]
            Validate[Валидация данных]
            Advise[Консультирование]
            Route[Маршрутизация задачи]
        end
        
        subgraph Data_Sources ["Источники данных"]
            OpData[(Рабочие данные<br/>Firebird 5.0)]
            NSI_Data[(НСИ<br/>Firebird 5.0)]
            Knowledge[(Знания<br/>Firebird 5.0)]
            MetaData[(Метаданные<br/>Firebird 5.0)]
        end
    end

    FormOpen --> Context
    NewRequest --> Context
    DataEntry --> Context
    UserQuestion --> Context
    
    Context --> Decision
    Decision --> Action
    Action --> Response
    
    Context --> Data_Sources
    Data_Sources --> Context
    
    Action --> FillForm
    Action --> Classify
    Action --> Validate
    Action --> Advise
    Action --> Route
```

### 11.2 Процесс выполнения AI агентом

```mermaid
sequenceDiagram
    autonumber
    participant User as "Пользователь"
    participant Client as "Толстый клиент"
    participant AI as "AI Агент"
    participant Op as "Рабочие данные"
    participant NSI as "НСИ"
    participant Know as "Знания"

    Note over User, Know: Сценарий 1: Автоматическое заполнение формы
    
    User->>Client: Открытие формы "Регистрация образца"
    Client->>AI: Триггер: форма открыта
    AI->>Op: Чтение связанных данных (заявка)
    Op-->>AI: Данные заявки
    AI->>NSI: Загрузка справочников (типы образцов)
    NSI-->>AI: Справочные значения
    AI->>AI: Анализ и предзаполнение полей
    AI-->>Client: Предложенные значения
    Client-->>User: Отображение заполненной формы
    User->>Client: Подтверждение/корректировка

    Note over User, Know: Сценарий 2: Обработка заявки
    
    User->>Client: Создание новой заявки
    Client->>AI: Триггер: новая заявка
    AI->>Know: Извлечение правил классификации
    Know-->>AI: Критерии классификации
    AI->>AI: Анализ текста заявки
    AI->>NSI: Определение методик
    NSI-->>AI: Список методик
    AI->>AI: Формирование заданий
    AI-->>Client: Предложение маршрута
    Client-->>User: Отображение маршрута

    Note over User, Know: Сценарий 3: Консультирование
    
    User->>Client: Вопрос "Как провести испытание X?"
    Client->>AI: Триггер: вопрос пользователя
    AI->>Know: Поиск методики X
    Know-->>AI: Описание методики
    AI->>Know: Извлечение пошаговой инструкции
    Know-->>AI: Пошаговый план
    AI->>AI: Формирование ответа
    AI-->>Client: Ответ с инструкцией
    Client-->>User: Отображение ответа
```

---

## 12. Использование AI агентом данных из базы

### 12.1 Источники данных для AI агента

**Рабочие данные:**
- История образцов и результатов для анализа трендов
- Сравнение текущих результатов с предыдущими измерениями
- Выявление отклонений и аномалий
- Прогноз сроков выполнения на основе загруженности

**НСИ:**
- Справочники для валидации введённых данных
- Классификаторы для категоризации объектов
- Сопоставление типов образцов с методиками
- Проверка единиц измерения и диапазонов

**Знания:**
- Методики для консультирования пользователей
- Правила интерпретации результатов
- Критерии приёмки/браковки
- Исторические прецеденты

**Метаданные:**
- Структура форм для понимания контекста
- Бизнес-логика на Pascal Script
- Валидаторы для проверки данных
- Конфигурация для модификации

```mermaid
flowchart TD
    subgraph AI_Data_Usage ["Использование данных AI агентом"]
        direction TB
        
        subgraph AI_Tasks ["Задачи AI агента"]
            Task1[Заполнение форм]
            Task2[Валидация данных]
            Task3[Консультирование]
            Task4[Принятие решений]
            Task5[Генерация отчётов]
        end
        
        subgraph AI_Agent ["AI Агент"]
            Query[Формирование SQL запросов]
            Extract[Извлечение информации]
            Analyze[Анализ данных]
            Synthesize[Синтез ответа]
        end
        
        subgraph Data_Layers ["Слои данных"]
            OpData[(Рабочие данные<br/>Firebird 5.0)]
            NSIData[(НСИ<br/>Firebird 5.0)]
            KnowledgeData[(Знания<br/>Firebird 5.0)]
            Metadata[(Метаданные<br/>Firebird 5.0)]
        end
        
        subgraph Results ["Результаты"]
            Res1[Предложенные значения]
            Res2[Проверенные данные]
            Res3[Инструкции и ответы]
            Res4[Рекомендации]
            Res5[Сформированные отчёты]
        end
    end

    Task1 --> Query
    Task2 --> Query
    Task3 --> Query
    Task4 --> Query
    Task5 --> Query
    
    Query --> Extract
    Extract --> Analyze
    Analyze --> Synthesize
    
    OpData --> Extract
    NSIData --> Extract
    KnowledgeData --> Extract
    Metadata --> Extract
    
    Synthesize --> Res1
    Synthesize --> Res2
    Synthesize --> Res3
    Synthesize --> Res4
    Synthesize --> Res5
```

### 12.2 Процесс извлечения и использования данных

```mermaid
sequenceDiagram
    autonumber
    participant Task as "Задача"
    participant AI as "AI Агент"
    participant Op as "Рабочие данные"
    participant NSI as "НСИ"
    participant Know as "Знания"
    participant Meta as "Метаданные"

    Note over Task, Meta: Задача: Валидация результата испытания
    
    Task->>AI: "Проверь результат: pH = 7.2 для воды питьевой"
    AI->>Meta: Чтение структуры данных (тип образца)
    Meta-->>AI: Тип: "Вода питьевая"
    AI->>NSI: Загрузка нормативов для воды
    NSI-->>AI: Норма pH: 6.0-9.0
    AI->>Op: Чтение истории результатов (контекст)
    Op-->>AI: Последние 5 измерений: 7.1, 7.3, 7.0, 7.2, 7.4
    AI->>Know: Извлечение правил интерпретации
    Know-->>AI: Правила: стабильность ±0.5, тренд
    AI->>AI: Анализ: 7.2 в норме, стабильно, без аномалий
    AI-->>Task: "Результат корректен. pH=7.2 в пределах нормы (6.0-9.0). Тренд стабильный."

    Note over Task, Meta: Задача: Рекомендация методики
    
    Task->>AI: "Какая методика для определения тяжёлых металлов в почве?"
    AI->>NSI: Поиск по типу образца "Почва"
    NSI-->>AI: Найдено: 3 методики
    AI->>Know: Загрузка описаний методик
    Know-->>AI: Методики: ААС, ICP-MS, XRF
    AI->>Op: Анализ исторических данных (что использовали)
    Op-->>AI: Частота: ICP-MS 70%, ААС 25%, XRF 5%
    AI->>Know: Извлечение критериев выбора
    Know-->>AI: Критерии: точность, стоимость, время
    AI->>AI: Синтез рекомендации
    AI-->>Task: "Рекомендую ICP-MS (используется в 70% случаев). Альтернатива: ААС для простых задач."
```

---

## 13. Интеграционная архитектура AI

### 13.1 Общая схема интеграции

```mermaid
C4Container
    title "AI Agent Integration Architecture"

    Person(user, "Пользователь", "Лаборант, руководитель")
    Person(dev, "Разработчик", "Настройка конфигурации")

    System_Boundary(jdx_system, "JDX System") {
        Container(thick_client, "Толстый клиент", "Data Express Client", "Основной интерфейс")
        Container(ai_gateway, "AI Gateway", "Integration Layer", "Управление AI-запросами")
        Container(process_engine, "Процессный движок", "Workflow Engine", "Выполнение бизнес-процессов")
        ContainerDb(metadata, "Метаданные", "Firebird 5.0", "Конфигурация системы, Pascal Script")
        ContainerDb(operational, "Рабочие данные", "Firebird 5.0", "Операционные данные")
        ContainerDb(nsi, "НСИ", "Firebird 5.0", "Справочники")
        ContainerDb(knowledge, "Знания", "Firebird 5.0", "Методики, правила")
    }

    System_Ext(llm, "Нейросеть", "LLM API", "GPT/Claude/локальная модель")

    Rel(user, thick_client, "Работа с системой")
    Rel(dev, thick_client, "Настройка конфигурации")
    
    Rel(thick_client, ai_gateway, "AI-запросы")
    Rel(ai_gateway, llm, "API вызовы", "HTTPS/REST")
    
    Rel(ai_gateway, metadata, "Чтение/запись метаданных")
    Rel(ai_gateway, operational, "Чтение данных")
    Rel(ai_gateway, nsi, "Использование справочников")
    Rel(ai_gateway, knowledge, "Извлечение знаний")
    
    Rel(process_engine, ai_gateway, "Вызов AI для процессов")
    Rel(process_engine, operational, "Обновление данных")
    
    Rel(thick_client, metadata, "Чтение конфигурации")
    Rel(thick_client, operational, "Работа с данными")
```

### 13.2 Компоненты AI Gateway

```mermaid
C4Component
    title "Component Diagram - AI Gateway"

    Container_Boundary(ai_gateway, "AI Gateway") {
        Component(prompt_engine, "Prompt Engine", "Template Manager", "Формирование запросов к LLM")
        Component(context_loader, "Context Loader", "Data Fetcher", "Загрузка контекста из метаданных")
        Component(code_parser, "Code Parser", "Validator", "Проверка сгенерированного Pascal Script")
        Component(diff_viewer, "Diff Viewer", "UI Component", "Отображение изменений в коде")
        Component(version_ctrl, "Version Control", "History", "Версионирование изменений метаданных")
    }

    System_Ext(llm_api, "LLM API", "External Service", "GPT/Claude/локальная модель")
    Container(script_editor, "Редактор скриптов", "Text Editor", "В толстом клиенте")
    ContainerDb(metadata_db, "Метаданные", "Firebird 5.0", "Хранение скриптов и модулей")

    Rel(prompt_engine, context_loader, "Запрашивает контекст")
    Rel(context_loader, metadata_db, "Чтение текущих метаданных")
    Rel(prompt_engine, llm_api, "Отправка prompt", "HTTPS/REST")
    Rel(llm_api, prompt_engine, "Возврат сгенерированного кода")
    Rel(code_parser, prompt_engine, "Валидация кода")
    Rel(diff_viewer, script_editor, "Интеграция в UI")
    Rel(version_ctrl, metadata_db, "Сохранение версий")
```

### 13.3 Поток данных при различных сценариях

```mermaid
flowchart LR
    subgraph Scenarios ["Сценарии использования"]
        S1[Редактирование метаданных]
        S2[Выполнение процессов]
        S3[Консультирование]
    end

    subgraph Data_Flow ["Поток данных"]
        direction TB
        
        subgraph Input ["Вход"]
            I1[Запрос пользователя]
            I2[Триггер процесса]
            I3[Событие в системе]
        end
        
        subgraph AI_Processing ["Обработка AI"]
            P1[Извлечение контекста]
            P2[Запрос к LLM]
            P3[Анализ ответа]
        end
        
        subgraph Data_Access ["Доступ к данным"]
            D1[Чтение метаданных]
            D2[Чтение рабочих данных]
            D3[Чтение НСИ]
            D4[Чтение знаний]
        end
        
        subgraph Output ["Выход"]
            O1[Изменённые метаданные]
            O2[Заполненные формы]
            O3[Ответы пользователю]
            O4[Рекомендации]
        end
    end

    S1 --> I1
    S2 --> I2
    S3 --> I3
    
    I1 --> P1
    I2 --> P1
    I3 --> P1
    
    P1 --> D1
    P1 --> D2
    P1 --> D3
    P1 --> D4
    
    D1 --> P2
    D2 --> P2
    D3 --> P2
    D4 --> P2
    
    P2 --> P3
    P3 --> O1
    P3 --> O2
    P3 --> O3
    P3 --> O4
```

---

## 14. Безопасность и контроль AI-действий

### 14.1 Схема контроля

```mermaid
flowchart TD
    subgraph AI_Control ["Контроль AI-действий"]
        direction TB
        
        subgraph Input_Validation ["Валидация входа"]
            IV1[Проверка запроса]
            IV2[Санитизация данных]
            IV3[Проверка прав доступа]
        end
        
        subgraph AI_Processing ["Обработка"]
            AP1[Формирование prompt]
            AP2[Вызов LLM]
            AP3[Парсинг ответа]
        end
        
        subgraph Output_Validation ["Валидация выхода"]
            OV1[Проверка синтаксиса Pascal Script]
            OV2[Тестирование в песочнице]
            OV3[Ручное подтверждение]
        end
        
        subgraph Audit ["Аудит"]
            AU1[Логирование запросов]
            AU2[Логирование изменений]
            AU3[Версионирование]
        end
    end

    IV1 --> IV2
    IV2 --> IV3
    IV3 --> AP1
    AP1 --> AP2
    AP2 --> AP3
    AP3 --> OV1
    OV1 --> OV2
    OV2 --> OV3
    OV3 --> AU1
    AU1 --> AU2
    AU2 --> AU3
```

### 14.2 Матрица безопасности

| Действие AI | Уровень риска | Меры контроля | Статус |
|-------------|---------------|---------------|--------|
| Чтение метаданных | Низкий | Аудит запросов | Обязательно |
| Чтение рабочих данных | Средний | Проверка прав доступа | Обязательно |
| Генерация Pascal Script | Высокий | Валидация синтаксиса + ручное подтверждение | Обязательно |
| Изменение метаданных | Критический | Тестирование + версионирование + откат | Обязательно |
| Заполнение форм | Средний | Валидация результатов | Обязательно |
| Консультирование | Низкий | Проверка источников | Рекомендуется |

### 14.3 Особенности валидации Pascal Script

**Проверка синтаксиса:**
- Компиляция в изолированной среде
- Проверка типов данных
- Валидация вызовов API платформы
- Проверка доступа к переменным

**Ограничения безопасности:**
- Запрет прямого доступа к файловой системе
- Ограничение вызовов внешних библиотек
- Контроль доступа к системным функциям
- Песочница для выполнения сгенерированного кода

---

# ЧАСТЬ III. ПЛАН ВНЕДРЕНИЯ AI

---

## 15. План внедрения AI в систему JDX

### 15.1 Цели внедрения

**Стратегические цели:**
1. Сокращение времени разработки конфигурации JDX на 40-60%
2. Автоматизация рутинных операций лаборантов
3. Повышение качества данных за счёт AI-валидации
4. Создание интеллектуального помощника для пользователей

**Тактические цели:**
- Внедрить AI-ассистент для генерации Pascal Script
- Автоматизировать заполнение типовых форм
- Реализовать AI-консультирование по методикам
- Настроить автоматическую классификацию заявок

### 15.2 Фазы внедрения

```mermaid
gantt
    title "План внедрения AI в систему JDX"
    dateFormat YYYY-MM-DD
    axisFormat %Y-%m
    
    section Фаза 0: Подготовка
    Анализ текущей архитектуры :2026-11-01, 30d
    Формирование команды :2026-11-15, 20d
    Выбор LLM-провайдера :2026-12-01, 30d
    Подготовка инфраструктуры :2026-12-15, 45d
    
    section Фаза 1: AI Gateway
    Разработка AI Gateway :2027-01-15, 60d
    Интеграция с LLM API :2027-02-15, 45d
    Настройка безопасности :2027-03-15, 30d
    Тестирование :2027-04-01, 30d
    
    section Фаза 2: AI-редактор кода
    Интеграция в толстый клиент :2027-04-15, 45d
    Prompt engineering для Pascal Script :2027-05-01, 60d
    Валидация сгенерированного кода :2027-06-15, 45d
    Пилотное тестирование :2027-07-15, 30d
    
    section Фаза 3: AI-исполнитель процессов
    Разработка процессного движка :2027-08-01, 60d
    Автозаполнение форм :2027-09-15, 45d
    Классификация заявок :2027-10-15, 45d
    Валидация результатов :2027-11-15, 45d
    
    section Фаза 4: AI-консультант
    База знаний методик :2027-12-15, 60d
    Интеграция с НСИ :2028-02-01, 45d
    Консультационный интерфейс :2028-03-01, 45d
    Обучение пользователей :2028-04-01, 30d
    
    section Фаза 5: Оптимизация
    Сбор обратной связи :2028-04-15, 30d
    Оптимизация промптов :2028-05-01, 45d
    Локальная LLM модель :2028-06-01, 90d
    Масштабирование :2028-08-15, 60d
```

### 15.3 Детализация фаз

#### Фаза 0: Подготовка (ноябрь 2026 — январь 2027)

**Задачи:**
- Детальный анализ архитектуры Data Express
- Инвентаризация существующих метаданных
- Формирование команды проекта
- Выбор LLM-провайдера (облачный vs локальный)
- Подготовка инфраструктуры (серверы, сеть, безопасность)

**Результаты:**
- Документ "Требования к AI-интеграции"
- Утверждённая архитектура AI Gateway
- Развёрнутая тестовая среда
- Команда проекта укомплектована

**Риски:**
- ⚠️ Ограничения платформы Data Express на прямое редактирование метаданных
- ⚠️ Требования безопасности лаборатории к внешним API
- ⚠️ Нехватка компетенций по LLM

**Митигация:**
- Провести PoC с платформой Data Express
- Рассмотреть гибридный подход (локальная LLM + облачная)
- Обучение команды или привлечение экспертов

---

#### Фаза 1: AI Gateway (январь — апрель 2027)

**Задачи:**
- Разработка компонента AI Gateway
- Интеграция с выбранным LLM API
- Настройка системы безопасности (аутентификация, авторизация, аудит)
- Реализация механизма версионирования изменений
- Тестирование производительности

**Результаты:**
- Рабочий AI Gateway
- API для взаимодействия с LLM
- Система аудита всех AI-действий
- Документация по интеграции

**Критерии приёмки:**
- ✅ Успешная генерация тестового Pascal Script кода
- ✅ Время ответа < 5 секунд для типовых запросов
- ✅ 100% аудит всех запросов к LLM
- ✅ Прохождение security-тестирования

---

#### Фаза 2: AI-редактор кода (апрель — август 2027)

**Задачи:**
- Интеграция AI-функционала в толстый клиент
- Разработка Prompt Engine для Pascal Script
- Создание валидатора синтаксиса Pascal Script
- Реализация diff-просмотрщика изменений
- Пилотное тестирование на ограниченной группе разработчиков

**Результаты:**
- AI-редактор в толстом клиенте
- Библиотека промптов для типовых задач
- Валидатор Pascal Script
- Обратная связь от пилотных пользователей

**Критерии приёмки:**
- ✅ Точность генерации кода > 80%
- ✅ Сокращение времени разработки скриптов на 50%
- ✅ Отсутствие критических ошибок в сгенерированном коде
- ✅ Положительная обратная связь от 80% пилотных пользователей

---

#### Фаза 3: AI-исполнитель процессов (август 2027 — январь 2028)

**Задачи:**
- Разработка процессного движка с AI-компонентом
- Реализация автозаполнения форм
- Настройка автоматической классификации заявок
- Внедрение AI-валидации результатов
- Интеграция с рабочими данными и НСИ

**Результаты:**
- Автоматическое заполнение типовых форм
- AI-классификатор заявок
- AI-валидатор результатов
- Интеграция со всеми слоями данных

**Критерии приёмки:**
- ✅ Точность автозаполнения > 85%
- ✅ Точность классификации заявок > 90%
- ✅ Выявление аномалий в результатах > 75%
- ✅ Сокращение времени обработки заявки на 30%

---

#### Фаза 4: AI-консультант (декабрь 2027 — май 2028)

**Задачи:**
- Формирование базы знаний методик
- Интеграция с НСИ и рабочими данными
- Разработка консультационного интерфейса
- Обучение модели на исторических данных
- Обучение пользователей

**Результаты:**
- AI-консультант по методикам
- Система рекомендаций
- Обученная модель на исторических данных
- Обученные пользователи

**Критерии приёмки:**
- ✅ Точность рекомендаций > 80%
- ✅ Сокращение времени поиска информации на 60%
- ✅ Положительная обратная связь от 75% пользователей
- ✅ Снижение количества ошибок при выборе методики

---

#### Фаза 5: Оптимизация и масштабирование (апрель — октябрь 2028)

**Задачи:**
- Сбор и анализ обратной связи
- Оптимизация промптов и моделей
- Развёртывание локальной LLM для конфиденциальных данных
- Масштабирование на все подразделения
- Подготовка к промышленной эксплуатации

**Результаты:**
- Оптимизированная AI-система
- Локальная LLM модель
- Полномасштабное развёртывание
- Документация по эксплуатации

**Критерии приёмки:**
- ✅ Система работает стабильно 99.5% времени
- ✅ Все пользователи обучены
- ✅ Полная документация
- ✅ План развития на следующий год

---

### 15.4 Команда проекта

| Роль | Количество | Ответственность |
|------|-----------|-----------------|
| Руководитель проекта | 1 | Общее управление, координация |
| Архитектор Data Express | 1 | Архитектура платформы, метаданные |
| Backend-разработчик | 2 | AI Gateway, интеграции |
| ML-инженер | 1 | Prompt engineering, LLM |
| Frontend-разработчик | 1 | Интеграция в толстый клиент |
| QA-инженер | 1 | Тестирование, валидация |
| Аналитик | 1 | Требования, документация |
| Специалист по безопасности | 0.5 | Аудит, security-тестирование |

**Итого:** 7.5 FTE

---

### 15.5 Бюджет (оценка)

| Статья | Стоимость (руб.) | Период |
|--------|-----------------|--------|
| LLM API (облачные модели) | 1 500 000 | 2 года |
| Серверное оборудование | 2 000 000 | Разово |
| Локальная LLM (GPU-сервер) | 3 000 000 | Разово |
| Лицензии на ПО | 500 000 | 2 года |
| Обучение команды | 400 000 | Разово |
| Консалтинг и экспертиза | 1 000 000 | 2 года |
| **ИТОГО** | **~8 400 000** | 2 года |

---

### 15.6 Матрица рисков

| Риск | Вероятность | Влияние | Митигация |
|------|-------------|---------|-----------|
| Ограничения платформы Data Express | Средняя | Высокое | PoC на ранней стадии, альтернативные подходы |
| Неточность генерации кода | Высокая | Среднее | Многоуровневая валидация, ручное подтверждение |
| Утечка конфиденциальных данных через LLM | Низкая | Критическое | Локальная LLM, санитизация данных |
| Сопротивление пользователей | Средняя | Среднее | Обучение, пилоты, обратная связь |
| Превышение бюджета | Средняя | Среднее | Поэтапное финансирование, контроль |
| Зависимость от внешнего LLM-провайдера | Средняя | Высокое | Локальная модель как резерв |
| Недостаток компетенций | Средняя | Высокое | Обучение, привлечение экспертов |

---

### 15.7 Критерии успеха проекта

**Количественные:**
- Сокращение времени разработки конфигурации на 40-60%
- Сокращение времени обработки заявки на 30%
- Точность AI-рекомендаций > 80%
- Uptime системы > 99.5%
- ROI проекта в течение 18 месяцев

**Качественные:**
- Положительная обратная связь от 80% пользователей
- Снижение количества ошибок в данных
- Повышение удовлетворённости клиентов лаборатории
- Формирование конкурентного преимущества

---

### 15.8 Дорожная карта развития после внедрения

```mermaid
gantt
    title "Дорожная карта развития AI после внедрения"
    dateFormat YYYY
    axisFormat %Y
    
    section 2029
    Мультимодальный AI (голос, изображения) :2029, 365d
    Интеграция с лабораторным оборудованием :2029, 365d
    
    section 2030
    Предиктивная аналитика :2030, 365d
    Автоматическая оптимизация методик :2030, 365d
    
    section 2031
    Полностью автономная лаборатория :2031, 365d
    AI-driven управление качеством :2031, 365d
```

---

## 16. Приложения

### 16.1 Глоссарий

| Термин | Определение |
|--------|-------------|
| JDX | Конфигурация системы учёта на платформе Data Express для лабораторий |
| Data Express | Low-code платформа для построения систем учёта (mydataexpress.ru) |
| Firebird 5.0 | Реляционная СУБД, используемая для хранения данных |
| RemObjects Pascal Script | Язык программирования для скриптов и бизнес-логики |
| Толстый клиент | Нативное Windows-приложение для работы с системой внутри контура |
| Метаданные | Конфигурация системы: формы, скрипты, запросы, валидаторы |
| НСИ | Нормативно-справочная информация: справочники, классификаторы |
| LLM | Large Language Model — большая языковая модель |
| AI Gateway | Компонент для интеграции AI-функционала в систему |
| DMZ | Демилитаризованная зона — сегмент сети между интернетом и внутренним контуром |
| Reverse Proxy | Сервер-посредник для обработки входящих запросов |
| Prompt Engineering | Искусство формирования запросов к LLM для получения нужного результата |

### 16.2 История изменений документа

| Версия | Дата | Автор | Изменения |
|--------|------|-------|-----------|
| 1.0 | 2026 | Аналитик | Первоначальная версия архитектуры |
| 2.0 | 28.10.2026 | Аналитик | Добавлены разделы по AI-интеграции и план внедрения |

### 16.3 Ссылки

- Платформа Data Express: https://mydataexpress.ru
- Документация Firebird 5.0: https://firebirdsql.org
- RemObjects Pascal Script: https://www.remobjects.com/ps.aspx
- Спецификация C4 Model: https://c4model.com
- Mermaid Live Editor: https://mermaid.live

---

> **Примечание для аккредитации:**  
> Данный документ может использоваться как часть технической документации при прохождении аккредитации лаборатории. Рекомендуется дополнить его:
> - Политикой резервного копирования и восстановления
> - Регламентом управления изменениями конфигурации
> - Инструкцией по реагированию на инциденты информационной безопасности
> - Планом непрерывности бизнеса с учётом AI-компонента

---

**Документ подготовлен в формате Markdown с диаграммами Mermaid.**  
Все диаграммы совместимы с рендерерами: GitHub, GitLab, Mermaid Live, Obsidian, Typora.

