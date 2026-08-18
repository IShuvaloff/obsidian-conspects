# Frontend Architecture — Interview Q&A

> Архитектурный ответ: **контекст → требования → варианты → trade-offs → решение**.
>
> Здесь важнее показать ход мысли, чем назвать «единственно правильную» технологию.

---

## 1. С чего начинать проектирование frontend-архитектуры?

Не со структуры папок и библиотек. Сначала выяснить:

- кто пользователи и ключевые сценарии;
- новый проект или legacy;
- SPA/SSR/hybrid, нужен ли SEO;
- какие backend API уже есть;
- auth, roles, permissions;
- где находится бизнес-логика: thin или thick client;
- есть ли mobile и общие контракты;
- performance requirements;
- размер команды, релизы, CI/CD.

**Interview:** архитектура начинается с требований и ограничений.

![[Pasted image 20260818221652.png|500]]

---

## 2. Какие вопросы задать перед проектированием?

### Бизнес
- кто пользователь;
- какие сценарии критичны;
- какие ошибки наиболее дорогие.

### Frontend
- SPA/SSR/hybrid;
- SEO;
- desktop/mobile;
- насколько сложный client state.

### Backend/API
- REST/GraphQL/RPC;
- кто владеет API-контрактом;
- насколько API стабилен;
- нужен ли realtime.

### Security
- authentication;
- authorization;
- roles/permissions.

### Delivery

**Delivery (software delivery)** — весь процесс доставки изменений от кода разработчика до production:

```text
code
→ review
→ tests/typecheck/lint
→ build
→ CI/CD
→ deploy/release
→ monitoring
```

**Release cadence** — частота/ритм релизов: например, несколько раз в день, ежедневно, раз в неделю или крупными релизами раз в месяц.

Уточнить:

- размер и устройство команды;
- tests/typecheck/lint;
- CI/CD;
- как часто происходят релизы;
- есть ли preview/staging;
- rollback;
- monitoring.

---

## 3. Thin client vs thick client

### Thin client

Frontend в основном:

- рисует UI;
- собирает input;
- вызывает API;
- показывает результат.

Критичная бизнес-логика — на backend.

### Thick client

Frontend содержит больше:

- business rules;
- local state;
- cache;
- optimistic updates;
- сложных workflows.

**Optimistic update** — клиент меняет UI/local state сразу, ещё до подтверждения backend, предполагая успех операции.

```text
user clicks
→ UI сразу изменился
→ request отправился
→ success: ничего не делаем
→ error: rollback + показываем ошибку
```

Например, toggle, like, reorder. Для критичных финансовых/необратимых операций применять осторожно.

**Хорошая формулировка:** критичные инварианты лучше оставлять backend source of truth, особенно если есть несколько клиентов: web/mobile/API.

---

## 4. Где должна находиться бизнес-логика?

### Backend — source of truth для

- permissions;
- финансовых расчётов;
- тарифных ограничений;
- переходов состояния;
- критичных validation rules.

### Frontend — для

- UI-state;
- presentation logic;
- client validation ради UX;
- wizard/form flow;
- optimistic UI.

> Frontend validation улучшает UX, backend validation обеспечивает корректность системы.

**UX (User Experience, пользовательский опыт)** — насколько пользователю понятно, быстро и удобно достигать цели в интерфейсе. «Улучшить UX» означает, например: уменьшить число лишних шагов, быстрее давать обратную связь, показывать понятные ошибки, не терять введённые данные, ускорять интерфейс и делать поведение предсказуемым.

---

## 5. REST, RPC или GraphQL?

### REST

Resource-oriented CRUD:

```text
GET /users/42
POST /orders
PATCH /profile
```

Прост, понятен, зрелая инфраструктура.

### RPC

Business operations:

```text
POST /orders/42/cancel
POST /hosting/migrate
POST /invoice/recalculate
```

API выражает операции, а не только ресурсы.

### GraphQL

Полезен, когда:

- клиентам нужны разные срезы данных;
- много связанных сущностей;
- overfetching/underfetching реально мешают.

**Правило:** не выбирать технологию только потому, что она модная.

---

## 6. Polling, SSE или WebSocket?

### Polling

```text
GET /report/status
GET /report/status
GET /report/status
```

Хорош, если обновления нечастые и задержка допустима.

### SSE

```text
server → client
```

Для:

- progress;
- logs;
- notifications;
- status stream.

### WebSocket

```text
client ↔ server
```

Для постоянного двунаправленного realtime:

- chat;
- collaborative editing;
- games;
- trading UI.

> Выбираю самую простую технологию, которая удовлетворяет realtime requirements.

---

## 7. Как организовать большой frontend?

Не только по техническим папкам:

```text
components/
stores/
services/
utils/
```

а группировать код по **feature/domain boundaries** — бизнес-функциям и предметным областям.

Например:

```text
modules/
  hotels/
    components/
    composables/
    pages/
    store.ts
    utils/
    index.ts

  booking/
    components/
    composables/
    pages/
    store.ts
```

Это уже можно назвать **feature/domain-oriented architecture**, если модуль действительно объединяет код одной бизнес-области, а не просто является случайной папкой.

> Код, который меняется по одной бизнес-причине, желательно держать рядом.

### Если используется терминология Feature-Sliced Design (FSD)

В FSD слои имеют более конкретный смысл:

```text
entities/
  user/
  tariff/
  server/

features/
  login/
  change-tariff/
  delete-server/

shared/
  ui/
  api/
  lib/
```

**Entity (сущность)** — базовое бизнес-понятие предметной области: `User`, `Tariff`, `Server`, `Hotel`. Внутри могут находиться типы/модель сущности, API для неё, небольшой entity-specific UI и helpers.

**Feature (фича)** — пользовательское действие/use-case над сущностями: `login`, `book-hotel`, `change-tariff`, `delete-user`.

**Shared** — переиспользуемый код без знания конкретного бизнеса: `Button`, `Modal`, HTTP client, formatters, generic utilities.

То есть:

```text
Entity
→ "что это за бизнес-объект?"

Feature
→ "что пользователь с ним делает?"

Shared
→ "общий технический/визуальный инструмент"
```

---

## 8. Что такое feature-based architecture?

Вместо глобальных technical folders код группируется по функциональности или бизнес-модулям:

```text
hotels/
  components/
  composables/
  pages/
  store.ts
  utils/
  index.ts
```

Если проект устроен как:

```text
modules/
  Hotels/
    components/
    pages/
    composables/
    store.ts
    utils/
    index.ts
```

это вполне можно назвать **feature-based / domain-oriented modular architecture**, потому что всё, относящееся к Hotels, локализовано внутри одного бизнес-модуля.

Важно не название папки `modules`, а принцип: **код группируется по причине изменения и бизнес-области**.

Плюсы:

- проще искать код feature/domain;
- проще менять или удалять модуль;
- понятнее ownership;
- меньше скрытых зависимостей.

---

## 9. Separation of concerns

Пример границ:

```text
Component
→ presentation + user interaction

Composable
→ reusable Vue/application logic

Store
→ shared client state

API layer
→ communication

Backend
→ authoritative business rules
```

Цель — понятные responsibilities, а не максимальное количество файлов.

---

## 10. Что положить в Pinia/store, а что оставить local?

### Store

Если state:

- нужен независимым компонентам;
- должен переживать route change;
- представляет application/session state.

Примеры:

```text
current user
permissions
cart
global filters
notifications
```

### Local state

```text
modal opened
selected tab
input value
temporary form state
```

> Не класть state в global store только потому, что это возможно.

---

## 11. Server state vs client state

### Server state

```text
users
orders
tariffs
hosting accounts
```

Требует:

- loading;
- caching;
- invalidation;
- refetch;
- stale handling.

### Client state

