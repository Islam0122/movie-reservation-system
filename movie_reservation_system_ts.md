# Movie Reservation System
## Technical Specification (ТЗ)

**Версия документа:** 1.0 (Draft for Team Approval)
**Дата:** 02.09.2026
**Команда:** Islam (PM / Tech Lead / Backend Developer), Khadizha (Backend Developer)
**Статус:** На рассмотрении — требует утверждения обеими сторонами перед началом разработки

---

## 1. Project Overview

**Название проекта:** Movie Reservation System

**Тип проекта:** Backend-ориентированный portfolio-проект (REST API), разработанный на основе задания с roadmap.sh, но расширенный до уровня полноценного production-style приложения.

**Краткое описание:** Система бронирования мест в кинотеатре, позволяющая пользователям просматривать фильмы, расписания сеансов и бронировать конкретные места в зале, а администраторам — управлять фильмами, кинотеатрами, залами, сеансами и получать аналитические отчёты.

**Проблема, которую решает система:** Ручное или неструктурированное управление сеансами и бронированием мест приводит к двойным бронированиям (overbooking), отсутствию прозрачности занятости зала и невозможности анализировать эффективность показов. Система централизует эти процессы и гарантирует целостность данных при конкурентном доступе.

**Основная цель:** Спроектировать и реализовать надёжный backend с корректной реляционной моделью данных, строгой бизнес-логикой бронирования, защитой от race conditions и полным набором инструментов разработки (тесты, CI/CD, документация), достаточным для портфолио уровня Strong Junior Backend Developer.

**Целевая аудитория:**
- Конечные пользователи (USER) — бронируют билеты на сеансы.
- Администраторы (ADMIN) — управляют контентом и получают отчётность.
- Косвенная аудитория: технические рекрутеры и интервьюеры, оценивающие проект как портфолио.

**Границы проекта:** Backend REST API без фронтенд-приложения. Взаимодействие с системой предполагается через API-клиенты (Postman/Swagger UI) и автотесты. Оплата эмулируется на уровне статуса резервирования, без интеграции с реальными платёжными системами.

**Что входит в MVP:**
- Регистрация/аутентификация/авторизация (JWT).
- CRUD фильмов, жанров, кинотеатров, залов, мест, сеансов (для ADMIN).
- Просмотр и фильтрация фильмов/сеансов (для USER).
- Создание, просмотр, отмена бронирований с защитой от overbooking.
- Отчётность для ADMIN.
- Docker-окружение, тесты, CI, документация.

**Что НЕ входит в MVP:**
- Реальная оплата и платёжные шлюзы.
- Frontend/мобильное приложение.
- Уведомления (email/SMS/push).
- Динамическое ценообразование, скидки, промокоды.
- Multi-tenancy / поддержка нескольких сетей кинотеатров с разными владельцами.
- Микросервисная архитектура.

---

## 2. Project Objectives

1. Спроектировать нормализованную реляционную схему данных (PostgreSQL), корректно отражающую сущности предметной области и их связи.
2. Реализовать REST API на Django REST Framework с чётким разделением на модули (apps) согласно принципам modular monolith.
3. Реализовать аутентификацию и авторизацию на основе JWT с ролевой моделью (USER / ADMIN).
4. Реализовать бизнес-логику бронирования мест, включая многошаговый выбор мест, расчёт стоимости и статусы резервирования.
5. Гарантировать защиту от конкурентного двойного бронирования (anti-overbooking) с использованием транзакций БД и механизмов блокировок.
6. Реализовать аналитическую отчётность для администраторов (выручка, занятость, топ фильмов и сеансов).
7. Обеспечить надёжное покрытие тестами: unit, integration, API, permission, concurrency.
8. Настроить контейнеризацию (Docker/Docker Compose) и автоматизацию (GitHub Actions CI/CD).
9. Подготовить полную техническую документацию (README, OpenAPI/Swagger).
10. Отработать процесс командной разработки: Git workflow, code review, task tracking через Trello.

---

## 3. Team & Responsibilities

### Islam — Project Manager / Tech Lead / Backend Developer

Responsibilities:
- Project planning и декомпозиция задач (ТЗ → Trello).
- Итоговые архитектурные решения (после обсуждения с командой).
- Проектирование схемы базы данных.
- Управление GitHub-репозиторием (ветки, релизы, доступы).
- Код-ревью Pull Request'ов Khadizha.
- Backend-разработка ключевых модулей (auth, reservations, anti-overbooking).
- Интеграция модулей между собой.
- Настройка Docker/CI/CD и деплой.
- Подготовка и проведение финальной презентации проекта.

