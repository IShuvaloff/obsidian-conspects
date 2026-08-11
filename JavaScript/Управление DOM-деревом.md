
Материалы можно почитать [здесь](https://learn.javascript.ru/dom-navigation).
1. Поиск **по вышестоящим**:
	1. непосредственный родитель элемента:
	   ```js
	   let parent = elem.parentElement;
		```
	2. [ближайший из предков с заданным селектором](https://learn.javascript.ru/searching-elements-dom#closest):
	   ```js
	   let parent = elem.closest('div')
		```
	1. 