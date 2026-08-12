#this #bind #apply #call #arrow #regular

```
                user.regular()
                      │
                      │ call site
                      ▼
              ┌────────────────┐
              │ name = "Ilya"  |
              |   regular()    │
              │  this = user   │
              └───────┬────────┘
	                внутри
          ┌───────────┴────────────┐
          │                        │
          ▼                        ▼
       () => {}                  func()
          │                        │
   своего this нет           свой this есть
          │                        │
   берёт у regular         call site = func()
          │                        │
          ▼                        ▼
     this = user            this = undefined
          │                        │
          ▼                        ▼
       "Ilya"                  TypeError
```

# JavaScript `this`: regular functions, arrow functions, call / apply / bind

> [!summary] 30-second Cheat Sheet
>
> ```text
> REGULAR FUNCTION
> → имеет собственный this
> → this определяется СПОСОБОМ ВЫЗОВА
>
> obj.method()
> → this = obj
>
> fn()
> → this = undefined в strict mode
>
> fn.call(obj)
> fn.apply(obj)
> → this = obj на один вызов
>
> fn.bind(obj)
> → создаётся новая bound function
> → this закреплён за obj
>
>
> ARROW FUNCTION
> → собственного this НЕТ
> → использует this внешнего контекста
>
> call / apply / bind
> → не могут заменить this у arrow
> ```

---

# 1. Главное правило `this`

При работе с `this` сначала необходимо определить:

```text
Что это за функция?
│
├── regular function
│   ↓
│   КАК ЕЁ ВЫЗВАЛИ?
│
└── arrow function
    ↓
    КАКОЙ this У ВНЕШНЕГО КОНТЕКСТА?
```

Это главный алгоритм решения большинства задач на `this`.

---

# 2. Regular function: `this` определяется при вызове

Для обычной функции место объявления функции само по себе не определяет её `this`.

```js
function showName() {
  console.log(this.name)
}
```

До момента вызова нельзя сказать, каким будет `this`.

## Вызов как метода

```js
const user = {
  name: 'Ilya',
  showName
}

user.showName()
```

Смотрим на объект непосредственно перед точкой:

```text
user.showName()
^^^^
this
```

Следовательно:

```text
this = user
```

Результат:

```text
Ilya
```

---

# 3. Одна функция может иметь разный `this`

```js
function showName() {
  console.log(this.name)
}

const user = {
  name: 'Ilya',
  showName
}

const admin = {
  name: 'Admin',
  showName
}

user.showName()
admin.showName()
```

Результат:

```text
Ilya
Admin
```

Функция одна и та же.

Но:

```text
user.showName()
→ this = user

admin.showName()
→ this = admin
```

Поэтому для regular function:

> `this` определяется не местом создания функции, а способом её вызова.

---

# 4. Потеря контекста

```js
'use strict'

const user = {
  name: 'Ilya',

  showName() {
    console.log(this.name)
  }
}

const fn = user.showName

fn()
```

При:

```js
const fn = user.showName
```

сама функция никуда не перемещается и не изменяется.

Просто создаётся ещё одна ссылка на неё:

```text
user.showName ───┐
                 ├──► function
fn ──────────────┘
```

Но вызов изменился.

Было:

```js
user.showName()
```

Стало:

```js
fn()
```

Для regular function:

```text
fn()
↓
plain function call
↓
strict mode
↓
this = undefined
↓
this.name
↓
TypeError
```

Это называется **потерей контекста**.

---

# 5. Важно: объект не является Lexical Environment

Очень легко смешать две разные системы.

```js
const user = {
  name: 'Ilya',

  show() {
    console.log(this.name)
  }
}
```

Объект:

```text
user
├── name
└── show
```

не создаёт отдельный lexical scope только потому, что функция записана внутри объекта.

Поэтому нельзя мыслить так:

```text
show
↓
user lexical environment
↓
global
```

Такого `user lexical environment` здесь нет.

---

## `name` и `this.name` — разные механизмы

### Обычный identifier

```js
console.log(name)
```

JS ищет его по lexical chain:

```text
Current Lexical Environment
↓
Outer Lexical Environment
↓
...
↓
Global
```

### Свойство через `this`

```js
console.log(this.name)
```

Здесь сначала определяется:

```text
this
```

а потом производится property lookup:

```text
this.name
```

Поэтому:

```text
name
→ lexical lookup

this.name
→ определить this
→ найти property name у объекта
```

---

# 6. `call`

`call` немедленно вызывает regular function и явно задаёт её `this`.

```js
function print(prefix) {
  console.log(prefix, this.name)
}

const user = {
  name: 'Ilya'
}

print.call(user, 'User:')
```

Схема:

```text
print.call(user, 'User:')
           ↓
       this = user
           ↓
       User: Ilya
```