### Khadizha — Backend Developer

Responsibilities:
- Backend-разработка закреплённых модулей (movies/cinemas/showtimes/reports — распределяется на этапе Trello-планирования).
- Написание тестов для своих модулей.
- Код-ревью Pull Request'ов Islam.
- Ведение документации по своим модулям (docstrings, README-разделы).
- Исправление багов, найденных на код-ревью или в CI.
- Участие в обсуждении архитектурных решений наравне с Islam.

**Важно:** проект — collaborative. Islam выполняет роль PM/Tech Lead и несёт финальную ответственность за архитектурные решения, но любые значимые архитектурные вопросы (например, раздел 14 "Data Model", Option A/B) должны обсуждаться и приниматься совместно.

---

## 4. Functional Requirements

### 4.1 Authentication

- **Registration** — регистрация нового пользователя (email, password, имя). Пароль должен проходить валидацию сложности.
- **Login** — вход по email/паролю, выдача access и refresh JWT-токенов.
- **Logout** — инвалидация refresh-токена (blacklist).
- **Token refresh** — обновление access-токена по действующему refresh-токену.
- **Current user** — получение данных текущего аутентифицированного пользователя (`/auth/me`).
- **Password security** — хранение паролей только в виде хэша (Django встроенный hasher), минимальные требования к сложности пароля.

### 4.2 Authorization

Роли:
- **USER** — базовая роль по умолчанию для всех зарегистрированных пользователей. Может просматривать каталог, бронировать и управлять только своими бронированиями.
- **ADMIN** — роль с полным доступом к управлению контентом (фильмы, кинотеатры, залы, сеансы) и отчётностью. Назначается вручную (например, через Django admin или management-команду), самостоятельная регистрация как ADMIN не предусмотрена.

Подробная матрица permissions — см. раздел 17 "Authorization Matrix".

---

## 5. Movie Management

**Сущности:**
- **Movie** — фильм: title, description, duration (минуты), release_date, poster (image/URL), genres (M2M), возможно age_rating.
- **Genre** — жанр фильма (например, Action, Drama), простая справочная сущность с уникальным названием.

**Функциональность:**
- Movie CRUD — создание, чтение, обновление, удаление (только ADMIN).
- Genre management — CRUD жанров (только ADMIN).
- Movie search — поиск по названию (частичное совпадение).
- Filtering — фильтрация по жанру, по диапазону дат релиза, по наличию активных сеансов.
- Pagination — постраничная выдача списка фильмов.
- Ordering — сортировка по названию, дате релиза, длительности.
- Poster — хранение изображения постера (файл или внешняя ссылка — решается на этапе реализации).
- Release date и duration — обязательные поля для планирования сеансов и валидации showtime.

**Доступ:**
- USER — только чтение (список, детали, поиск, фильтрация).
- ADMIN — полный CRUD над Movie и Genre.

---

## 6. Cinema Management

**Иерархия сущностей:**

```text
Cinema
 └── Hall
      └── Seat
```

- **Cinema** — кинотеатр: name, address, город (опционально).
- **Hall** — зал внутри кинотеатра: name/номер, cinema (FK), capacity (вычисляется или задаётся явно).
- **Seat** — место внутри зала: hall (FK), row, seat_number, возможно seat_type (обычное/VIP — опционально, вне MVP по умолчанию).

**Операции:**
- Создание, изменение, удаление Cinema/Hall/Seat — доступно только ADMIN.
- Capacity — вместимость зала определяется количеством связанных Seat; при удалении Hall/Cinema с активными будущими showtimes должна применяться защита (soft-constraint) от случайного удаления.
- Уникальность — комбинация (hall, row, seat_number) должна быть уникальной в рамках одного Hall.
- Ограничения — Seat не может существовать без Hall; Hall не может существовать без Cinema (обязательные FK, on_delete политика определяется на этапе реализации, рекомендуется PROTECT/RESTRICT для сущностей с историей бронирований).

---

## 7. Showtime Management

**Сущность Showtime** связывает:

```text
Movie + Hall + Start Time + End Time + Price
```

**Обязательные поля:** movie (FK), hall (FK), start_time, end_time, price.

**Бизнес-правила:**
- Нельзя создать showtime в прошлом (start_time не может быть раньше текущего момента на момент создания).
- `start_time < end_time`.
- `price > 0`.
- Один hall не может иметь пересекающиеся showtimes (по времени) — при создании/обновлении showtime система должна проверять отсутствие временного пересечения с другими showtime в этом же hall.
- Movie должен существовать (валидный FK, статус фильма — активен).
- Hall должен существовать (валидный FK).

