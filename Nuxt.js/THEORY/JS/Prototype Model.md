
# 🚀 30-second Cheat Sheet

## Главная схема

```text
instance
↓ [[Prototype]]
Constructor.prototype
↓ [[Prototype]]
Object.prototype
↓
null
```

## `.prototype` vs `[[Prototype]]`

```text
Constructor.prototype
→ обычное свойство constructor function / class (т.к. function - это тоже объект со свойствами)
→ объект, который станет прототипом экземпляров
```

```text
obj.[[Prototype]]
→ внутренняя ссылка конкретного объекта
→ куда JS идёт искать свойство дальше
```

При:

```js
const user = new User()
```

получаем:

```js
Object.getPrototypeOf(user) === User.prototype
// true
```

---

## Поиск свойства

```text
obj
→ есть property?
→ нет
↓
obj.[[Prototype]]
→ есть property?
→ нет
↓
дальше по prototype chain
↓
null
```

---

## Own vs inherited

```js
Object.hasOwn(obj, key)
```

→ property лежит непосредственно в `obj`.

```js
key in obj
```

→ property есть либо в `obj`, либо где-то выше по prototype chain.

---

## `new`

```text
new User()
↓
1. создаёт новый объект
2. связывает его [[Prototype]] с User.prototype
3. вызывает User с this, равным созданному объекту
4. возвращает объект
```

---

## `class`

```text
class
→ удобный синтаксис поверх prototype model
```

```js
class User {
  constructor(name) {
    this.name = name
  }

  sayHello() {}
}
```

```text
name
→ own property экземпляра

sayHello
→ User.prototype.sayHello
```

---

## `instanceof`

```text
obj instanceof Constructor
≈
есть ли Constructor.prototype
в prototype chain объекта obj
```

---

## Современные API

```text
Object.getPrototypeOf(obj)
→ получить [[Prototype]]

Object.hasOwn(obj, key)
→ проверить own property
```

```text
__proto__
→ исторический accessor к [[Prototype]]

obj.hasOwnProperty(key)
→ старый способ проверки own property
```

<br> 

---

<br>

# Быстрое восстановление за 2–3 минуты

## 1. Функция в JavaScript — тоже объект

```js
function User() {}
```

`User` — function object.

Поэтому у неё могут быть свойства:

```js
User.name
User.length
User.prototype
```

Обычная constructable function имеет внутренний метод:

```text
[[Construct]]
```

который позволяет использовать:

```js
new User()
```

`.prototype` и `[[Construct]]` — разные вещи.

---

## 2. Что такое `User.prototype`

```js
function User() {}
```

У `User` есть свойство:

```js
User.prototype
```

Его значение — объект.

При:

```js
const user = new User()
```

создаётся связь:

```text
user
↓ [[Prototype]]
User.prototype
```

То есть:

```js
Object.getPrototypeOf(user) === User.prototype
```

---

## 3. Prototype chain

```js
user.sayHello()
```

Если собственного `sayHello` у `user` нет:

```text
user
→ sayHello? ❌

User.prototype
→ sayHello? ✅
```

Если не найдено и там:

```text
User.prototype
↓
Object.prototype
↓
null
```

Так работает prototype chain.

---

## 4. Own и inherited properties

```js
function User(name) {
  this.name = name
}

User.prototype.sayHello = function () {}
```

После:

```js
const user = new User('Ilya')
```

структура:

```text
user
├── name                ← own
└── [[Prototype]]
       ↓
   User.prototype
   └── sayHello          ← inherited
```

Проверка:

```js
Object.hasOwn(user, 'name')      // true
Object.hasOwn(user, 'sayHello')  // false
```

Но:

```js
'sayHello' in user // true
```

потому что `in` смотрит всю prototype chain.

---

## 5. Shadowing

```js
User.prototype.role = 'user'

user.role // 'user'
```

Если затем:

```js
user.role = 'admin'
```

получаем:

```text
user
├── role = 'admin'
└── [[Prototype]]
       ↓
   User.prototype
   └── role = 'user'
```

Теперь:

```js
user.role // 'admin'
```

