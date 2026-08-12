
# 🚀 30-second Cheat Sheet

## 1. Главная схема

```
SCOPE / FUNCTION
      ↓
EXECUTION CONTEXT
      ↓
LEXICAL ENVIRONMENT
      ↓
BINDINGS
      ↓
CREATION PHASE
      ↓
┌───────────────────────────────┐
│ function → function           │
│ var      → undefined          │
│ let      → TDZ                │
│ const    → TDZ                │
└───────────────────────────────┘
      ↓
EXECUTION PHASE
      ↓
код сверху вниз
```

![[Pasted image 20260812093319.png]]

---

## 2. Hoisting — мгновенная таблица

|Код|До строки декларации|
|---|---|
|`function foo(){}`|`foo → function`|
|`var x`|`x → undefined`|
|`let x`|`x → TDZ`|
|`const x`|`x → TDZ`|
|`var foo = () => {}`|`foo → undefined`|
|`const foo = () => {}`|`foo → TDZ`|

---

## 3. Ошибки — узнавай по шаблону

```ts
foo()

function foo() {}
```

```
✅ работает
```

---

```ts
foo()

var foo = function () {}
```

```
foo = undefined
undefined()
→ TypeError
```

---

```ts
foo()

const foo = () => {}
```

```
foo в TDZ
→ ReferenceError
```

---

## 4. TDZ (Temporal Dead Zone)

```
начало scope
    ↓
binding уже существует
    ↓
let / const НЕ initialized
    ↓
████████ TDZ (до момента декларации) ████████
    ↓
let x = ...
const x = ...
    ↓
можно использовать
```

---

## 5. Shadowing

```ts
const x = 'global'

function foo() {
  const x = 'local'
  console.log(x)
}
```

```
foo scope:
x → local
↓
нашли x
↓
STOP
```

В global уже не идём.

---

## 6. CALL STACK vs LEXICAL SCOPE (лекс./стат. обл. видимости)

### САМАЯ ВАЖНАЯ СХЕМА

```
CALL STACK
=
КТО КОГО ВЫЗВАЛ
```

```
LEXICAL SCOPE
=
ГДЕ ФУНКЦИЯ ОБЪЯВЛЕНА
```

---

### Пример

```ts
const value = 'global'

function create() {
  const value = 'created'

  return function show() {
    console.log(value)
  }
}

const show = create() 

function run() {
  const value = 'run'
  show()
}

run()
```

### Call Stack

> [!TIP]- Формирование Call Stack
> 1. `global` стартует - **добавлен в Call Stack**
> 2. вызывается `create()` - **добавляется в Call Stack**
> 3. `create()` возвращает функцию `show`
> 4. `create()` ЗАВЕРШАЕТСЯ, ==**удаляется из Call Stack**==! 
> 5. переменная `show` теперь хранит возвращённую функцию 
> 6. вызывается `run()` - **добавляется в Call Stack**
> 7. внутри `run()`  вызывается `show()` - **добавляется в Call Stack**
> 8. дальше аналогично по завершении каждой функции она удаляется из Call Stack по методу LIFO

```
show
 ↓
run
 ↓
global
```

### Lexical Scope

> [!TIP]- Почему из Lexical Scope выпал `run`
> 1. потому что `show` не был объявлен внутри `run`
> 2. где физически написана функция `show`:
>    ```
>    global
>    ├── create
>    │    └── show
>    └── run
>    ```
> 3. `run` и `create` - соседи, `show` - ребенок `create`, `run` выпал из цепочки

```
show
 ↓
create
 ↓
global
```

### Ответ

```
created
```

---

## 7. Универсальное правило

```
НУЖНО НАЙТИ ПЕРЕМЕННУЮ?
        ↓
НЕ СМОТРИ НА CALL STACK
        ↓
СМОТРИ, ГДЕ ФУНКЦИЯ ОБЪЯВЛЕНА
        ↓
иди по lexical scope вверх
```

---

## 8. Call Stack — только вызовы

```
a()
  → b()
      → c()
```