Доступ: USER — только чтение (список, детали, фильтрация по фильму/дате/кинотеатру); ADMIN — полный CRUD.

---

## 8. Reservation System

Центральная часть проекта.

**Функциональность:**
- **Создание reservation** — пользователь выбирает showtime и один или несколько seats, система создаёт запись бронирования.
- **Выбор нескольких seats** — один reservation может включать несколько мест (например, для группы/семьи), связь реализуется через промежуточную сущность (см. раздел 14).
- **Просмотр reservation** — получение деталей конкретного бронирования (владельцем или ADMIN).
- **Просмотр своих reservations** — список всех бронирований текущего пользователя с фильтрацией по статусу.
- **Cancellation** — отмена бронирования согласно правилам раздела 11.
- **Reservation status** — например: `PENDING`/`CONFIRMED`, `CANCELLED` (точный набор статусов уточняется на этапе реализации, но обязательно должен различать активные и отменённые бронирования).
- **Total price** — рассчитывается как сумма цены showtime за количество забронированных мест (или price × количество seats).

**API requirements (без конкретной реализации):**
- Endpoint для создания бронирования должен принимать showtime_id и список seat_id.
- Система должна атомарно проверять доступность всех выбранных мест перед подтверждением.
- Ответ должен включать итоговую стоимость и статус бронирования.
- USER видит только свои бронирования; ADMIN может видеть все.

---

## 9. Seat Availability

Система должна позволять пользователю увидеть для каждого места конкретного showtime один из статусов:

```text
AVAILABLE
RESERVED
```

**Ключевое бизнес-требование:** Seat физически принадлежит Hall (статичная сущность), но его **availability** (доступность) — это динамическая характеристика, зависящая от пары:

```text
Showtime + Seat
```

Одно и то же физическое место может быть `AVAILABLE` для одного showtime и `RESERVED` для другого. Это должно быть явно отражено в модели данных (см. раздел 14, Option A/B) и в API (endpoint получения карты мест для конкретного showtime).

---

## 10. Anti-Overbooking (Критическое требование)

Система **не должна** позволять двум пользователям одновременно успешно забронировать одно и то же место на один и тот же showtime.

**Иллюстрация:**

```text
User A ──┐
         ├── Seat A5, Showtime #42
User B ──┘

Результат:
User A → SUCCESS
User B → CONFLICT (409)
```

Это фиксируется как обязательное business requirement верхнего приоритета.

Конкретная реализация не диктуется на этапе ТЗ, но архитектура **обязана** учитывать:
- **Database transactions** — операция бронирования должна выполняться в рамках единой транзакции.
- **Concurrency** — возможность одновременных запросов на одно и то же место должна быть предусмотрена в тестах и логике.
- **Race conditions** — необходимо избегать окна между "проверкой доступности" и "созданием бронирования", в котором возможна гонка.
- **Database constraints** — рекомендуется рассмотреть уникальные ограничения на уровне БД (например, unique constraint на пару showtime+seat в таблице занятых мест) как последний рубеж защиты.
- **Atomic operations** — операция резервирования нескольких мест должна быть all-or-nothing: либо бронируются все запрошенные места, либо ни одно.

Конкретный механизм (select_for_update, database-level unique constraint, optimistic locking и т.д.) — предмет отдельного архитектурного обсуждения командой перед началом Phase 6/7.

---

## 11. Reservation Cancellation

Правила отмены бронирования, привязанные к состоянию showtime и текущему статусу reservation:

```text
Upcoming (showtime ещё не начался)  → can cancel
Started (showtime уже идёт/начался) → cannot cancel
Finished (showtime завершён)        → cannot cancel
Cancelled (уже отменено)            → cannot cancel again
```

Дополнительно рекомендуется рассмотреть (на усмотрение команды при реализации): минимальное время до начала сеанса, после которого отмена уже недоступна (например, "нельзя отменить менее чем за N минут до начала") — фиксируется как опциональное расширение, не обязательное для MVP.

---

## 12. Reporting

ADMIN должен иметь возможность получать следующие отчёты (сгруппированы по категориям):

**Reservations Analytics**
- Total reservations.
- Active reservations.
- Cancelled reservations.
- Reservations by date (динамика по датам/периодам).

**Financial Analytics**
- Total revenue.
- Revenue by date.
- Average ticket price.

