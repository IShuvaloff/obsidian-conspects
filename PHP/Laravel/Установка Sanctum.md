#sanctum

Система быстрой и безопасной авторизации. В тесной связке с фронтендом. [Установка](https://laravel.com/docs/12.x/sanctum) (см. блок конфигурации SPA-приложения, т.к. используем Vue):
1. `php artisan install:api` - устанавливает санктум, выполняет миграции и другие операции;
2. добавить в `AppServiceProvider` код в метод `boot`:
   
	```php
	use App\Models\Sanctum\PersonalAccessToken;
	use Laravel\Sanctum\Sanctum;
	
	public function boot(): void {
	    Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
	}
	```

1. добавить в файл `/bootstrap/app.php` запрос `statefulApi()`:

	```php
	->withMiddleware(function (Middleware $middleware): void {
		...
		$middleware->statefulApi();
		...
	})
	```

1. также добавить в этот файл (В СЛУЧАЕ ОТСУТСТВИЯ, что маловероятно, т.к. появляется с добавлением первого api-маршрута) переменную с апи-маршрутами:

	```php
	->withRouting(
		...
		api: __DIR__ . '/../routes/api.php',
		...
	)
	```

2. установить пакет для быстрого выполнения REST API запросов с удобной встроенной системой проверок авторизации `npm i axios`;
3. создать файл `/resources/js/lib/axios.ts`:
   
	```ts
	import axios from 'axios';
	
	axios.defaults.withCredentials = true;
	axios.defaults.withXSRFToken = true;
	axios.defaults.headers.common['Accept'] = 'application/json';
	
	// если хочешь, можно зафиксировать baseURL под APP_URL
	axios.defaults.baseURL = import.meta.env.VITE_API_URL ?? window.location.origin;
	
	export default axios;
	```
   
4. можно импортировать `import axios from '@/lib/axios'`, а можно добавить глобальную переменную в `/resources/js/app.ts`:

	```ts
	import axios from '@/lib/axios';
	// либо сразу здесь, либо внутри функции setup:
	createInertiaApp({
		...
		setup({ el, App, props, plugin }) {
	        (window as any).axios = axios; // назначаем глобальную переменную axios
	        createApp({ render: () => h(App, props) })
	            .use(plugin)
	            .mount(el);
	    },
	    ...
	});
	```

5. использование при авторизации - всегда требуется вначале сделать запрос на получение csrf-токена и запись его в куку. После этого данный токен будет подключаться ко всем уходящим на сервер запросам => вызываем `/login` и далее выполняем любой запрос:

	```ts
	import axios from '@/lib/axios';
	
	export async function login(email: string, password: string) {
	  // шаг 1: инициализировать CSRF
	  await axios.get('/sanctum/csrf-cookie');
	
	  // шаг 2: логин
	  await axios.post('/login', { email, password });
	
	  // шаг 3: получить пользователя
	  const { data: user } = await axios.get('/api/user');
	  return user;
	}
	```

6. проверка авторизации будет проходить на любом запросе, который на сервере в файле `/routes/api.php` будет обернут в миддлвар санктума (например, получение информации о пользователе):

	```php
	Route::get('/user', function (Request $request) {
	    return $request->user();
	})->middleware('auth:sanctum');
	```

7. проверить авторизацию можно на встроенной системе авторизации, стоящей по умолчанию в проекте, зарегистрировав пользователя, затем залогинившись и после этого:
	- проверить в DevToolz -> Application -> Cookies для текущего сайта наличие XSRF-TOKEN и localhost-session (вместо локалхоста будет название приложения);
	- вбив в консоль (после того, как выставили axios глобальным) строку `await axios.get('/api/user');` - она должна вывести объект текущего пользователя. 