# Руководство по проекту partners-admin

> **Стек:** October CMS 3.x · Laravel 9 · PHP 8.0+ · MySQL  
> **Назначение:** Партнёрский административный портал курорта «Завидово» — управление отелями, ресторанами, услугами, публикациями, паркингом, а также набор JSON API-эндпоинтов для внешнего сайта.

На дату: **5 мая 2026 г.**

---

## Содержание

1. [Структура проекта](#1-структура-проекта)
2. [Как работает October CMS](#2-как-работает-october-cms)
3. [Плагины: полный список и зависимости](#3-плагины-полный-список-и-зависимости)
4. [Навигация в admin-панели (backend)](#4-навигация-в-admin-панели-backend)
5. [Как настраиваются страницы в backend-Admin](#5-как-настраиваются-страницы-в-backend-admin)
6. [Модели и база данных](#6-модели-и-база-данных)
7. [API-маршруты (внешний сайт)](#7-api-маршруты-внешний-сайт)
8. [Frontend-часть (тема blank)](#8-frontend-часть-тема-blank)
9. [Консольные команды и планировщик](#9-консольные-команды-и-планировщик)
10. [Внешние интеграции и сервисы](#10-внешние-интеграции-и-сервисы)
11. [Конфигурация окружения (.env)](#11-конфигурация-окружения-env)
12. [Быстрый справочник: «что и где менять»](#12-быстрый-справочник-что-и-где-менять)

---

## 1. Структура проекта

```
partners-admin/
├── app/                          # Глобальные сервисы приложения
│   ├── Provider.php              # Регистрация глобального ServiceProvider
│   ├── Services/
│   │   ├── ParkingApiClient.php  # HTTP-клиент к внешней парковочной системе
│   │   └── RabbitMqPublisher.php # Публикация событий в RabbitMQ
│   └── assets/                   # JS/CSS/изображения для backend (общие)
├── config/                       # Laravel/October конфиги (database, services, cors, …)
├── plugins/                      # ВСЯ бизнес-логика — здесь живёт код
│   ├── gromit/popupbuilder/      # Вспомогательный поведенческий плагин (попапы)
│   ├── integration/restaurantconnectors/ # Интеграция с внешней ресторанной системой
│   ├── platform/                 # ОСНОВНЫЕ плагины (см. раздел 3)
│   │   ├── api/                  # Только API-маршруты и HTTP-контроллеры
│   │   ├── common/               # Справочники (локации, адреса, промо, документы…)
│   │   ├── hotels/               # Отели, номера, тарифы, бронирование
│   │   ├── notifications/        # Email + Telegram-уведомления, шаблоны
│   │   ├── partners/             # Партнёры, кабинет, мероприятия, парковки
│   │   ├── posts/                # Публикации, новости, мероприятия, галереи, SEO-страницы
│   │   ├── restaurants/          # Рестораны, кухни, бронирование столов
│   │   ├── services/             # Предложения партнёров (услуги + категории)
│   │   └── users/                # Внешние пользователи (мобильное приложение / сайт)
│   └── security/parking/         # Модуль паркинга — пользователи, абонементы, пропуска
├── themes/blank/                 # CMS-тема: frontend партнёрского кабинета + монитор охраны
│   ├── layouts/                  # auth.htm, main.htm, security.htm
│   ├── pages/                    # Отдельные страницы (см. раздел 8)
│   └── partials/                 # Переиспользуемые блоки
├── modules/                      # Ядро October CMS (не трогать)
└── vendor/                       # Composer-зависимости (не трогать)
```

---

## 2. Как работает October CMS

### Два «мира» в одном Laravel-приложении

| Слой | URL-префикс | Что делает |
|---|---|---|
| **Backend (Admin)** | `/backend/*` | Административная панель — CRUD для данных |
| **CMS (Frontend)** | `/*` (тема `blank`) | HTML-страницы партнёрского кабинета и монитора охраны |
| **API** | `/api/v1/*`, `/api/v4/*` | JSON-эндпоинты для внешнего сайта |

### Ключевые понятия

- **Plugin** — единица кода. Каждый плагин имеет `Plugin.php`, где регистрируются меню, права, компоненты, mail-шаблоны.
- **Controller (backend)** — PHP-класс + набор YAML-конфигов, которые описывают, какие поля показывать в форме и таблице.
- **Model** — Eloquent-модель + папка `models/modelname/` с `fields.yaml`, `columns.yaml` и `rules.php`.
- **Component** — PHP-компонент для CMS-страниц (в этом проекте почти не используется, логика встроена прямо в `.htm`-файлы страниц).
- **Theme** — набор `.htm`-файлов с шаблонами Twig + PHP-блоками (`==` разделители).

---

## 3. Плагины: полный список и зависимости

### Граф зависимостей

```
Platform.Common
    └── Platform.Partners
            └── Platform.Users
                    ├── Platform.Hotels
                    └── Platform.Restaurants
                            └── Integration.RestaurantConnectors

Platform.Posts       (нет зависимостей)
Platform.Services    (нет зависимостей)
Platform.Api         (использует модели из всех Platform.*)
Platform.Notifications (использует модели из Hotels, Partners, Restaurants)
Security.Parking     (нет зависимостей от Platform.*)
GromIT.PopupBuilder  (утилита, нет зависимостей)
```

### Описание каждого плагина

#### `platform/common` — Справочники

**Назначение:** базовые справочные данные курорта.  
**Навигация:** раздел **«Справочники»** в боковом меню.

| Пункт меню | URL backend | Модель |
|---|---|---|
| Удобства (отели) | `platform/hotels/amenities` | `Hotels\Models\Amenity` |
| Услуги (отели) | `platform/hotels/servicecategories` | `Hotels\Models\ServiceCategory` |
| Возрастные категории | `platform/hotels/roomagecategories` | `Hotels\Models\RoomAgeCategory` |
| Удобства (рестораны) | `platform/restaurants/restaurantservices` | `Restaurants\Models\RestaurantService` |
| Кухни | `platform/restaurants/cuisines` | `Restaurants\Models\Cuisine` |
| Категории предложений | `platform/services/servicecategories` | `Services\Models\ServiceCategory` |
| Удобства/иконки услуг | `platform/services/serviceamenities` | `Services\Models\ServiceAmenity` |
| Правовые документы | `platform/common/legaldocuments` | `Common\Models\LegalDocument` |
| Локации курорта | `platform/common/locations` | `Common\Models\Location` |
| Адреса | `platform/common/addresses` | `Common\Models\Address` |
| Типы рассылок | `platform/users/subscriptions` | `Users\Models\Subscription` |

**Глобальные ресурсы (подключаются ко всем backend-страницам):**
- `jquery.inputmask` — маски для полей ввода
- `jquery.suggestions` + `dadata.js` — подсказки адресов через DaData API

---

#### `platform/partners` — Партнёры

**Назначение:** управление партнёрами, их кабинетами, мероприятиями и парковочными абонементами партнёров.

**Навигация:**
- Для **суперадмина/менеджера** → раздел **«Партнёры»** (список всех партнёров)
- Для **пользователя с ролью partner** → раздел **«Кабинет партнёра»** (только своя карточка)

**Контроллеры:**

| Класс | URL backend | Что делает |
|---|---|---|
| `Partners` | `platform/partners/partners` | CRUD партнёров (Form + List + Relation) |
| `PartnerUsers` | `platform/partners/partnerusers` | Пользователи партнёрского кабинета (не backend-пользователи!) |
| `PartnerEvents` | `platform/partners/partnerevents` | Мероприятия в календаре |

**Модели:**

| Модель | Таблица | Назначение |
|---|---|---|
| `Partner` | `platform_partners` | Карточка партнёра |
| `PartnerBranch` | `platform_partner_branches` | Связь партнёра с конкретным объектом (Hotel/Restaurant/Service — полиморфная) |
| `PartnerUser` | `platform_partner_users` | Пользователь фронт-кабинета (логин/пароль для `/partner/auth`) |
| `PartnerEvent` | `platform_partner_events` | Мероприятие в календаре партнёра |
| `PartnerFeature` | `platform_partner_features` | Преимущества партнёра (список тезисов) |
| `PartnerParking` | `platform_partner_parkings` | Привязка партнёра к парковочной зоне |
| `PartnerParkingAbonement` | `platform_partner_parking_abonements` | Выданный абонемент на парковку |
| `PartnerParkingSession` | `platform_partner_parking_sessions` | Сессии въезда/выезда |
| `PartnerTemplate` | `platform_partner_templates` | Шаблоны для уведомлений конкретного партнёра |

**Права доступа:**

| Право | Описание |
|---|---|
| `platform.partners.view` | Доступ к своей странице (для роли partner) |

---

#### `platform/hotels` — Отели

**Навигация:** раздел **«Отели»**. В боковом меню динамически генерируется список каждого отеля + пункт «Добавить отель».

**Контроллеры:**

| Класс | URL backend | Что делает |
|---|---|---|
| `Hotels` | `platform/hotels/hotels` | CRUD отелей (Form + List + Relation) |
| `RoomTypes` | `platform/hotels/roomtypes` | Типы номеров |
| `Amenities` | `platform/hotels/amenities` | Удобства (справочник) |
| `ServiceCategories` | `platform/hotels/servicecategories` | Категории услуг (справочник) |
| `RoomAgeCategories` | `platform/hotels/roomagecategories` | Возрастные категории гостей |

**Модели:**

| Модель | Назначение |
|---|---|
| `Hotel` | Карточка отеля (звёзды, описание, галерея, Traveline ID) |
| `RoomType` | Тип номера (площадь, вместимость, удобства, цены) |
| `Rate` / `BasePrice` / `AdditionalPrice` | Тарифы и цены |
| `Restriction` | Ограничения бронирования |
| `Availability` | Доступность номеров |
| `Reservation` | Бронирование номера |
| `ReservationGuest` | Гости в брони |
| `ServiceCategory` / `ServiceItem` | Услуги отеля |
| `Amenity` | Удобства (Wi-Fi, бассейн и т.д.) |

**Ключевое поле:** `tl_id` в модели `Hotel` — ID для интеграции с **Traveline** (система онлайн-бронирования).  
**CSS** отелей (`style.less`) подключается глобально ко всем backend-страницам через `Plugin.php`.

---

#### `platform/restaurants` — Рестораны

**Навигация:** раздел **«Рестораны»**. В боковом меню — динамический список ресторанов. Партнёр видит только свои рестораны.

**Контроллеры:**

| Класс | URL backend | Что делает |
|---|---|---|
| `Restaurants` | `platform/restaurants/restaurants` | CRUD ресторанов |
| `Cuisines` | `platform/restaurants/cuisines` | Справочник кухонь |
| `RestaurantServices` | `platform/restaurants/restaurantservices` | Справочник удобств ресторана |

**Модели:**

| Модель | Назначение |
|---|---|
| `Restaurant` | Карточка ресторана |
| `ReservationTable` | Столы для бронирования |
| `ReservationSection` | Зоны (летняя терраса, основной зал и т.д.) |
| `Reservation` | Бронирование стола |
| `Cuisine` / `RestaurantService` | Справочники |

---

#### `platform/posts` — Публикации

**Навигация:** раздел **«Публикации»** → подразделы:

| Пункт | URL backend | Тип контента |
|---|---|---|
| Публикации (новости, мероприятия) | `platform/posts/posts` | Тип `Post::POST_TYPE_NEWS` / `POST_TYPE_EVENT` |
| Спецпредложения | `platform/posts/specialoffers` | `SpecialOffer` |
| Страницы (блог, SEO) | `platform/posts/pages` | `Page` |
| Статичные страницы | `platform/posts/staticpages` | `StaticPage` |
| Галереи | `platform/posts/galleries` | `Gallery` |
| Календарь мероприятий | `platform/partners/partnerevents` | `PartnerEvent` (в плагине partners) |

**Модели:**

| Модель | Назначение |
|---|---|
| `Post` | Новость или мероприятие (поле `type` разделяет) |
| `SubPost` | Вложенные блоки публикации |
| `EventSchedule` | Расписание мероприятия (дни, время) |
| `Gallery` | Фотогалерея |
| `Page` | Страница блога или SEO-страница |
| `StaticPage` | Информационная страница |
| `SpecialOffer` | Специальное предложение |
| `RelatedEntity` | Полиморфная связь публикации с другим объектом |

**Особенность:** контроллер `Posts` использует `GromIT\PopupBuilder` для попапа копирования мероприятий (`copy_post_event`).

---

#### `platform/services` — Предложения партнёров

**Навигация:** раздел **«Предложения партнёров»**.

**Контроллеры:** `Services`, `ServiceCategories`, `ServiceAmenities`

**Модели:**

| Модель | Назначение |
|---|---|
| `Service` | Конкретное предложение (тур, SPA, активность) |
| `ServiceCategory` | Категория предложений |
| `ServiceAmenity` | Иконки/удобства для карточки услуги |

---

#### `platform/users` — Пользователи

> Это **пользователи мобильного приложения/сайта**, НЕ сотрудники в backend.

**Навигация:** раздел **«Пользователи»**.

**Контроллеры:** `Users`, `Subscriptions`

**Модели:**

| Модель | Назначение |
|---|---|
| `User` | Пользователь (аккаунт в мобильном приложении) |
| `Account` | Привязанный аккаунт (Apple ID, Google и т.д.) |
| `Subscription` | Подписка на рассылки |
| `Device` | Устройство пользователя (push-уведомления) |
| `Transaction` / `Level` | Программа лояльности |
| `TIDSession` | Сессии Traveline-авторизации |
| `UserLegalDocument` | Принятые юридические документы |

---

#### `platform/notifications` — Уведомления

**Назначение:** отправка Email и Telegram-уведомлений по событиям (бронирование и т.д.).

**Контроллеры:** `MessageTemplates` (редактирование шаблонов уведомлений)

**Зарегистрированные mail-шаблоны:**

| Ключ | Описание |
|---|---|
| `email_verification` | Подтверждение email |
| `hotel_reservation` | Бронирование номера |
| `restaurant_reservation` | Бронирование стола |

**Twig-фильтр:** `human_date` — форматирует дату в читаемый вид (учитывает +3 часа Moscow time).

**Модели:** `MessageTemplate`, `RestaurantReservation`, `TelegramUser`

---

#### `security/parking` — Паркинг

**Навигация:** раздел **«Парковка»** (виден только пользователям с правом `security.parking`).

**Контроллеры:**

| Класс | URL backend | Что делает |
|---|---|---|
| `ParkingUsers` | `security/parking/parkingusers` | Пользователи системы (Import/Export, попапы изменения пропуска) |
| `ParkingTickets` | `security/parking/parkingtickets` | Абонементы |
| `ParkingPermitTypes` | `security/parking/parkingpermittypes` | Типы пропусков |
| `ParkingPermits` | `security/parking/parkingpermits` | Категории доступа |
| `ParkingOperators` | `security/parking/parkingoperators` | Внешние парковочные системы |

**Модели:**

| Модель | Назначение |
|---|---|
| `ParkingUser` | Пользователь парковки |
| `ParkingTicket` | Абонемент |
| `ParkingPermit` / `ParkingPermitType` | Пропуск и его тип |
| `ParkingZone` | Парковочная зона |
| `ParkingOperator` | Внешняя система (AutoMarshal, СКУД и т.д.) |
| `AutoMarshalEvent` | Событие от системы распознавания номеров |
| `ParkingEmployeeSession` | Сессии сотрудников |

**Права:**

| Право | Описание |
|---|---|
| `security.parking` | Просмотр раздела |
| `security.parking.management` | Управление (изменение данных) |

---

#### `platform/api` — API-слой

**Назначение:** только HTTP-контроллеры и маршруты для внешнего сайта. Не регистрирует меню и права. Данные читает из моделей других плагинов.

---

#### `gromit/popupbuilder` — Утилита

**Назначение:** поведение `PopupController` для backend-контроллеров. Позволяет открывать модальные окна с формами прямо в списке/форме. Используется в `Posts` и `ParkingUsers`.

---

#### `integration/restaurantconnectors` — Интеграция ресторанов

**Назначение:** синхронизация данных с внешней ресторанной системой.  
**Зависит от:** `Platform.Restaurants`.

---

## 4. Навигация в admin-панели (backend)

Полное меню после авторизации (URL `/backend`):

```
Кабинет партнёра        → /backend/platform/partners/partners/update/{id}
                           (только для роли "partner" — прямой переход в свою карточку)

Партнёры                → /backend/platform/partners/partners
  (для суперадмина/менеджера)

Отели                   → /backend/platform/hotels/hotels
  ├── [имя отеля 1]     → /backend/platform/hotels/hotels/update/{id}
  ├── [имя отеля 2]     → ...
  └── Добавить отель    → /backend/platform/hotels/hotels/create

Рестораны               → /backend/platform/restaurants/restaurants
  └── [имена ресторанов по аналогии с отелями]

Публикации              → /backend/platform/posts/posts
  ├── Публикации        → /backend/platform/posts/posts
  ├── Спецпредложения   → /backend/platform/posts/specialoffers
  ├── Страницы (блог, SEO) → /backend/platform/posts/pages
  ├── Статичные страницы → /backend/platform/posts/staticpages
  ├── Галереи           → /backend/platform/posts/galleries
  └── Календарь мероприятий → /backend/platform/partners/partnerevents

Предложения партнёров   → /backend/platform/services/services

Пользователи            → /backend/platform/users/users

Парковка                → /backend/security/parking/parkingusers
  ├── Пользователи      → /backend/security/parking/parkingusers
  ├── Абонементы        → /backend/security/parking/parkingtickets
  ├── Типы пропусков    → /backend/security/parking/parkingpermittypes
  ├── Категории доступа → /backend/security/parking/parkingpermits
  └── Паркинг-системы   → /backend/security/parking/parkingoperators

Справочники             → (раздел без URL — только подпункты)
  ├── [Отели] Удобства  → /backend/platform/hotels/amenities
  ├── [Отели] Услуги    → /backend/platform/hotels/servicecategories
  ├── [Отели] Возрастные категории → /backend/platform/hotels/roomagecategories
  ├── [Рестораны] Удобства → /backend/platform/restaurants/restaurantservices
  ├── [Рестораны] Кухни → /backend/platform/restaurants/cuisines
  ├── [Услуги] Категории → /backend/platform/services/servicecategories
  ├── [Услуги] Удобства и иконки → /backend/platform/services/serviceamenities
  ├── Правовые документы → /backend/platform/common/legaldocuments
  ├── Локации курорта   → /backend/platform/common/locations
  ├── Адреса            → /backend/platform/common/addresses
  └── Типы рассылок     → /backend/platform/users/subscriptions
```

### Роли backend-пользователей

| Роль | Что видит |
|---|---|
| Суперадмин (`is_superuser = true`) | Полный доступ ко всему |
| Менеджер | Всё, кроме управления пользователями и прав |
| `partner` (роль `code = 'partner'`) | Только «Кабинет партнёра», свои рестораны, свои мероприятия |
| `security.parking` | Раздел «Парковка» (только просмотр) |
| `security.parking.management` | Раздел «Парковка» (полное управление) |

---

## 5. Как настраиваются страницы в backend-Admin

### Принцип работы

Каждый backend-контроллер использует **три behaviour (поведения)**:
- `FormController` — форма создания/редактирования записи
- `ListController` — таблица-список записей
- `RelationController` — вкладки с вложенными связями

Все три управляются **YAML-конфигами** в папке контроллера.

### Файловая структура контроллера (пример: Hotels)

```
plugins/platform/hotels/controllers/
├── Hotels.php                      # PHP-класс контроллера
└── hotels/                         # Папка конфигов (имя = lowercase класса)
    ├── config_form.yaml            # Конфиг формы
    ├── config_list.yaml            # Конфиг списка
    ├── config_relation.yaml        # Конфиг вкладок со связями
    ├── create.php / update.php     # Опциональные переопределения методов
    ├── index.php                   # Опциональное переопределение списка
    ├── preview.php                 # Страница просмотра
    └── _list_toolbar.php           # HTML кнопок над таблицей
```

### config_form.yaml — что настраивает

```yaml
name: Hotel                                    # Заголовок в хлебных крошках
form: $/platform/hotels/models/hotel/fields.yaml  # Путь к YAML с полями ($ = корень проекта)
modelClass: Platform\Hotels\Models\Hotel        # Класс модели

create:
    title: Новый отель
    redirect: platform/hotels/hotels/update/:id # После сохранения — редирект на редактирование

update:
    title: Редактирование отеля
    redirect: platform/hotels/hotels
```

### fields.yaml — как добавить/изменить поле формы

Файл лежит в: `plugins/{vendor}/{plugin}/models/{modelname}/fields.yaml`

```yaml
fields:                         # Поля вне табов (верхняя часть формы)
    name:
        label: Название отеля
        span: left              # left / right / full / row

    is_active:
        label: Опубликован
        type: switch            # Типы: text, number, textarea, richeditor, switch,
        span: right             #       dropdown, radio, balloon-selector, datepicker,
                                #       fileupload, relation, partial, hint, …

tabs:                           # Вкладки формы
    defaultTab: Описание отеля
    fields:
        description:
            label: Описание
            type: richeditor
            tab: Описание отеля  # К какой вкладке относится поле

        photos:
            label: Фото
            type: fileupload
            mode: image
            tab: Медиа
```

**Суффиксы полей:**
- `fieldname@create` — поле показывается только при создании
- `fieldname@update` — поле показывается только при редактировании

### columns.yaml — как настроить таблицу-список

Файл: `plugins/{vendor}/{plugin}/models/{modelname}/columns.yaml`

```yaml
columns:
    id:
        label: ID
        searchable: true

    name:
        label: Название
        sortable: true
        searchable: true

    is_active:
        label: Активен
        type: switch
```

### Где какие fields/columns лежат

| Сущность | Путь к fields.yaml |
|---|---|
| Партнёр | `plugins/platform/partners/models/partner/fields.yaml` |
| Отель | `plugins/platform/hotels/models/hotel/fields.yaml` |
| Тип номера | `plugins/platform/hotels/models/roomtype/fields.yaml` |
| Ресторан | `plugins/platform/restaurants/models/restaurant/fields.yaml` |
| Публикация | `plugins/platform/posts/models/post/fields.yaml` |
| Спецпредложение | `plugins/platform/posts/models/specialoffer/fields.yaml` |
| Страница (блог/SEO) | `plugins/platform/posts/models/page/fields.yaml` |
| Статичная страница | `plugins/platform/posts/models/staticpage/fields.yaml` |
| Галерея | `plugins/platform/posts/models/gallery/fields.yaml` |
| Услуга | `plugins/platform/services/models/service/fields.yaml` |
| Пользователь (внешний) | `plugins/platform/users/models/user/fields.yaml` |
| Шаблон уведомления | `plugins/platform/notifications/models/messagetemplate/fields.yaml` |
| Пользователь парковки | `plugins/security/parking/models/parkinguser/fields.yaml` |
| Абонемент парковки | `plugins/security/parking/models/parkingticket/fields.yaml` |

### Как добавить новый пункт в меню backend

Метод `registerNavigation()` в `Plugin.php`:

```php
public function registerNavigation()
{
    return [
        'my_section' => [                          // Ключ раздела (уникальный)
            'label'       => 'Мой раздел',          // Название в меню
            'url'         => Backend::url('vendor/plugin/controller'),
            'icon'        => 'ph ph-star',          // Phosphor Icons (ph-*)
            'permissions' => ['vendor.plugin.*'],   // Требуемые права
            'order'       => 500,                   // Порядок сортировки
            'sideMenu'    => [                      // Подпункты
                'item1' => [
                    'label' => 'Подпункт',
                    'url'   => Backend::url('vendor/plugin/controller'),
                    'icon'  => 'ph ph-list',
                ],
            ],
        ],
    ];
}
```

### Иконки

Проект использует **Phosphor Icons** (`ph ph-*`). Примеры: `ph ph-buildings`, `ph ph-coffee`, `ph ph-newspaper`, `ph ph-car`, `ph ph-users`.

---

## 6. Модели и база данных

### Миграции

Все миграции хранятся в папке `updates/` каждого плагина:
```
plugins/platform/hotels/updates/
├── create_hotels_table.php
├── create_room_types_table.php
└── version.yaml          # Файл версий — ОБЯЗАТЕЛЬНЫЙ, описывает порядок применения
```

Запуск миграций: `php artisan october:migrate`

### version.yaml — ключевой файл плагина

```yaml
1.0.0:
    - create_hotels_table.php
1.0.1:
    - add_tl_id_to_hotels.php
```

Без записи в `version.yaml` миграция не выполнится.

### Связи между моделями (важные)

```
Partner  ──< PartnerBranch >──  Hotel / Restaurant / Service  (полиморфная, через entity_type/entity_id)
Partner  ──< PartnerUser          (пользователи фронт-кабинета)
Partner  ──< PartnerEvent         (мероприятия)
Partner  ──< PartnerParking >──  ParkingZone
Hotel    ──< RoomType ──< Rate / BasePrice
Hotel    ──< Reservation ──< ReservationGuest
Restaurant ──< ReservationTable ──< Reservation (ресторанная)
Post     ──< RelatedEntity >──  Hotel / Restaurant / Service / Partner  (полиморфная)
ParkingUser  ──< ParkingUserPermit >── ParkingPermit
ParkingUser  ──< ParkingUserVehicle
```

---

## 7. API-маршруты (внешний сайт)

### Обзор групп маршрутов

| Файл | Префикс | Middleware | Назначение |
|---|---|---|---|
| `plugins/platform/api/routes.php` | `/api/v4` | CORS | Основное публичное API для сайта |
| `plugins/platform/partners/routes.php` | `/api/v1/partners` | `web` | API партнёрского кабинета (парковочные абонементы) |
| `plugins/platform/notifications/routes.php` | `/api/v1/notification` | `web` | Отправка уведомлений через salt-токен |

---

### `/api/v4` — Основное API сайта

> Файл: `plugins/platform/api/routes.php`  
> Контроллеры: `plugins/platform/api/http/controllers/`

| Метод | URL | Контроллер | Описание |
|---|---|---|---|
| GET | `/api/v4/blog` | `BlogController@index` | Список страниц блога (тип `blog`, пагинация) |
| GET | `/api/v4/blog/{slug}` | `BlogController@entry` | Одна запись блога |
| GET | `/api/v4/commercial/{segments}` | `CommercialPageController@entry` | Коммерческая страница по пути |
| GET | `/api/v4/services` | `ServiceController@index` | Предложения партнёров (с фильтрами) |
| GET | `/api/v4/services/{slug}` | `ServiceController@entry` | Одна услуга |
| GET | `/api/v4/static` | `StaticPageController@index` | Список статичных страниц |
| GET | `/api/v4/static/{slug}` | `StaticPageController@entry` | Одна статичная страница |
| GET | `/api/v4/partners` | `PartnerController@index` | Список партнёров |
| GET | `/api/v4/offers/special` | `SpecialOffersController@index` | Спецпредложения |
| GET | `/api/v4/galleries` | `GalleryController@index` | Список галерей |
| GET | `/api/v4/galleries/{slug}` | `GalleryController@entry` | Одна галерея |
| GET | `/api/v4/posts` | `PostController@index` | Новости/мероприятия (параметр `type`) |
| GET | `/api/v4/posts/{slug}` | `PostController@entry` | Одна публикация |
| GET | `/api/v4/promocodes/check` | `PromoController@check` | Проверка промокода |
| GET | `/api/v4/promocodes/apply` | `PromoController@codeApply` | Применение промокода |
| GET | `/api/v4/hotels/tl` | `HotelController@tl` | ID отелей для Traveline |
| GET | `/api/v4/booking/parking/check` | `ParkingController@checkParkingAvailability` | Проверка наличия парковочных мест |
| GET | `/api/v4/booking/parking/create` | `ParkingController@createParkingAbonements` | Создание абонемента |
| GET/POST | `/api/v4/security/automarshal/event` | `ParkingController@AutoMarshalDetect` | Webhook от AutoMarshal |
| GET/POST | `/api/v4/security/automarshal/last` | `ParkingController@AutoMarshalLastEvent` | Последнее событие AutoMarshal |
| GET | `/api/v4/security/automarshal/list` | `ParkingController@AutoMarshalList` | Список событий |
| GET | `/api/v4/n8n/partner/user/get` | Closure | Проверка партнёрского пользователя для n8n (требует `salt=n8n`) |

**Параметры `PostController@index`:**

| Параметр | Описание |
|---|---|
| `type` | `news` или `event` |
| `isArchive` | `true`/`false` — архивные мероприятия |
| `fromDate` / `toDate` | Фильтр по датам |

---

### `/api/v1/partners` — API партнёрского кабинета

> Файл: `plugins/platform/partners/routes.php`

| Метод | URL | Описание |
|---|---|---|
| GET | `/api/v1/partners/parking/abonements` | Список абонементов партнёра. Параметры: `user_id`, `limit`, `offset`, `search`, `filterModel` (JSON AG Grid), `sortModel` (JSON) |

Поддерживает сортировку и фильтрацию в формате **AG Grid** (компонент React-таблицы на фронтенде кабинета партнёра).

---

### `/api/v1/notification` — API уведомлений

> Файл: `plugins/platform/notifications/routes.php`

| Метод | URL | Описание |
|---|---|---|
| POST | `/api/v1/notification/send` | Отправка уведомления. Требует `salt` из `NOTIFICATION_SALT` в `.env` |

**Параметры:**

| Параметр | Описание |
|---|---|
| `salt` | Секретный токен из `.env` `NOTIFICATION_SALT` |
| `event` | Ключ события (из `MessageTemplate::EVENTS`) |
| `type` | `hotel` или `restaurant` |
| `content` | JSON с данными события |
| `email` | Адрес получателя (для email-уведомлений) |

---

### Как добавить новый API-эндпоинт

1. Открыть файл маршрутов нужного плагина (или создать `routes.php` в новом плагине).
2. Добавить маршрут в соответствующую группу `Route::group`:
   ```php
   Route::get('/my-endpoint', function () {
       return response()->json(['data' => ...]);
   });
   // или через контроллер:
   Route::get('/my-endpoint', [MyController::class, 'index']);
   ```
3. Если нужен новый HTTP-контроллер — создать в `plugins/{vendor}/{plugin}/http/controllers/`.
4. Убедиться, что CORS настроен в `config/cors.php` (сейчас открыт `['*']`).

---

## 8. Frontend-часть (тема blank)

Тема `blank` — это **не публичный сайт**, а:
1. Партнёрский фронт-кабинет (авторизация + просмотр абонементов)
2. Монитор охраны (просмотр событий AutoMarshal в реальном времени)

Главная страница `/` (файл `main.htm`) сразу редиректит на `/backend`.

### Страницы темы

| Файл | URL | Layout | Назначение |
|---|---|---|---|
| `main.htm` | `/` | — | Редирект на backend |
| `partner-auth.htm` | `/partner/auth` | `auth` | Форма входа для пользователей фронт-кабинета |
| `partner-lk.htm` | `/partner/lk` | `main` | Главная страница кабинета (дашборд партнёра) |
| `partner-events.htm` | `/partner/events` | `main` | Мероприятия партнёра |
| `partner-parking.htm` | `/partner/parking` | `main` | Абонементы на парковку (AG Grid-таблица) |
| `security-auth.htm` | `/security/auth` | `auth` | Авторизация для монитора охраны |
| `security_monitor.htm` | `/security/monitor` | `security` | Монитор событий AutoMarshal (real-time polling) |

### Структура .htm-файлов

```
url = "/partner/lk"       ← URL страницы
layout = "main"           ← Используемый layout
title = "Заголовок"
==
<?php
// PHP-код (onStart, обработчики AJAX-форм onLogin, onSubmit, …)
public function onStart() { ... }
public function onLogin() { ... }
?>
==
{# Twig-шаблон #}
{% partial 'header' %}
<div>{{ variable }}</div>
```

### Layouts

| Файл | Назначение |
|---|---|
| `layouts/main.htm` | Основной layout кабинета партнёра (с header/footer) |
| `layouts/auth.htm` | Layout страницы авторизации (без меню) |
| `layouts/security.htm` | Layout монитора охраны |

### Аутентификация на фронте

- Хранится в **PHP-сессии** (`Session::put('partner_user', ...)`)
- Проверяется в `onStart()` каждой страницы: если нет сессии — `Redirect::to('/partner/auth')`
- Пользователи — модель `PartnerUser` (таблица в БД), NOT backend-пользователи

### JS-зависимости темы

- **AG Grid Enterprise** — таблицы для партнёрского кабинета и монитора охраны (`themes/blank/assets/js/ag/`)
- Tailwind CSS (utility-классы в разметке)
- OctoberCMS flash-сообщения (`oc.flashMsg(...)`)

---

## 9. Консольные команды и планировщик

### Зарегистрированные команды

| Команда | Класс | Описание |
|---|---|---|
| `partners:parking-sessions` | `Platform\Partners\Console\GetAbonementSessions` | Получение сессий парковки для партнёров |
| `partners:event_request` | `Platform\Partners\Console\EventRequestMail` | Рассылка уведомлений о мероприятиях |
| `security:parking-sessions` | `Security\Parking\Console\GetEmployeeSessions` | Получение сессий паркинга для сотрудников |
| `telegram:polling` | `Platform\Notifications\Console\TelegramPolling` | Long polling Telegram Bot |
| `common:dump_db` | `Platform\Common\Console\CreateDbDump` | Дамп базы данных |

### Планировщик (cron)

В `app/Console/Kernel.php` / через `registerSchedule()` в плагинах:

| Команда | Расписание |
|---|---|
| `partners:parking-sessions` | Каждый час |
| `security:parking-sessions` | Каждый час |

Для работы планировщика в cron должна быть запись:
```
* * * * * php /path/to/project/artisan schedule:run
```

### Telegram Bot (Supervisor)

Файл `supervisor-tg-bot.conf` — конфиг Supervisor для запуска `telegram:polling` как демона.  
Watchdog-скрипт: `scripts/artisan_telegram_watchdog.sh`

---

## 10. Внешние интеграции и сервисы

| Сервис | Конфиг | Назначение |
|---|---|---|
| **Traveline** (TL) | `Hotel::tl_id` | Онлайн-бронирование номеров. ID хранится в `tl_id` у каждого отеля |
| **DaData** | `services.dadata.token` (.env: `DADATA_TOKEN`) | Автодополнение адресов в backend-формах |
| **Telegram Bot** | `services.telegram.*` (.env: `TELEGRAM_BOT_TOKEN`) | Уведомления партнёрам и охране |
| **Exchange (EWS)** | `config/ews-mail-server.php` | Отправка email через Microsoft Exchange |
| **RabbitMQ** | `app/Services/RabbitMqPublisher.php` | Очередь событий |
| **AutoMarshal** | `plugins/platform/api/http/controllers/ParkingController.php` | Система распознавания номеров (паркинг) |
| **Restaurant Connector** | `services.restaurant_connector.*` | Внешняя система ресторанных данных |
| **n8n** | `/api/v4/n8n/partner/user/get` | Интеграция с n8n (автоматизация) через статический salt |

---

## 11. Конфигурация окружения (.env)

Основные переменные для корректной работы:

```env
# База данных
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_database
DB_USERNAME=your_user
DB_PASSWORD=your_password

# October CMS
APP_KEY=base64:...
APP_URL=https://your-domain.com
APP_ENV=production

# Telegram Bot
TELEGRAM_BOT_TOKEN=...
TELEGRAM_BOT_USERNAME=...
TELEGRAM_BOT_SALT=...

# Уведомления
NOTIFICATION_SALT=...           # Используется как Bearer-токен для /api/v1/notification/send

# DaData (автодополнение адресов в backend)
DADATA_TOKEN=...
DADATA_SECRET=...

# Ресторанная интеграция
RESTAURANT_CONNECTOR_URL=...
RESTAURANT_CONNECTOR_TOKEN=...
```

---

## 12. Быстрый справочник: «что и где менять»

### Хочу изменить поля формы в backend (например, добавить поле к отелю)

→ Открыть: `plugins/platform/hotels/models/hotel/fields.yaml`  
→ Добавить поле в нужный таб по образцу существующих

### Хочу изменить колонки в таблице-списке

→ Открыть: `plugins/platform/hotels/models/hotel/columns.yaml`

### Хочу изменить меню в левом сайдбаре backend

→ Открыть `Plugin.php` нужного плагина → метод `registerNavigation()`

### Хочу добавить нового партнёра

→ Backend → **Партнёры** → кнопка «Создать»  
→ Заполнить карточку: название, тип, описание, логотип, обложка, контакты, объекты (вкладка «Объекты»)  
→ Создать пользователя для фронт-кабинета: вкладка «Пользователи» → добавить `PartnerUser` с логином/паролем

### Хочу добавить отель

→ Backend → **Отели** → «Добавить отель»  
→ Заполнить название, звёзды, описание, медиа, удобства  
→ Привязать к партнёру: Backend → **Партнёры** → карточка партнёра → вкладка «Объекты» → добавить объект типа Hotel

### Хочу добавить/изменить новость или мероприятие

→ Backend → **Публикации** → «Публикации (новости, мероприятия)»  
→ Поле `Тип публикации` определяет, куда попадает запись: `news` → новость, `event` → мероприятие

### Хочу изменить страницу блога/SEO

→ Backend → **Публикации** → «Страницы (блог, SEO)»

### Хочу добавить/изменить предложение партнёра (услугу)

→ Backend → **Предложения партнёров**  
→ Категории услуг: **Справочники** → «Категории предложений партнеров»

### Хочу изменить шаблон уведомления (email/telegram)

→ Backend → **Уведомления** (если есть пункт в меню для вашей роли) → `MessageTemplates`  
→ Либо напрямую: `plugins/platform/notifications/models/messagetemplate/`

### Хочу добавить API-эндпоинт

→ Открыть `plugins/platform/api/routes.php` → добавить маршрут в блок `Route::group`  
→ При необходимости создать контроллер в `plugins/platform/api/http/controllers/`

### Хочу изменить страницу партнёрского кабинета (фронт)

→ Открыть `themes/blank/pages/partner-*.htm`  
→ PHP-логика — в блоке `<?php ... ?>` между первыми `==`  
→ HTML/Twig — после вторых `==`

### Хочу добавить парковочного пользователя

→ Backend → **Парковка** → «Пользователи» → создать / импортировать через Excel (есть ImportExport)

### Где находятся миграции

→ `plugins/{vendor}/{plugin}/updates/` — PHP-файлы миграций  
→ `plugins/{vendor}/{plugin}/updates/version.yaml` — порядок применения  
→ Команда: `php artisan october:migrate`

### Как запустить локальный сервер

```bash
composer run dev
# или
php artisan serve
# или через Docker:
docker compose up
```

---

*Документ создан на основе анализа кода проекта. Актуален для состояния репозитория на май 2026 г.*