**Occupancy & Performance**
- Occupancy rate (заполняемость залов по сеансам).
- Cinema/hall capacity utilization.
- Top movies (по числу бронирований/выручке).
- Top showtimes (наиболее заполняемые сеансы).

Отчёты доступны только ADMIN, с поддержкой фильтрации по периоду (date range) как минимум для финансовых и occupancy-отчётов.

---

## 13. Non-Functional Requirements

**Performance**
- Пагинация обязательна для всех list-endpoint'ов.
- Эффективные запросы к БД (select_related/prefetch_related там, где это уместно).
- Избегание N+1 запросов.
- Индексы на часто фильтруемых/связываемых полях (FK, поля дат, поля поиска).

**Security**
- Хэширование паролей (Django default hasher, никогда не plain text).
- JWT для аутентификации (access + refresh).
- Проверка авторизации на уровне каждого endpoint (permission classes).
- Управление секретами через переменные окружения, никогда не в коде.
- Секреты и `.env` не должны попадать в GitHub (обязательный `.gitignore`).

**Reliability**
- Использование транзакций БД для операций, критичных к целостности (в первую очередь — бронирование).
- Гарантия целостности данных (foreign key constraints, валидация на уровне модели и сериализатора).
- Защита от конкурентных конфликтов (см. раздел 10).

**Maintainability**
- Модульная архитектура Django-приложений (apps по доменам).
- Чистый, читаемый код, соответствующий PEP8 (рекомендуется линтер).
- Docstrings и комментарии там, где логика нетривиальна.
- Актуальный набор тестов, сопровождающий каждый модуль.

**Scalability**
- Базовые требования без overengineering: приложение должно быть стейтлесс (кроме БД/кэша), готово к горизонтальному масштабированию через несколько инстансов за балансировщиком в будущем, но реализация Kubernetes/микросервисов на этом этапе не предусмотрена.

---

## 14. Data Model / ERD

**Сущности:**

```text
User
Movie
Genre
Cinema
Hall
Seat
Showtime
Reservation
ReservationSeat
```

**Предварительные связи:**

- `User` 1—* `Reservation` (один пользователь может иметь много бронирований).
- `Movie` *—* `Genre` (many-to-many через промежуточную таблицу).
- `Cinema` 1—* `Hall` (один кинотеатр содержит много залов).
- `Hall` 1—* `Seat` (один зал содержит много мест).
- `Movie` 1—* `Showtime`, `Hall` 1—* `Showtime` (сеанс всегда привязан к одному фильму и одному залу).
- `Reservation` 1—* `ReservationSeat` (одно бронирование может включать несколько мест).
- `Showtime` 1—* `Reservation` (опосредованно, через ReservationSeat — один showtime может иметь много бронирований).

**Unique constraints (предварительно):**
- `Seat`: unique_together (hall, row, seat_number).
- `Genre`: unique(name).
- Занятость места на конкретный showtime — должна быть защищена уникальным ограничением (конкретное поле/таблица — предмет решения в Option A/B ниже).

**Два архитектурных варианта моделирования занятости мест — для обсуждения командой:**

### Option A (проще)

```text
ReservationSeat → Seat
```

`ReservationSeat` хранит прямую ссылку на `Reservation`, `Showtime` и `Seat`. Уникальный constraint накладывается на пару (showtime, seat) в таблице `ReservationSeat` (с учётом того, что отменённые бронирования не должны блокировать место — например, через partial unique index или проверку статуса).

*Плюсы:* меньше таблиц, проще миграции и запросы.
*Минусы:* логика "свободно ли место" зависит от статуса связанных reservation, что усложняет unique constraint (нужен partial/conditional index).

### Option B (через промежуточную сущность занятости)

```text
Reservation
      ↓
ReservationSeat
      ↓
ShowtimeSeat
      ↓
Seat
```

Вводится дополнительная сущность `ShowtimeSeat`, явно представляющая "место в контексте конкретного showtime" (по сути — материализация доступности). `ReservationSeat` ссылается на `ShowtimeSeat`, а не напрямую на `Seat`. Уникальность на (showtime, seat) естественным образом обеспечивается уникальностью строки `ShowtimeSeat`, а её "занятость" можно отслеживать явным статусным полем или наличием связанного `ReservationSeat`.

*Плюсы:* явное моделирование доступности, проще constraint без partial index, удобнее для будущего расширения (например, блокировка мест на обслуживание).
*Минусы:* дополнительная таблица, требуется генерировать/поддерживать ShowtimeSeat при создании showtime, больше сложности в MVP.

