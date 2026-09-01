# KHADIZHA_TASKS.md
## Movie Reservation System — Задачи разработки
### Ответственный: Khadizha (Backend Developer)

Источник истины: утверждённое Техническое задание (Movie Reservation System TS, v1.0).
Файл структурирован по схеме `EPIC → FEATURE → TASK` для прямого переноса в Trello.

---

## EPIC: Основа проекта и архитектура

### Feature: Окружение и участие в архитектуре

#### KHD-SETUP-001 — Настройка локального окружения и онбординг в репозиторий

**Description:** Склонировать репозиторий, настроенный Исламом (ISL-SETUP-001/002), сконфигурировать локальный `.env`, убедиться, что проект запускается локально, разобраться в модульной структуре приложений.

**Why:** Предпосылка для всей последующей разработки; ТЗ §23 предполагает, что оба разработчика работают в одной модели ветвления.

**Dependencies:** ISL-SETUP-001, ISL-SETUP-002

**Priority:** P0

**Complexity:** XS

**Acceptance Criteria:**
- Проект запускается локально на машине Хадижи (через Docker или локальный venv).
- Локальный `.env` создан на основе `.env.example` (после появления в ISL-DEVOPS-002) — до этого используется минимальная локальная конфигурация.
- Подтверждён доступ на запись к репозиторию и понимание правил именования веток.

**Testing Requirements:** N/A (настроечная задача).

**Git Branch:** N/A

**Suggested Commit:** N/A

**Learning:** Онбординг в модульную Django-кодовую базу.

---

#### KHD-ARCH-001 — Ревью ERD и участие в решении Option A/B

**Description:** Изучить черновик ERD (ISL-ARCH-001) и активно участвовать в сессии решения Option A/B (ISL-ARCH-002), внося анализ компромиссов, актуальных для модулей Showtime/Cinema, которыми она владеет.

**Why:** ТЗ §14 требует совместного принятия решения, а Хадижа владеет приложениями `Showtime`/`Cinema`, которые напрямую взаимодействуют с этой моделью.

**Dependencies:** ISL-ARCH-001

**Priority:** P0

**Complexity:** S

**Owner:** BOTH (см. SHARED-002)

**Acceptance Criteria:**
- Письменные комментарии/фидбэк по черновику ERD предоставлены до сессии принятия решения.
- Согласие зафиксировано в итоговом документе ADR-001.

**Testing Requirements:** N/A.

**Git Branch:** N/A

**Suggested Commit:** N/A

**Learning:** Чтение и критика ERD, компромиссы реляционного моделирования.

---

## EPIC: Каталог фильмов

### Feature: Управление жанрами

#### KHD-MOV-001 — Реализация модели Genre и CRUD

**Description:** Реализовать модель `Genre` (уникальное название) и полный CRUD API согласно ТЗ §5.

**Why:** ТЗ §5 (Movie Management — управление жанрами), необходима до того, как Movie сможет ссылаться на жанры.

**Dependencies:** ISL-SETUP-002, ISL-AUTH-006 (классы разрешений)

**Priority:** P0

**Complexity:** S

**Acceptance Criteria:**
- Модель `Genre` с уникальным полем `name`.
- Полный CRUD эндпоинтов (`/genres`) с доступом на запись только для ADMIN, доступом на чтение — публичным/для USER (ТЗ §17).
- Создание жанра с дублирующимся названием возвращает 400.
- Миграция создана и чисто применена.

**Testing Requirements:** API-тесты на успешные сценарии CRUD, отклонение дублирующегося названия и проверки прав доступа (USER не может создавать/изменять/удалять).

**Git Branch:** `feature/genre-crud`

**Suggested Commit:** `feat(movies): add Genre model and CRUD endpoints`

**Learning:** Базовый DRF ModelViewSet, применение классов разрешений.

---

### Feature: Управление фильмами

#### KHD-MOV-002 — Реализация модели Movie