```text
opened modal
active tab
sidebar collapsed
draft filters
```

Не стоит превращать store в копию backend database.

---

## 12. Как организовать API layer?

Не размазывать `$fetch/fetch` по компонентам.

```text
component
↓
composable/store
↓
API module
↓
HTTP
```

Пример:

```ts
export function getUser(id: number) {
  return $fetch<User>(`/api/users/${id}`)
}
```

Плюсы:

- единые contracts;
- auth headers;
- error mapping;
- testing;
- backend changes локализуются.

---

## 13. DTO vs domain model

Полезно разделять, если backend shape неудобен frontend.

Backend:

```ts
type UserDto = {
  first_name: string
  created_at: string
}
```

Frontend:

```ts
type User = {
  firstName: string
  createdAt: Date
}
```

```text
DTO
↓
mapper
↓
domain model
```

Для простого CRUD это иногда лишняя прослойка.

---

## 14. Что делать с нестабильным API?

**Нестабильный API** — API, контракт которого часто или непредсказуемо меняется: названия/типы полей, структура response, endpoints, semantics, error format. В результате frontend регулярно ломается или вынужден подстраиваться.

Минимизировать распространение backend-specific shape:

```text
API response
↓
adapter/mapper
↓
frontend model
```

Также полезны:

- versioning;
- OpenAPI/generated contracts;
- contract tests;
- согласованный процесс изменения API.

---

## 15. Как обрабатывать API errors?

Разделять:

```text
transport
→ timeout, 500, 502

auth
→ 401, 403

validation
→ field errors

business
→ operation prohibited
```

Не сводить всё к:

```ts
catch {
  alert('Something went wrong')
}
```

Нужен baseline централизованной обработки + domain-specific UI.

---

## 16. Где loading/error/empty state?

Обычно рядом с feature, владеющей запросом.

Reusable list может рисовать shell:

```text
loading
empty
error placeholder
```

Но domain-specific message лучше формировать выше.

---

## 17. Как избегать огромных компонентов?

Если компонент одновременно:

- делает много API calls;
- содержит business rules;
- управляет form;
- рисует table;
- мутирует global state;

разделить:

```text
page/container
→ orchestration

composable
→ logic

presentational components
→ rendering

API module
→ requests
```

---

## 18. Когда нужен composable?

Когда есть reusable logic на Vue Composition API:

```text
usePagination
usePermissions
useInjectedUser
useAsyncData
useDebounce
```

Composable не обязан возвращать UI.

---

## 19. Composable vs utility

### Utility

```ts
formatMoney(100)
```

Не зависит от Vue reactivity/lifecycle.

### Composable

Использует:

```text
ref
computed
watch
inject
onMounted
```

Например `useInjectedUser()` логичнее composable, потому что внутри `inject()`.

---

## 20. Как проектировать reusable component?

Определить минимальный public API:

```text
props
events
slots
v-model
```

Не превращать component в набор из 30 boolean props.

Если rendering сильно меняется, чаще лучше composition/slot.

---

## 21. Когда использовать slot?

Когда component контролирует structure/behavior, но rendering части UI должен определить parent.

```vue
<GenericList :items="users">
  <template #item="{ item }">
    <UserCard :user="item" />
  </template>
</GenericList>
```

List отвечает за:

```text
layout
pagination
loading
selection
```

Parent — за карточку.

---

## 22. Как не переусложнить reusable component?

Спросить:

> Это действительно одна ответственность или просто похожий CSS?

Если совпадает только grid — возможно достаточно layout component.

Если совпадают:

```text
pagination
loading
selection
sorting
empty state
```

generic list уже оправдан.

---

## 23. Coupling и cohesion

### Coupling — связанность / степень зависимости модулей

Насколько сильно один модуль зависит от деталей другого.

Желательно:

```text
low coupling
→ низкая связанность между модулями
```

### Cohesion — связность / внутренняя целостность модуля

Насколько элементы одного модуля относятся к одной ответственности и логически принадлежат друг другу.