**Рекомендация:** решение между Option A и Option B должно быть принято совместно Islam и Khadizha на отдельной архитектурной сессии (Phase 2/7), с фиксацией итогового решения и обоснования в ADR (Architecture Decision Record) или в README проекта. Данное ТЗ намеренно не предопределяет выбор.

---

## 15. API Requirements

Предполагаемый список REST endpoints (без деталей реализации).

### /auth
| Method | Endpoint | Auth | Назначение |
|---|---|---|---|
| POST | /auth/register | Public | Регистрация нового пользователя |
| POST | /auth/login | Public | Вход, выдача JWT пары |
| POST | /auth/logout | User | Инвалидация refresh-токена |
| POST | /auth/token/refresh | Public (valid refresh) | Обновление access-токена |
| GET | /auth/me | User | Данные текущего пользователя |

### /movies
| Method | Endpoint | Auth | Назначение |
|---|---|---|---|
| GET | /movies | Public/User | Список фильмов (поиск, фильтр, пагинация) |
| GET | /movies/{id} | Public/User | Детали фильма |
| POST | /movies | Admin | Создание фильма |
| PUT/PATCH | /movies/{id} | Admin | Обновление фильма |
| DELETE | /movies/{id} | Admin | Удаление фильма |

### /genres
| Method | Endpoint | Auth | Назначение |
|---|---|---|---|
| GET | /genres | Public/User | Список жанров |
| POST | /genres | Admin | Создание жанра |
| PUT/PATCH | /genres/{id} | Admin | Обновление жанра |
| DELETE | /genres/{id} | Admin | Удаление жанра |

### /cinemas
| Method | Endpoint | Auth | Назначение |
|---|---|---|---|
| GET | /cinemas | Public/User | Список кинотеатров |
| GET | /cinemas/{id} | Public/User | Детали кинотеатра |
| POST | /cinemas | Admin | Создание кинотеатра |
| PUT/PATCH | /cinemas/{id} | Admin | Обновление кинотеатра |
| DELETE | /cinemas/{id} | Admin | Удаление кинотеатра |

### /halls
| Method | Endpoint | Auth | Назначение |
|---|---|---|---|
| GET | /halls | Public/User | Список залов (возможна фильтрация по cinema) |
| GET | /halls/{id} | Public/User | Детали зала |
| POST | /halls | Admin | Создание зала |
| PUT/PATCH | /halls/{id} | Admin | Обновление зала |
| DELETE | /halls/{id} | Admin | Удаление зала |

### /seats
| Method | Endpoint | Auth | Назначение |
|---|---|---|---|
| GET | /seats | Public/User | Список мест (фильтрация по hall) |
| POST | /seats | Admin | Создание места |
| PUT/PATCH | /seats/{id} | Admin | Обновление места |
| DELETE | /seats/{id} | Admin | Удаление места |

### /showtimes
| Method | Endpoint | Auth | Назначение |
|---|---|---|---|
| GET | /showtimes | Public/User | Список сеансов (фильтр по фильму/дате/кинотеатру) |
| GET | /showtimes/{id} | Public/User | Детали сеанса |
| GET | /showtimes/{id}/seats | Public/User | Карта мест (available/reserved) для сеанса |
| POST | /showtimes | Admin | Создание сеанса |
| PUT/PATCH | /showtimes/{id} | Admin | Обновление сеанса |
| DELETE | /showtimes/{id} | Admin | Удаление сеанса |

### /reservations
| Method | Endpoint | Auth | Назначение |
|---|---|---|---|
| POST | /reservations | User | Создание бронирования (showtime + список seats) |
| GET | /reservations | User/Admin | Список своих бронирований (User) / всех (Admin) |
| GET | /reservations/{id} | Owner/Admin | Детали бронирования |
| POST | /reservations/{id}/cancel | Owner/Admin | Отмена бронирования |

### /reports
| Method | Endpoint | Auth | Назначение |
|---|---|---|---|
| GET | /reports/reservations | Admin | Статистика по бронированиям |
| GET | /reports/revenue | Admin | Отчёт по выручке |
| GET | /reports/occupancy | Admin | Отчёт по заполняемости |
| GET | /reports/top-movies | Admin | Топ фильмов |
| GET | /reports/top-showtimes | Admin | Топ сеансов |

---

## 16. API Business Rules & Errors

Система должна корректно обрабатывать следующие ошибки и edge cases (используя стандартные HTTP-коды; конкретные форматы тела ответа — на этапе реализации):