**Description:** Реализовать модель `Movie` согласно ТЗ §5: title, description, duration, release_date, poster, genres (M2M к Genre).

**Why:** ТЗ §5 и §14 (Data Model).

**Dependencies:** KHD-MOV-001

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Модель `Movie` со всеми полями, перечисленными в ТЗ §5, поле `genres` как many-to-many связь с `Genre`.
- Поля `duration` и `release_date` обязательны (необходимы для валидации showtime далее по цепочке).
- Миграция создана и чисто применена.
- Зарегистрировано в Django admin.

**Technical Notes:** Держать поле `poster` гибким (ImageField или URLField) — согласовать финальный подход с Исламом, так как это влияет на конфигурацию хранилища (локальное vs. внешнее), но не блокировать задачу — URLField допустим как вариант MVP по умолчанию.

**Testing Requirements:** Unit-тесты на создание модели, назначение жанров через M2M, валидацию обязательных полей.

**Git Branch:** `feature/movie-model`

**Suggested Commit:** `feat(movies): add Movie model with genre relationship`

**Learning:** Связи many-to-many в Django ORM.

---

#### KHD-MOV-003 — Реализация CRUD-эндпоинтов Movie

**Description:** Реализовать полный CRUD для `/movies` согласно ТЗ §5/§15, доступ на запись только ADMIN, чтение — публично/для USER.

**Why:** ТЗ §5 и §17 (Authorization Matrix).

**Dependencies:** KHD-MOV-002, ISL-AUTH-006

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- `POST/PUT/PATCH/DELETE /movies` доступны только ADMIN (403 для USER/анонимных).
- `GET /movies` и `GET /movies/{id}` доступны всем (публично или для аутентифицированного USER).
- Жанры можно назначать/обновлять через сериализатор фильма (вложенная запись или запись по списку ID).
- Соответствующие ошибки валидации для отсутствующих/некорректных полей.

**Testing Requirements:** API-тесты на полный жизненный цикл CRUD и применение прав доступа.

**Git Branch:** `feature/movie-crud`

**Suggested Commit:** `feat(movies): add Movie CRUD endpoints`

**Learning:** Сериализация вложенных/связанных полей в DRF.

---

#### KHD-MOV-004 — Реализация поиска, фильтрации, пагинации, сортировки для Movie

**Description:** Добавить к `GET /movies` поиск по названию, фильтрацию по жанру/диапазону дат релиза, пагинацию и сортировку (title/release_date/duration) согласно ТЗ §5.

**Why:** ТЗ §5 явно указывает поиск/фильтрацию/пагинацию/сортировку как обязательные возможности каталога фильмов; ТЗ §13 требует пагинацию повсеместно.

**Dependencies:** KHD-MOV-003

**Priority:** P1

**Complexity:** M

**Acceptance Criteria:**
- Поиск по query-параметру по названию (частичное совпадение, без учёта регистра).
- Фильтрация по одному или нескольким жанрам.
- Фильтрация по диапазону release_date.
- Пагинация применяется по умолчанию (размер страницы настраивается в settings).
- Сортировка поддерживается через query-параметр по title/release_date/duration.

**Technical Notes:** Использовать `django-filter` (или аналог) для согласованности; сверить именование query-параметров с SHARED-003 (согласование API-контракта).

**Testing Requirements:** API-тесты для каждого фильтра отдельно и в комбинации; метаданные пагинации проверены в ответе.

**Git Branch:** `feature/movie-search-filter`

**Suggested Commit:** `feat(movies): add search, filtering, pagination and ordering`

**Learning:** django-filter, классы пагинации DRF, оптимизация запросов при фильтрации.

---

## EPIC: Структура кинотеатра

### Feature: Иерархия Cinema, Hall, Seat

#### KHD-CIN-001 — Реализация модели Cinema и CRUD

**Description:** Реализовать модель `Cinema` (name, address) и полный CRUD согласно ТЗ §6.

**Why:** ТЗ §6 (Cinema Management), корень иерархии Cinema → Hall → Seat.

