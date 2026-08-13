# Краткий конспект

## 1. Что такое closure

Closure — функция, которая сохраняет доступ к внешнему Lexical Environment даже после завершения внешней функции.

```js
function create(value) {
  return () => console.log(value)
}

const fn = create(10)
fn() // 10
```

Схема:

```text
fn
↓
returned function
↓
Lexical Environment create
↓
value = 10
```

Execution Context `create()` уже ушёл из Call Stack, но его LE продолжает жить, пока доступен через closure.

---

## 2. Closure сохраняет binding, а не копию значения

```js
function create() {
  let value = 10

  const show = () => console.log(value)

  value = 20

  return show
}

create()() // 20
```

```text
show
↓
не хранит "10"
↓
хранит доступ к binding value
↓
value: 10 → 20
```

---

## 3. Каждый вызов factory → своё окружение

```js
const a = createCounter()
const b = createCounter()
```

```text
a → LE #1 → count₁
b → LE #2 → count₂
```

Изменения одного счётчика не влияют на другой.

Но несколько closures, созданных **в одном вызове**, могут работать с одной общей переменной:

```text
        create LE
        value
        ↑   ↑
       get set
```

---

## 4. `var` в цикле

```js
const functions = []

for (var i = 0; i < 3; i++) {
  functions.push(() => console.log(i))
}

functions[0]()
functions[1]()
functions[2]()
```

Результат:

```text
3
3
3
```

Почему:

```text
var
→ один binding i на весь цикл

i: 0 → 1 → 2 → 3

fn0 ─┐
fn1 ─┼──► один i = 3
fn2 ─┘
```

Проблема не в `setTimeout` как таковом.

Главное:

```text
один binding
+
функции вызываются после изменения binding
```

---

## 5. `let` в заголовке `for`

```js
for (let i = 0; i < 3; i++) {
  functions.push(() => console.log(i))
}
```

Для каждой итерации создаётся отдельный binding:

```text
fn0 → i₀ = 0
fn1 → i₁ = 1
fn2 → i₂ = 2
```

Поэтому:

```text
0
1
2
```

Это специальная **per-iteration semantics** `let` в `for`.

---

## 6. Внешний `let` — уже другой случай

```js
let i = 0

for (; i < 3; i++) {
  functions.push(() => console.log(i))
}
```

Здесь binding только один:

```text
один i
0 → 1 → 2 → 3
```

Поэтому после цикла:

```text
3
3
3
```

Ключевая разница:

```text
for (let i = ...)
→ новый binding на каждую итерацию

let i = ...
for (...)
→ один внешний binding
```

---

## 7. Создание closure ≠ вызов closure

```js
functions[0]()
functions[0]()
functions[1]()
```

Это:

```text
2 созданных closure
3 вызова
```

Повторный вызов:

```text
НЕ создаёт новое внешнее LE
↓
работает с уже сохранённым состоянием
```

---

## 8. Closure и память

```js
function create() {
  const data = { value: 100 }

  return () => data.value
}

let fn = create()
```

Пока `fn` достижима:

```text
fn
↓
closure
↓
create LE
↓
data
```

`data` тоже остаётся достижимым.

Если все ссылки на closure исчезнут:

```js
fn = null
```

и других ссылок нет, окружение и связанные данные **могут быть собраны GC**.

---

# Быстрый алгоритм

```text
Вижу closure
↓
ГДЕ функция была объявлена?
↓
какой внешний LE она сохраняет?
↓
какие bindings находятся там?
↓
binding один или отдельный для каждого вызова/итерации?
```

## Главные формулы

```text
Closure
→ сохраняет доступ к binding
→ не копию значения
```

```text
factory вызвали два раза
→ два независимых LE
```

```text
var в цикле
→ один binding
```

```text
for (let i = ...)
→ binding на каждую итерацию
```

```text
Lexical Chain
→ зависит от места ОБЪЯВЛЕНИЯ функции

Call Stack
→ зависит от того, КТО КОГО ВЫЗВАЛ
```