| Ситуация | Ожидаемый статус |
|---|---|
| Invalid seat (несуществующий seat_id) | 400/404 |
| Seat принадлежит другому hall, не связанному с showtime | 400 |
| Seat уже зарезервирован на этот showtime | 409 |
| Showtime уже завершён | 400 |
| Showtime в прошлом (при создании showtime) | 400 |
| Invalid price (≤ 0) | 400 |
| Overlapping showtime в одном hall | 400/409 |
| Unauthorized access (нет токена / истёк токен) | 401 |
| Forbidden access (нет прав на действие) | 403 |
| Отмена уже отменённого/начатого/завершённого reservation | 400 |
| Duplicate reservation (повторная попытка забронировать то же место в рамках одного запроса) | 400 |
| Ресурс не найден | 404 |

---

## 17. Authorization Matrix

| Resource | USER | ADMIN |
|---|---|---|
| Movies | Read | CRUD |
| Genres | Read | CRUD |
| Cinemas | Read | CRUD |
| Halls | Read | CRUD |
| Seats | Read | CRUD |
| Showtimes | Read | CRUD |
| Own Reservations | CRUD according to rules (create/read/cancel своих) | Read all, может отменять любые (по решению команды) |
| Reports | No | Read |

Предложение к обсуждению: явно зафиксировать, что ADMIN не может создавать reservation "от имени" другого пользователя в MVP (может быть добавлено позже как расширение). Финальное решение — на усмотрение команды.

---

## 18. Technology Stack

| Технология | Назначение |
|---|---|
| Python 3.12+ | Основной язык разработки backend |
| Django 5+ | Web-framework, ORM, admin-панель |
| Django REST Framework | Построение REST API (сериализация, views, permissions) |
| PostgreSQL | Основная реляционная база данных, поддержка транзакций и constraints |
| JWT (например, djangorestframework-simplejwt) | Аутентификация без сессий на сервере, access/refresh токены |
| Redis | Кэш и брокер сообщений для Celery |
| Celery | Асинхронные/фоновые задачи (например, будущие уведомления, отложенная обработка отчётов) |
| Docker | Контейнеризация приложения и зависимостей |
| Docker Compose | Локальный оркестр сервисов (web, db, redis, worker) |
| Pytest / pytest-django | Тестирование (unit, integration, API) |
| drf-spectacular | Автогенерация OpenAPI/Swagger-документации |
| GitHub Actions | CI/CD: автотесты, линтинг при каждом PR |

Технологии подобраны по принципу необходимой достаточности — без добавления инструментов "для количества".

---

## 19. Architecture

Предлагается **modular monolith** — единое Django-приложение, разделённое на слабо связанные модули (apps) по доменным границам:

```text
apps/
├── users/         — модели пользователя, аутентификация, роли
├── movies/        — Movie, Genre
├── cinemas/       — Cinema, Hall, Seat
├── showtimes/     — Showtime и связанная бизнес-логика (пересечения, валидация времени)
├── reservations/  — Reservation, ReservationSeat, anti-overbooking логика
└── reports/       — агрегационные запросы и отчётность для ADMIN
```

Каждое приложение отвечает только за свою предметную область (модели, сериализаторы, views, тесты). Межмодульные зависимости (например, showtimes → movies/cinemas, reservations → showtimes) допустимы, но должны быть однонаправленными там, где это возможно, чтобы избежать циклических зависимостей.

Микросервисная архитектура намеренно не используется — она не оправдана масштабом проекта и добавила бы overengineering.

---

## 20. Testing Requirements

**Unit tests** — тестирование отдельных функций/методов бизнес-логики (например, расчёт total_price, валидация пересечения showtime).

**Integration tests** — тестирование взаимодействия между компонентами (например, создание showtime и последующее появление мест в seat map).

**API tests** — тестирование HTTP-эндпоинтов через DRF test client (статус-коды, форматы ответов, edge cases из раздела 16).

**Permission tests** — проверка, что USER не может выполнять ADMIN-действия и наоборот, что чужие reservation недоступны другому пользователю.

**Database tests** — проверка constraints, каскадного поведения, уникальности.

**Concurrency tests** — отдельно и подробно фиксируется как обязательное требование:

> **Concurrent reservation test:** два пользователя одновременно пытаются забронировать один и тот же Seat на один и тот же Showtime. Тест должен подтверждать, что только один запрос завершается успешно (SUCCESS), а второй получает конфликт (CONFLICT/409), независимо от порядка выполнения потоков.

---

## 21. DevOps Requirements