`call` не изменяет исходную функцию навсегда.

```js
print.call(user)
print.call(admin)
```

могут использовать разные `this`.

---

# 7. `apply`

По смыслу `apply` аналогичен `call`.

Различие — передача аргументов.

```js
fn.call(obj, arg1, arg2)

fn.apply(obj, [arg1, arg2])
```

Например:

```js
calculate.apply(user, [10, 20, 30])
```

Современный аналог через spread:

```js
calculate.call(user, ...numbers)
```

> `...numbers` при вызове функции — **spread**, а не rest.

---

# 8. `bind`

`bind` отличается от `call/apply`.

```text
call / apply
→ сразу вызывают функцию

bind
→ НЕ вызывает функцию
→ создаёт новую bound function
```

Пример:

```js
function introduce(prefix, age) {
  console.log(prefix, this.name, age)
}

const user = {
  name: 'Ilya'
}

const boundIntroduce =
  introduce.bind(user, 'Developer:')

boundIntroduce(38)
boundIntroduce(40)
```

Результат:

```text
Developer: Ilya 38
Developer: Ilya 40
```

Схема:

```text
introduce
   │
   │ bind(user, 'Developer:')
   ▼
boundIntroduce
   │
   ├── this = user
   └── prefix = 'Developer:'
```

Также:

```js
boundIntroduce !== introduce
```

потому что `bind()` создаёт **новую функцию**.

---

# 9. Arrow function: собственного `this` нет

Это главное отличие arrow.

```js
const arrow = () => {
  console.log(this)
}
```

У arrow нет собственного binding для `this`.

Она использует `this` окружающего контекста.

Коротко:

```text
REGULAR
→ this приходит ПРИ ВЫЗОВЕ

ARROW
→ собственного this нет
→ использует this СНАРУЖИ
```

---

# 10. Arrow внутри regular method

```js
const user = {
  name: 'Ilya',

  showName() {
    const inner = () => {
      console.log(this.name)
    }

    inner()
  }
}

user.showName()
```

Сначала вызывается regular method:

```text
user.showName()
^^^^
this = user
```

Затем создаётся arrow:

```text
user.showName()
      │
      ▼
showName
this = user
      │
      │ arrow использует внешний this
      ▼
inner
this = user
```

Поэтому:

```text
Ilya
```

---

# 11. Почему `inner()` работает, хотя перед ним нет объекта?

Это важный момент.

```js
inner()
```

Если бы `inner` была regular function:

```text
inner()
→ this = undefined
```

Но это arrow.

Для arrow форма вызова:

```js
inner()
```

не задаёт новый `this`.

Arrow спрашивает:

```text
Есть собственный this?
↓
нет
↓
какой this у окружающего контекста?
↓
user
```

---

# 12. Regular function внутри regular method

Сравним:

```js
const user = {
  name: 'Ilya',

  showName() {
    function inner() {
      console.log(this.name)
    }

    inner()
  }
}

user.showName()
```

`showName`:

```text
user.showName()
→ this = user
```

Но `inner` — regular function.

Поэтому для неё снова рассматривается **её собственный call site**:

```text
inner()
↓
regular function
↓
plain call
↓
this = undefined
↓
this.name
↓
TypeError
```

То, что `inner` была объявлена внутри функции с `this = user`, для regular-function её `this` не закрепляет.

---

# 13. Центральная схема: regular vs arrow

```text
                 outerMethod
                 this = user
                      │
            ┌─────────┴──────────┐
            │                    │
            ▼                    ▼
      regularInner           arrowInner
            │                    │
      свой this ЕСТЬ         своего this НЕТ
            │                    │
            ▼                    ▼
   зависит от своего         использует this
      CALL SITE              outerMethod
```

Запоминать именно так:

```text
REGULAR
→ КАК ВЫЗВАЛИ?

ARROW
→ КАКОЙ this СНАРУЖИ?
```

---

# 14. Arrow как callback

Arrow особенно удобны в callback внутри методов:

```js
const user = {
  name: 'Ilya',

  start() {
    setTimeout(() => {
      console.log(this.name)
    }, 100)
  }
}

user.start()
```

При:

```js
user.start()
```

получаем:

```text
start: this = user
```

Arrow callback использует этот `this`:

```text
start
this = user
   ↓
arrow callback
   ↓
this = user
```

---

# 15. Но arrow не спасает потерянный `this` родителя

```js
const user = {
  name: 'Ilya',

  start() {
    const arrow = () => {
      console.log(this.name)
    }

    arrow()
  }
}

const callback = user.start

callback()
```

`start` — regular function.

Она вызвана:

```text
callback()
↓
this = undefined
```

Затем arrow использует именно этот внешний `this`:

```text
start
this = undefined
      ↓
arrow
      ↓
this = undefined
      ↓
this.name
      ↓
TypeError
```