```
c
b
a
global
```

После `c()`:

```
b
a
global
```

После `b()`:

```
a
global
```

---

## 9. Lexical Scope — только объявления

```ts
function outer() {
  function inner() {}
}
```

```
inner
 ↓
outer
 ↓
global
```

Не важно, откуда потом вызвали `inner`.

---

## 10. Closure

```
FUNCTION
+
ССЫЛКА НА ВНЕШНИЙ LEXICAL ENVIRONMENT
=
CLOSURE
```

```ts
function create() {
  let count = 0

  return () => ++count
}
```

```
inner function
      │
      └────→ create Lexical Environment
                  count
```

---

## 11. Алгоритм решения задачи

```
1. Найди scope
      ↓
2. Выпиши bindings
      ↓
3. Creation Phase:
   function → function
   var      → undefined
   let/const→ TDZ
      ↓
4. Execution сверху вниз
      ↓
5. Если ищешь переменную:
   current lexical env
      ↓
   outer 1
      ↓
   ...outers
      ↓
   outer N
      ↓
   global
      ↓
   null
```

###  Результат поиска

```
нашли binding?
   ├─ да → используем его и STOP
   └─ нет → идём в [[OuterEnv]]
                 ↓
              повторяем
                 ↓
        дошли до null и не нашли
                 ↓
            ReferenceError
```

---

## 12. Три формулы, которые надо помнить

```
HOISTING
=
bindings создаются ДО execution
```

```
CALL STACK
=
цепочка ВЫЗОВОВ
```

```
LEXICAL SCOPE
=
цепочка ОБЪЯВЛЕНИЙ
```

---

## 💡13. Самая короткая версия

```
function → function
var      → undefined
let      → TDZ
const    → TDZ

Call Stack    → кто кого вызвал
Lexical Scope → где объявлена функция

Переменные ищем по Lexical Scope,
НЕ по Call Stack.

Closure =
функция помнит внешний Lexical Environment.
```


<br>

---
---

<br>

# 📜 Summary / Cheat Sheet

## 0. Главная опорная схема

```
JS входит в scope / function
        ↓
[1] создаётся Execution Context
        ↓
[2] внутри него создаётся Lexical Environment
        ↓
[3] в нём создаются bindings
        ↓
[4] проходит Creation Phase
        ↓
  function declaration → сразу function
  var                  → undefined
  let / const          → binding есть, но TDZ
        ↓
[5] начинается Execution Phase
        ↓
код выполняется сверху вниз
```

---

## 1. Базовые понятия

### 1.1. Scope / Область видимости

**Вопрос:** где переменная доступна?

```
Scope = область, внутри которой можно использовать переменную
```

#### Виды scope

```
Global Scope
Function Scope
Block Scope
```

#### Пример

```ts
const a = 1      // global scope

function test() {
  var b = 2      // function scope
  if (true) {
    let c = 3    // block scope
    const d = 4  // block scope
  }
}
```

#### Таблица

|Что объявили|Где живёт|
|---|---|
|`var`|function scope|
|`let`|block scope|
|`const`|block scope|
|`function declaration`|scope, где объявлена|

---

### 1.2. Execution Context

**Вопрос:** в какой среде сейчас выполняется код?

```
Execution Context = runtime-среда выполнения текущего кода
```

#### Что в нём важно для нас

```
Execution Context включает:
- Lexical Environment
- this
- служебную информацию выполнения
```

#### Когда создаётся

```
1. При старте глобального кода → Global Execution Context
2. При каждом вызове функции  → New Function Execution Context
```

---

### 1.3. Lexical Environment

**Вопрос:** какие bindings доступны здесь и где искать внешние?

```
Lexical Environment = таблица bindings + ссылка на outer environment
```

#### Упрощённая модель

```
Lexical Environment:
- a → 10
- foo → function
- b → <uninitialized>
- outer → ссылка на внешнее окружение
```

---

## 2. Scope vs Execution Context vs Lexical Environment

### Коротко

