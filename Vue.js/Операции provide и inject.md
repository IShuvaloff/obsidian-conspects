#provide #inject

Во вью передача данных от родителя к потомку может производиться двумя способами:
1. ступенчатая сверху вниз - через `props`;
2. с помощью инжектов. 

==Порядок передачи через provide/inject==:
1. в родительском компоненте добавить переменную в блок **provide: {...}** (Options API) или передав в функцию **provide(...)** (Composition API):
   
```ts
	// options API
	provide: {
		location: 'Северный полюс',
		geolocation: {
		  longitude: 90,
		  latitude: 135
		}
	}

	// composition API
	setup() {
		provide('location', 'Северный полюс')
		provide('geolocation', {
		  longitude: 90,
		  latitude: 135
		})
	}
```
   
2. в любом из дочерних компонентов извлечь переменную в блоке **inject: [...]** (Options API) или с помощью функции **inject(...)**:
   
```ts
	// options API
	inject: ['location', 'geolocation']

	// composition API
	setup() {
	    const userLocation = inject('location', 'Вселенная')
	    const userGeolocation = inject('geolocation')
	}
```
   
3. по умолчанию передаваемые переменные не реактивные. Чтобы добавить реактивность, требуется передать внутрь `provide` прокси-переменную, обернутую в **ref** (обычные данные) или **reactive** (объект):
   
```ts
setup() {
	const location = ref('Северный полюс')
	const geolocation = reactive({
	  longitude: 90,
	  latitude: 135
	})
	
	provide('location', location)
	provide('geolocation', geolocation)
}
```
   
4. изменение полученных реактивных свойств см. в [оф. документации](https://v3.ru.vuejs.org/ru/guide/composition-api-provide-inject.html#%D0%B8%D0%B7%D0%BC%D0%B5%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D1%80%D0%B5%D0%B0%D0%BA%D1%82%D0%B8%D0%B2%D0%BD%D1%8B%D1%85-%D1%81%D0%B2%D0%BE%D0%B8%D1%81%D1%82%D0%B2).