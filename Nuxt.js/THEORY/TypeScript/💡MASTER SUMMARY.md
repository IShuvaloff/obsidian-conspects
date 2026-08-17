# TypeScript — Final Summary

Короткий конспект для быстрого повторения перед собеседованием.

---

## 1. `any`, `unknown`, `void`, `never`

```ts
let a: any
let b: unknown
```

- `any` отключает type safety;
- `unknown` требует narrowing перед использованием;
- `void` — функция нормально завершается, но полезного значения нет;
- `never` — нормального завершения/значения не существует.

```ts
function fail(message: string): never {
  throw new Error(message)
}
```

---

## 2. `type` vs `interface`

`interface` удобен для объектных контрактов, `extends`, declaration merging.

```ts
interface User {
  id: number
}
```

`type` умеет aliases, unions, intersections, tuples.

```ts
type UserId = number
type Status = 'loading' | 'success'
type Pair = [number, string]
```

---

## 3. Union и Intersection

```ts
type A = User | Admin
```

`A` должен быть совместим хотя бы с одним вариантом.

```ts
type B = User & Admin
```

`B` должен одновременно удовлетворять обоим контрактам.

---

## 4. Narrowing

Основные инструменты:

```ts
typeof
instanceof
in
null checks
discriminated unions
custom type guards
```

---

## 5. Discriminated union

```ts
type ApiState<T> =
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error }
```

```ts
if (state.status === 'success') {
  state.data
  // T
}
```

> [!INFO]- Почему это лучше объекта с nullable-полями
> Discriminated union запрещает невозможные состояния и даёт точное narrowing по discriminator.
> 
> ```ts
> type Result<T> =
>   | { success: true; data: T }
>   | { success: false; error: Error }
> ```
> 
> Объект вида:
> 
> ```ts
> {
>   success: boolean
>   data: T | null
>   error: Error | null
> }
> ```
> 
> проще для некоторых UI/reactive сценариев, но допускает противоречивые комбинации вроде `success: true` + `error`.

---

## 6. Custom type guard

```ts
function isUser(
  value: unknown
): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    typeof value.id === 'number' &&
    typeof value.name === 'string'
  )
}
```

`value is User` — **type predicate**.

---

## 7. Runtime safety

Вот это:

```ts
const user = data as User
```

не валидирует runtime-данные.

Для внешнего API:

```text
unknown
→ runtime validation
→ User
```

Использовать:

- custom guards;
- Zod;
- Valibot.

---

## 8. Generics

```ts
function identity<T>(value: T): T {
  return value
}
```

Главная идея: **сохранить связь типов**.

```ts
function pair<T, U>(
  first: T,
  second: U
): [T, U] {
  return [first, second]
}
```

---

## 9. Generic constraints

```ts
function foo<
  T extends { id: number }
>(value: T) {}
```

`T` обязан быть совместим с указанным контрактом.

---

## 10. `keyof` + `T[K]`

```ts
function getValue<
  T,
  K extends keyof T
>(
  obj: T,
  key: K
): T[K] {
  return obj[key]
}
```

- `K extends keyof T` — можно передать только существующий ключ;
- `T[K]` — результат связан с конкретным выбранным ключом.

```ts
getValue(user, 'name')
// string

getValue(user, 'id')
// number
```

---

## 11. Очень полезные шаблоны `typeof`

### Union значений массива

```ts
const statuses = [
  'loading',
  'success',
  'error'
] as const

type Status =
  typeof statuses[number]

// 'loading' | 'success' | 'error'
```

### Union ключей объекта

```ts
const roles = {
  admin: 100,
  editor: 50,
  viewer: 10
} as const

type Role =
  keyof typeof roles

// 'admin' | 'editor' | 'viewer'
```

### Union значений объекта

```ts
type Level =
  (typeof roles)[keyof typeof roles]

// 100 | 50 | 10
```

Короткое правило:

```ts
// array values
type X = typeof array[number]

// object keys
type K = keyof typeof object

// object values
type V =
  (typeof object)[keyof typeof object]
```

---

## 12. `as const`

```ts
const statuses = [
  'loading',
  'success'
] as const
```

Получаем примерно:

```ts
readonly [
  'loading',
  'success'
]
```

`as const`:

- делает literal structure readonly;
- сохраняет literal types;
- не расширяет `'loading'` до `string`.

---

## 13. Indexed access types

```ts
type Name = User['name']
```

```ts
type UserValues =
  User[keyof User]
```