|Понятие|Что означает|
|---|---|
|`Scope`|где переменная доступна|
|`Execution Context`|среда выполнения текущего кода|
|`Lexical Environment`|bindings + ссылка на внешнее окружение|

---

## 3. Две фазы выполнения

### 3.1. Creation Phase

```
Creation Phase:
- создаются bindings
- function declaration сразу получает function
- var получает undefined
- let/const получают binding, но остаются uninitialized (TDZ)
```

### 3.2. Execution Phase

```
Execution Phase:
- код выполняется сверху вниз
- происходят присваивания
- вызываются функции
- читаются значения
```

---

## 4. Hoisting — самая важная схема

### Правильное определение

```
Hoisting ≠ "переменные физически переносятся наверх"

Hoisting = bindings создаются ДО выполнения строк кода
```

---

### 4.1. Таблица hoisting

|Конструкция|Binding создаётся заранее?|Начальное значение на Creation Phase|Можно использовать до строки декларации?|Что будет|
|---|---|---|---|---|
|`function foo() {}`|✅|сама функция|✅|работает|
|`var x`|✅|`undefined`|✅|получим `undefined`|
|`let x`|✅|`<uninitialized>`|❌|`ReferenceError`|
|`const x`|✅|`<uninitialized>`|❌|`ReferenceError`|
|`var foo = function(){}`|✅|`undefined`|вызовить нельзя|`TypeError: foo is not a function`|
|`const foo = () => {}`|✅|`<uninitialized>`|❌|`ReferenceError`|

---

### 4.2. Три эталонных примера

#### 1. Function declaration

```ts
foo()

function foo() {
  console.log('foo')
}
```

```
Creation:
foo → function

Execution:
foo() → OK
```

---

#### 2. Function expression через var

```ts
foo()

var foo = function () {
  console.log('foo')
}
```

```
Creation:
foo → undefined

Execution:
foo() → undefined()
→ TypeError
```

---

#### 3. Arrow function через const

```ts
foo()

const foo = () => {
  console.log('foo')
}
```

```
Creation:
foo → <uninitialized>

Execution:
foo()
→ ReferenceError
```

---

## 5. TDZ — Temporal Dead Zone

### Опорная схема

```
Начало scope
    ↓
binding уже создан
    ↓
НО let/const ещё не инициализированы
    ↓
ЭТО TDZ
    ↓
строка let/const
    ↓
после неё переменную можно читать
```

---

### Пример

```ts
console.log(x)
let x = 10
```

#### Что происходит

```
Creation:
x → <uninitialized>

Execution:
console.log(x)
→ ReferenceError
```

---

### Главное правило

```
TDZ есть у let и const
TDZ нет у var
```

---

## 6. Shadowing — затенение переменной

### Схема

```
Если во внутреннем scope есть локальный binding,
он затеняет внешний binding с тем же именем.
```

### Пример с var

```ts
var x = 1

function test() {
  console.log(x)
  var x = 2
  console.log(x)
}

test()
console.log(x)
```

#### Ответ

```
undefined
2
1
```

#### Почему

```
Внутри test на Creation Phase:
x → undefined

Поэтому первый console.log(x) в test
читает ЛОКАЛЬНЫЙ x,
а не внешний глобальный x
```

---

### Пример с let

```ts
let x = 1

function test() {
  console.log(x)
  let x = 2
}

test()
```

#### Ответ

```ts
ReferenceError
```

#### Почему

```
Внутри test уже существует локальный binding x,
но он в TDZ.

Lookup находит локальный x
и НЕ идёт искать глобальный x.
```

---

## 7. Call Stack

### Определение

```
Call Stack = стек активных вызовов / execution contexts
```

---

### Главная функция Call Stack

```
Call Stack отвечает на вопрос:
"какая функция сейчас выполняется
и куда вернуться после её завершения?"
```

---

### Схема

```ts
function first() {
  second()
}

function second() {
  third()
}

function third() {}

first()
```

