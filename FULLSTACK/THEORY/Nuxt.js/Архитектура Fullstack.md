#app

---

## Базовая архитектура фронтенда:

```r
app/
├── assets/              # стили, изображения, шрифты
├── components/          # UI-компоненты
│   ├── UI/              # атомарные (Button, Input, Modal и т.д.)
│   ├── common/          # часто используемые (Header, Footer)
│   ├── feature/         # фича-компоненты
│   └── layout/          # специфичные для layout
├── composables/         # вся бизнес-логика и переиспользуемая логика
├── features/            # (при больших проектах) — feature-sliced design
├── layouts/             # layouts
├── middleware/          # middleware (auth, admin и т.д.)
├── pages/               # страницы (file-based routing)
├── plugins/             # client/server plugins
├── server/
│   ├── api/             # ← здесь вся работа с внешними API (самое важное!)
│   ├── routes/          # кастомные Nitro routes (если нужны)
│   ├── middleware/      # server middleware
│   ├── utils/           # server-only утилиты
│   └── plugins/         # Nitro plugins
├── stores/              # Pinia stores (только если действительно нужно глобальное состояние)
├── utils/               # client + server утилиты
├── types/               # глобальные TypeScript типы
└── constants/
```

#### Пример серверной архитектуры

```r
server/api/
├── auth/
│   ├── login.post.ts
│   └── me.get.ts
├── products/
│   ├── index.get.ts
│   ├── [id].get.ts
│   └── search.get.ts
├── cart/
│   └── add.post.ts
└── utils/
    └── fetchWithAuth.ts   # обёртка с токеном
```



## Базовая логика

|Слой|Где лежит|Что содержит|Кто может использовать|
|---|---|---|---|
|**Presentation**|`components/`, `pages/`|Только UI и минимальная оркестрация|Компоненты|
|**Business Logic**|`composables/`|Вся логика, валидация, трансформации|Везде|
|**Data Fetching**|`composables/useApi*` + `server/api/`|Запросы к API, кэширование, обработка ошибок|Только через composables|
|**State**|Pinia (stores/) или `useNuxtData` + `useState`|Глобальное состояние|По необходимости|
|**Server-only**|`server/`|API-ключи, sensitive логика, rate limiting|Только сервер|

**Главное правило**:

> «Компонент не должен знать, откуда приходят данные. Он должен получать их через composable.»