- **Docker** — Dockerfile для backend-приложения (multi-stage build рекомендуется, но не обязателен для MVP).
- **Docker Compose** — оркестрация сервисов: web (Django), db (PostgreSQL), redis, celery worker.
- **PostgreSQL container** — с volume для персистентности данных.
- **Redis container** — для Celery и (опционально) кэширования.
- **Celery worker** — отдельный сервис в docker-compose.
- **Environment variables** — все конфигурационные и секретные значения (DB credentials, SECRET_KEY, JWT settings) выносятся в переменные окружения.
- **.env.example** — файл-шаблон с перечнем необходимых переменных без реальных значений, коммитится в репозиторий.
- **GitHub Actions** — pipeline минимум с шагами: install dependencies → lint → run tests, запускается на каждый PR в develop/main.
- **Migrations** — все изменения схемы БД проходят через Django migrations, применяются автоматически или через отдельный шаг при деплое.
- **Production deployment** — целевая платформа (например, Railway, Render, VPS) определяется на этапе Phase 11, конкретика не фиксируется в данном ТЗ.

---

## 22. Documentation Requirements

### README должен включать:
- description (описание проекта);
- features (ключевые возможности);
- architecture (краткое описание модульной структуры);
- stack (используемые технологии);
- installation (шаги установки);
- environment variables (список и назначение);
- Docker (инструкции по запуску через docker-compose);
- migrations (как применять);
- tests (как запускать);
- API documentation (ссылка на Swagger UI);
- deployment (краткое описание процесса деплоя).

### API documentation
Автогенерируемая через drf-spectacular, доступная как Swagger UI / ReDoc, отражающая все endpoints, схемы запросов/ответов, коды ошибок.

---

## 23. Git & GitHub Workflow

**Структура веток:**

```text
main       — стабильная, production-ready версия
develop    — интеграционная ветка для текущей разработки
feature/*  — новая функциональность (например, feature/reservations-api)
bugfix/*   — исправление багов (например, bugfix/seat-uniqueness)
```

**Правила:**
- Прямые пуши в `main` запрещены.
- Все изменения проходят через Pull Request в `develop` (а `develop` → `main` — при релизе/финализации этапа).
- Каждый PR требует минимум одного code review от второго участника команды.
- Коммиты должны быть осмысленными, отражающими суть изменения.
- Именование веток: `feature/<короткое-описание>`, `bugfix/<короткое-описание>`.

**Примеры commit messages:**
```text
feat(reservations): add seat availability endpoint
fix(showtimes): prevent overlapping showtime creation
test(reservations): add concurrent booking test
docs(readme): add docker setup instructions
refactor(movies): extract genre filtering into queryset method
```

**PR requirements:** описание изменений, ссылка на связанную Trello-карточку/задачу, чеклист (тесты добавлены/пройдены, документация обновлена при необходимости).

---

## 24. Trello Workstreams

ТЗ структурировано так, чтобы после утверждения быть разбитым на:

```text
EPICS → FEATURES → TASKS
```

Основные workstreams (без полной декомпозиции — сама декомпозиция на Features/Tasks выполняется отдельно после утверждения ТЗ):

```text
Project Setup
Authentication
Movies
Cinema
Showtimes
Reservations
Reports
Testing
DevOps
Documentation
Deployment
```

Каждый workstream в дальнейшем становится Epic-карточкой (или списком) в Trello, из которой создаются Feature- и Task-карточки на этапе спринт-планирования.

---

## 25. Project Phases

```text
Phase 1  — Planning (утверждение ТЗ, архитектурные решения, Trello board)
Phase 2  — Foundation (репозиторий, Docker, базовая структура apps, CI skeleton)
Phase 3  — Authentication (регистрация, login, JWT, роли)
Phase 4  — Movie & Cinema Management (Movie, Genre, Cinema, Hall, Seat CRUD)
Phase 5  — Showtime Management (Showtime CRUD + бизнес-правила пересечений)
Phase 6  — Reservation System (создание/просмотр/отмена бронирований)
Phase 7  — Concurrency & Data Integrity (anti-overbooking, транзакции, concurrency-тесты)
Phase 8  — Reporting (аналитические endpoints для ADMIN)
Phase 9  — Testing (полное покрытие: unit/integration/API/permission/concurrency)
Phase 10 — DevOps (финализация Docker Compose, GitHub Actions)
Phase 11 — Deployment (production deployment)
Phase 12 — Final Review (документация, финальная презентация, оценка по критериям раздела 29)
```

---

## 26. Definition of Done

Задача (task) считается Done только если выполнены **все** пункты:

- [ ] Implementation complete — функциональность реализована согласно ТЗ.
- [ ] Tests written — написаны соответствующие тесты (unit/integration/API, где применимо).
- [ ] Edge cases considered — обработаны граничные случаи и ошибки (раздел 16).
- [ ] Documentation updated — обновлены README/docstrings/Swagger-описания при необходимости.
- [ ] Code reviewed — проведено код-ревью вторым участником команды.
- [ ] PR approved — Pull Request одобрен ревьюером.
- [ ] PR merged — изменения влиты в целевую ветку.
- [ ] No critical bugs — отсутствуют известные критические дефекты.

---

## 27. Acceptance Criteria

Итоговый чек-лист для проверки готовности проекта:

- [ ] Пользователь может зарегистрироваться, войти, получить и обновить JWT-токены.
- [ ] ADMIN может выполнять полный CRUD над Movie, Genre, Cinema, Hall, Seat, Showtime.
- [ ] USER может просматривать фильмы, кинотеатры, залы, сеансы с поиском/фильтрацией/пагинацией.
- [ ] Showtime не может быть создан в прошлом, с некорректным временем/ценой или с пересечением по залу.
- [ ] USER может создать бронирование с одним или несколькими местами и увидеть итоговую стоимость.
- [ ] Карта мест для showtime корректно отражает AVAILABLE/RESERVED.
- [ ] Конкурентная попытка забронировать одно место двумя пользователями гарантированно даёт один SUCCESS и один CONFLICT (подтверждено тестом).
- [ ] Отмена бронирования работает согласно правилам раздела 11 и запрещена в недопустимых состояниях.
- [ ] ADMIN может получить все указанные в разделе 12 отчёты.
- [ ] Все API-эндпоинты защищены корректными permission-проверками (согласно разделу 17).
- [ ] Проект запускается локально через `docker-compose up` без ручных дополнительных шагов (кроме `.env`).
- [ ] CI (GitHub Actions) успешно проходит на каждом PR (тесты + линт).
- [ ] Тестовое покрытие включает unit, integration, API, permission и concurrency тесты.
- [ ] README и Swagger-документация полны и актуальны.
- [ ] История коммитов и PR отражает реальную командную работу (осмысленные сообщения, ревью).

---

## 28. Final Project Quality

Проект должен выглядеть как реальный backend portfolio project.

**Не стремимся к:**
- excessive abstraction (излишняя абстракция ради абстракции);
- unnecessary microservices;
- Kubernetes без явной необходимости;
- excessive design patterns (паттерны ради паттернов).

**Стремимся к:**
- clean architecture (понятное разделение ответственности);
- readable code (читаемый, поддерживаемый код);
- correct database design (нормализация, корректные constraints);
- strong business logic (надёжная реализация бронирования и anti-overbooking);
- reliable reservation system;
- good tests (осмысленное, а не формальное покрытие);
- documentation (README и API-документация, которые реально помогают);
- production-oriented practices (переменные окружения, CI, Docker).

---

## 29. Final Challenge / Reward Agreement

В проекте участвуют два человека: Islam и Khadizha. По завершении проекта проводится совместная итоговая оценка по следующим критериям:

| Критерий | Вес |
|---|---|
| Code Quality | 20% |
| Architecture | 20% |
| Task Completion | 20% |
| Testing | 15% |
| Git/GitHub | 10% |
| Documentation | 5% |
| Teamwork | 10% |
| **TOTAL** | **100%** |

**Reward:** 5 000 KGS или Tour (по договорённости команды при достижении удовлетворительного результата по итоговой оценке).

Это неформальное командное соглашение (challenge agreement) о взаимной мотивации, а не юридический договор.

---

## 30. Final Checklist (Team Sign-off)

Перед стартом Phase 2 (Foundation) команда должна:

- [ ] Совместно прочитать и обсудить данное ТЗ.
- [ ] Принять решение по Option A / Option B (раздел 14).
- [ ] Распределить модули/ответственность более детально в рамках workstreams (раздел 24).
- [ ] Создать Trello board с Epics согласно workstreams.
- [ ] Создать GitHub-репозиторий, настроить ветки `main`/`develop`, добавить обоих участников.
- [ ] Согласовать окончательный набор reservation-статусов и конкретный механизм anti-overbooking (раздел 10) перед началом Phase 6/7.
- [ ] Подтвердить утверждение ТЗ подписью/сообщением обоих участников (Islam, Khadizha) перед стартом разработки.

---

*Конец документа. Данное ТЗ является черновиком (Draft) и вступает в силу только после явного утверждения обеими сторонами.*