**Dependencies:** ISL-AUTH-006

**Priority:** P0

**Complexity:** S

**Acceptance Criteria:**
- Модель `Cinema` с полями name, address.
- Полный CRUD эндпоинтов (`/cinemas`), доступ на запись только ADMIN, чтение — публично/для USER.
- Миграция создана и чисто применена.

**Testing Requirements:** API-тесты на жизненный цикл CRUD и проверки прав доступа.

**Git Branch:** `feature/cinema-crud`

**Suggested Commit:** `feat(cinemas): add Cinema model and CRUD endpoints`

**Learning:** Моделирование корневой сущности в иерархическом домене.

---

#### KHD-CIN-002 — Реализация модели Hall и CRUD

**Description:** Реализовать модель `Hall` (name/номер, cinema FK, capacity) и полный CRUD согласно ТЗ §6.

**Why:** ТЗ §6 (связь Cinema → Hall).

**Dependencies:** KHD-CIN-001

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Модель `Hall`, связанная FK с `Cinema`, с полем capacity (вычисляемым или явно заданным — задокументировать выбранный подход).
- Полный CRUD эндпоинтов (`/halls`), фильтруемых по `cinema`, доступ на запись только ADMIN.
- Удаление `Cinema` с существующими `Hall` обрабатывается согласно определённой политике `on_delete` (рекомендуется PROTECT согласно примечанию в ТЗ §6).

**Testing Requirements:** API-тесты на CRUD, фильтрацию по cinema и поведение защиты от удаления.

**Git Branch:** `feature/hall-crud`

**Suggested Commit:** `feat(cinemas): add Hall model and CRUD endpoints`

**Learning:** Стратегии FK on_delete, защита целостности ссылок для сущностей с зависимой историей.

---

#### KHD-CIN-003 — Реализация модели Seat и CRUD с ограничениями уникальности

**Description:** Реализовать модель `Seat` (hall FK, row, seat_number) с полным CRUD и ограничением уникальности из ТЗ §6.

**Why:** ТЗ §6 (сущность Seat, требование уникальности) — эта модель является прямой зависимостью для ISL-RES-002/003.

**Dependencies:** KHD-CIN-002

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Модель `Seat`, связанная FK с `Hall`, с полями `row` и `seat_number`.
- Ограничение `unique_together` (или эквивалент) на (`hall`, `row`, `seat_number`).
- Полный CRUD эндпоинтов (`/seats`), фильтруемых по `hall`, доступ на запись только ADMIN.
- Попытка создать дубликат (hall, row, seat_number) возвращает 400.

**Testing Requirements:** API-тесты на CRUD, отклонение дублирующегося места, фильтрацию по hall.

**Git Branch:** `feature/seat-crud`

**Suggested Commit:** `feat(cinemas): add Seat model with uniqueness constraint and CRUD`

**Learning:** Составные ограничения уникальности, целостность данных на уровне БД.

---

## EPIC: Управление сеансами

### Feature: Планирование сеансов

#### KHD-SHOW-001 — Реализация модели Showtime с валидацией бизнес-правил

**Description:** Реализовать модель `Showtime` (movie FK, hall FK, start_time, end_time, price) со всеми правилами валидации из ТЗ §7.

**Why:** ТЗ §7 (Showtime Management) — прямая зависимость для ISL-RES-001/002/003/004.

**Dependencies:** KHD-MOV-002, KHD-CIN-003, ISL-ARCH-002 (если выбран Option B, задача также создаёт строки `ShowtimeSeat` при создании)

**Priority:** P0

**Complexity:** L

**Acceptance Criteria:**
- Модель `Showtime` с полями `movie`, `hall`, `start_time`, `end_time`, `price`.
- Валидация на уровне модели/сериализатора: `start_time` не в прошлом, `start_time < end_time`, `price > 0`.
- **Предотвращение пересечений:** создание/обновление сеанса, пересекающегося по времени с существующим сеансом в том же зале, отклоняется (ТЗ §7 — "один зал не может иметь пересекающиеся сеансы").
- Если выбран Option B (ТЗ §14): создание `Showtime` автоматически генерирует соответствующие строки `ShowtimeSeat` для каждого места в зале.
- Миграция создана и чисто применена.