`T[K]` — тип значения поля `K`.

---

## 14. Utility types

```ts
Partial<T>
Pick<T, K>
Omit<T, K>
Readonly<T>
Record<K, V>
Exclude<T, U>
Extract<T, U>
NonNullable<T>
ReturnType<T>
Parameters<T>
```

Пример:

```ts
type UserPatch =
  Partial<
    Pick<User, 'name' | 'email'>
  >
```

Результат:

```ts
{
  name?: string
  email?: string
}
```

Плюс производный тип остаётся связан с исходным `User`.

---

## 15. `Exclude` / `Extract`

```ts
type Permission =
  'read' | 'write' | 'delete'
```

```ts
type Safe =
  Exclude<
    Permission,
    'delete'
  >
// 'read' | 'write'
```

```ts
type Dangerous =
  Extract<
    Permission,
    'write' | 'delete'
  >
// 'write' | 'delete'
```

```text
Exclude → вычесть
Extract → оставить пересечение
```

---

## 16. `Parameters` / `ReturnType`

```ts
type Params =
  Parameters<
    (id: number, name: string) => boolean
  >
// [id: number, name: string]
```

```ts
type Result =
  ReturnType<
    (id: number, name: string) => boolean
  >
// boolean
```

---

## 17. Mapped types

```ts
type Nullable<T> = {
  [K in keyof T]: T[K] | null
}
```

```ts
type FeatureFlags<T> = {
  [K in keyof T]: boolean
}
```

```ts
type Getters<T> = {
  [K in keyof T]: () => T[K]
}
```

```ts
type FormErrors<T> = {
  [K in keyof T]?: string
}
```

Принцип:

```text
keyof T
→ пройти по каждому ключу
→ преобразовать тип значения
```

---

## 18. `Partial<T>` vs `Nullable<T>`

```ts
Partial<User>
```

Поле может отсутствовать:

```ts
{
  id?: number
}
```

```ts
Nullable<User>
```

Поле обязательно существует, но значение может быть `null`:

```ts
{
  id: number | null
}
```

---

## 19. Conditional types

```ts
type IsString<T> =
  T extends string
    ? true
    : false
```

```ts
type A = IsString<string>
// true

type B = IsString<number>
// false
```

`extends` здесь проверяет совместимость типов.

Не путать с generic constraint:

```ts
function foo<
  T extends Something
>() {}
```

---

## 20. `satisfies` vs annotation vs `as`

### Type annotation

```ts
const config: Config = {
  theme: 'light'
}
```

Проверяет контракт, переменная получает тип `Config`.

### `satisfies`

```ts
const config = {
  theme: 'light'
} satisfies Config
```

Проверяет контракт, но старается сохранить более узкий inferred type.

Хорошо для config maps, route maps, feature flags, словарей.

Нюанс: если значение должно потом мутировать от `'light'` к `'dark'`, слишком узкий inferred literal иногда неудобен.

### `as`

```ts
const config = {
  theme: 'light'
} as Config
```

Type assertion: «считай это Config».

Не runtime validation.

---

## 21. Structural typing

```ts
const employee = {
  id: 1,
  name: 'Alex',
  department: 'Frontend'
}

const user: User = employee
```

Допустимо, если обязательные поля `User` совместимы.

### Excess property checking

Fresh literal:

```ts
const user: User = {
  id: 1,
  name: 'Alex',
  department: 'Frontend'
}
```

может получить excess property error.

---

## 22. Function overloads

```ts
function parse(value: string): string[]
function parse(value: number): number[]

function parse(
  value: string | number
): string[] | number[] {
  // ...
}
```

Первые сигнатуры — overload signatures.

Последняя — **implementation signature**.

Важно:

> implementation signature не становится дополнительной публичной overload-сигнатурой.

---

## 23. Exhaustive checking

```ts
type Status =
  'loading'
  | 'success'
  | 'error'

switch (status) {
  case 'loading':
    break
  case 'success':
    break
  case 'error':
    break
  default: {
    const exhaustive: never = status
  }
}
```

Если добавить `'cancelled'` и забыть `case`, TypeScript не позволит присвоить `'cancelled'` в `never`.

---

## 24. Merge и `T & U`

```ts
function merge<
  T extends object,
  U extends object
>(
  left: T,
  right: U
): T & U {
  return {
    ...left,
    ...right
  }
}
```

При конфликтующих ключах runtime-semantics:

```ts
{
  ...left,
  ...right
}
```

