# ISLAM_TASKS.md
## Movie Reservation System — Задачи разработки
### Ответственный: Islam (Project Manager / Tech Lead / Backend Developer)

Источник истины: утверждённое Техническое задание (Movie Reservation System TS, v1.0).
Файл структурирован по схеме `EPIC → FEATURE → TASK` для прямого переноса в Trello.

---

## EPIC: Основа проекта и архитектура

### Feature: Структура репозитория и проекта

#### ISL-SETUP-001 — Инициализация репозитория и стратегии веток

**Description:** Создать GitHub-репозиторий, настроить ветки `main`/`develop`, правила защиты веток (запрет прямых пушей в `main`/`develop`), пригласить Хадижу как коллаборатора.

**Why:** ТЗ §23 (Git & GitHub Workflow) требует защищённую модель ветвления до начала любой разработки.

**Dependencies:** Нет (первая задача проекта).

**Priority:** P0

**Complexity:** S

**Acceptance Criteria:**
- Репозиторий создан с ветками `main` и `develop`.
- Включена защита веток `main` и `develop` (требуется PR + ревью).
- Настроен `.gitignore` (шаблон Python/Django, исключает `.env`).
- Хадижа добавлена с правами записи.
- Закоммичен скелет `README.md` (разделы пустые, будут заполнены согласно ТЗ §22).

**Technical Notes:** Использовать стандартный `.gitignore` для Python/Django. Не коммитить файл `.env`, только `.env.example` (создаётся в ISL-DEVOPS-002).

**Testing Requirements:** N/A (инфраструктурная задача).

**Git Branch:** `feature/repo-setup`

**Suggested Commit:** `chore(repo): initialize repository structure and branch protection`

**Learning:** Стратегии ветвления в Git, администрирование репозитория GitHub.

---

#### ISL-SETUP-002 — Создание модульной структуры Django-проекта

**Description:** Инициализировать Django-проект и создать пакет `apps/` с пустыми скелетами приложений: `users`, `movies`, `cinemas`, `showtimes`, `reservations`, `reports`, согласно ТЗ §19 (Architecture).

**Why:** ТЗ §19 требует модульный монолит с разделением по доменам до начала работы над функциональностью.

**Dependencies:** ISL-SETUP-001

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Django-проект создан с разделением настроек (base/local/production) или эквивалентной конфигурацией через переменные окружения.
- Все шесть приложений зарегистрированы в `INSTALLED_APPS` под `apps.<name>`.
- Проект запускается локально (`runserver`) без ошибок на пустой БД.
- Скелет базовой маршрутизации (`/api/v1/...`) на месте, по одному маршруту на приложение (заглушки допустимы).

**Technical Notes:** С самого начала держать настройки управляемыми через переменные окружения (см. ISL-DEVOPS-002), чтобы избежать переделок позже.

**Testing Requirements:** Smoke-тест — проект запускается, корневой API-маршрут возвращает ожидаемый статус (200/404), без 500-х ошибок.

**Git Branch:** `feature/project-structure`

**Suggested Commit:** `chore(core): bootstrap modular Django project structure`

**Learning:** Дизайн модульного монолита, разделение проекта/приложений в Django.

---

### Feature: Архитектура данных

#### ISL-ARCH-001 — Черновик первичной ERD (до реализации)

**Description:** Подготовить диаграмму связей сущностей (ERD) для всех сущностей из ТЗ §14 (`User`, `Movie`, `Genre`, `Cinema`, `Hall`, `Seat`, `Showtime`, `Reservation`, `ReservationSeat`), включая оба подхода к моделированию доступности мест — Option A и Option B.

**Why:** ТЗ §14 требует задокументированную ERD и явное сравнение Option A/B до начала реализации.

**Dependencies:** ISL-SETUP-002

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Диаграмма ERD (в любом инструменте — изображение или текстовый формат, например Mermaid/dbdiagram), охватывающая все сущности и связи.
- Задокументированы оба варианта — Option A и Option B — с плюсами/минусами, согласно ТЗ §14.
- Диаграмма передана Хадиже на ревью (см. ISL-ARCH-002 / SHARED).

**Technical Notes:** Пока не писать миграции — это проектный артефакт (в стиле ADR), коммитится в `/docs/architecture/erd.md`.

**Testing Requirements:** N/A (проектный артефакт).

**Git Branch:** `feature/erd-draft`

**Suggested Commit:** `docs(architecture): add initial ERD draft with Option A/B comparison`

**Learning:** Реляционное моделирование БД, компромиссы между many-to-many и промежуточной сущностью.

---

#### ISL-ARCH-002 — Проведение архитектурной сессии решения (Option A vs Option B)

**Description:** Провести совместную с Хадижой сессию для выбора между Option A и Option B моделирования доступности мест/сеансов (ТЗ §14) и задокументировать итоговое решение в виде ADR.

**Why:** ТЗ §14 прямо требует, чтобы это решение принималось совместно, а не единолично Исламом.

**Dependencies:** ISL-ARCH-001

**Priority:** P0

**Complexity:** S

