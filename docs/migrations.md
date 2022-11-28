## Migrations

- [Порядок выполнения миграций](#Порядок-выполнения-миграций)
- [Поля миграции с часовыми поясами](#Поля-миграции-с-часовыми-поясами)
- [Типы столбцов миграции базы данных](#Типы-столбцов-миграции-базы-данных)
- [Значения timestamp по умолчанию](#Значения-timestamp-по-умолчанию)
- [Статус миграций](#Статус-миграций)
- [Написание имени миграции с пробелами](#Написание-имени-миграции-с-пробелами)
- [Создать столбец после другого столбца](#Создать-столбец-после-другого-столбца)
- [Создать миграцию для существующей таблицы](#Создать-миграцию-для-существующей-таблицы)
- [Вывод SQL перед запуском миграций](#Вывод-SQL-перед-запуском-миграций)
- [Анонимные миграции](#Анонимные-миграции)
- [Комментарий для столбца](#Комментарий-для-столбца)
- [Проверка на существования таблицы или столбца](#Проверка-на-существования-таблицы-или-столбца)
- [Проверка на наличие перед добавлением или удалением столбца](#Проверка-на-наличие-перед-добавлением-или-удалением-столбца)
- [Переименование поля PostgreSQL](#Переименование-поля-PostgreSQL)

### Порядок выполнения миграций

Если вы хотите изменить порядок выполнения миграций, то просто поменяйте время в названии файла миграции, например, `2023_08_04_070443_create_posts_table.php` измените на `2023_07_04_070443_create_posts_table.php` (внимательно, меняется дата `2023_08_04` to `2023_07_04`).

Они запускаются в алфавитном порядке.

[Вернуться к списку](#Migrations)
___

### Поля миграции с часовыми поясами

Знаете ли, что в миграция доступны не только `timestamps()`, но и `timestampsTz()`, для часовых поясов?

```php
Schema::create('employees', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
    $table->string('email');
    $table->timestampsTz();
});
```

Также есть такие типы для колонок `dateTimeTz()`, `timeTz()`, `timestampTz()`, `softDeletesTz()`.

[Вернуться к списку](#Migrations)
___

### Типы столбцов миграции базы данных

Существует множество любопытных типов полей для создаваемых миграций, приведу парочку в качестве примеров.

```php
$table->geometry('positions');
$table->ipAddress('visitor');
$table->macAddress('device');
$table->point('position');
$table->uuid('id');
```

Посмотреть все доступные типы можно в [официальной документации](https://laravel.com/docs/master/migrations#creating-columns).

[Вернуться к списку](#Migrations)
___

### Значения timestamp по умолчанию

При создании миграции вы можете использовать вместе с `timestamp()` следующие параметры `useCurrent()` и `useCurrentOnUpdate()`, 
они помогут установить `CURRENT_TIMESTAMP` в качестве значения по умолчанию.

```php
$table->timestamp('created_at')->useCurrent();
$table->timestamp('updated_at')->useCurrentOnUpdate();
```

[Вернуться к списку](#Migrations)
___

### Статус миграций

Если вы хотите посмотреть статус выполнения миграций, то нет нужды заходить в таблицу миграций, достаточно ввести команду `php artisan migrate:status`.

Результат выполнения команды:

```
Migration name .......................................................................... Batch / Status  
2023_12_01_000000_create_users_table ........................................................... [1] Ran  
2023_12_02_100000_create_password_resets_table ................................................. [1] Ran  
2023_12_03_000000_create_failed_jobs_table ..................................................... [1] Ran    
```

[Вернуться к списку](#Migrations)
___

### Написание имени миграции с пробелами

When typing `make:migration` command, you don't necessarily have to use underscore `_` symbol between parts, like `create_transactions_table`. You can put the name into quotes and then use spaces instead of underscores.

```php
// Это работает
php artisan make:migration create_transactions_table

// Но это также будет работать
php artisan make:migration "create transactions table"
```

Источник: [Steve O on Twitter](https://twitter.com/stephenoldham/status/1353647972991578120)

[Вернуться к списку](#Migrations)
___

### Создать столбец после другого столбца

_Внимание: Только для MySQL_

Если вы добавляете новый столбец, то не обязательно, чтобы он был последним в списке. Можно указать, после какого столбца он должен быть создан. 

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->after('email');
});
```

Также можно создать новый столбец, чтобы он находился перед указанным столбцом.

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->before('created_at');
});
```

Если же вы хотите, чтобы новый столбец был первым в таблице, используйте метод `first()`.

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('uuid')->first();
});
```

Также метод `after()` позволяет использовать добавление нескольких полей, тем самым объединяя их в группу.

```php
Schema::table('users', function (Blueprint $table) {
    $table->after('remember_token', function ($table){
        $table->string('card_brand')->nullable();
        $table->string('card_last_four', 4)->nullable();
    });
});
```

[Вернуться к списку](#Migrations)
___

### Создать миграцию для существующей таблицы

Если вы хотите создать миграцию для существующей таблицы и хотите, чтобы Laravel 
сгенерировал для вас Schema::table(), то достаточно добавить "\_in_xxxxx_table" or "\_to_xxxxx_table"
в название миграции или использовать параметр "--table", где укажите название вашей таблицы.

По умолчанию `php artisan change_fields_products_table` генерирует пустой класс.

```php
class ChangeFieldsProductsTable extends Migration
{
    public function up()
    {
        //
    }
}
```

Но стоит добавить `in_xxxxx_table` `php artisan make:migration change_fields_in_products_table` и автоматически создастся `Schema::table()` с предварительно заполненным именем таблицы.

```php
class ChangeFieldsProductsTable extends Migration
{
    public function up()
    {
        Schema::table('products', function (Blueprint $table) {
            //
        })
    };
}
```

Also you can specify `--table` parameter `php artisan make:migration whatever_you_want --table=products`

```php
class WhateverYouWant extends Migration
{
    public function up()
    {
        Schema::table('products', function (Blueprint $table) {
            //
        })
    };
}
```

[Вернуться к списку](#Migrations)
___

### Вывод SQL перед запуском миграций

При написании команды `migrate --pretend`, вы можете вывести запрос, который впоследствии будет выполняться. 
Это интересный способ для отладки запросов при необходимости.

```php
php artisan migrate --pretend
```

Совет от [@zarpelon](https://github.com/zarpelon)

[Вернуться к списку](#Migrations)
___

### Анонимные миграции

Команда Laravel выпустила Laravel 8.37 с поддержкой анонимной миграции, 
которая решает проблему GitHub с конфликтами имен классов миграции.

Суть проблемы в том, что если несколько миграций имеют одно и то же имя класса, это вызовет проблемы при попытке воссоздать базу данных с нуля.

Вот пример из [pull request](https://github.com/laravel/framework/pull/36906):

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(
    {
        Schema::table('people', function (Blueprint $table) {
            $table->string('first_name')->nullable();
        });
    }

    public function down()
    {
        Schema::table('people', function (Blueprint $table) {
            $table->dropColumn('first_name');
        });
    }
};
```

Совет от [@nicksdot](https://twitter.com/nicksdot/status/1432340806275198978)

[Вернуться к списку](#Migrations)
___

### Комментарий для столбца

Вы можете добавить «комментарий» о столбце внутри ваших миграций, тем самым предоставите полезную информацию.

Если базой данных управляет кто-то, кроме разработчиков, они могут просматривать комментарии в структуре таблицы перед выполнением каких-либо операций.

```php
$table->unsignedInteger('interval')
    ->index()
    ->comment('This column is used for indexing.')
```

Совет от [@nicksdot](https://twitter.com/nicksdot/status/1432340806275198978)

[Вернуться к списку](#Migrations)
___

### Проверка на существования таблицы или столбца

Вы можете проверить наличие таблицы или столбца, используя методы hasTable и hasColumn:

```php
if (Schema::hasTable('users')) {
    // Таблица "users" существует...
}

if (Schema::hasColumn('users', 'email')) {
    // Таблица "users" найдена и имеет поле "email"...
}
```

Совет от [@dipeshsukhia](https://github.com/dipeshsukhia)

[Вернуться к списку](#Migrations)
___

### Проверка на наличие перед добавлением или удалением столбца

Теперь вы можете добавить столбец в таблицу базы данных, только если он отсутствует, и можете удалить его, если он присутствует. Для этого вводятся следующие методы:

👉 whenTableDoesntHaveColumn

👉 whenTableHasColumn

Доступно для Laravel 9.6.0

```php
return new class extends Migration {
    public function up()
    {
        Schema::whenTableDoesntHaveColumn('users', 'name', function (Blueprint $table) {
            $table->string('name', 30);
        });
    }

    public function down()
    {
        Schema::whenTableHasColumn('users', 'name', function (Blueprint $table) {
            $table->dropColumn('name');
        });
    }
}
```

Совет от [@iamharis010](https://twitter.com/iamharis010/status/1510579415163432961)

[Вернуться к списку](#Migrations)
___

### Переименование поля PostgreSQL

Если вам потребуется переименовать поле, то для PostgreSQL работает следующий прием.

```php
return new class extends Migration {
    public function up()
    {
        DB::transaction(function () {
            DB::statement('ALTER TABLE settings_workflows RENAME COLUMN after_hiring TO after_date');
        });
    }

    public function down()
    {
        DB::transaction(function () {
            DB::statement('ALTER TABLE settings_workflows RENAME COLUMN after_date TO after_hiring');
        });
    }
}
```

Для MySQL поможет такой вариант.

```php
return new class extends Migration {
    public function up()
    {
        Schema::table('settings_workflows', function(Blueprint $table) {
            $table->renameColumn('after_hiring', 'after_date');
        });
    }


    public function down()
    {
        Schema::table('settings_workflows', function(Blueprint $table) {
            $table->renameColumn('after_date', 'after_hiring');
        });
    }
}
```

[Вернуться к списку](#Migrations)
___