означает: последнее значение побеждает.

А `T & U` может дать конфликтный тип вроде:

```ts
number & string
// never
```

Более точная модель overwrite-семантики:

```ts
Omit<T, keyof U> & U
```

---

# Vue / Nuxt + TypeScript — быстрый блок

## 25. `defineProps`

```ts
const props = defineProps<Props>()
```

Optional prop:

```ts
title?: string
// string | undefined
```

Defaults:

```ts
withDefaults(
  defineProps<Props>(),
  {
    title: 'Default'
  }
)
```

---

## 26. `defineEmits`

```ts
const emit = defineEmits<{
  save: [id: number]
  close: []
}>()
```

---

## 27. `defineModel`

```ts
const model = defineModel<string>()
```

Заменяет ручной `modelValue + update:modelValue`.

Несколько моделей:

```vue
<MyComponent
  v-model:title="title"
  v-model:count="count"
/>
```

```ts
const title = defineModel<string>('title')
const count = defineModel<number>('count')
```

---

## 28. `ref` / `reactive`

```ts
const user = ref<User | null>(null)
```

```ts
const state = reactive<State>({
  user: null,
  loading: false
})
```

Эвристика:

```text
ref
→ одиночные / replaceable values

reactive
→ объектное состояние
```

---

## 29. `computed`

```ts
const name = computed(
  () => user.value?.name ?? 'Guest'
)
```

Обычно inference достаточно: `ComputedRef<string>`.

Явный generic — для закрепления compile-time контракта.

---

## 30. `InjectionKey<T>`

```ts
export const userKey:
  InjectionKey<Ref<User | null>> =
    Symbol('user')
```

Использовать **тот же экземпляр Symbol** в `provide` и `inject`.

```ts
const user = inject(userKey)
```

Тип содержит `undefined`, потому что provider может отсутствовать.

Обязательную dependency удобно оборачивать composable:

```ts
function useInjectedUser() {
  const user = inject(userKey)

  if (!user) {
    throw new Error('User provider is missing')
  }

  return user
}
```

---

## 31. Typed slots

```ts
defineSlots<{
  item(props: {
    item: User
    index: number
  }): any
}>()
```

Child:

```vue
<slot
  name="item"
  :item="user"
  :index="index"
/>
```

Parent:

```vue
<template #item="{ item, index }">
  {{ item.name }}
</template>
```

Типизируется **slot payload**, а не markup, который рисует родитель.

---

## 32. Generic SFC

> SFC = Single File Component.

```vue
<script
  setup
  lang="ts"
  generic="T extends { id: number }"
>
```

Один `T` может связать:

```text
props.items    → T[]
slot.item      → T
emit payload   → T
v-model        → T | null
```

---

## 33. Universal generic list pattern

```vue
<script
  setup
  lang="ts"
  generic="T extends { id: number }"
>
defineProps<{
  items: T[]
}>()

const selected = defineModel<T | null>()

defineSlots<{
  item(props: {
    item: T
  }): any
}>()

const emit = defineEmits<{
  select: [item: T]
}>()
</script>
```

Если родитель передаёт `User[]`, то `T = User`, и TypeScript автоматически связывает slot item, emit payload и model с `User`.

---

## 34. Generic `K extends keyof T`

```vue
<script
  setup
  lang="ts"
  generic="T, K extends keyof T"
>
```

```ts
labelKey: K
```

гарантирует, что переданный ключ действительно существует в `T`.

```ts
item[labelKey]
```

имеет тип:

```ts
T[K]
```

---

## 35. Nuxt generic не валидирует runtime

```ts
useFetch<User>('/api/user')
```

означает compile-time контракт, но **не проверяет**, что backend реально вернул `User`.

Для внешних данных нужен runtime-validation layer.

---

# Финальные формулировки для интервью

### Generics

> Generics нужны не просто для универсальности, а для сохранения типовой связи между входами и выходами.

### `keyof` + `T[K]`

> `K extends keyof T` ограничивает ключ существующими ключами объекта, а `T[K]` сохраняет точную зависимость между выбранным ключом и типом значения.

### `unknown`

> `unknown` — безопасная альтернатива `any`: значение неизвестно, пока мы не сузим тип.

### Runtime validation

> TypeScript types стираются после компиляции, поэтому внешние данные нельзя считать безопасными только из-за generic или `as`.

### Discriminated union

> Хорошо моделирует взаимоисключающие состояния и позволяет компилятору запрещать невозможные комбинации.