**Owner:** BOTH (ведёт Islam)

**Acceptance Criteria:**
- Закоммичен документ ADR (`/docs/architecture/adr-001-seat-availability-model.md`) с решением, обоснованием и отметкой согласия обоих участников.
- Решение отражено в документе ERD из ISL-ARCH-001.

**Technical Notes:** Эта задача блокирует все задачи по написанию моделей для приложений `showtimes` и `reservations`.

**Testing Requirements:** N/A.

**Git Branch:** `feature/adr-seat-model`

**Suggested Commit:** `docs(architecture): record ADR-001 seat availability model decision`

**Learning:** Практика Architecture Decision Records (ADR), совместное принятие технических решений.

---

## EPIC: Аутентификация и авторизация

### Feature: Управление пользователями

#### ISL-AUTH-001 — Создание кастомной модели User

**Description:** Реализовать кастомную модель `User` согласно ТЗ §4.1/§4.2: уникальный email, обязательные поля, поле `role` с поддержкой `USER`/`ADMIN`, безопасное хэширование пароля.

**Why:** ТЗ §4.1–4.2 требуют модель пользователя с ролями как основу для всей работы по аутентификации/авторизации.

**Dependencies:** ISL-SETUP-002

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Email уникален и используется как основной идентификатор для входа.
- Поле `role` с вариантами `USER`/`ADMIN`, по умолчанию `USER`.
- Пароль хранится только через хэшер Django.
- Реализован кастомный менеджер пользователя (`create_user`, `create_superuser`).
- Миграция создана и чисто применяется на пустой БД.
- Зарегистрировано в Django admin.
- Написаны unit-тесты для поведения модели/менеджера.

**Technical Notes:** Установить `AUTH_USER_MODEL` до первой миграции — это должна быть одна из самых первых задач по моделям в проекте.

**Testing Requirements:** Unit-тесты: уникальность email, роль по умолчанию, хэширование пароля, создание суперпользователя.

**Git Branch:** `feature/custom-user`

**Suggested Commit:** `feat(users): add custom user model with role field`

**Learning:** Кастомная модель пользователя в Django, кастомные менеджеры пользователей, внутреннее устройство хэширования паролей.

---

#### ISL-AUTH-002 — Реализация эндпоинта регистрации

**Description:** Реализовать `POST /auth/register` согласно ТЗ §15, с валидацией входных данных и созданием нового аккаунта с ролью `USER`.

**Why:** ТЗ §4.1 (Registration) и ТЗ §15 (таблица эндпоинтов `/auth`).

**Dependencies:** ISL-AUTH-001

**Priority:** P0

**Complexity:** S

**Acceptance Criteria:**
- Эндпоинт валидирует формат/уникальность email и сложность пароля.
- Новые пользователи всегда создаются с ролью `USER` (без возможности повышения роли через клиента).
- Успешная регистрация возвращает публичное представление созданного пользователя (без пароля).
- Соответствующие ошибки 400 для невалидных/дублирующихся данных (ТЗ §16).

**Technical Notes:** Отклонять любое поле `role`, присутствующее в теле запроса, а не молча игнорировать его — во избежание edge-кейсов с повышением привилегий.

**Testing Requirements:** API-тесты на успех, дублирующийся email, слабый пароль, отсутствующие поля.

**Git Branch:** `feature/auth-registration`

**Suggested Commit:** `feat(auth): add user registration endpoint`

**Learning:** Сериализаторы/валидаторы DRF, санитизация входных данных против повышения привилегий.

---

#### ISL-AUTH-003 — Реализация Login и выдачи JWT

**Description:** Реализовать `POST /auth/login`, выдающий пару access/refresh JWT-токенов согласно ТЗ §4.1/§15.

**Why:** ТЗ §4.1 (Login) и §18 (JWT как выбранный механизм аутентификации).

**Dependencies:** ISL-AUTH-001

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Корректные учётные данные возвращают access + refresh токены.
- Некорректные учётные данные возвращают 401 без утечки информации о существовании email.
- Access-токен содержит claims, необходимые для определения роли пользователя при проверке прав.
- Время жизни токенов настраивается через переменные окружения/настройки.

**Technical Notes:** Использовать `djangorestframework-simplejwt` или аналог; конфигурацию держать централизованно в настройках, а не хардкодить.

**Testing Requirements:** API-тесты: успешный вход, неверный пароль, несуществующий пользователь, неактивный пользователь (если применимо).

**Git Branch:** `feature/auth-login`

**Suggested Commit:** `feat(auth): add login endpoint with JWT issuance`

**Learning:** Поток JWT-аутентификации, дизайн access/refresh токенов.

---

#### ISL-AUTH-004 — Реализация Token Refresh и Logout

**Description:** Реализовать `POST /auth/token/refresh` и `POST /auth/logout` (blacklisting refresh-токена) согласно ТЗ §15.

**Why:** ТЗ §4.1 (Token refresh, Logout).

**Dependencies:** ISL-AUTH-003

**Priority:** P1

**Complexity:** S

