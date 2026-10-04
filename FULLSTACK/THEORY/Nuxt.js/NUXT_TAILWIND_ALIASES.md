# Aliases Nuxt And Tailwind

В проекте используются два вида алиасов, поскольку TypeScript/Vue и директивы Tailwind CSS обрабатываются разными резолверами.

## Nuxt And TypeScript

В `nuxt.config.ts` объявлены алиасы Vite/Nuxt:

- `@assets` -> `app/assets`;
- `@styles` -> `app/assets/css`.

```json
alias: {
	"@assets": fileURLToPath(new URL("./app/assets", import.meta.url)),
	"@styles": fileURLToPath(new URL("./app/assets/css", import.meta.url)),
},
```

Используйте их в `.ts`, `.vue` и обычных CSS-импортах, которые проходят через Nuxt/Vite:

```ts
import logoUrl from "@assets/logo.svg";
import "@styles/main.css";
```

Стандартные Nuxt-алиасы `@/` и `~/` также указывают на `app`.

## Tailwind CSS

`@reference`, `@import` и другие CSS-директивы Tailwind не используют Nuxt-алиасы. Для них в `package.json` настроены package imports:

- `#assets/*` -> `app/assets/*`;
- `#styles/*` -> `app/assets/css/*`.

```json
"imports": {
	"#assets/*": "./app/assets/*",
	"#styles/*": "./app/assets/css/*"
},
```

Для `@apply` или `@variant` в scoped CSS Vue-компонента используйте:

```vue
<style lang="postcss" scoped>
@reference "#styles/main.css";

.action {
    @apply transition-colors duration-200;
}
</style>
```

`@tailwindcss/postcss` подключен в `nuxt.config.ts`, поэтому эти директивы обрабатываются и внутри Vue SFC. Для вложенных правил доступен `postcss-nested`.
