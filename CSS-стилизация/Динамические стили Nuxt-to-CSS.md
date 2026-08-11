#v-bind #useCssVar

Способы задания стилей в Vue/Nuxt (классами, условными классами, массивом классов, объектами с полями-классами, инлайновыми стилями)
   
```ts
const isAvailable = ref<boolean>(true);
const classVar = ref('className');
const classes = ref({
   'classNameNew': isAvailable,
   'classNameOld': !isAvailable,
});

const fontSize = ref(30);
const objectFit = ref('cover');
const styles = ref({
   objectFit: objectFit.value,
   fontSize: fontSize.value,
});
```
```html
<div class="className"> // стандарт, название класса
<div :class="classVar"> // переменная с названием класса
<div :class="{'classOld classNew': isAvailable}"> // условные классы
<div :class="{[classVar]: isAvailable}"> // условный класс из переменной
<div :class="[classVar, {'className': isAvailable}]"> // комбинация
<div :class="classes"> // в виде готового объекта

<div :style="{fontSize: fontSize + 'px', 'object-fit': objectFit}">
<div :style="styles">

<!-- все способы можно комбинировать -->
<div 
   class="one" 
   :class="['two', classVar, {'className': isAvailable}]" 
   style="background-color:red;" 
   :style="[styles, otherStyles]"
></div>
```
    
Чтобы ==напрямую в **блоке со стилями**== в компоненте Nuxt 3 использовать значение **переменной** из блока **script**, можно поступить двумя способами:

1. использовать плагин [`useCssVar`](https://vueuse.org/core/useCssVar/), где мы создаем в скриптах переменную через привязку ее к CSS-переменной, затем записываем в эту переменную нужное значение, и после этого используем в стилях эту переменную (см. пример в проекте Завидово);
2. использовать в стилях переменную из скриптов напрямую через [`v-bind`](https://ru.vuejs.org/api/sfc-css-features.html#v-bind-in-css):
   ```css
   width: v-bind(widthVariable);
	```

	Особенности:
	* лучше хранить в переменной сразу полное значение css-свойства (включая размерности);
	* можно цеплять поля объектов, например, `width: v-bind('obj.field');`, но для этого не забываем оборачивать выражение в одинарные кавычки;
	* формируемое CSS-свойство таким образом является реактивным и будет изменяться с изменением значения переменной;