**Acceptance Criteria:**
- Валидный refresh-токен возвращает новый access-токен.
- Истёкший/невалидный refresh-токен возвращает 401.
- Logout добавляет переданный refresh-токен в blacklist, после чего он не может быть использован снова.
- Повторное использование blacklisted-токена возвращает 401.

**Technical Notes:** Требует приложение/таблицу blacklist токенов, если используется blacklist-приложение simplejwt.

**Testing Requirements:** API-тесты: успешный refresh, refresh с истёкшим токеном, logout и повторное использование токена.

**Git Branch:** `feature/auth-refresh-logout`

**Suggested Commit:** `feat(auth): add token refresh and logout with blacklisting`

**Learning:** Управление жизненным циклом токенов, отзыв на основе blacklist.

---

#### ISL-AUTH-005 — Реализация эндпоинта текущего пользователя

**Description:** Реализовать `GET /auth/me`, возвращающий данные профиля аутентифицированного пользователя согласно ТЗ §15.

**Why:** ТЗ §4.1 (Current user).

**Dependencies:** ISL-AUTH-003

**Priority:** P2

**Complexity:** XS

**Acceptance Criteria:**
- Аутентифицированный запрос возвращает id, email, роль и другие публичные поля текущего пользователя.
- Неаутентифицированный запрос возвращает 401.

**Testing Requirements:** API-тесты: успех при аутентификации, 401 без аутентификации.

**Git Branch:** `feature/auth-current-user`

**Suggested Commit:** `feat(auth): add current user endpoint`

**Learning:** Классы разрешений DRF, использование request.user.

---

#### ISL-AUTH-006 — Реализация фреймворка RBAC-разрешений

**Description:** Построить переиспользуемые классы разрешений (`IsAdmin`, `IsOwnerOrAdmin` и т.д.), которые будут использовать все остальные приложения для реализации ТЗ §17 (Authorization Matrix).

**Why:** ТЗ §4.2 и §17 требуют согласованный контроль доступа на основе ролей по всем ресурсам.

**Dependencies:** ISL-AUTH-001

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Класс разрешений `IsAdmin` ограничивает доступ на запись только ролью `ADMIN`.
- Класс разрешений `IsOwnerOrAdmin` разрешает действия владельцу ресурса или `ADMIN`.
- Классы разрешений задокументированы (docstring) с примерами использования для другого разработчика.
- Применены как схема разрешений по умолчанию для admin-only viewset'ов (movies/genres/cinemas и т.д. — согласовано с Хадижой).

**Technical Notes:** Централизовать это в общем `apps/users/permissions.py` (или `apps/core/`), чтобы оба разработчика импортировали из одного места, а не дублировали логику.

**Testing Requirements:** Unit-тесты для каждого класса разрешений против запросов USER/ADMIN/анонимных.

**Git Branch:** `feature/rbac-permissions`

**Suggested Commit:** `feat(users): add reusable RBAC permission classes`

**Learning:** Архитектура разрешений DRF, паттерны проектирования RBAC.

---

#### ISL-AUTH-007 — Механизм повышения до ADMIN

**Description:** Реализовать management-команду (или задокументированную процедуру через Django admin) для повышения `USER` до `ADMIN`, согласно ТЗ §4.2 (роль ADMIN недоступна через self-service).

**Why:** ТЗ §4.2 явно указывает, что роль ADMIN не получается через самостоятельную регистрацию.

**Dependencies:** ISL-AUTH-001

**Priority:** P2

**Complexity:** XS

**Acceptance Criteria:**
- `python manage.py promote_admin <email>` (или аналог) устанавливает роль в `ADMIN`.
- Задокументировано в README в разделе установки/настройки администратора.
- Команда корректно обрабатывает случай отсутствия email.

**Testing Requirements:** Unit-тест для management-команды (успех + пользователь не найден).

**Git Branch:** `feature/admin-promotion`

**Suggested Commit:** `feat(users): add admin promotion management command`

**Learning:** Management-команды Django.

---

## EPIC: Ядро системы бронирования

### Feature: Модель данных бронирования

#### ISL-RES-001 — Реализация моделей Reservation и ReservationSeat

**Description:** Реализовать модели `Reservation` и `ReservationSeat` согласно решению ADR из ISL-ARCH-002 (Option A или B), включая поле статуса и поля расчёта цены.

**Why:** ТЗ §8 (Reservation System) и §14 (Data Model).

**Dependencies:** ISL-ARCH-002, KHD-SHOW-001 (модель Showtime должна существовать)

**Priority:** P0

**Complexity:** L

**Acceptance Criteria:**
- Модель `Reservation`: user (FK), showtime (FK), status, total_price, created_at.
- Модель `ReservationSeat` связывает бронирование с конкретным(и) местом(-ами) согласно выбранному варианту ADR.
- Поле статуса ограничено определённым набором значений (например, `PENDING`/`CONFIRMED`, `CANCELLED`).
- Миграция создана и чисто применена.
- Реализованы `__str__` модели и регистрация в admin для отладки.