Желательно:

```text
high cohesion
→ высокая внутренняя связность
```

Хорошая архитектура:

```text
high cohesion
+
low coupling
```

По-русски удобно говорить: **«высокая внутренняя связность и низкая связанность между модулями»**.

---

## 24. Dependency direction

Высокоуровневая бизнес-логика не должна хаотично зависеть от UI details.

Упрощённо:

```text
UI
↓
application/domain abstraction
↓
infrastructure/API
```

Не обязательно строить формальную Clean Architecture, но dependency boundaries должны быть понятны.

---

## 25. Clean Architecture на frontend?

Полезны принципы:

- **separation of concerns** — разделение ответственности;
- **dependency boundaries** — чёткие границы зависимостей между слоями/модулями;
- **dependency direction** — контролируемое направление зависимостей;
- **testability** — тестируемость;
- **isolation of domain logic** — изоляция доменной/бизнес-логики от UI и инфраструктурных деталей.

**Boundary (граница)** — место, где мы специально ограничиваем прямую связанность двух частей системы и заставляем взаимодействовать их через понятный контракт/API.

Но механический перенос backend Clean Architecture в небольшой frontend может стать overengineering.

> Использовать принципы, а не религию.

---

## 26. Что такое overengineering?

Когда сложность решения выше сложности задачи.

Например для 3 CRUD pages без причин вводить:

```text
event bus
CQRS
DI container
microfrontends
```

Коротко:

- **Event Bus (шина событий)** — общий механизм publish/subscribe: одна часть приложения публикует событие, другие подписчики реагируют без прямого вызова друг друга.
- **CQRS (Command Query Responsibility Segregation)** — разделение операций изменения состояния (`commands`) и чтения данных (`queries`) на разные модели/пути.
- **DI Container (Dependency Injection Container, контейнер внедрения зависимостей)** — реестр/механизм, который создаёт объекты и автоматически передаёт им нужные зависимости.
- **Microfrontends (микрофронтенды)** — разбиение frontend на несколько относительно автономных приложений/частей, которыми могут владеть и независимо поставлять разные команды.

Хорошая архитектура уменьшает стоимость изменений.

---

## 27. Когда вводить abstraction?

Проверить:

1. одинаковая ли ответственность;
2. меняется ли код по одной причине;
3. одинаковы ли требования;
4. abstraction проще duplication?

Иногда три похожие функции лучше одной generic-функции с 10 flags.

---

## 28. SPA, SSR или hybrid?

### SPA

Для:

- internal dashboards;
- control panels;
- authenticated apps;
- SEO не нужен.

### SSR

Для:

- SEO;
- fast first content;
- public pages;
- social previews.

### Hybrid

В Nuxt часто естественно:

```text
public pages → SSR/SSG
control panel → client-heavy
```

---

## 29. Что учитывать при SSR?

Нельзя бездумно использовать:

```ts
window
document
localStorage
```

Код может выполняться на сервере.

Также:

- hydration;
- duplicated requests;
- cookies/auth;
- server/client state consistency.

---

## 30. Hydration

**Коротко для интервью:** hydration — это процесс, когда Vue на клиенте «оживляет» уже отрендеренный сервером HTML: привязывает к нему reactive state и event handlers, не создавая всю разметку заново.

```text
SSR HTML
↓
browser
↓
Vue attaches reactivity/events
↓
interactive app
```

Hydration mismatch — server и client сгенерировали разную разметку.

---

## 31. Authentication vs authorization

```text
authentication
→ кто пользователь?

authorization
→ что ему можно?
```

Frontend может скрыть кнопку ради UX:

```vue
<button v-if="canDelete">
```

Но backend обязан повторно проверить permission.

> Frontend не является security boundary.

---

## 32. Как хранить permissions?

Типичный вариант: после authentication backend возвращает пользователя и его права, а frontend хранит их в session/auth store:

```ts
permissions: [
  'users.read',
  'users.delete'
]
```