Чтобы исправить:

```js
const callback = user.start.bind(user)
```

Теперь:

```text
bind(user)
   ↓
start: this = user
   ↓
arrow
   ↓
использует this = user
```

---

# 16. `call/apply/bind` и arrow

У arrow нет собственного `this`.

Поэтому:

```js
arrow.call(obj)
arrow.apply(obj)
arrow.bind(obj)
```

не могут заменить её lexical `this`.

Пример:

```js
function create() {
  return () => {
    console.log(this.name)
  }
}

const user = {
  name: 'Ilya'
}

const admin = {
  name: 'Admin'
}

const fn = create.call(user)

fn.call(admin)
```

Сначала:

```text
create.call(user)
↓
create: this = user
```

Внутри создаётся arrow:

```text
create
this = user
   ↓
arrow
   ↓
использует this = user
```

Затем:

```text
fn.call(admin)
```

не меняет `this` arrow:

```text
admin пытаются передать как this
↓
arrow собственного this не имеет
↓
используется внешний this
↓
user
```

Результат:

```text
Ilya
```

---

# 17. Таблица для собеседования

| Ситуация | `this` |
|---|---|
| `obj.method()` — regular | `obj` |
| `fn()` — regular, strict mode | `undefined` |
| `fn.call(obj)` | `obj` |
| `fn.apply(obj)` | `obj` |
| `fn.bind(obj)` | новая функция с `this = obj` |
| arrow function | собственного `this` нет |
| arrow внутри method | использует `this` внешнего method |
| `arrow.call(obj)` | lexical `this` arrow не меняется |
| `arrow.bind(obj)` | lexical `this` arrow не меняется |

---

# 18. Алгоритм решения задачи на `this`

При виде функции не пытаться одновременно думать про:

- Call Stack;
- Lexical Environment;
- scope;
- объект;
- closure;
- `this`.

Для задачи именно на `this` сначала выполнить простой алгоритм.

## Шаг 1. Определить тип функции

```text
regular?
или
arrow?
```

## Шаг 2. Если regular

Посмотреть на call site:

```text
obj.fn()
→ this = obj

fn()
→ this = undefined

fn.call(obj)
→ this = obj

boundFn()
→ this задан bind
```

## Шаг 3. Если arrow

Не смотреть на объект перед точкой для определения `this`.

```text
arrow
↓
своего this нет
↓
смотрим на внешний контекст
```

---

# 19. Не смешивать четыре механизма

```text
Обычная переменная:
x
→ Lexical Scope / Lexical Environment

Call Stack:
→ кто кого сейчас вызвал

this regular function:
→ как вызвали функцию

this arrow function:
→ внешний this
```

Короткая шпаргалка:

```text
x
→ ГДЕ ОБЪЯВИЛИ?

Call Stack
→ КТО КОГО ВЫЗВАЛ?

regular this
→ КАК ВЫЗВАЛИ?

arrow this
→ КАКОЙ this СНАРУЖИ?
```

---

# 20. Сложный контрольный пример

```js
'use strict'

const user = {
  name: 'Ilya',

  regularMethod() {
    console.log('A:', this.name)

    function regularInner() {
      console.log('B:', this.name)
    }

    const arrowInner = () => {
      console.log('C:', this.name)
    }

    return {
      regularInner,
      arrowInner
    }
  }
}

const admin = {
  name: 'Admin'
}

const helpers = user.regularMethod()

helpers.arrowInner()

helpers.regularInner()

const detachedRegular = helpers.regularInner
detachedRegular()

const boundRegular = helpers.regularInner.bind(admin)
boundRegular()

helpers.arrowInner.call(admin)
```

---

## Вопрос 1

Что происходит здесь?

```js
const helpers = user.regularMethod()
```

`regularMethod` — regular function.

Вызов:

```text
user.regularMethod()
^^^^
this = user
```

Поэтому:

```js
console.log('A:', this.name)
```

выведет:

```text
A: Ilya
```

Во время выполнения `regularMethod` создаются две функции:

```js
function regularInner() { ... }

const arrowInner = () => { ... }
```

Они создаются в одном месте, но правила `this` у них разные.

---

## Вопрос 2

Что выведет:

```js
helpers.arrowInner()
```

`arrowInner` — arrow function.

Она была создана во время:

```js
user.regularMethod()
```

А в этом вызове:

```text
regularMethod: this = user
```

Arrow собственного `this` не имеет:

```text
regularMethod
this = user
      │
      │ внешний this
      ▼
arrowInner
      │
      ▼
this = user
```

Поэтому:

```text
C: Ilya
```

Важно:

```js
helpers.arrowInner()
```

не делает `this = helpers` для arrow.

Объект перед точкой здесь не назначает arrow новый `this`.