### Generic Vue component

> Generic SFC позволяет одним `T` связать props, slots, emits и `v-model`, сохраняя type safety reusable-компонента без `any`.

---

# Дополнение: темы из всех 5 блоков, которые стоит держать в master-summary

## 36. Tuple и type alias

Tuple — массив фиксированной структуры:

```ts
const user: [number, string] = [42, 'Alex']
```

Type alias — новое имя существующего типа:

```ts
type UserId = number
type Email = string
```

Alias не создаёт новый runtime-тип.

---

## 37. Контексты `extends`

Одно слово используется в разных механизмах:

```ts
interface Admin extends User {}
// interface inheritance

function foo<T extends object>() {}
// generic constraint

function get<T, K extends keyof T>() {}
// K ограничен ключами T

type X<T> = T extends string ? true : false
// conditional type
```

Главное: в generic constraint `T extends X` означает «T должен быть совместим с X».

---

## 38. Trust Boundary

Внешние данные лучше считать недоверенными:

```text
API
localStorage
URL/query params
user input
third-party SDK
```

Модель:

```text
external data
↓
unknown
↓
runtime validation
↓
trusted domain type
```

---

## 39. Generic `fetchData<T>` с runtime guard

Сам generic API не валидирует:

```ts
async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url)
  return response.json()
}
```

Безопаснее передать настоящий runtime-validator:

```ts
async function fetchData<T>(
  url: string,
  guard: (data: unknown) => data is T
): Promise<T> {
  const response = await fetch(url)
  const data: unknown = await response.json()

  if (!guard(data)) {
    throw new Error('Invalid API response')
  }

  return data
}
```

```text
JSON → unknown → guard → T
```

Нельзя сделать:

```ts
data instanceof T
```

потому что `T` исчезает после компиляции.

---

## 40. Runtime schema validators

При большом API ручные `isUser`, `isProduct`, `isOrder` начинают дублироваться.

Типичный подход — schema validator: Zod, Valibot и аналоги.

```ts
const UserSchema = z.object({
  id: z.number(),
  name: z.string()
})
```

Сильная сторона: runtime-schema и TypeScript type могут опираться на один источник истины.

---

## 41. `throw` vs `Result<T>`

Exception:

```ts
async function getUser(id: number): Promise<User> {
  if (failed) {
    throw new Error('Failed')
  }

  return user
}
```

Ожидаемая бизнес-ветка:

```ts
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string }
```

Эвристика:

```text
ожидаемый бизнес-результат → Result<T>
исключительная ситуация   → throw
```

И:

```ts
T | never
```

упрощается до `T`, поэтому `never` не является «возвращаемым типом ошибки».

---

## 42. Compile-time error ≠ runtime error

```ts
function merge<T extends object>(value: T) {}
```

constraint не генерирует runtime-код вроде:

```js
if (typeof value !== 'object') {
  throw new Error()
}
```

TypeScript проверяет compile-time контракт. Runtime определяется JavaScript-кодом и build pipeline.

---

## 43. Optional, default и rest parameters

Optional:

```ts
function greet(name?: string) {}
// name: string | undefined
```

Default:

```ts
function greet(name = 'Guest') {}
// name inferred as string
```

Rest:

```ts
function sum(...values: number[]): number {
  return values.reduce((a, b) => a + b, 0)
}
```

---

## 44. Callback typing

```ts
function processUsers(
  users: User[],
  callback: (user: User) => void
) {
  users.forEach(callback)
}
```

`(user: User) => void` — контракт callback-функции.

---

## 45. Generic callback

```ts
function mapItems<T, U>(
  items: T[],
  mapper: (item: T, index: number) => U
): U[] {
  return items.map(mapper)
}
```

Связь:

```text
items         → T[]
mapper input  → T
mapper output → U
result        → U[]
```

`T` и `U` не обязаны быть объектами.

---

## 46. Overload vs JavaScript

TypeScript overload — несколько compile-time способов вызвать одну runtime-функцию.

```text
overload signatures
→ публичный контракт

implementation signature
→ единая runtime-реализация
```

Повторное объявление одноимённых функций в JavaScript само по себе не даёт typed overload dispatch как в Java/C#/C++.

---

# Vue / Nuxt + TypeScript — расширение master-summary

## 47. SFC

**SFC = Single File Component** — `.vue`-файл с `<script>`, `<template>` и `<style>`.

---

## 48. `withDefaults`