Над ними делается единая функция:

```ts
function can(
  permission: Permission
): boolean {
  return permissions.includes(
    permission
  )
}
```

После этого в любой части приложения можно использовать:

```ts
can('users.delete')
```

или composable/computed-обёртку.

Это лучше, чем размазывать:

```ts
if (role === 'admin')
```

по компонентам.

**Важно:** frontend `can()` нужен для UX. Реальную authorization backend обязан проверить повторно.

---

## 33. Role-based UI

Плохо:

```ts
if (user.role === 'admin')
```

везде.

Лучше:

```ts
const canDeleteUser = computed(
  () => permissions.can('users.delete')
)
```

UI зависит от capability.

---

## 34. Feature flags

**Feature flag (флаг функциональности)** — переключатель, позволяющий включать/выключать новую функцию без отдельной сборки приложения.

Например:

```ts
featureFlags.isEnabled(
  'newBilling'
)
```

Зачем:

- выкатывать новую функцию постепенно;
- включать только части пользователей;
- делать A/B/эксперименты;
- быстро выключить проблемную feature;
- деплоить код раньше, чем показывать функцию всем.

Полезны:

- typed keys;
- централизованный доступ;
- ownership;
- удаление старых flags после rollout.

---

## 35. Performance approach

Сначала измерить, потом оптимизировать.

Смотреть:

- bundle size;
- rendering;
- network waterfall;
- duplicate requests;
- large lists;
- unnecessary watchers;
- expensive computed.

Инструменты:

```text
code splitting
lazy loading
virtualization
caching
debounce/throttle
memoization
```

---

## 36. Когда virtual list?

**Virtual list / виртуализированный список** нужен, когда данных очень много, но пользователь одновременно видит только небольшую часть.

Без virtualization:

```text
10000 records
→ 10000 DOM nodes/rows
```

С virtualization:

```text
10000 records в данных
↓
на экране видно ~20
↓
в DOM реально держим, например, ~30–40 rows
```

При прокрутке старые DOM-строки переиспользуются/заменяются новыми.

Это уменьшает:

- стоимость создания DOM;
- memory usage;
- layout/paint work.

То есть число **данных** может быть 10 000, а число реально **отрисованных DOM-элементов** — лишь несколько десятков.

Для 20 элементов virtualization обычно лишняя сложность.

---

## 37. Code splitting

Не грузить весь JS одним bundle.

```text
main
+
billing chunk
+
admin chunk
```

Часто реализуется через **dynamic/lazy import**:

```ts
const module =
  await import(
    './heavy-module'
  )
```

Для Vue-компонента:

```ts
const HeavyChart =
  defineAsyncComponent(
    () =>
      import(
        './HeavyChart.vue'
      )
  )
```

Router/Nuxt также умеют автоматически создавать отдельные chunks для страниц/routes.

**Смысл:** код загружается только тогда, когда он реально понадобится пользователю.

Nuxt/Vite многое делают автоматически, но imports всё равно влияют на bundle graph.

---

## 38. Caching

Trade-off:

```text
speed
vs
freshness + complexity
```

Нужно определить:

- TTL;
- invalidation;
- stale behavior;
- ownership.

**Stale behavior** — что приложение делает с устаревшими (stale) данными в кеше. Например:

```text
показать старые данные сразу
+
в фоне запросить свежие
```

или:

```text
не показывать stale data
→ сначала дождаться refetch
```

Самая сложная часть cache обычно — invalidation.

---

## 39. Optimistic UI

Client показывает ожидаемый результат до server response.

Хорошо:

```text
like
toggle
simple reorder
```

Осторожно:

```text
payments
billing
irreversible actions
```

Нужен rollback.

---

## 40. Архитектура forms

Разделять:

```text
form UI state
validation
DTO mapping
submit
server errors
```

Client validation → UX.

Server validation → source of truth.

---

## 41. Сложный wizard