**Technical Notes:** Запрос проверки пересечений должен корректно обрабатывать все случаи пересечения интервалов (не только точные совпадения) — это один из наиболее нетривиальных элементов валидации в проекте; при неясности обсудить подход к запросу с Исламом (применимы соглашения SHARED-003/004).

**Testing Requirements:** Unit/API-тесты на: отклонение прошлого start_time, отклонение start>=end, отклонение price<=0, отклонение пересекающегося сеанса в том же зале, принятие непересекающегося сеанса в том же зале, принятие сеанса в другом зале в то же время.

**Git Branch:** `feature/showtime-model-validation`

**Suggested Commit:** `feat(showtimes): add Showtime model with scheduling validation`

**Learning:** Логика запросов на пересечение интервалов, сложная валидация на уровне модели/сериализатора, (при необходимости) паттерны массового создания для ShowtimeSeat.

---

#### KHD-SHOW-002 — Реализация CRUD и фильтрации эндпоинтов Showtime

**Description:** Реализовать CRUD `/showtimes`, а также фильтрацию по фильму/дате/кинотеатру и пагинацию согласно ТЗ §7/§15.

**Why:** ТЗ §7 и §15 (таблица эндпоинтов `/showtimes`).

**Dependencies:** KHD-SHOW-001

**Priority:** P0

**Complexity:** M

**Acceptance Criteria:**
- Доступ на запись только ADMIN, чтение — публично/для USER (ТЗ §17).
- Фильтрация по `movie`, `date` (или диапазону дат) и `cinema`/`hall`.
- Пагинация применяется по умолчанию.
- Детальный эндпоинт возвращает полную информацию о сеансе, включая контекст зала/кинотеатра.

**Testing Requirements:** API-тесты на жизненный цикл CRUD, каждый фильтр, пагинацию и применение прав доступа.

**Git Branch:** `feature/showtime-crud-filtering`

**Suggested Commit:** `feat(showtimes): add CRUD and filtering endpoints`

**Learning:** Кросс-сущностная фильтрация (фильтрация по связанной сущности связанной сущности — hall→cinema).

---

## EPIC: Тестирование

### Feature: Покрытие тестами модулей и интеграции

#### KHD-TEST-001 — Полное тестовое покрытие модулей Movie и Cinema

**Description:** Обеспечить полное unit- и API-тестовое покрытие для всех эндпоинтов Movie/Genre/Cinema/Hall/Seat, закрыв пробелы, оставленные задачами по функциональности выше.

**Why:** ТЗ §20 (Testing Requirements — Unit, API, Permission, Database тесты) и ТЗ §26 (Definition of Done требует тестов для каждой задачи).

**Dependencies:** KHD-MOV-004, KHD-CIN-003

**Priority:** P1

**Complexity:** M

**Acceptance Criteria:**
- Все успешные/неуспешные сценарии CRUD покрыты для Genre, Movie, Cinema, Hall, Seat.
- Все границы прав доступа покрыты (USER vs ADMIN vs анонимный) для этих модулей.
- Включены тесты ограничений БД (уникальность Seat).
- Отчёт о покрытии не показывает значимых пробелов в этих приложениях (целевой уровень согласуется с Исламом, без произвольных цифр сверх того, что подразумевает ТЗ).

**Testing Requirements:** Эта задача *и есть* работа по тестированию; см. Acceptance Criteria.

**Git Branch:** `test/movies-cinema-coverage`

**Suggested Commit:** `test(movies,cinemas): expand coverage for CRUD and permissions`

**Learning:** Систематический аудит тестового покрытия, фикстуры/фабрики pytest-django.

---

#### KHD-TEST-002 — Полное тестовое покрытие модуля Showtime

