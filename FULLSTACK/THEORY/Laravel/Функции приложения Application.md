#app #application #container

Функции, действительные для объекта глобального приложения *Application* и объектов *Container*:
1. `app()->singleton( SomeClass::class, function($app) { return new SomeClass(); } )` - возвращает объект указанного класс, созданный только один раз (при первом обращении к функции singleton). Если идет повторный вызов singleton, идет поиск данного объекта в приложении Application, и при наличии возвращается только он;
2. 