Поиск останавливается на ближайшем найденном property.

---

## 6. Зачем методы кладут в prototype

Плохой в плане дублирования вариант:

```js
function User(name) {
  this.name = name

  this.sayHello = function () {
    console.log(this.name)
  }
}
```

Каждый экземпляр получает отдельную функцию:

```text
user1.sayHello → function #1
user2.sayHello → function #2
```

```js
user1.sayHello === user2.sayHello
// false
```

Через prototype:

```js
User.prototype.sayHello = function () {
  console.log(this.name)
}
```

получаем:

```text
user1 ─┐
       ├──► User.prototype.sayHello
user2 ─┘
```

```js
user1.sayHello === user2.sayHello
// true
```

Данные экземпляров разные, метод общий.

---

## 7. Что делает `new`

```js
function User(name) {
  this.name = name
}

const user = new User('Ilya')
```

Концептуальная модель:

```text
1. создать новый объект

2. новый объект.[[Prototype]]
   → User.prototype

3. вызвать User
   → this = новый объект

4. вернуть новый объект
```

Поэтому:

```js
this.name = name
```

создаёт own property у экземпляра.

Если constructor явно возвращает объект, он может заменить автоматически созданный instance.

Если возвращает primitive или ничего — возвращается созданный instance.

---

## 8. `class` и prototype model

```js
class User {
  constructor(name) {
    this.name = name
  }

  sayHello() {
    return this.name
  }
}
```

Это более удобный синтаксис поверх той же prototype model.

```text
user.name
→ own state

User.prototype.sayHello
→ общий метод
```

Поэтому:

```js
const a = new User('Ilya')
const b = new User('Alex')

a.sayHello === b.sayHello
// true
```

---

## 9. `extends`

```js
class User {
  sayHello() {}
}

class Admin extends User {
  deleteUser() {}
}

const admin = new Admin()
```

Prototype chain:

```text
admin
↓
Admin.prototype
↓
User.prototype
↓
Object.prototype
↓
null
```

Поэтому `admin` получает доступ и к:

```text
Admin.prototype.deleteUser
```

и к:

```text
User.prototype.sayHello
```

---

## 10. `instanceof`

```js
admin instanceof Admin
admin instanceof User
admin instanceof Object
```

могут быть одновременно `true`.

`instanceof` проверяет:

```text
встречается ли Constructor.prototype
в prototype chain объекта?
```

Например:

```text
admin
↓
Admin.prototype      ← instanceof Admin
↓
User.prototype       ← instanceof User
↓
Object.prototype     ← instanceof Object
```

---

## 11. `Object.hasOwn()` vs `hasOwnProperty()`

По смыслу оба проверяют:

```text
является ли property собственным
а не унаследованным
```

Старый вариант:

```js
obj.hasOwnProperty('name')
```

Современный предпочтительный:

```js
Object.hasOwn(obj, 'name')
```

Почему `Object.hasOwn()` надёжнее:

- не зависит от того, есть ли `hasOwnProperty` у самого объекта;
- объект мог переопределить `hasOwnProperty`;
- объект может быть создан без `Object.prototype`.

Например:

```js
const obj = Object.create(null)

obj.name = 'Ilya'

Object.hasOwn(obj, 'name') // true
```

---

## 12. `__proto__` vs `Object.getPrototypeOf()`

```js
Object.getPrototypeOf(user)
```

→ современный стандартный способ получить `[[Prototype]]`.

```js
user.__proto__
```

→ исторический accessor к тому же механизму.

Для понимания DevTools `__proto__` знать нужно, но в рабочем коде предпочтительнее:

```js
Object.getPrototypeOf(user)
```

---

# Итоговая модель

```text
.prototype
→ свойство constructor function / class

[[Prototype]]
→ внутренняя ссылка экземпляра

new
→ связывает instance.[[Prototype]]
  с Constructor.prototype

prototype chain
→ механизм поиска properties

class
→ удобный синтаксис поверх prototype model

instanceof
→ ищет Constructor.prototype в prototype chain

Object.hasOwn
→ проверяет own property

Object.getPrototypeOf
→ получает [[Prototype]]
```