#nuxt-modules #nuxt-routing 

1. Библиотека глобальных модулей - https://nuxt.com/modules. Каждый глобальный nuxt-модуль обозначается префиксом `@nuxt(js)/...`. Примеры таких модулей:
	* [VueUse](https://vueuse.org/) - различные композаблы, используемые в приложении;
	* [Icon](https://nuxt.com/modules/icon) - функционал по оптимизации работы с svg-изображениями;
1. Локальные модули: ...
2. Установка модулей:
	1. стандартная установка через npm;
	2. добавление установленных модулей в файл `nuxt.config.ts` в массив модулей: `modules: [...]`:
		```ts
		modules: [
			"@emanuele-em/nuxt-swipe",
			"nuxt-svgo",
			"./src/modules/gde-poest",
		],
		```
4. Создание папки `modules/` в корне проекта;
5. Создание файла `index.ts` в папке `modules/`;
6. Регистрация модуля в главном файле `index.ts` с помощью функции **`defineNuxtModule`** и расширения внутри глобального списка страниц с помощью функции **`extendPages`**:
	```ts
	import { defineNuxtModule, extendPages } from "@nuxt/kit";
	import { resolve } from "pathe";

	export default defineNuxtModule({
	    setup() {
	        extendPages((pages) => {
	            pages.push(
	                {
	                    name: "otdyhayshim-chem-zanyatsya",
	                    path: "/otdyhayshim/chem-zanyatsya",
	                    file: resolve(__dirname, "./pages/index.vue"),
	                },
	                {
	                    name: "otdyhayshim-chem-zanyatsya-slug",
	                    path: "/otdyhayshim/chem-zanyatsya/:slug",
	                    file: resolve(__dirname, "./pages/duty-detail.vue"),
	                }
	            );
	        });
	    },
	});
	```
	1. расширяем список страниц, добавляя в параметр `pages` нужные модули;
	2. указываем имя, по которому будем вызывать переход по маршруту с помощью атрибута `to` в кнопке или ссылке при нажатии;
	3. указываем путь, который будет отображаться в браузере (вместо *параметров с префиксом двоеточия* будет выставляться переданная строка);
	4. указываем физический путь через функцию **`resolve`**, пропуская только корневой `baseUrl`;
7. Ручной переход по маршрутам см. [[Страницы (pages)|здесь]].