Если много состояний и переходов, лучше явно моделировать state:

```text
idle
→ editing
→ validating
→ submitting
→ success/error
```

вместо набора независимых booleans.

---

## 42. Loading/error state model

### Простой вариант

```ts
{
  loading: boolean
  data: T | null
  error: Error | null
}
```

### Строгий вариант

```ts
type State<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error }
```

Второй запрещает противоречивые состояния.

---

## 43. Testing architecture

### Unit

```text
pure functions
composables
domain logic
```

### Component

```text
render
events
behavior
```

### Integration

```text
feature + mocked API
```

### E2E

```text
critical user flows
```

Не всё тестировать через E2E: они дорогие и медленные.

---

## 44. Что покрывать E2E?

Критичные сценарии:

```text
login
checkout/payment
tariff change
critical settings
main business workflows
```

Не каждую мелкую кнопку.

---

## 45. Observability frontend

Нужны:

```text
error monitoring
performance metrics
request correlation
release/version identification
frontend events/logs
```

Полезно знать:

```text
какая версия frontend
какой route
какой endpoint
какая ошибка
```

---

## 46. Logging errors

Baseline централизуется:

```text
global error handler
HTTP layer
monitoring SDK
```

Domain-specific ошибки должны оставаться понятными пользователю.

---

## 47. Работа с legacy

Не начинать с rewrite.

1. Найти самые дорогие проблемы.
2. Зафиксировать critical behavior тестами.
3. Создавать **boundaries (границы)** вокруг legacy.
4. Постепенно вытаскивать features.
5. Уменьшать coupling.

**Создать boundary вокруг legacy** — значит перестать позволять новому коду напрямую зависеть от внутренностей старого модуля и поставить между ними понятный интерфейс/adapter.

```text
new code
↓
adapter / facade
↓
legacy module
```

Тогда внутренности legacy можно постепенно менять, не ломая весь новый код.

Rewrite — только если цена миграции действительно ниже эволюции.

---

## 48. Tech debt

Tech debt лучше описывать через бизнес-стоимость.

Не:

> Код некрасивый.

А:

> Из-за этого модуля каждая feature занимает на два дня больше и ломает соседние экраны.

Стратегии простыми словами:

- **Boy Scout Rule / правило бойскаута** — оставлять код немного лучше, чем он был до твоего изменения;
- **refactor вокруг активных features** — улучшать проблемный код тогда, когда команда всё равно меняет эту область ради новой задачи;
- **debt backlog** — вести явный список технического долга с приоритетами;
- **ownership** — понимать, какая команда/разработчик отвечает за конкретный модуль и его состояние;
- **metrics** — измерять последствия долга: частоту багов, время разработки, сложность изменений, performance и т.п.

---

## 49. Если разработчики спорят об архитектуре

Перевести спор в критерии:

```text
complexity
performance
maintainability
team familiarity
migration cost
testability
delivery time
```

Если оба решения нормальные — часто выбрать более простое и знакомое команде.

---

## 50. Как аргументировать решение?

Шаблон:

```text
Есть requirement X.

Варианты A/B/C.

A проще, но...
B сложнее, зато...

Выбираю B, потому что...

Если requirement изменится,
решение можно пересмотреть.
```

Сильнее, чем:

> Это best practice.

---

## 51. Source of truth

Авторитетное место состояния.

Пример:

```text
backend
→ tariff/balance/permissions

Pinia
→ client session

URL
→ shareable filters/page
```

Проблема — когда одна сущность независимо хранится в нескольких местах.

---

## 52. Filter state в URL?

Полезно, если состояние должно:

- переживать refresh;
- работать с back/forward;
- быть shareable.

```text
/catalog?page=3&sort=price&status=active
```

Для временного modal state URL обычно не нужен.

---

## 53. Duplicate state

Плохо:

```text
route.query.page = 3
store.page = 3
component.page = 3
```

три независимых source of truth.

Лучше выбрать один главный источник, остальные derive/sync контролируемо.