**Description:** Обеспечить полное тестовое покрытие для валидации и CRUD Showtime, особенно логики предотвращения пересечений.

**Why:** ТЗ §20 и нетривиальная бизнес-логика, введённая в KHD-SHOW-001.

**Dependencies:** KHD-SHOW-002

**Priority:** P1

**Complexity:** M

**Acceptance Criteria:**
- Все правила валидации из ТЗ §7 покрыты как валидными, так и невалидными случаями.
- Явно протестированы граничные случаи: сеанс, начинающийся точно в момент окончания другого (граничный случай), сеанс, полностью вложенный в другой, сеанс, частично пересекающийся в начале/конце.
- Включены тесты прав доступа и фильтрации.

**Testing Requirements:** Эта задача *и есть* работа по тестированию; см. Acceptance Criteria.

**Git Branch:** `test/showtime-coverage`

**Suggested Commit:** `test(showtimes): expand coverage for scheduling validation edge cases`

**Learning:** Дизайн тестов граничных условий для логики интервалов.

---

#### KHD-TEST-003 — Написание набора тестов на конкурентное бронирование

**Description:** Написать тест(ы) на конкурентность, требуемые ТЗ §20/§10: два одновременных запроса на бронирование одного места одного сеанса, с проверкой ровно одного SUCCESS и одного CONFLICT.

**Why:** ТЗ §20 явно выделяет этот тест как обязательный, детально описанный; это самая критичная гарантия корректности проекта.

**Dependencies:** ISL-RES-004 (реализация должна существовать для тестирования)

**Priority:** P0

**Complexity:** L

**Acceptance Criteria:**
- Тест симулирует два действительно конкурентных запроса (например, через потоки, `multiprocessing` или транзакционные паттерны тест-клиента, подходящие для Django/pytest) на одну пару (showtime, seat).
- Тест проверяет, что ровно один запрос завершается успешно (бронирование создано), а другой получает ответ с конфликтом.
- Тест детерминирован/стабилен — задокументирован подход к предотвращению "мигания" (например, использование барьеров/синхронизации для принудительного вызова гонки).
- Тест выполняется в составе стандартного набора CI (или явно помеченной более медленной группы тестов, если требуется время выполнения).

**Technical Notes:** Этому тесту, вероятно, потребуется обойти стандартное оборачивание тестовой транзакции Django (например, `TransactionTestCase` или `transactional_db` в pytest-django), поскольку требуются реальные конкурентные транзакции БД, а не одна обёрнутая транзакция. Тесно координироваться с Исламом (ISL-RES-007) — это парная задача.

**Testing Requirements:** Эта задача *и есть* работа по тестированию; см. Acceptance Criteria. Дополнительно проверить, что тест падает при наивной (без блокировок) реализации — для подтверждения того, что он действительно обнаруживает гонку.

**Git Branch:** `test/concurrent-reservation`

**Suggested Commit:** `test(reservations): add concurrent double-booking test suite`

**Learning:** Тестирование конкурентности в Django, `TransactionTestCase`, симуляция race condition, примитивы синхронизации потоков.

---

#### KHD-TEST-004 — Вклад в тестирование бронирований и отчётов

**Description:** Внести вклад в интеграционные тесты потока бронирования (создание → обновление доступности → отмена → обновление доступности) и базовые тесты корректности эндпоинтов отчётов на основе фикстур.

**Why:** ТЗ §20 (Integration tests) и ТЗ §12 (корректность отчётности).

**Dependencies:** ISL-RES-006, ISL-REP-001

**Priority:** P1

**Complexity:** M

**Acceptance Criteria:**
- Интеграционный тест, проходящий полный жизненный цикл бронирования от начала до конца.
- Минимум один тест на каждый эндпоинт отчёта, проверяющий корректность на известном наборе фикстур.

**Testing Requirements:** Эта задача *и есть* работа по тестированию; см. Acceptance Criteria.

**Git Branch:** `test/reservation-reports-integration`

