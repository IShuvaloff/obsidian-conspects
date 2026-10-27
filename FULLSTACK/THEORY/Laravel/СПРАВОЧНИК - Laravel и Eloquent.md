
|**Команда**|**Объяснение**|
|---|---|
|`Model::find($id)`|найти модель по PK|
|`Model::findOrFail($id)`|найти или выбросить 404-подобное исключение|
|`->where(...)`|добавить условие запроса|
|`->when(condition, callback)`|добавь кусок query только если условие истинно|
|`->with('student')`|eager loading relation (подключить объект к таблице HasMany по PK)|
|`->get()`|выполнить query и получить Collection|
|`->first()`|получить первую запись в коллекции|
|`->create([...])`|создать запись|
|`$parent->children()->create([...])`|создать дочернюю запись через parent с автоматической установкой FK|
|`$query->whereHas()`|фильтровать parent по условию relation|
|`$model->relation()`|Relation/query object, например, HasMany или BelongsTo - можно продолжить запрос|
|`$model->relation`|загруженная связанная Model/Collection - фактический результат запроса|
|`app(SomeClass::class)`|resolve из Service Container|
|`DB::transaction(...)`|атомарно выполнить несколько DB operations|
|||
|||
|||
|||
|||
|||
|||
|||