```
Старт:
[ Global ]

first():
[ first ]
[ Global ]

second():
[ second ]
[ first  ]
[ Global ]

third():
[ third  ]
[ second ]
[ first  ]
[ Global ]

возврат:
[ second ]
[ first  ]
[ Global ]

[ first  ]
[ Global ]

[ Global ]
```

---

## 8. Lexical Scope / Scope Chain

### Определение

```
Lexical Scope определяется МЕСТОМ ОБЪЯВЛЕНИЯ функции,
а не местом её вызова.
```

---

### Главная функция Scope Chain

```
Scope Chain отвечает на вопрос:
"где искать переменную?"
```

---

### Схема

```ts
const a = 'global'

function outer() {
  const b = 'outer'

  function inner() {
    console.log(a)
    console.log(b)
  }

  inner()
}

outer()
```

#### Поиск переменных в `inner`

```
inner scope
   ↓
outer scope
   ↓
global scope
   ↓
null
```

---

## 9. САМОЕ СЛОЖНОЕ: Call Stack ≠ Lexical Scope

### Главное различие

|Механизм|Отвечает на вопрос|
|---|---|
|`Call Stack`|кто кого сейчас вызвал|
|`Lexical Scope`|где искать переменную|

---

### Формула

```
Call Stack = цепочка ВЫЗОВОВ
Lexical Scope = цепочка ОБЪЯВЛЕНИЙ
```

---

### 9.1. Когда они совпадают

```ts
function outer() {
  function inner() {
    console.log(x)
  }

  inner()
}

outer()
```

```
Call Stack:   inner → outer → global
Lexical Scope: inner → outer → global
```

Здесь легко спутать, потому что цепочки одинаковые.

---

### 9.2. Когда они РАСХОДЯТСЯ

#### Эталонный пример №1

```ts
const value = 'global'

function createFn() {
  const value = 'created'

  return function show() {
    console.log(value)
  }
}

const show = createFn()

function run() {
  const value = 'run'
  show()
}

run()
```

#### Что выведется

```
created
```

#### Почему

##### Call Stack в момент `console.log`

```
show
run
global
```

##### Lexical Scope для `show`

```
show
↓
createFn
↓
global
```

#### Вывод

```
show вызвана из run
НО объявлена внутри createFn
поэтому ищет value в createFn, а не в run
```

---

#### Эталонный пример №2

```ts
const x = 'global'

function foo() {
  const x = 'foo'
  bar()
}

function bar() {
  console.log(x)
}

foo()
```

#### Что выведется

```
global
```

#### Почему

##### Call Stack

```
bar
foo
global
```

##### Lexical Scope для `bar`

```
bar
↓
global
```

#### Вывод

```
bar объявлена в global
поэтому ищет x в global
а не в foo
```

---

## 10. Замыкание (Closure)

### Определение

```
Closure = функция + доступ к её внешнему lexical environment
```

---

### Эталонный пример

```ts
function createCounter() {
  let count = 0

  return function () {
    count++
    return count
  }
}

const counter = createCounter()

console.log(counter()) // 1
console.log(counter()) // 2
console.log(counter()) // 3
```

---

### Схема

```
createCounter() вызвалась
    ↓
создался Lexical Environment:
count → 0
    ↓
вернули inner function
    ↓
inner function сохранила ссылку
на этот Lexical Environment
    ↓
поэтому count не исчез
```

---

### Пример с двумя методами

```ts
function create() {
  let count = 0

  return {
    inc() {
      count++
    },
    print() {
      console.log(count)
    }
  }
}

const counter = create()
counter.inc()
counter.inc()
counter.print()
```

#### Ответ

```
2
```

#### Почему

```
inc и print созданы внутри ОДНОГО вызова create()
→ обе функции замкнули ОДИН и тот же Lexical Environment
→ count один и тот же
```

---

## 11. Object literal не создаёт lexical scope

### Важно

```
Объект сам по себе НЕ создаёт новую lexical scope
```

### Неправильная мысль

```
"внутри объекта есть своя область видимости"
```

### Правильная мысль

```
Функции внутри object literal
объявлены в том lexical environment,
где сам объект создаётся
```