**Suggested Commit:** `test(reservations,reports): add integration coverage for full lifecycle and analytics`

**Learning:** Интеграционное тестирование между модулями, проверка аналитики на основе фикстур.

---

## EPIC: API-документация

#### KHD-DOC-001 — Вклад в Swagger/OpenAPI-аннотации для своих модулей

**Description:** Обеспечить, чтобы drf-spectacular генерировал понятную, полную схему документации для эндпоинтов Movies, Genres, Cinemas, Halls, Seats и Showtimes (описания, примеры значений, схемы ответов там, где автогенерации недостаточно).

**Why:** ТЗ §22 (API-документация через Swagger/OpenAPI).

**Dependencies:** KHD-SHOW-002

**Priority:** P2

**Complexity:** S

**Acceptance Criteria:**
- Все свои эндпоинты корректно отображаются в Swagger UI с осмысленными описаниями.
- Query-параметры (фильтры, поиск, сортировка) задокументированы, а не только выведены автоматически.
- Отсутствуют предупреждения о сломанной/отсутствующей схеме от drf-spectacular для этих приложений.

**Testing Requirements:** Ручная проверка через Swagger UI; команда валидации схемы drf-spectacular (если доступна) выполняется без ошибок.

**Git Branch:** `docs/swagger-catalog-modules`

**Suggested Commit:** `docs(api): improve OpenAPI schema for catalog and showtime endpoints`

**Learning:** Написание схемы OpenAPI, кастомизация drf-spectacular.

---

#### KHD-DOC-002 — Написание разделов README для своих модулей

**Description:** Заполнить раздел "Features" README, описывающий возможности каталога фильмов, структуры кинотеатра и планирования сеансов, согласно ТЗ §22.

**Why:** ТЗ §22 (Documentation Requirements — features).

**Dependencies:** KHD-DOC-001

**Priority:** P2

**Complexity:** XS

**Acceptance Criteria:**
- Раздел "Features" README точно отражает реализованные возможности модулей Movie/Cinema/Showtime.

**Testing Requirements:** N/A.

**Git Branch:** `docs/readme-features`

**Suggested Commit:** `docs(readme): add features section for catalog and showtime modules`

**Learning:** Техническая документация.

---

## ОБЩИЕ ЗАДАЧИ (Owner: BOTH)

Они идентичны перечисленным в `ISLAM_TASKS.md` — создать **одну** карточку Trello на каждый пункт, а не две.

#### SHARED-001 — Ревью архитектуры
Полное описание см. в ISLAM_TASKS.md. **Priority:** P0 | **Complexity:** S

#### SHARED-002 — Ревью базы данных / ERD
Полное описание см. в ISLAM_TASKS.md. **Priority:** P0 | **Complexity:** S

#### SHARED-003 — Согласование API-контракта
Полное описание см. в ISLAM_TASKS.md. **Priority:** P0 | **Complexity:** S

#### SHARED-004 — Ревью дизайна бронирования
Полное описание см. в ISLAM_TASKS.md. **Priority:** P0 | **Complexity:** S

#### SHARED-005 — Код-ревью (постоянно)
Полное описание см. в ISLAM_TASKS.md. **Priority:** P0 | **Complexity:** N/A (повторяющаяся)

#### SHARED-006 — Интеграционное тестирование
Полное описание см. в ISLAM_TASKS.md. **Priority:** P0 | **Complexity:** M

#### SHARED-007 — Финальный проход тестирования
Полное описание см. в ISLAM_TASKS.md. **Priority:** P0 | **Complexity:** M

#### SHARED-008 — Финальная презентация
Полное описание см. в ISLAM_TASKS.md. **Priority:** P1 | **Complexity:** M

---

## РАСПРЕДЕЛЕНИЕ ПО СПРИНТАМ (сторона Khadizha)

