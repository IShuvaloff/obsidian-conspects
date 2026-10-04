#group #group-hover #group-focus #group-focus-within #group-focus-visible #group-focus-active #group-disabled #group-active #group-open #group-data #group-aria

`group` - служебный класс Tailwind для передачи состояния родительского элемента его потомкам.

```html
<div class="group">
  <button class="opacity-0 group-hover:opacity-100">
    Удалить
  </button>
</div>
```

`group-hover:opacity-100` означает: когда курсор находится над родителем с классом `group`, дочерняя кнопка становится видимой.

## Частые варианты

- `group-hover:` - курсор над родителем.
- `group-focus:` - фокус на родителе.
- `group-focus-within:` - фокус на родителе или любом его потомке.
- `group-focus-visible:` - фокус с клавиатуры.
- `group-active:` - родитель нажат.
- `group-disabled:` - родитель disabled.
- `group-open:` - родительский `<details>` раскрыт.
- `group-data-[state=value]:` - у родителя есть `data-state="value"`.
- `group-aria-[expanded=true]:` - у родителя есть `aria-expanded="true"`.

После `group-*:` указывается обычный класс Tailwind: `opacity-*`, `text-*`, `bg-*`, `rotate-*`, `scale-*`, `block`, `flex` и другие.

```html
<div class="group">
  <span class="group-hover:text-[var(--blue)]">Название</span>
  <svg class="group-hover:rotate-180" />
  <button class="opacity-0 group-hover:opacity-100 group-focus-within:opacity-100">
    Удалить
  </button>
</div>
```

Для скрытых действий обычно достаточно:

```html
<div class="opacity-0 group-hover:opacity-100 group-focus-within:opacity-100">...</div>
```

На мобильных, где нет hover, действие лучше показывать сразу:

```html
<div class="opacity-100 md:opacity-0 md:group-hover:opacity-100">...</div>
```