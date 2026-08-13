
# 🚀 30-second Cheat Sheet

## `this`

```text
regular function
→ this определяется СПОСОБОМ ВЫЗОВА

arrow function
→ собственного this нет
→ использует this СНАРУЖИ
```

```text
obj.method()
→ this = obj

const fn = obj.method
fn()
→ context loss
```

## `call / apply / bind`

```text
call
→ вызвать сейчас
→ args через запятую

apply
→ вызвать сейчас
→ args массивом

bind
→ НЕ вызывает
→ создаёт новую bound function
```

---

## Closure

```text
closure
→ функция сохраняет доступ к внешнему LE
→ сохраняет binding, не snapshot значения
```

```text
разные factory-вызовы
→ разные LE

несколько closures одного вызова
→ могут работать с одними bindings
```

---

## `var` / `let` в цикле

```text
var
→ один binding

for (let i = ...)
→ отдельный binding на итерацию
```

---

## Declaration / Expression / Arrow

```text
function A() {}
→ Function Declaration
→ A сразу связан с обычной функцией
→ доступна до строки объявления
```

```text
const B = function () {}
→ B после инициализации хранит ссылку
  на обычную функцию
→ TDZ до объявления
```

```text
const C = () => {}
→ C хранит ссылку на arrow function
→ TDZ до объявления
```

---

## Regular vs Arrow

```text
regular function
→ свой this
→ свой arguments
→ может быть constructable
→ может иметь [[Construct]]
→ есть .prototype

arrow
→ нет собственного this
→ нет собственного arguments
→ нет [[Construct]]
→ нельзя new
→ нет собственного .prototype
```

<br>

---

<br>

# Быстрое восстановление за 2–3 минуты

## 1. Regular function и `this`

Для обычной функции `this` определяется не местом объявления, а **формой вызова**:

```js
const user = {
  name: 'Ilya',

  show() {
    console.log(this.name)
  }
}

user.show()
```

```text
user.show()
→ this = user
```

Но:

```js
const fn = user.show
fn()
```

контекст теряется.

В strict mode:

```text
this = undefined
```

### Главное разделение механизмов

```text
Lexical Scope
→ ГДЕ объявили?

Call Stack
→ КТО КОГО вызвал?

regular this
→ КАК вызвали?

arrow this
→ КАКОЙ this снаружи?
```

---

## 2. Arrow function

Arrow function собственного `this` не создаёт:

```js
const user = {
  name: 'Ilya',

  show() {
    const inner = () => {
      console.log(this.name)
    }

    inner()
  }
}
```

`show()` — regular method:

```text
user.show()
→ this = user
```

`inner` — arrow:

```text
собственного this нет
→ использует this внешней show
→ user
```

Важно:

```text
call / apply / bind
НЕ могут заменить lexical this arrow
```

---

## 3. `call`, `apply`, `bind`

```js
function show(greeting) {
  console.log(greeting, this.name)
}
```

### `call`

```js
show.call(user, 'Hello')
```

```text
вызвать сейчас
this = user
аргументы через запятую
```

### `apply`

```js
show.apply(user, ['Hello'])
```

```text
вызвать сейчас
this = user
аргументы массивом
```

### `bind`

```js
const bound = show.bind(user)

bound('Hello')
```

```text
исходную функцию сразу НЕ вызывает
↓
создаёт новую bound function
```

---

## 4. Closure

```js
function create(value) {
  return () => console.log(value)
}

const fn = create(10)
```

После завершения `create()` её Execution Context исчезает из Call Stack.

Но возвращённая функция сохраняет доступ к её Lexical Environment:

```text
fn
↓
returned function
↓
create LE
↓
value
```

Closure работает с **binding**, поэтому:

```js
function create() {
  let value = 10

  const show = () => console.log(value)

  value = 20

  return show
}

create()() // 20
```

---

## 5. Несколько closures

Разные вызовы factory:

```js
const a = createCounter()
const b = createCounter()
```

создают разные окружения:

```text
a → LE #1 → count₁
b → LE #2 → count₂
```

Но closures одного вызова могут работать с общей переменной:

```text
        create LE
        value
        ↑   ↑
       get set
```

---

## 6. `var` / `let` в циклах

### `var`

```js
for (var i = 0; i < 3; i++) {
  functions.push(() => console.log(i))
}
```

Все closures используют один binding:

```text
fn0 ─┐
fn1 ─┼──► i
fn2 ─┘
```

После цикла:

```text
i = 3
```

поэтому:

```text
3
3
3
```

### `for (let i = ...)`

```js
for (let i = 0; i < 3; i++) {
  functions.push(() => console.log(i))
}
```

работает per-iteration binding:

```text
fn0 → i₀ = 0
fn1 → i₁ = 1
fn2 → i₂ = 2
```

---

## 7. Function Declaration / Expression / Arrow

### Function Declaration

```js
function A() {}
```

На этапе создания окружения:

```text
A → function
```

Поэтому функция доступна до строки объявления.

---

### Function Expression

```js
const B = function () {}
```

Здесь важно разделять:

```text
B
→ переменная

function () {}
→ обычная функция

B
→ после инициализации хранит ссылку на эту функцию
```

Function Expression не мешает функции быть обычной constructable function.

---

### Arrow Function

```js
const C = () => {}
```

`C` тоже хранит ссылку на функцию, но уже на **arrow function**, у которой другие свойства языка.

---

## 8. Named Function Expression

```js
const factorial = function calc(n) {
  if (n <= 1) return 1

  return n * calc(n - 1)
}
```

```text
factorial
→ внешняя переменная

calc
→ внутреннее имя функции
→ доступно внутри самой функции
```

Это удобно, например, для рекурсии.

---

## 9. `arguments`

Обычная функция имеет собственный `arguments`:

```js
function show() {
  console.log(arguments[0])
}
```

Arrow собственного `arguments` не имеет.

Если arrow находится внутри regular function:

```js
function outer(a) {
  const arrow = () => {
    console.log(arguments[0])
  }

  arrow()
}
```

она может использовать `arguments` внешней `outer`.

В современном коде обычно удобнее:

```js
(...args) => {}
```

---

## 10. `new`, `[[Construct]]`, `.prototype`

Обычная функция может быть constructable:

```js
function User(name) {
  this.name = name
}

const user = new User('Ilya')
```

Такая функция имеет внутренний механизм:

```text
[[Construct]]
```

который позволяет использовать:

```js
new User()
```

Arrow function:

```js
const User = () => {}
```

не имеет `[[Construct]]`.

Поэтому:

```js
new User()
```

даёт:

```text
TypeError
```

### Не путать

```text
[[Construct]]
≠
.prototype
```

`[[Construct]]` — внутренний механизм функции-объекта.

`.prototype` — отдельное свойство constructable function.

---

# Главная итоговая таблица

| Механизм | Regular function | Arrow |
|---|---|---|
| Собственный `this` | ✅ | ❌ |
| Собственный `arguments` | ✅ | ❌ |
| `call/apply/bind` меняют `this` | ✅ | ❌ |
| Можно `new` | ✅* | ❌ |
| `[[Construct]]` | ✅* | ❌ |
| Собственный `.prototype` | ✅* | ❌ |

`*` — для обычных constructable functions.

---

# 📥Главное, что нужно помнить на интервью

```text
regular this
→ как вызвали?

arrow this
→ какой this снаружи?

closure
→ какой внешний binding сохранён?

var в цикле
→ один binding

for (let ...)
→ binding на итерацию

Function Declaration
→ функция доступна заранее

Function Expression
→ переменная хранит ссылку на обычную функцию

Arrow
→ другая семантика:
  no own this
  no own arguments
  no [[Construct]]
  no own .prototype
```