---

## 54. Derived state

Если значение можно вычислить, часто не надо хранить отдельно.

```ts
const fullName = computed(
  () => `${user.firstName} ${user.lastName}`
)
```

Иначе появляется synchronization problem.

---

## 55. Duplication vs плохая abstraction

Иногда:

```text
немного duplication
<
abstraction с 12 flags
```

Abstraction должна объединять общую ответственность, а не просто похожий код.

---

## 56. Component API design

Хороший API:

```text
minimal
predictable
typed
composable
```

Пример:

```vue
<DataTable
  :rows="rows"
  v-model:selected="selected"
  @sort="onSort"
>
  <template #row="{ row }">
    ...
  </template>
</DataTable>
```

---

## 57. Controlled vs uncontrolled component

### Controlled

State у parent:

```vue
<MySelect v-model="selected" />
```

### Uncontrolled

Component сам хранит state.

Controlled удобнее, если значение важно другим частям приложения.

---

## 58. Когда provide/inject?

Для dependency, естественно принадлежащей subtree:

```text
form context
tabs context
theme
feature context
```

Но provide/inject не должен становиться скрытым global store.

---

## 59. Provide/inject vs Pinia

### Provide/inject

Scoped dependency:

```text
Form
└─ nested fields
```

### Pinia

Application-wide/shared state:

```text
user
auth
global notifications
```

Если dependency имеет естественного parent-provider, provide/inject часто лучше global store.

---

## 60. Как mobile влияет на architecture?

Для web + mobile важно не дублировать authoritative business rules.

Общие вещи:

- API contracts;
- auth;
- permissions;
- backend validation;
- domain semantics.

Presentation остаётся client-specific.

---

## 61. Кто владеет API-контрактом?

Нужно определить процесс:

- кто инициирует изменения;
- versioning;
- как frontend узнаёт об изменении;
- OpenAPI/schema;
- contract tests.

Плохо:

```text
backend changed JSON
↓
frontend noticed in production
```

---

## 62. Generated API client

При стабильной OpenAPI/schema generation даёт:

- typed DTO;
- меньше boilerplate;
- synchronized contracts.

Но generated client не отменяет domain layer и runtime trust boundary автоматически.

---

## 63. Что спросить про CI/CD?

```text
tests
typecheck
lint
build
preview environment
rollback
feature flags
release cadence
```

Delivery constraints тоже влияют на архитектуру.

---

## 64. Release cadence и architecture

**Release cadence** — частота и ритм поставки новых версий пользователям.

Например:

```text
continuous / несколько раз в день
daily
weekly
monthly major release
```

Частые релизы требуют:

- small changes;
- feature flags;
- backward-compatible API;
- хорошей автоматизации tests/CI/CD.

Редкие большие релизы увеличивают integration risk.

---

## 65. Microfrontends — когда нужны?

Обычно не из-за размера кода, а из-за организационных границ:

- несколько автономных команд;
- independent deploy;
- отдельный ownership.

Цена:

```text
runtime integration
shared dependencies
UX consistency
routing
deployment complexity
```

Для одной команды часто overengineering.

---

## 66. Monorepo

**Monorepo (монорепозиторий)** — один Git-репозиторий, в котором живут несколько приложений и/или библиотек.

Например:

```text
repo/
  apps/
    web/
    admin/
    mobile-web/
  packages/
    ui/
    api-client/
    types/
```

### Плюсы

- shared packages;
- atomic changes сразу в нескольких проектах;
- common tooling;
- easier refactoring.

### Минусы

- tooling complexity;
- дорогой CI без оптимизации;
- ownership надо поддерживать дисциплиной.

---

## 67. Что выносить в shared?

**`shared/`** — условная директория для общего переиспользуемого кода, который не принадлежит конкретной бизнес-фиче/сущности.

Хорошо:

```text
shared/
  ui/
    Button
    Modal
  api/
    httpClient
  lib/
    formatMoney
  types/
```