```ts
const props = withDefaults(
  defineProps<{
    title?: string
    items?: string[]
  }>(),
  {
    title: 'Default',
    items: () => []
  }
)
```

После default optional prop внутри компонента может иметь уже non-undefined тип.

---

## 49. Composables: inference vs explicit return

Для внутреннего composable обычно достаточно inference:

```ts
function useUser() {
  const user = ref<User | null>(null)
  const loading = ref(false)

  async function load() {}

  return { user, loading, load }
}
```

Явный return type особенно полезен, когда composable — публичный API, нужен стабильный контракт или хочется скрыть часть реализации.

```ts
interface UseUserReturn {
  user: Readonly<Ref<User | null>>
  loading: Readonly<Ref<boolean>>
  load: () => Promise<void>
}
```

---

## 50. Generic composable

```ts
function useAsyncData<T>(loader: () => Promise<T>) {
  const data = ref<T | null>(null)
  return { data }
}
```

Если:

```ts
fetchUsers(): Promise<User[]>
```

то:

```text
T = User[]
data ≈ Ref<User[] | null>
```

---

## 51. Pinia setup store

Обычно не надо вручную описывать гигантский тип всего store:

```ts
export const useUserStore = defineStore('user', () => {
  const user = ref<User | null>(null)
  const loading = ref(false)
  const isAuthorized = computed(() => user.value !== null)

  return { user, loading, isAuthorized }
})
```

Vue/Pinia inference обычно достаточно.

### Ограничить внешнюю мутацию

```ts
const userData = ref<User | null>(null)
const user = computed(() => userData.value)

return {
  user,
  loadUser
}
```

Mutable `userData` остаётся внутри store, наружу выходит getter-only computed.

---

## 52. `InjectionKey<T>`: production-pattern

```ts
// injectionKeys/user.ts
export const userKey: InjectionKey<Ref<User | null>> =
  Symbol('user')
```

Provider:

```ts
provide(userKey, user)
```

Consumer:

```ts
const user = inject(userKey)
// Ref<User | null> | undefined
```

`undefined` остаётся, потому что TypeScript не может доказать наличие provider.

Для обязательной dependency:

```ts
export function useInjectedUser() {
  const user = inject(userKey)

  if (!user) {
    throw new Error('User provider is missing')
  }

  return user
}
```

Это логично держать в composable: функция зависит от Vue Composition API.

**Важно:** provider и consumer должны импортировать один и тот же `Symbol`; два `Symbol('user')` — разные ключи.

---

## 53. DOM event typing

```ts
function onInput(event: Event) {
  const target = event.target

  if (target instanceof HTMLInputElement) {
    console.log(target.value)
  }
}
```

`event.target` — `EventTarget | null`; базовый `EventTarget` не гарантирует `.value`.

Вариант с assertion:

```ts
const target = event.target as HTMLInputElement
```

Когда важен именно элемент, на котором зарегистрирован handler, семантически полезно помнить про `currentTarget`.

---

## 54. Template refs

```ts
const inputRef = ref<HTMLInputElement | null>(null)
```

`null` отражает lifecycle:

```text
before mount → null
mounted      → HTMLInputElement
after unmount → null
```

Ref на child component:

```ts
const modalRef = ref<InstanceType<typeof MyModal> | null>(null)
```

Child:

```ts
defineExpose({ open })
```

Parent:

```ts
modalRef.value?.open()
```

---

## 55. Typed scoped slots — ментальная модель

`defineSlots` типизирует **slot props**, которые child отдаёт шаблону parent.

Child:

```ts
defineSlots<{
  item(props: {
    item: User
    index: number
  }): any
}>()
```

```vue
<slot
  name="item"
  :item="user"
  :index="index"
/>
```

Parent:

```vue
<template #item="{ item, index }">
  {{ item.name }}
</template>
```

Ментальная модель:

```ts
slot({ item, index })
```

Child решает **когда** вызвать slot и какие данные передать; parent решает **как** их отрисовать.

### Быстрые кандидаты на scoped slots

<details>
<summary>DataTable</summary>

```vue
<DataTable :rows="users">
  <template #row="{ row }">
    <UserTableRow :user="row" />
  </template>
</DataTable>
```

</details>

<details>
<summary>Select</summary>

```vue
<MySelect :options="users">
  <template #option="{ option }">
    <img :src="option.avatar" />
    {{ option.name }}
  </template>
</MySelect>
```

</details>

