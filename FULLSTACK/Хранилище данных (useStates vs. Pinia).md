#nuxt-state #nuxt-pinia

Хранение данных предусмотрено в Nuxt 3 в специальном хранилище, доступ к которому обеспечивается через функцию **useState**, базирующемся на Vuex. Размещение states производится в файле `composables/states.ts`. Примеры:

```ts
export const useCounter = () => useState<number>("counter", () => 0);
export const useColor = () => useState<string>("color", () => "green");
```

В примерах используются хранящиеся в хранилище переменные со счетчиком и цветом, доступ к которым обеспечивается из любой точки приложения.

Особенности:
1. use-функции не требуют импорта;
2. при изменениях синхронизируются во всех местах, где используются;
3. ==способ использования==:

	```vue
	<script setup lang='ts'>
		const counter = useCounter();
	</script>
	
	<template>
		<div>
			Counter: {{ counter }}:
			<button @click="counter++"> + </button>
			<button @click="counter--"> - </button>
		</div>
	</template>
	```

4. ==Альтернатива== - использование модуля **Pinia** ([@pinia/nuxt](https://nuxt.com/modules/pinia)], устанавливаемого из глобальной библиотеки модулей Nuxt.js. Отличия можно прочесть подробно в [статье](https://www.vuemastery.com/blog/nuxt-3-state-mangement-pinia-vs-usestate/) и описать двумя пунктами:
	* для *больших приложений* лучше использовать **Pinia**;
	* для *маленьких приложений* более подходящим является **useState**;
	* использование библиотеки Pinia:
		  ![[Хранилище Pinia]] 