---

## 12. Алгоритм решения любой задачи на hoisting / scope / closures

### Алгоритм

#### Шаг 1. Найди scope

```
Где global?
Где function scope?
Где block scope?
```

---

#### Шаг 2. Определи Creation Phase

Для каждого scope выпиши bindings:

```
function → function
var      → undefined
let/const→ TDZ
```

---

#### Шаг 3. Ищи shadowing

```
Есть ли локальная переменная
с тем же именем, что и внешняя?
Если да — внешняя затенена.
```

---

#### Шаг 4. Разделяй:

```
Call Stack = кто кого вызвал
Lexical Scope = где функция объявлена
```

---

#### Шаг 5. Для каждой переменной делай lookup так:

```
1. ищу в текущем lexical environment
2. если нет → иду во внешний
3. если нет → ещё выше
4. если дошёл до global → проверяю там
5. если нигде нет → ReferenceError
```

---

## 13. Быстрые диагностические схемы

### 13.1. Почему `undefined`?

```
Если увидел:
console.log(x)
var x = ...

→ ответ почти всегда:
var hoisted
x = undefined
```

---

### 13.2. Почему `ReferenceError`?

```
Если увидел:
console.log(x)
let x = ...
или
const x = ...

→ ответ почти всегда:
binding есть, но TDZ
```

---

### 13.3. Почему `TypeError: ... is not a function`?

```
Если увидел:
foo()
var foo = function () {}

→ answer:
foo hoisted as undefined
получилось undefined()
```

---

### 13.4. Почему функция взяла "не ту" переменную?

```
Не смотри на то, КТО вызвал функцию.
Смотри на то, ГДЕ функция была ОБЪЯВЛЕНА.
```

---

## 14. Мини-дерево по функциям

### Как понять, откуда функция возьмёт переменную

```
Функция вызвана
   ↓
Смотрю НЕ на call stack
   ↓
Смотрю, где функция ОБЪЯВЛЕНА
   ↓
Беру её lexical environment
   ↓
Иду вверх по outer chain
   ↓
Нахожу нужный binding
```

---

## 🚨15. Три золотых правила блока

### 📜 Правило 1

```
Bindings появляются раньше, чем начинается выполнение строк кода.
```

---

### 📜 Правило 2

```
Call Stack = вызовы
Lexical Scope = объявления
Это РАЗНЫЕ вещи.
```

---

### 📜 Правило 3

```
Функция помнит не место вызова,
а место объявления.
```

---

## 16. Сверхкороткая версия блока

```
1. JS сначала создаёт Execution Context
2. В нём создаётся Lexical Environment
3. В нём создаются bindings
4. function → function
5. var → undefined
6. let/const → TDZ
7. Потом начинается выполнение сверху вниз
8. Call Stack = активные вызовы
9. Lexical Scope = цепочка мест объявления
10. Closure = функция помнит внешнее lexical environment
```

---

## 17. Карманная таблица “вижу код → сразу понимаю”

|Если в коде вижу|Значит думаю|
|---|---|
|`function foo(){}` до вызова|function hoisted целиком|
|`var x` до присваивания|`x === undefined`|
|`let/const x` до строки декларации|TDZ → `ReferenceError`|
|`var foo = function(){}` и вызов выше|`foo === undefined` → `TypeError`|
|функция вызвана из другого места|не важно, смотри место объявления|
|возвращённая внутренняя функция|скорее всего closure|
|переменная не взялась из внешнего scope|возможно её затенил локальный binding|

---

## 18. Одна супер-опорная схема именно для твоей сложной темы

### Call Stack vs Lexical Scope

```
CALL STACK
=
"КТО КОГО ВЫЗВАЛ?"

show
run
global
```

```
LEXICAL SCOPE
=
"ГДЕ ФУНКЦИЯ БЫЛА ОБЪЯВЛЕНА?"

show
createFn
global
```

### Вывод

```
Функция БЕРЁТ ПЕРЕМЕННЫЕ
НЕ из call stack,
А из lexical scope.
```