То есть:

```text
Button
Modal
API primitives
formatters
generic types
```

Опасно:

```text
shared/business/
```

куда складывают всё подряд. Если код знает о `HotelBooking`, `TariffChange` или другой конкретной бизнес-области, он, скорее всего, уже не truly shared.

---

## 68. Как определить feature boundary?

По бизнес-ответственности:

```text
billing
hosting
domains
auth
```

а не только по техническому типу:

```text
tables
forms
requests
```

---

## 68.1. Что такое DDD?

**DDD = Domain-Driven Design (предметно-ориентированное проектирование).**

Идея: архитектура и код строятся вокруг бизнес-предметной области и её языка, а не только вокруг технических слоёв.

Например, вместо абстрактных:

```text
services/
managers/
helpers/
```

в коде явно появляются бизнес-понятия:

```text
Booking
Hotel
Tariff
Invoice
Domain
HostingAccount
```

Ключевые идеи DDD:

- **domain** — предметная область бизнеса;
- **domain model** — модель бизнес-понятий и правил;
- **ubiquitous language** — единый язык терминов между разработчиками и бизнесом;
- **bounded context** — граница, внутри которой термины и модель имеют однозначный смысл.

Для frontend не обязательно применять «полный DDD». Полезнее сама идея: строить модули вокруг понятных бизнес-доменов и не смешивать их правила.

---

## 69. Как отвечать на architecture/system design question?

Сначала уточнить:

```text
кто пользователь?
какие сценарии?
масштаб?
SSR/SEO?
realtime?
backend/API?
auth?
team?
performance?
```

Потом назвать решение и trade-offs.

---

## 70. Полезная interview-фраза

> Я бы сначала уточнил требования, потому что архитектурный выбор здесь зависит от нескольких факторов.

После этого задать 2–4 реально важных вопроса.

Это инженерный подход, а не уход от ответа.

---

# Быстрая карта архитектуры

```text
Requirements
↓
Business scenarios
↓
Data ownership
↓
API contracts
↓
State ownership
↓
Module boundaries
↓
Rendering strategy
↓
Performance
↓
Testing
↓
Delivery / monitoring
```

---

# Что где хранить

```text
Backend
→ authoritative business rules
→ permissions
→ financial/state invariants

API layer
→ HTTP communication
→ DTO/contracts
→ common error handling

Store
→ shared client/application state

Composable
→ reusable Vue logic

Component
→ UI + interaction

URL
→ shareable/navigation state

Local component state
→ temporary UI state
```

---

# Transport cheat sheet

```text
REST
→ resources / CRUD

RPC
→ business operations

Polling
→ periodic status checks

SSE
→ server → client stream

WebSocket
→ continuous bidirectional realtime

GraphQL
→ flexible client-selected data shape
```

---

# Сильная структура ответа

```text
1. Уточнить требования.
2. Назвать ограничения.
3. Предложить варианты.
4. Объяснить trade-offs.
5. Выбрать самый простой достаточный вариант.
6. Назвать условие, при котором решение пришлось бы изменить.
```

---

# Что НЕ делать на архитектурном интервью

Не начинать с:

> Я бы взял Pinia, Axios, FSD и WebSocket.

пока не ясны requirements.

Не говорить:

> Так принято.

Лучше:

> Выбираю это из-за такого-то ограничения.

Не применять автоматически:

```text
microfrontends
Clean Architecture
DDD
event bus
WebSocket
global store
```

> Архитектура — управление сложностью, а не максимизация количества паттернов.

---

# Финальная формулировка для интервью

> Для меня frontend-архитектура — это прежде всего управление границами ответственности и стоимостью изменений. Я сначала стараюсь понять бизнес-сценарии, ownership данных, API и требования к rendering/state, а уже потом выбираю структуру приложения и инструменты. Хорошая архитектура должна помогать команде быстрее и безопаснее менять продукт, а не просто соответствовать набору паттернов.