<details>
<summary>Autocomplete</summary>

```vue
<Autocomplete :items="users">
  <template #suggestion="{ item }">
    {{ item.name }} — {{ item.email }}
  </template>
</Autocomplete>
```

</details>

<details>
<summary>List</summary>

```vue
<GenericList :items="products">
  <template #item="{ item }">
    <ProductCard :product="item" />
  </template>
</GenericList>
```

</details>

<details>
<summary>Tree</summary>

```vue
<TreeView :nodes="categories">
  <template #node="{ node }">
    <CategoryNode :category="node" />
  </template>
</TreeView>
```

</details>

<details>
<summary>VirtualList</summary>

```vue
<VirtualList :items="messages">
  <template #item="{ item }">
    <MessageRow :message="item" />
  </template>
</VirtualList>
```

</details>

<details>
<summary>Carousel</summary>

```vue
<Carousel :items="products">
  <template #slide="{ item }">
    <ProductPreview :product="item" />
  </template>
</Carousel>
```

</details>

<details>
<summary>Dropdown</summary>

```vue
<Dropdown :items="actions">
  <template #item="{ item }">
    <ActionButton :action="item" />
  </template>
</Dropdown>
```

</details>

---

## 56. Generic SFC с constraint

```vue
<script
  setup
  lang="ts"
  generic="T extends { id: number }"
>
```

`generic="T"` — объявление generic-параметра всего SFC. Без него `T` в `defineProps<{ items: T[] }>()` не объявлен.

---

## 57. Универсальный generic component: props + slot + emit + model

```vue
<script
  setup
  lang="ts"
  generic="T extends { id: number }"
>
defineProps<{
  items: T[]
}>()

const selected = defineModel<T | null>()

defineSlots<{
  item(props: {
    item: T
    selected: boolean
  }): any
}>()

const emit = defineEmits<{
  select: [item: T]
}>()

function selectItem(item: T) {
  selected.value = item
  emit('select', item)
}
</script>
```

Один `T` связывает:

```text
props.items    → T[]
v-model        → T | null
slot.item      → T
emit('select') → T
```

Parent с `User[]` автоматически получает:

```text
T = User
slot item = User
select payload = User
model = User | null
```

---

## 58. Production pattern: GenericList для разных карточек

Если одинаковы:

```text
grid
pagination
loading/error/empty
common controls
selection/sort shell
```

а отличается только карточка, логично иметь один generic list и typed scoped slot:

```vue
<GenericList :items="products">
  <template #item="{ item }">
    <ProductCard :product="item" />
  </template>
</GenericList>
```

```text
GenericList
  ├─ layout
  ├─ pagination
  ├─ common states
  └─ slot<T>
       ├─ UserCard
       ├─ ProductCard
       ├─ HotelCard
       └─ ...
```

Не стоит превращать GenericList в бизнес-комбайн: domain-specific filters/API/business rules лучше оставить странице/feature/composable.

---

## 59. Финальная карта TypeScript

```text
TypeScript
│
├─ Core contracts
│  ├─ type / interface
│  ├─ union / intersection
│  ├─ unknown / never
│  └─ structural typing
│
├─ Type transformations
│  ├─ generics
│  ├─ keyof
│  ├─ T[K]
│  ├─ utility types
│  ├─ mapped types
│  └─ conditional types
│
├─ Narrowing
│  ├─ typeof
│  ├─ instanceof
│  ├─ in
│  ├─ type guards
│  └─ discriminated unions
│
├─ Runtime boundary
│  ├─ external data → unknown
│  ├─ guard/schema validation
│  └─ TS types are erased
│
├─ Functions
│  ├─ callbacks
│  ├─ generic callbacks
│  └─ overloads
│
└─ Vue/Nuxt contracts
   ├─ defineProps
   ├─ defineEmits
   ├─ defineModel
   ├─ defineSlots
   ├─ ref/reactive/computed
   ├─ InjectionKey
   └─ generic SFC
```

---

# Последние 15 секунд перед собеседованием

```text
unknown > any для внешних данных
never = невозможный return/value
| = OR, & = AND
keyof T = ключи
T[K] = значение по ключу
generic = связь типов
as const = literals + readonly
satisfies = check + preserve inference
as = assertion, не validation
external data = unknown → validate → domain type
overload implementation signature не публична
defineModel = typed v-model contract
defineSlots = typed slot payload
generic SFC = props/slot/emit/model через один T
useFetch<T> не валидирует backend runtime
```
