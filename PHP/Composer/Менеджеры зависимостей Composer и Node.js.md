#composer #nodejs 

**Composer** - менеджер зависимостей для PHP. Технически, это аналог Node.js для JS. 

==Сравнение Composer и Node.js:==

Язык | JavaScript | PHP
---: | --- | ---
Менеджер | [[Менеджер зависимостей Node.js\|Node.js]] | Composer
Сайт с библиотеками | [npmjs.com](https://www.npmjs.com/) | [Packagist.org](https://packagist.org/)
[Установка библиотеки](https://getcomposer.org/doc/01-basic-usage.md#basic-usage) | `npm i [-D] library` | <ol><li>прописать название и версию библиотеки в файл `composer.json`: ![[Pasted image 20231026144335.png]]</li><li>вызвать команду `php composer.phar update`^[это пропишет конкретную рабочую версию библиотеки в файл `composer.lock`]</li><ol>
Установленные библиотеки | `package.json` | `composer.json`
Конкретные версии установленных библиотек | `package.lock` | `composer.lock`
Папка с файлами библиотек | `/node_modules` | `/vendor` 
Импорт библиотеки в модуль | `import library from 'library';` | `require __DIR__ . '/vendor/autoload.php';`
Переключатель версий менеджера^[позволяет быстро переключать версии менеджера пакетов для работы в различных проектах] | Менеджер&nbsp;`nvm` | 