
|**Команда**|**Объяснение**|
|---|---|
|`Model::find($id)`|найти модель по PK|
|`Model::findOrFail($id)`|найти или выбросить 404-подобное исключение|
|`->where(...)`|добавить условие запроса|
|`->whereNull(...)`|значение колонки пустое|
|`->when(condition, callback)`|добавь кусок к SQL-запросу только если условие истинно|
|`->get()`|выполнить query и получить Collection|
|`->first()`|получить первую запись в коллекции|
|`->create([...])`|создать запись|
|`$parent->children()->create([...])`|создать дочернюю запись через parent с автоматической установкой FK|
|`$query->whereHas()`|фильтровать parent по условию relation|
|`$model->relation()`|Relation/query object, например, HasMany или BelongsTo - можно продолжить запрос|
|`$model->relation`|загруженная связанная Model/Collection - фактический результат запроса|
|`->with('student')`|eager loading: заранее загрузить relation для набора models; помогает избежать N+1, не равен обязательному ручному SQL JOIN|
|N+1|один query на parent records плюс отдельный query для relation каждой строки; обычно исправляется eager loading|
|`ILIKE`|PostgreSQL case-insensitive pattern matching; DB-specific оператор, не универсальный SQL Laravel API|
|`->paginate($perPage)`|вернуть `LengthAwarePaginator`: текущие `data` и metadata (`total`, `current_page`, `per_page`, links)|
|`->withQueryString()`|сохранить текущие query params в URLs paginator|
|`router.reload()`|Inertia visit текущего URL; можно запросить свежие props без full browser reload|
|partial reload / `only`|на same-page Inertia visit запросить только перечисленные top-level page props, например `only: ['relationships']`|
|`preserveScroll` / `preserveState` / `replace`|не сбрасывать scroll / сохранить local state page / заменить текущую запись browser history|
|`return back()`|redirect response после Inertia mutation; Inertia следует ему новым GET и получает fresh props|
|`useHttp`|request-oriented API текущего Inertia starter kit: Promise-based request для локального `try/catch/finally`, а не Inertia navigation|
|`204 No Content`|успешная HTTP operation без response body; не означает, что backend ничего не сделал|
|optimistic UI / rollback|сначала показать ожидаемый local result, затем оставить его при success или вернуть previous state при error|
|pending state / double submit|state выполняющегося action; disabled/guard не дают повторно отправить ту же mutation|
|`try/catch/finally`|success path / обработка rejection и rollback / гарантированная очистка pending state|
|`HttpResponseError`|ошибка request layer с HTTP response; в текущем `useHttp` flow её обрабатывают в `catch`|
|`JSON.parse(response.data)`|нужен в текущем helper, потому что error body приходит строкой JSON; parse должен быть защищён `try/catch` и fallback|
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