**Technical Notes:** Если выбран Option B — задача также включает модель `ShowtimeSeat` и логику её генерации при создании `Showtime` (согласовать с приложением `showtimes` Хадижи).

**Testing Requirements:** Unit-тесты на создание модели, переходы статусов на уровне модели (пока без API).

**Git Branch:** `feature/reservation-models`

**Suggested Commit:** `feat(reservations): add Reservation and ReservationSeat models`

**Learning:** Сложное реляционное моделирование, моделирование временной доступности.

---

#### ISL-RES-002 — Реализация ограничений БД против overbooking

**Description:** Добавить уникальные ограничения на уровне БД (согласно ТЗ §10), гарантирующие, что одно место не может быть дважды забронировано на один и тот же сеанс среди активных бронирований.

**Why:** ТЗ §10 (Anti-Overbooking) явно требует ограничений на уровне БД как дополнительного рубежа защиты, а не только логики приложения.

**Dependencies:** ISL-RES-001

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Уникальное ограничение (partial/conditional индекс при Option A, либо естественная уникальность при Option B) предотвращает две активные строки `ReservationSeat` для одной пары (showtime, seat).
- Попытка обойти уровень приложения и вставить дубликат напрямую на уровне БД завершается ошибкой (проверено тестом).
- Ограничение не блокирует повторное бронирование места, чьё предыдущее бронирование было отменено.

**Technical Notes:** Это "последний рубеж защиты", описанный в ТЗ §10, дополняющий (а не заменяющий) логику транзакций/блокировок из ISL-RES-004.

**Testing Requirements:** Тест на уровне БД, вставляющий конфликтующие строки напрямую, для подтверждения срабатывания ограничения.

**Git Branch:** `feature/reservation-constraints`

**Suggested Commit:** `feat(reservations): add unique constraint preventing double booking`

**Learning:** Partial/conditional уникальные индексы PostgreSQL, глубокоэшелонированная защита в дизайне БД.

---

### Feature: Бизнес-логика бронирования

#### ISL-RES-003 — Реализация эндпоинта доступности мест

**Description:** Реализовать `GET /showtimes/{id}/seats`, возвращающий статусы `AVAILABLE`/`RESERVED` для каждого места зала в контексте конкретного сеанса (ТЗ §9).

**Why:** ТЗ §9 (Seat Availability) — ключевая зависимость пары `Showtime + Seat`.

**Dependencies:** ISL-RES-001, KHD-CIN-003 (модель Seat), KHD-SHOW-001 (модель Showtime)

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Эндпоинт возвращает все места, принадлежащие залу сеанса, со статусом `AVAILABLE` или `RESERVED`.
- Только места с активными (неотменёнными) бронированиями показывают статус `RESERVED`.
- Ответ формируется эффективно (без N+1 запросов — проверено тестом с подсчётом запросов).

**Technical Notes:** Использовать `select_related`/`prefetch_related` для избежания N+1 (ТЗ §13 Performance).

**Testing Requirements:** API-тест, проверяющий корректное разделение AVAILABLE/RESERVED; тест на количество запросов.

**Git Branch:** `feature/seat-availability`

**Suggested Commit:** `feat(reservations): add seat availability endpoint for showtime`

**Learning:** Оптимизация запросов, select_related/prefetch_related, предотвращение N+1.

---

#### ISL-RES-004 — Реализация сервиса создания бронирования (транзакционного)

**Description:** Реализовать основной поток создания бронирования: `POST /reservations`, принимающий `showtime_id` + список `seat_id`, с валидацией и атомарным созданием бронирования согласно ТЗ §8/§10.

**Why:** Это центральное бизнес-требование всего проекта (ТЗ §8, §10).

**Dependencies:** ISL-RES-002, ISL-RES-003

**Priority:** P0

**Complexity:** XL

**Acceptance Criteria:**
- Запрос принимает `showtime_id` и непустой список `seat_id`.
- Все запрошенные места принадлежат залу данного сеанса (ТЗ §16 — invalid seat / место из другого зала).
- Операция обёрнута в единую транзакцию БД — бронируются все места или ни одно (атомарность, ТЗ §10).
- Используется блокировка на уровне строк (например, `select_for_update`) или эквивалентный подход, устраняющий окно race condition (ТЗ §10).
- Итоговая цена рассчитывается корректно (цена сеанса × количество мест).
- Возвращает 409 при конфликте, если любое из мест уже забронировано на этот сеанс.
- Возвращает 400 для сеанса в прошлом/завершённого сеанса (ТЗ §16).

**Technical Notes:** Эта задача напрямую реализует механизм, концептуально выбранный в ISL-ARCH-002/ТЗ §10. Задокументировать выбранную стратегию блокировки в кратком комментарии кода/дополнении к ADR.

**Testing Requirements:** API-тесты на: успешный сценарий (одно место), успешный сценарий (несколько мест), одно место уже занято (весь запрос отклоняется), сеанс в прошлом, завершённый сеанс, невалидное место, место из чужого зала.

**Git Branch:** `feature/reservation-creation`

