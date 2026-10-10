# Storybook в Vue 3 / Nuxt 3–4

> [!summary]
> **Storybook** — изолированная среда для разработки, просмотра и документирования UI-компонентов в их конкретных состояниях. Он помогает развивать компонент без запуска всей страницы, бэкенда и реальных данных. Сам по себе Storybook — не замена unit/E2E-тестам; но истории можно превратить в проверяемые interaction-тесты.

```text
Компонент + props + mocks + state
              ↓
          Story (история)
              ↓
Controls / Docs / ручная проверка
              ↓
play() + expect() → автоматическая interaction-проверка
```

## 1. Зачем он нужен

Обычная страница приложения часто требует роутинга, авторизации, API, store и конкретных данных. Из-за этого маленький UI-компонент трудно быстро проверить во всех состояниях.

Storybook позволяет описать эти состояния явно:

```text
Button
├─ Default
├─ Loading
├─ Disabled
└─ With long label

UserCard
├─ Default
├─ Admin online
├─ Without avatar
└─ Error / empty state — если это ответственность компонента
```

Практическая ценность:

- быстрее верстать и отлаживать component API;
- видеть `props`, `slots`, `emits`, responsive-состояния и крайние случаи;
- документировать варианты компонента для команды;
- вручную проверять UI до подключения страницы;
- постепенно добавлять interaction-, accessibility- и visual-тесты.

> [!warning] Что Storybook не заменяет
> Он не доказывает, что реальный API, авторизация, серверный рендеринг, роутинг всего приложения или платёжный сценарий работают в production. Для этого всё ещё нужны unit/integration/E2E-тесты и ручная проверка интеграции.

## 2. Как он связан с тестированием

Есть три разных уровня, которые легко спутать:

| Уровень | Что происходит | Это автоматический тест? |
|---|---|---|
| Story + Controls | Человек вручную меняет `args` и смотрит на UI | Нет |
| Render test | Storybook проверяет, что story успешно отрисовывается | Да, но базовый |
| Interaction test | `play()` имитирует действия и делает `expect()` | Да |

Например, показ кнопки в состоянии `disabled` — хорошая story. Проверка, что нажатие не вызывает `emit('save')`, — interaction-тест.

## 3. Установка для Nuxt

Для Nuxt нужно использовать официальный модуль `@nuxtjs/storybook`, а не настраивать чистый `@storybook/vue3-vite` как отдельное Vue-приложение.

```bash
npx nuxi@latest module add storybook
```

Модуль поддерживает Nuxt 3/4 и Storybook 10. После установки сначала используй обычный dev server Nuxt:

```bash
npm run dev
```

Стандартный маршрут модуля — `/_storybook`; настройки можно менять только при реальной необходимости:

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/storybook'],

  storybook: {
    host: 'http://localhost',
    route: '/_storybook',
    port: 6006,
  },
})
```

> [!tip] Не добавляй `--legacy-peer-deps` «на всякий случай»
> Это обход конфликта зависимостей, а не нормальная часть установки. Сначала проверить версии Node, Nuxt, Storybook и текст конфликта. Временный workaround допустим только после понимания, какую совместимость он обходит.

> [!note] Nuxt-контекст
> Модуль интегрирует Storybook с Nuxt. Но внешние зависимости компонента всё равно должны быть контролируемыми: API-запросы, store, feature flags, текущий маршрут и глобальные плагины часто требуют mock, decorator или тестовую настройку. «Компонент запускается в Storybook» не означает, что ему безопасно ходить в настоящий API.

## 4. Основные сущности

| Термин | Смысл |
|---|---|
| Component | Сам `.vue`-компонент, который нужно показать или проверить |
| Story | Конкретное начальное состояние компонента |
| `args` | Входные данные story; обычно props и callbacks для emits |
| Controls | Панель для ручного изменения `args` |
| `argTypes` | Описание того, как показывать аргумент в Controls/Docs |
| `render` | Кастомный способ отрисовать story, например чтобы передать slot |
| `play` | Сценарий действий пользователя после рендера |
| `expect` | Проверка ожидаемого результата в `play` |
| `fn()` | Spy-функция, которой можно проверить emit/callback |

## 5. Комплексный пример: компонент и истории

### Компонент `app/components/UserCard.vue`

```vue
<script setup lang="ts">
type UserRole = 'admin' | 'user' | 'moderator'

defineProps<{
  username: string
  avatarUrl?: string
  role: UserRole
  isOnline?: boolean
}>()

const emit = defineEmits<{
  actionClick: [type: 'message' | 'block']
}>()
</script>

<template>
  <article class="user-card" :class="{ 'user-card--online': isOnline }">
    <header class="user-card__header">
      <img
        :src="avatarUrl || 'https://placehold.co/100'"
        :alt="`Аватар ${username}`"
        class="user-card__avatar"
      >
      <span class="user-card__badge" :class="`user-card__badge--${role}`">
        {{ role }}
      </span>
    </header>

    <div class="user-card__content">
      <h3>{{ username }}</h3>
      <slot />
    </div>

    <div class="user-card__actions">
      <button type="button" @click="emit('actionClick', 'message')">
        Написать
      </button>
      <button type="button" @click="emit('actionClick', 'block')">
        Блок
      </button>
    </div>

    <footer class="user-card__footer">
      <!-- Для Storybook разумнее проверить href, а не навигацию всего приложения. -->
      <NuxtLink :to="`/users/${username}`">Открыть профиль</NuxtLink>
    </footer>
  </article>
</template>
```

### Истории `app/components/UserCard.stories.ts`

Storybook 10 использует пакет `storybook/test` для `fn` и `expect`.

```ts
import type { Meta, StoryObj } from '@storybook/vue3'
import { expect, fn } from 'storybook/test'