---

## Вопрос 3

Что выведет:

```js
helpers.regularInner()
```

Вот здесь принципиальное отличие.

`regularInner` — **regular function**.

Поэтому спрашиваем:

```text
КАК ЕЁ ВЫЗВАЛИ?
```

Вызвали:

```js
helpers.regularInner()
```

Значит:

```text
this = helpers
```

Что такое `helpers`?

`regularMethod` вернула:

```js
{
  regularInner,
  arrowInner
}
```

То есть примерно:

```text
helpers
├── regularInner
└── arrowInner
```

Свойства:

```text
name
```

у `helpers` нет.

Поэтому:

```js
this.name
```

это:

```js
helpers.name
```

а результат:

```text
undefined
```

Следовательно:

```text
B: undefined
```

Не `TypeError`, потому что `this` — существующий объект `helpers`.

Просто у него нет свойства `name`.

---

## Вопрос 4

Что произойдёт здесь?

```js
const detachedRegular = helpers.regularInner

detachedRegular()
```

`regularInner` — regular function.

После отделения:

```text
helpers.regularInner
         │
         ▼
detachedRegular
```

вызывается:

```js
detachedRegular()
```

Объекта перед точкой нет.

В strict mode:

```text
detachedRegular()
↓
regular function
↓
plain function call
↓
this = undefined
```

Далее:

```js
this.name
```

становится:

```js
undefined.name
```

И возникает:

```text
TypeError
```

---

## Вопрос 5

Что выведет:

```js
const boundRegular =
  helpers.regularInner.bind(admin)

boundRegular()
```

`regularInner` — regular function, поэтому её `this` можно закрепить через `bind`.

```text
regularInner
     │
     │ bind(admin)
     ▼
boundRegular
     │
     ▼
this = admin
```

Поэтому:

```js
this.name
```

это:

```js
admin.name
```

Результат:

```text
B: Admin
```

---

## Вопрос 6

Что выведет:

```js
helpers.arrowInner.call(admin)
```

`arrowInner` — arrow function.

Она уже использует внешний `this`, полученный от `regularMethod`:

```text
arrowInner
↓
this = user
```

Попытка:

```js
.call(admin)
```

не может заменить lexical `this` arrow.

```text
arrowInner.call(admin)
          │
          └── admin не становится this
                 ↓
         используется внешний this
                 ↓
                user
```

Результат:

```text
C: Ilya
```

---

## Вопрос 7 — главный

Почему:

```js
function regularInner() {
  console.log(this.name)
}
```

и:

```js
const arrowInner = () => {
  console.log(this.name)
}
```

были созданы внутри одной `regularMethod`, но их `this` ведёт себя по-разному?

Потому что это **разные типы функций с разными правилами `this`**.

```text
                 regularMethod
                 this = user
                      │
            ┌─────────┴──────────┐
            │                    │
            ▼                    ▼
     regularInner           arrowInner
            │                    │
     свой this ЕСТЬ         своего this НЕТ
            │                    │
            ▼                    ▼
     определяется             использует
    НОВЫМ call site          внешний this
```

Поэтому:

```js
helpers.regularInner()
```

означает:

```text
REGULAR
↓
смотрим call site
↓
helpers.regularInner()
↓
this = helpers
↓
helpers.name
↓
undefined
```

А:

```js
helpers.arrowInner()
```

означает:

```text
ARROW
↓
собственного this нет
↓
call site для this не используется
↓
берётся внешний this regularMethod
↓
user
↓
Ilya
```

---

# 21. Итог сложного примера

До возникновения ошибки программа выводит:

```text
A: Ilya
C: Ilya
B: undefined
```

После:

```js
detachedRegular()
```

возникает:

```text
TypeError
```

и обычное выполнение программы прекращается.

Поэтому строки ниже фактически уже не выполнятся:

```js
boundRegular()
helpers.arrowInner.call(admin)
```

Но если рассматривать их отдельно, они дали бы:

```text
B: Admin
C: Ilya
```

---

# 22. Финальная схема

```text
              ФУНКЦИЯ
                 │
        ┌────────┴────────┐
        │                 │
     REGULAR            ARROW
        │                 │
  свой this есть     своего this нет
        │                 │
        ▼                 ▼
 КАК ВЫЗВАЛИ?        КАКОЙ this
                     СНАРУЖИ?
        │                 │
 ┌──────┼──────┐          │
 │      │      │          │
obj.fn fn()   bind        │
 │      │      │          │
obj undefined fixed       │
                         ▼
                    внешний this
```

> [!tip] Главное правило для собеседования
>
> ```text
> REGULAR FUNCTION
> → this определяется call site
>
> ARROW FUNCTION
> → собственного this нет
> → используется внешний this
> ```

Этой модели достаточно для большинства практических задач на `this`.