```text
Sprint 1 — Foundation
Khadizha: KHD-SETUP-001, KHD-ARCH-001
Shared: SHARED-001 (совместно с Исламом)

Sprint 2 — Movies & Genres
Khadizha: KHD-MOV-001, KHD-MOV-002, KHD-MOV-003
Shared: SHARED-002, SHARED-003
(Parallel with: ISL-AUTH-001, ISL-AUTH-002, ISL-AUTH-003)

Sprint 3 — Cinema Structure
Khadizha: KHD-MOV-004, KHD-CIN-001, KHD-CIN-002, KHD-CIN-003
(Parallel with: ISL-AUTH-004, ISL-AUTH-005, ISL-AUTH-007)

Sprint 4 — Showtimes
Khadizha: KHD-SHOW-001, KHD-SHOW-002
Shared: SHARED-004
(Parallel with: ISL-RES-001, ISL-RES-002)

Sprint 5 — Reservation Support & Early Testing
Khadizha: KHD-TEST-001, KHD-TEST-002
(Parallel with: ISL-RES-003, ISL-RES-004, ISL-RES-005)

Sprint 6 — Concurrency & Reports Testing
Khadizha: KHD-TEST-003 (в паре с ISL-RES-007), KHD-TEST-004
(Parallel with: ISL-RES-006, ISL-REP-001)

Sprint 7 — Testing & DevOps
Khadizha: KHD-DOC-001, KHD-DOC-002, SHARED-006, SHARED-007
(Parallel with: ISL-REP-002, ISL-DEVOPS-003, ISL-DEVOPS-004)

Sprint 8 — Deployment & Final Review
Khadizha: поддержка проверки критериев приёмки (прохождение ТЗ §27), SHARED-008
(Parallel with: ISL-DEVOPS-005, ISL-DOC-001)
```

---

## СВОДКА ПО ЗАГРУЗКЕ

```text
Всего задач (за Khadizha): 18
Распределение по сложности:
  XS: 2
  S:  3
  M:  9
  L:  4
  XL: 0

Распределение по приоритету:
  P0: 9
  P1: 5
  P2: 4
  P3: 0
```

```text
Загрузка Khadizha: сконцентрирована в полном каталоге Movie/Genre (включая нетривиальную
  логику поиска/фильтрации/пагинации/сортировки), всей иерархии Cinema→Hall→Seat с её
  ограничениями уникальности/целостности, и модуле Showtime — который включает один из
  наиболее нетривиальных элементов бизнес-логики проекта (валидацию пересечения интервалов,
  KHD-SHOW-001, сложность L). Она также владеет написанием набора тестов на конкурентность
  (KHD-TEST-003, сложность L) — задачей с высокой обучающей ценностью, напрямую связанной в
  пару с работой Ислама по укреплению отказоустойчивости (ISL-RES-007), плюс значимым
  вкладом в интеграционное/отчётное тестирование и API-документацию.

Загрузка Islam: см. ISLAM_TASKS.md — сконцентрирована в Authentication/RBAC, наиболее
  рискованной логике ядра Reservation (включая единственную XL-задачу, ISL-RES-004),
  Reporting и полном владении DevOps/CI/CD/Deployment.

Оценка баланса: 18 задач Khadizha намеренно не ограничены "простым CRUD" — KHD-SHOW-001 и
  KHD-TEST-003 являются существенными (L) задачами с реальной обучающей ценностью для
  backend-разработки (запросы на пересечение интервалов, тестирование конкурентности), что
  соответствует цели ТЗ — создать полноценный обучающий проект уровня Strong Junior, а не
  формальное распределение задач. 26 задач Islam несут больший суммарный вес сложности, что
  отражает дополнительную ответственность за архитектуру, DevOps и наиболее рискованную
  логику бронирования, присущую роли PM/Tech Lead (ТЗ §3). У обоих разработчиков в каждом
  спринте есть непрерывная, параллелизуемая работа без периодов простоя, и оба вносят
  прямой вклад в две самые критичные гарантии проекта: механизм anti-overbooking (Islam
  реализует, Khadizha тестирует) и целостность планирования сеансов, которая делает этот
  механизм осмысленным.
```