**Suggested Commit:** `feat(reservations): implement transactional reservation creation with anti-overbooking`

**Learning:** Транзакции БД, блокировка на уровне строк (`select_for_update`), гарантии атомарности, предотвращение race condition.

---

#### ISL-RES-005 — Реализация эндпоинтов получения бронирований

**Description:** Реализовать `GET /reservations` (список: свои для USER / все для ADMIN) и `GET /reservations/{id}` (только владелец или ADMIN) согласно ТЗ §8/§15/§17.

**Why:** ТЗ §8 (просмотр своих бронирований / просмотр бронирования).

**Dependencies:** ISL-RES-004

**Priority:** P1

**Complexity:** M

**Acceptance Criteria:**
- USER видит в списке только свои бронирования.
- ADMIN видит все бронирования, с возможностью фильтрации по пользователю/статусу (базовая фильтрация).
- Детальный эндпоинт возвращает 403 для USER, не являющегося владельцем, 200 — для владельца или ADMIN.
- Список пагинирован (ТЗ §13 Performance).

**Testing Requirements:** API-тесты: USER видит только свой список, ADMIN видит все, доступ к деталям чужого бронирования запрещён, наличие пагинации.

**Git Branch:** `feature/reservation-list-detail`

**Suggested Commit:** `feat(reservations): add reservation list and detail endpoints`

**Learning:** Ограничение queryset по роли, пагинация DRF.

---

#### ISL-RES-006 — Реализация отмены бронирования

**Description:** Реализовать `POST /reservations/{id}/cancel` с соблюдением правил отмены из ТЗ §11 (Upcoming → можно отменить; Started/Finished/Cancelled → нельзя отменить).

**Why:** ТЗ §11 (Reservation Cancellation).

**Dependencies:** ISL-RES-004

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Бронирование на сеанс, который ещё не начался, может быть отменено владельцем (или ADMIN).
- Бронирование на начавшийся/завершённый сеанс при попытке отмены возвращает 400.
- Уже отменённое бронирование при повторной попытке отмены возвращает 400.
- Отмена бронирования освобождает связанное(ые) место(а) для этого сеанса (отражается в эндпоинте доступности мест).

**Testing Requirements:** API-тесты, покрывающие все четыре состояния из ТЗ §11, плюс последующая проверка доступности, подтверждающая освобождение мест.

**Git Branch:** `feature/reservation-cancellation`

**Suggested Commit:** `feat(reservations): implement reservation cancellation with state rules`

**Learning:** Валидация в стиле конечного автомата, реализация бизнес-правил в сервисном слое.

---

#### ISL-RES-007 — Ревью и укрепление отказоустойчивости бронирования при конкурентности

**Description:** После того как Хадижа напишет набор тестов на конкурентность (KHD-TEST-003), укрепить реализацию ISL-RES-004 до тех пор, пока тест на конкурентное бронирование не будет проходить стабильно.

**Why:** ТЗ §10 и §20 требуют доказанной, проверенной тестами гарантии, а не только теоретического дизайна.

**Dependencies:** ISL-RES-004, KHD-TEST-003

**Priority:** P0

**Complexity:** M

**Owner:** BOTH (Islam реализует/исправляет, Хадижа пишет/валидирует тест)

**Acceptance Criteria:**
- Тест на конкурентное бронирование (два одновременных запроса на одно место/сеанс) проходит стабильно при многократных запусках.
- Не наблюдается deadlock'ов при паттерне нагрузки теста.
- Исправление/корректировка задокументирована в дополнении к ADR, если исходный подход потребовал изменений.

**Testing Requirements:** Повторно запустить KHD-TEST-003 минимум 10 раз в CI, чтобы подтвердить стабильность (отсутствие "мигающих" тестов).

**Git Branch:** `feature/reservation-concurrency-hardening`

**Suggested Commit:** `fix(reservations): harden anti-overbooking under concurrent load`

**Learning:** Отладка race condition, анализ deadlock'ов, методология тестирования конкурентности.

---

## EPIC: Отчётность

### Feature: Аналитика для администратора

#### ISL-REP-001 — Реализация отчётов по бронированиям и финансам

**Description:** Реализовать `GET /reports/reservations` и `GET /reports/revenue` согласно ТЗ §12/§15, охватывающие total/active/cancelled бронирования, общую выручку, выручку по датам, среднюю цену билета.

**Why:** ТЗ §12 (Reporting — категории Reservations Analytics, Financial Analytics).

**Dependencies:** ISL-RES-004, ISL-AUTH-006

**Priority:** P1

**Complexity:** L