import UserCard from './UserCard.vue'

const meta = {
  title: 'UI/UserCard',
  component: UserCard,
  args: {
    onActionClick: fn(),
  },
  argTypes: {
    role: {
      control: 'select',
      options: ['user', 'moderator', 'admin'],
      description: 'Роль пользователя в системе',
    },
    isOnline: {
      control: 'boolean',
      description: 'Показывает online-состояние',
    },
  },
} satisfies Meta<typeof UserCard>

export default meta
type Story = StoryObj<typeof meta>

export const Default: Story = {
  args: {
    username: 'Иван Иванов',
    role: 'user',
    isOnline: false,
    avatarUrl: 'https://placehold.co/100',
  },
}

export const AdminOnlineWithSlot: Story = {
  args: {
    ...Default.args,
    username: 'Алексей Админ',
    role: 'admin',
    isOnline: true,
  },
  render: (args) => ({
    components: { UserCard },
    setup: () => ({ args }),
    template: `
      <UserCard v-bind="args">
        <p>Доступ: главная панель, базы данных</p>
      </UserCard>
    `,
  }),
}

export const EmitsMessage: Story = {
  args: Default.args,
  play: async ({ args, canvas, userEvent }) => {
    await userEvent.click(
      canvas.getByRole('button', { name: 'Написать' }),
    )

    await expect(args.onActionClick).toHaveBeenCalledWith('message')
  },
}
```

Что проверяет последняя story:

```text
рендер компонента
→ пользователь нажимает «Написать»
→ компонент отправляет ожидаемый emit
```

Это уже не просто демонстрация UI: при запуске test runner assertion может упасть, если поведение сломано.

## 6. Что проверять в Storybook

### Ручная проверка

- обязательные, optional и длинные props;
- loading / empty / error / disabled состояния — если ими владеет компонент;
- адаптивность через viewport;
- читаемость, focus, названия кнопок, alt-текст и контраст;
- slots, emits, варианты темы и локали;
- отсутствие случайных обращений к production API.

### Автоматизировать постепенно

1. Начать с render stories для важных вариантов.
2. Добавить `play()` к критическим интеракциям: click, input, validation, emit.
3. Добавить accessibility-проверки.
4. Для дизайн-системы или критичных UI использовать visual regression tests.
5. Запускать Storybook tests вместе с обычными тестами в CI.

> [!tip] Хорошая граница
> Storybook лучше всего подходит для компонентов и небольших связок UI. Полный пользовательский путь — login → API → database → redirect — обычно остаётся задачей E2E-теста.

## 7. Частые ошибки

| Ошибка | Почему плохо | Что делать |
|---|---|---|
| Считать Controls тестами | Controls лишь меняют входные данные вручную | Для поведения писать `play()` и `expect()` |
| Рендерить реальный API | История становится нестабильной, медленной и может менять данные | Mock API/модули и передавать fixture-данные |
| Делать один универсальный компонент-комбайн | Storybook скрывает плохую архитектуру, но не исправляет её | Разделять domain-логику и переиспользуемый UI |
| Кликать `NuxtLink`, ожидая полноценную навигацию | Story не обязана иметь весь роутинг приложения и нужную страницу | Проверять контракт ссылки или покрывать маршрут E2E-тестом |
| Использовать устаревший `@storybook/test` | В актуальном Storybook используется `storybook/test` | Импортировать `fn` и `expect` из `storybook/test` |
| Не сбрасывать общий state | Одна story может загрязнить следующую | Настроить reset/mock cleanup в preview `beforeEach` |

## 8. Краткое резюме

```text
Storybook
→ изолированная разработка и документация UI

Story
→ состояние компонента

args
→ props и callbacks story

Controls
→ ручная проверка вариантов

play + userEvent + expect
→ interaction-тест

fn()
→ проверка emit/callback

Storybook
≠ замена integration/E2E-тестов
```

## 9. Связи и дальнейшие темы

- Предпосылки: [[Композаблы]], [[Прокидывание пропсов (Provide-Inject)]], [[TypeScript/5. TS in Vue and Nuxt|TypeScript в Vue/Nuxt]].
- Дальше: Vue Test Utils/Vitest, Playwright, accessibility и visual regression testing.
- Официальные источники: [Nuxt Storybook module](https://github.com/nuxt-modules/storybook), [Storybook interaction tests](https://storybook.js.org/docs/writing-tests/interaction-testing).

## Для проверки знаний

### Термины

- Storybook, story, `args`, Controls, `argTypes`, `render`, `play`, `fn`, interaction test.

### Что нужно уметь объяснить

- Почему Storybook позволяет развивать UI быстрее, но не заменяет E2E.
- Чем ручная проверка через Controls отличается от interaction-теста.
- Почему API/store/route нужно mock-ать или контролировать в story.

### Типичные ошибки

- «Если компонент открылся в Storybook, значит приложение полностью протестировано».
- «Controls автоматически проверяют поведение компонента».
- «Storybook должен выполнять реальные запросы к production API».

### Мини-практика

- Создать story для `Loading` и `Empty` состояний компонента списка.
- Добавить `play()` к кнопке и проверить ожидаемый emit через `fn()` и `expect()`.
- Объяснить, почему проверка реального перехода по `NuxtLink` относится скорее к E2E.

### Связи

- Предпосылки: [[Композаблы]], [[Прокидывание пропсов (Provide-Inject)]], [[TypeScript/5. TS in Vue and Nuxt|TypeScript в Vue/Nuxt]].
- Дальше: Vitest, Vue Test Utils, Playwright, accessibility и visual regression testing.