**Acceptance Criteria:**
- `/reports/reservations` возвращает счётчики total/active/cancelled, опционально с фильтром по диапазону дат.
- `/reports/revenue` возвращает общую выручку, разбивку выручки по датам и среднюю цену билета.
- Из расчёта выручки исключаются только отменённые бронирования.
- Эндпоинты доступны только `ADMIN` (403 для USER).
- Запросы используют агрегацию на уровне БД (без ручного суммирования в Python по большим queryset'ам).

**Technical Notes:** Использовать агрегацию Django ORM (`Sum`, `Count`, `Avg`, `annotate`) для сохранения производительности согласно ТЗ §13.

**Testing Requirements:** API-тесты, проверяющие рассчитанные показатели против известных фикстур; тест на запрет доступа для USER.

**Git Branch:** `feature/reports-financial`

**Suggested Commit:** `feat(reports): add reservation and revenue analytics endpoints`

**Learning:** Агрегация Django ORM, дизайн ADMIN-only аналитического API.

---

#### ISL-REP-002 — Реализация отчётов по заполняемости и топ-показателям

**Description:** Реализовать `GET /reports/occupancy`, `GET /reports/top-movies`, `GET /reports/top-showtimes` согласно ТЗ §12.

**Why:** ТЗ §12 (категория Occupancy & Performance).

**Dependencies:** ISL-REP-001

**Priority:** P2

**Complexity:** L

**Acceptance Criteria:**
- `/reports/occupancy` возвращает процент заполнения по сеансу/залу (забронированные места ÷ вместимость зала).
- `/reports/top-movies` ранжирует фильмы по числу бронирований или выручке.
- `/reports/top-showtimes` ранжирует сеансы по числу забронированных мест.
- Все три эндпоинта доступны только `ADMIN`.
- Разумная пагинация/лимит на результаты "топ N".

**Testing Requirements:** API-тесты с фикстурами, проверяющие порядок ранжирования и проценты заполняемости.

**Git Branch:** `feature/reports-occupancy`

**Suggested Commit:** `feat(reports): add occupancy and top-performers analytics endpoints`

**Learning:** Сложные аннотации/джойны ORM, запросы ранжирования.

---

## EPIC: DevOps и инфраструктура

### Feature: Контейнеризация

#### ISL-DEVOPS-001 — Докеризация приложения

**Description:** Написать `Dockerfile` для Django-приложения согласно ТЗ §21.

**Why:** ТЗ §21 (DevOps Requirements — Docker).

**Dependencies:** ISL-SETUP-002

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Приложение собирается и запускается внутри контейнера.
- Образ использует зафиксированную базовую версию Python 3.12+.
- Процесс приложения запускается от имени non-root пользователя (базовая гигиена безопасности).

**Testing Requirements:** Ручная проверка — контейнер собирается и обслуживает приложение; smoke-тест эндпоинта отвечает.

**Git Branch:** `feature/dockerfile`

**Suggested Commit:** `chore(docker): add application Dockerfile`

**Learning:** Сборка Docker-образов для приложений Python/Django.

---

#### ISL-DEVOPS-002 — Docker Compose и конфигурация окружения

**Description:** Создать `docker-compose.yml`, оркеструющий сервисы `web`, `db` (PostgreSQL), `redis` и Celery worker, а также `.env.example` согласно ТЗ §21.

**Why:** ТЗ §21 (Docker Compose, контейнеры PostgreSQL/Redis, переменные окружения, `.env.example`).

**Dependencies:** ISL-DEVOPS-001

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- `docker-compose up` успешно запускает web, db, redis и Celery worker без ручных шагов, кроме предоставления `.env`.
- PostgreSQL использует именованный volume для персистентности.
- `.env.example` перечисляет все необходимые переменные с placeholder-значениями, закоммичен в репозиторий.
- Ни один секрет нигде не закоммичен в репозиторий.

**Testing Requirements:** Ручная проверка запуска полного стека; подтверждение подключения приложения к db/redis внутри сети compose.

**Git Branch:** `feature/docker-compose`

**Suggested Commit:** `chore(docker): add docker-compose orchestration and env template`

**Learning:** Оркестрация нескольких сервисов в контейнерах, конфигурация через переменные окружения.

---

#### ISL-DEVOPS-003 — Настройка интеграции Celery и Redis

**Description:** Подключить Celery к использованию Redis в качестве брокера, с базовой health-check/тестовой задачей, согласно ТЗ §18/§21.

**Why:** ТЗ §18 указывает Celery/Redis как обязательные компоненты стека для будущей асинхронной обработки.

**Dependencies:** ISL-DEVOPS-002

**Priority:** P2

**Complexity:** S

**Acceptance Criteria:**
- Приложение Celery настроено и обнаруживается Django.
- Тривиальная тестовая задача успешно выполняется через worker-контейнер.
- Конфигурация управляется через переменные окружения (URL брокера и т.д.).

**Testing Requirements:** Интеграционный тест или ручная проверка, что отправленная задача подхватывается и завершается worker'ом.

**Git Branch:** `feature/celery-setup`

**Suggested Commit:** `chore(celery): configure Celery with Redis broker`

**Learning:** Интеграция Celery/Redis, асинхронные очереди задач.

---

#### ISL-DEVOPS-004 — Настройка CI-пайплайна GitHub Actions

**Description:** Создать workflow GitHub Actions, выполняющий install → lint → tests при каждом PR, согласно ТЗ §21/§23.

**Why:** ТЗ §21 (GitHub Actions) и §23 (требования к PR завязаны на CI).

**Dependencies:** ISL-SETUP-001

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Workflow запускается на PR в `develop` и `main`.
- Шаги: установка зависимостей, lint (например, flake8/ruff), запуск pytest с сервисным контейнером PostgreSQL.
- Провалившиеся тесты/lint блокируют слияние PR (обязательная проверка статуса).
- Статус workflow отображается на каждом PR.

**Testing Requirements:** Проверить, что пайплайн падает при намеренно сломанном тесте/нарушении lint, а затем проходит после исправления.

**Git Branch:** `feature/ci-pipeline`

**Suggested Commit:** `ci: add GitHub Actions pipeline for lint and tests`

**Learning:** Дизайн CI/CD-пайплайна, GitHub Actions, сервисные контейнеры в CI.

---

#### ISL-DEVOPS-005 — Продакшн-деплой

**Description:** Развернуть приложение на выбранной хостинг-платформе (согласно ТЗ §21, платформа определяется на Phase 11), включая миграции и настройку секретов окружения.

**Why:** ТЗ §21 (Production deployment) и ТЗ Phase 11.

**Dependencies:** ISL-DEVOPS-004, все P0-задачи функциональности завершены

**Priority:** P1

**Complexity:** L

**Acceptance Criteria:**
- Приложение доступно по публичному URL.
- Миграции применяются автоматически или через задокументированный шаг деплоя.
- Секреты окружения настроены на хостинг-платформе (не в исходном коде).
- Документация Swagger/OpenAPI доступна в продакшене.

**Testing Requirements:** Smoke-тест критичных эндпоинтов (auth, список фильмов, создание бронирования) на развёрнутом инстансе.

**Git Branch:** `feature/deployment`

**Suggested Commit:** `chore(deploy): configure production deployment`

**Learning:** Практики продакшн-деплоя, управление секретами вне системы контроля версий.

---

## EPIC: Документация и финальное ревью

#### ISL-DOC-001 — Написание README (разделы Setup, Architecture, Deployment)

**Description:** Заполнить разделы README, за которые отвечает Islam, согласно ТЗ §22: architecture, stack, installation, environment variables, Docker, migrations, deployment.

**Why:** ТЗ §22 (Documentation Requirements).

**Dependencies:** ISL-DEVOPS-005

**Priority:** P1

**Complexity:** M

**Acceptance Criteria:**
- Все разделы README, за которые отвечает Islam, заполнены и соответствуют фактической реализации.
- Инструкции проверены путём буквального выполнения на чистом checkout'е.

**Testing Requirements:** N/A (задача по документации); проверяется вручную путём выполнения инструкций.

**Git Branch:** `docs/readme-architecture`

**Suggested Commit:** `docs(readme): add architecture, setup, and deployment sections`

**Learning:** Техническая документация для онбординга разработчиков.

---

#### ISL-FINAL-001 — Подготовка финальной презентации

**Description:** Подготовить финальную презентацию/демо проекта, охватывающую архитектуру, ключевые решения (ADR-001), поток бронирования, демонстрацию гарантии конкурентности.

**Why:** ТЗ Phase 12 (Final Review) и ТЗ §29 (критерии Final Challenge включают итоговую сдачу проекта).

**Dependencies:** Все P0/P1-задачи завершены

**Priority:** P1

**Complexity:** M

**Owner:** BOTH (ведёт Islam)

**Acceptance Criteria:**
- Презентация охватывает: обзор архитектуры, ERD, демонстрацию anti-overbooking, сводку тестирования, деплой.
- Живая или записанная демонстрация сценария конкурентного бронирования (пример из ТЗ §10).

**Testing Requirements:** N/A.

**Git Branch:** N/A

**Suggested Commit:** N/A

**Learning:** Навыки технической презентации.

---

## ОБЩИЕ ЗАДАЧИ (Owner: BOTH)

Эти задачи идентично присутствуют в `KHADIZHA_TASKS.md`; не дублировать как отдельные карточки Trello — создать одну общую карточку на каждый пункт.

#### SHARED-001 — Ревью архитектуры

**Description:** Совместное ревью и утверждение модульной архитектуры (ТЗ §19) до начала полноценной работы над функциональностью.

**Dependencies:** ISL-SETUP-002

**Priority:** P0 | **Complexity:** S

**Acceptance Criteria:** Оба разработчика явно подтверждают структуру приложений в письменном виде (комментарий в PR или отметка в ADR).

---

#### SHARED-002 — Ревью базы данных / ERD

**Description:** Совместное ревью ERD и решения Option A/B (ТЗ §14) — операционно покрыто задачей ISL-ARCH-002.

**Dependencies:** ISL-ARCH-001

**Priority:** P0 | **Complexity:** S

---

#### SHARED-003 — Согласование API-контракта

**Description:** Согласовать единые соглашения по запросам/ответам (формат ошибок, формат пагинации, query-параметры фильтрации) по всем приложениям до начала параллельной разработки эндпоинтов.

**Dependencies:** ISL-SETUP-002

**Priority:** P0 | **Complexity:** S

**Acceptance Criteria:** Закоммичен краткий документ соглашений (`/docs/api-conventions.md`), на который оба разработчика ссылаются в своих сериализаторах/views.

---

#### SHARED-004 — Ревью дизайна бронирования

**Description:** Совместное ревью дизайна anti-overbooking (ТЗ §10) до начала реализации ISL-RES-004.

**Dependencies:** ISL-ARCH-002, KHD-SHOW-001

**Priority:** P0 | **Complexity:** S

---

#### SHARED-005 — Код-ревью (постоянно)

**Description:** Каждый значимый Pull Request проверяется другим разработчиком перед слиянием, согласно ТЗ §23.

**Dependencies:** Постоянно на протяжении всего проекта.

**Priority:** P0 | **Complexity:** N/A (повторяющаяся задача, не привязана к спринту)

---

#### SHARED-006 — Интеграционное тестирование

**Description:** Совместная проверка, что модули обоих разработчиков корректно работают вместе (например, showtimes + reservations, movies + showtimes).

**Dependencies:** Завершение функциональности интегрируемых модулей.

**Priority:** P0 | **Complexity:** M

---

#### SHARED-007 — Финальный проход тестирования

**Description:** Полный регрессионный проход по чек-листу критериев приёмки (ТЗ §27) перед финальной сдачей.

**Dependencies:** Вся функциональность завершена.

**Priority:** P0 | **Complexity:** M

---

#### SHARED-008 — Финальная презентация

**Description:** См. ISL-FINAL-001.

**Dependencies:** Вся функциональность завершена.

**Priority:** P1 | **Complexity:** M

---

## РАСПРЕДЕЛЕНИЕ ПО СПРИНТАМ (сторона Islam)

```text
Sprint 1 — Foundation
Islam: ISL-SETUP-001, ISL-SETUP-002, ISL-ARCH-001
Shared: SHARED-001 (частично, ожидает ревью от KHD)

Sprint 2 — Authentication
Islam: ISL-AUTH-001, ISL-AUTH-002, ISL-AUTH-003, ISL-AUTH-006
Shared: SHARED-002, SHARED-003, ISL-ARCH-002 (завершение SHARED-002)

Sprint 3 — RBAC & Reservation Scaffolding
Islam: ISL-AUTH-004, ISL-AUTH-005, ISL-AUTH-007, ISL-RES-001 (только модель, ожидает KHD-SHOW-001)

Sprint 4 — Showtimes & Reservation Design
Islam: ISL-RES-002, SHARED-004
(Parallel with: KHD-SHOW-001, KHD-SHOW-002)

Sprint 5 — Reservation Core
Islam: ISL-RES-003, ISL-RES-004, ISL-RES-005

Sprint 6 — Concurrency & Reports
Islam: ISL-RES-006, ISL-RES-007, ISL-REP-001
(Parallel with: KHD-TEST-003)

Sprint 7 — Testing & DevOps
Islam: ISL-REP-002, ISL-DEVOPS-003, ISL-DEVOPS-004, SHARED-006, SHARED-007

Sprint 8 — Deployment & Final Review
Islam: ISL-DEVOPS-005, ISL-DOC-001, ISL-FINAL-001, SHARED-008
```

*Примечание: ISL-DEVOPS-001 и ISL-DEVOPS-002 выполняются в Sprint 1 параллельно с ISL-ARCH-001, так как настройка Docker не зависит от решения по ERD.*

---

## СВОДКА ПО ЗАГРУЗКЕ

```text
Всего задач (за Islam): 26
Распределение по сложности:
  XS: 2
  S:  5
  M:  13
  L:  4
  XL: 1

Распределение по приоритету:
  P0: 16
  P1: 6
  P2: 4
  P3: 0
```

```text
Загрузка Islam: наибольшая концентрация в Authentication, ядре Reservation (включая
  самую сложную задачу проекта — ISL-RES-004), Reporting и DevOps/CI/CD/Deployment.
  Это соответствует роли Tech Lead: владение архитектурой, самая критичная логика,
  связанная с конкурентностью, и инфраструктура.

Загрузка Khadizha: см. KHADIZHA_TASKS.md — сконцентрирована в Movies, Cinema, Showtimes
  и значимой доле Testing (включая написание набора тестов на конкурентность, от которого
  зависит ISL-RES-007), плюс вклад в API-документацию.

Оценка баланса: Islam несёт больше суммарных баллов сложности (за счёт ISL-RES-004/XL и
  нескольких M/L задач по reservation+reports+devops), что соответствует роли Tech
  Lead/PM, несущей наиболее рискованные технические решения. Загрузка Khadizha
  намеренно не ограничена простым CRUD — логика планирования/пересечений сеансов и набор
  тестов на конкурентность являются существенными задачами с высокой обучающей ценностью.
  Количество задач распределено примерно поровну; вес сложности намеренно смещён в
  сторону Islam согласно ролям из ТЗ §3, при этом задачи Khadizha подобраны так, чтобы
  максимизировать обучающую ценность согласно цели ТЗ "Strong Junior Backend Developer".
```
