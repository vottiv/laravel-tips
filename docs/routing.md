## Routing

- [Объявление групп внутри групп](#Объявление-групп-внутри-групп)
- [Объявление метода resolveRouteBinding в вашей модели](#Объявление-метода-resolveRouteBinding-в-вашей-модели)
- [assign withTrashed() to Route::resource() method](#Назначить-withTrashed-для-метода-Routeresource)
- [Пропустить нормализацию ввода](#Пропустить-нормализацию-ввода)
- [Субдомены с подстановочными знаками](#Поддомены-с-подстановочными-знаками)
- [Что скрывается за маршрутами?](#Что-скрывается-за-маршрутами)
- [Привязка модели маршрута: вы можете определить ключ](#Привязка-модели-маршрута-вы-можете-определить-ключ)
- [Возврат маршрута: если нет другого маршрута](#Возврат-маршрута-если-нет-другого-маршрута)
- [Проверка параметров маршрута с помощью RegExp](#Проверка-параметров-маршрута-с-помощью-RegExp)
- [Ограничение количества запросов](#Ограничение-количества-запросов)
- [Параметры строки запроса для маршрутов](#Параметры-строки-запроса-для-маршрутов)
- [Отдельные маршруты по файлам](#Отдельные-маршруты-по-файлам)
- [Перевести глаголы ресурса](#Перевести-глаголы-ресурса)
- [Пользовательские имена маршрутов ресурсовs](#Пользовательские-имена-маршрутов-ресурсов)
- [Нетерпеливые отношения нагрузки](#Нетерпеливые-отношения-нагрузки)
- [Именование контроллеров ресурсов](#Именование-контроллеров-ресурсов)
- [Легко выделяйте меню панели навигации](#Легко-выделяйте-меню-панели-навигации)
- [Сгенерировать абсолютный путь с помощью помощника route()](#Сгенерировать-абсолютный-путь-с-помощью-помощника-route)
- [Переопределите преобразователь привязки маршрута для каждой из ваших моделей](#Переопределите-преобразователь-привязки-маршрута-для-каждой-из-ваших-моделей)
- [Если вам нужен общедоступный URL, но вы хотите, чтобы они были защищены](#Если-вам-нужен-общедоступный-URL-но-вы-хотите-чтобы-они-были-защищены)
- [Использование Gate в методе промежуточного программного обеспечения](#Использование-Gate-в-методе-промежуточного-программного-обеспечения)
- [Простой маршрут со стрелочной функцией](#Простой-маршрут-со-стрелочной-функцией)
- [Отдача представления напрямую в роуте](#Отдача-представления-напрямую-в-роуте)
- [Каталог маршрута вместо файла маршрута](#Каталог-маршрута-вместо-файла-маршрута)
- [Группировка ресурсов маршрута](#Группировка-ресурсов-маршрута)
- [Пользовательские привязки маршрутов](#Пользовательские-привязки-маршрутов)
- [Два способа проверить имя маршрута](#Два-способа-проверить-имя-маршрута)
- [Привязка модели маршрута к мягко-удаленным моделям](#Привязка-модели-маршрута-к-мягко-удаленным-моделям)
- [Получить URL без параметров запроса](#Получить-URL-без-параметров-запроса)
- [Настройка поведения отсутствующей модели в привязках модели маршрута](#Настройка-поведения-отсутствующей-модели-в-привязках-модели-маршрута)
- [Исключить посредники из роута](#Исключить-посредники-из-роута)
- [Группы контроллеров](#Группы-контроллеров)
___
### Объявление групп внутри групп

Порой необходимо установить свои правила для роутов, которые находятся внутри группы. Laravel позволяет это реализовать без проблем.
```php
Route::group(['prefix' => 'account', 'as' => 'account.'], function() {
    Route::get('login', [AccountController::class, 'login']);
    Route::get('register', [AccountController::class, 'register']);
    Route::group(['middleware' => 'auth'], function() {
        Route::get('edit', [AccountController::class, 'edit']);
    });
});
```
[Вернуться к списку](#Routing)
___

### Объявление метода resolveRouteBinding в вашей модели

Привязка модели маршрута в Laravel — это здорово, но бывают случаи, 
когда мы не можем просто позволить пользователям легко получать доступ к ресурсам по идентификатору. 
Возможно, нам потребуется подтвердить их право собственности на ресурс.

Вы можете объявить метод resolveRouteBinding в своей модели и добавить туда свою пользовательскую логику.

```php
public function resolveRouteBinding($value, $field = null)
{
     $user = request()->user();

     return $this->where([
          ['user_id' => $user->id],
          ['id' => $value]
     ])->firstOrFail();
}
```

Совет от [@notdylanv](https://twitter.com/notdylanv/status/1567296232183447552/)

[Вернуться к списку](#Routing)
___
### Назначить withTrashed() для метода Route::resource()

До Laravel 9.35 - только для Route::get()
```php
Route::get('/users/{user}', function (User $user) {
     return $user->email;
})->withTrashed();
```

С версии Laravel 9.35 - доступно и для `Route::resource()`
```php
Route::resource('users', UserController::class)
     ->withTrashed();
```

Или для конкретного метода
```php
Route::resource('users', UserController::class)
     ->withTrashed(['show']);
```

[Вернуться к списку](#Routing)
___

### Пропустить нормализацию ввода

Laravel автоматически обрезает все входящие строковые поля в запросе. Это называется нормализация ввода.

Иногда вам может не хотеться такого поведения.

Вы можете использовать метод skipWhen в посреднике TrimStrings и вернуть true, чтобы пропустить его.

```php
public function boot()
{
     TrimStrings::skipWhen(function ($request) {
          return $request->is('admin/*);
     });
}
```

Совет от [@Laratips1](https://twitter.com/Laratips1/status/1580210517372596224)

[Вернуться к списку](#Routing)
___

### Поддомены с подстановочными знаками

Вы можете создать группу маршрутов по имени динамического поддомена и передать его значение каждому маршруту.

```php
Route::domain('{username}.workspace.com')->group(function () {
    Route::get('user/{id}', function ($username, $id) {
        //
    });
});
```

[Вернуться к списку](#Routing)
___

### Что скрывается за маршрутами?

Если вы используете [Laravel UI package](https://github.com/laravel/ui), то вы вероятно желаете узнать какие маршруты скрываются за `Auth::routes()`?

Откройте файл `/vendor/laravel/ui/src/AuthRouteMethods.php`.

```php
public function auth()
{
    return function ($options = []) {
        // Authentication Routes...
        $this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
        $this->post('login', 'Auth\LoginController@login');
        $this->post('logout', 'Auth\LoginController@logout')->name('logout');
        // Registration Routes...
        if ($options['register'] ?? true) {
            $this->get('register', 'Auth\RegisterController@showRegistrationForm')->name('register');
            $this->post('register', 'Auth\RegisterController@register');
        }
        // Password Reset Routes...
        if ($options['reset'] ?? true) {
            $this->resetPassword();
        }
        // Password Confirmation Routes...
        if ($options['confirm'] ?? class_exists($this->prependGroupNamespace('Auth\ConfirmPasswordController'))) {
            $this->confirmPassword();
        }
        // Email Verification Routes...
        if ($options['verify'] ?? false) {
            $this->emailVerification();
        }
    };
}
```

Использование этой функции по умолчанию просто так:

```php
Auth::routes(); // без параметров
```

Но вы можете указать параметры для включения или отключения определенных маршрутов:

```php
Auth::routes([
    'login'    => true,
    'logout'   => true,
    'register' => true,
    'reset'    => true,  // для сброса паролей
    'confirm'  => false, // для дополнительных подтверждений пароля
    'verify'   => false, // для проверки электронной почты
]);
```

Совет взят из [предложения](https://github.com/LaravelDaily/laravel-tips/pull/57) от [MimisK13](https://github.com/MimisK13)

[Вернуться к списку](#Routing)
___

### Привязка модели маршрута: вы можете определить ключ

Вы можете выполнить привязку модели в маршруте, например, `Route::get('api/users/{user}', function (User $user) { … }` - но не только по полю ID. Если вы хотите использовать `{user}` из названия `username` поля, поместите это в модель:

```php
public function getRouteKeyName() {
    return 'username';
}
```

[Вернуться к списку](#Routing)
___

### Возврат маршрута: если нет другого маршрута

Если вы хотите указать дополнительную логику для ненайденных маршрутов, вместо того, чтобы просто создавать страницу 404 по умолчанию, вы можете создать для этого специальный маршрут в самом конце вашего файла маршрутов.
```php
Route::group(['middleware' => ['auth'], 'prefix' => 'admin', 'as' => 'admin.'], function () {
    Route::get('/home', [HomeController::class, 'index']);
    Route::resource('tasks', [Admin\TasksController::class]);
});

Route::fallback(function() {
    // код выполнится, если ни один из указанных маршрутов не был найден
});
```

[Вернуться к списку](#Routing)
___

### Проверка параметров маршрута с помощью RegExp

Мы можем проверять параметры непосредственно в маршруте с параметром “where”.
Довольно типичным случаем является префикс ваших маршрутов по языковому стандарту, такие как`fr/blog` или `en/article/333`.

Как мы можем гарантировать, что эти две первые буквы не используются для чего-то другого, кроме языка?

`routes/web.php`:

```php
Route::group([
    'prefix' => '{locale}',
    'where' => ['locale' => '[a-zA-Z]{2}']
], function () {
    Route::get('/', [HomeController::class, 'index']);
    Route::get('article/{id}', [ArticleController::class, 'show']);
});
```

[Вернуться к списку](#Routing)
___

### Ограничение количества запросов

Вы можете ограничить некоторый URL-адрес, который будет вызываться не более 60 раз в минуту, используйте `throttle:60,1`:

```php
Route::middleware('auth:api', 'throttle:60,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

Но также вы можете сделать это отдельно для публичных и для авторизованных пользователей:

```php
// максимум 10 запросов для гостей, 60 для аутентифицированных пользователей
Route::middleware('throttle:10|60,1')->group(function () {
    //
});
```
Кроме того, у вас может быть поле БД users.rate_limit, тем самым, можно ввести лимит для конкретного пользователя:

```php
Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

[Вернуться к списку](#Routing)
___

### Параметры строки запроса для маршрутов

Если вы передадите дополнительные параметры маршруту в массиве, 
эти пары ключ / значение будут автоматически добавлены в строку запроса сгенерированного URL-адреса.

```php
Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']); // Результат: /user/1/profile?photos=yes
```

[Вернуться к списку](#Routing)
___

### Отдельные маршруты по файлам

Если у вас есть набор маршрутов, относящихся к определенному «разделу», 
вы можете выделить их в специальный файл `routesXXXXX.php` и просто включить его в `routes/web.php`

Для примера `routes/auth.php` в [Laravel Breeze](https://github.com/laravel/breeze/blob/1.x/stubs/default/routes/web.php) от самого Тейлора Отвела:

```php
Route::get('/', function () {
    return view('welcome');
});

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth'])->name('dashboard');

require __DIR__.'/auth.php';
```

Где в `routes/auth.php`:

```php
use App\Http\Controllers\Auth\AuthenticatedSessionController;
use App\Http\Controllers\Auth\RegisteredUserController;
// ... больше контроллер

use Illuminate\Support\Facades\Route;

Route::get('/register', [RegisteredUserController::class, 'create'])
                ->middleware('guest')
                ->name('register');

Route::post('/register', [RegisteredUserController::class, 'store'])
                ->middleware('guest');

// ... И еще маршруты
```

Но вы должны использовать этот `include()` только тогда, когда этот отдельный файл маршрута имеет 
те же настройки для префиксов/посредников, иначе необходимо сгруппировать их в `app/Providers/RouteServiceProvider`:

```php
public function boot()
{
    $this->configureRateLimiting();

    $this->routes(function () {
        Route::prefix('api')
            ->middleware('api')
            ->namespace($this->namespace)
            ->group(base_path('routes/api.php'));

        Route::middleware('web')
            ->namespace($this->namespace)
            ->group(base_path('routes/web.php'));

        // ... Ваш файл маршрутов указан где-то тут
    });
}
```

[Вернуться к списку](#Routing)
___

### Перевести глаголы ресурса

Если вы используете контроллеры ресурса, но хотите изменить глаголы URL на неанглийские для целей SEO, 
и желаете вместо `create` использовать испанский `crear`,  то
 можете настроить его с помощью метода `Route::resourceVerbs()` в `App\Providers\RouteServiceProvider`:

```php
public function boot()
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```

[Вернуться к списку](#Routing)
___

### Пользовательские имена маршрутов ресурсов

При использовании контроллеров ресурсов в `routes/web.php` вы можете указать параметр `->names()`, 
поэтому префикс URL-адреса в браузере и префикс имени маршрута, который вы используете во всем проекте Laravel, могут отличаться.

```php
Route::resource('p', ProductController::class)->names('products');
```

Таким образом, приведенный выше код будет генерировать такие URL-адреса, как `p`, `p{id}`, `p{id}edit` и т.д.
Но вы должны вызывать их в коде с помощью `route('products.index')`, маршрут('products.create')` и т. д.

[Вернуться к списку](#Routing)
___

### Нетерпеливые отношения нагрузки

Если вы используете привязку модели маршрута и считаете, что не можете использовать нетерпеливую загрузку для отношений, подумайте еще раз. 

Итак, вы используете привязку модели маршрута

```php
public function show(Product $product) {
    //
}
```

Но у вас есть отношения belongsTo и вы не можете использовать `product->with('category')` с нетерпеливой загрузкой? 

Вы действительно можете! Загрузите отношение с помощью `->load()`

```php
public function show(Product $product) {
    $product->load('category');
    //
}
```

[Вернуться к списку](#Routing)
___

### Именование контроллеров ресурсов

В контроллерах ресурсов в `routes/web.php` можно указать параметр `->names()`, поэтому префикс URL и префикс имени маршрута могут отличаться. Это создаст такие URL-адреса, как `p`, `p{id}`, `p{id}edit` и т. д. Но вы бы назвали их:

- route('products.index)
- route('products.create)
- и т.д.

```php
Route::resource('p', \App\Http\Controllers\ProductController::class)->names('products');
```

[Вернуться к списку](#Routing)
___

### Легко выделяйте меню панели навигации

Используйте `Route::is('route-name')`, чтобы легко выделить меню панели навигации.

```blade
<ul>
    <li @if(Route::is('home')) class="active" @endif>
        <a href="/">Home</a>
    </li>
    <li @if(Route::is('contact-us')) class="active" @endif>
        <a href="/contact-us">Contact us</a>
    </li>
</ul>
```

Совет от [@anwar_nairi](https://twitter.com/anwar_nairi/status/1443893957507747849)

[Вернуться к списку](#Routing)
___

### Сгенерировать абсолютный путь с помощью помощника route()

```php
route('page.show', $page->id);
// http://laravel.test/pages/1

route('page.show', $page->id, false);
// /pages/1
```

Совет от [@oliverds\_](https://twitter.com/oliverds_/status/1445796035742240770)

[Вернуться к списку](#Routing)
___

### Переопределите преобразователь привязки маршрута для каждой из ваших моделей

Вы можете переопределить преобразователь привязки маршрута для каждой из ваших моделей.

В этом примере у меня нет контроля над знаком @ в URL-адресе, поэтому с помощью метода `resolveRouteBinding` я могу удалить знак @ и разрешить модель.

```php
// Route
Route::get('{product:slug}', Controller::class);

// Request
https://nodejs.pub/@unlock/hello-world

// Product Model
public function resolveRouteBinding($value, $field = null)
{
    $value = str_replace('@', '', $value);

    return parent::resolveRouteBinding($value, $field);
}
```

Совет от [@Philo01](https://twitter.com/Philo01/status/1447539300397195269)

[Вернуться к списку](#Routing)
___

### Если вам нужен общедоступный URL, но вы хотите, чтобы они были защищены

Если вам нужен общедоступный URL-адрес, но вы хотите, чтобы он был защищен, используйте подписанный URL-адрес Laravel.

```php
class AccountController extends Controller
{
    public function destroy(Request $request)
    {
        $confirmDeleteUrl = URL::signedRoute('confirm-destroy', [
            $user => $request->user()
        ]);
        // Отправить ссылку по электронной почте...
    }

    public function confirmDestroy(Request $request, User $user)
    {
        if (! $request->hasValidSignature()) {
            abort(403);
        }

        // Пользователь подтверждается, нажав на письмо
        $user->delete();

        return redirect()->route('home');
    }
}
```

Совет от [@anwar_nairi](https://twitter.com/anwar_nairi/status/1448239591467589633)

[Вернуться к списку](#Routing)
___

### Использование Gate в методе промежуточного программного обеспечения

Вы можете использовать гейты, указанные в `App\Providers\AuthServiceProvider` в методе промежуточного программного обеспечения.

Для этого нужно просто поставить внутрь `can:` и имена нужных гейтов.

```php
Route::put('/post/{post}', function (Post $post) {
    // Текущий пользователь может обновить сообщение...
})->middleware('can:update,post');
```

[Вернуться к списку](#Routing)
___

### Простой маршрут со стрелочной функцией

Вы можете использовать функцию стрелки php в маршрутизации без использования анонимной функции.

Для этого вы можете использовать `fn() =>`, это выглядит проще.

```php
// Вместо
Route::get('/example', function () {
    return User::all();
});

// Вы можете
Route::get('/example', fn () => User::all());
```

[Вернуться к списку](#Routing)
___

### Отдача представления напрямую в роуте

Вы можете использовать `Route::view(uri, bladePage)`, чтобы вернуть представление напрямую, без использования функции контроллера.

```php
//this will return home.blade.php view
Route::view('/home', 'home');
```

[Вернуться к списку](#Routing)
___

### Каталог маршрута вместо файла маршрута

Вы можете создать каталог _/routes/web/_ и заполнить _/routes/web.php_ только:

```php
foreach(glob(dirname(__FILE__).'/web/*', GLOB_NOSORT) as $route_file){
    include $route_file;
}
```

Теперь каждый файл внутри _/routes/web/_ действует как файл веб-маршрутизатора, и вы можете организовать свои маршруты в разные файлы.

[Вернуться к списку](#Routing)
___

### Группировка ресурсов маршрута

Если в ваших маршрутах много контроллеров ресурсов, вы можете сгруппировать их и вызвать один Route::resources() вместо множества одиночных операторов Route::resource().

```php
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

[Вернуться к списку](#Routing)
___

### Пользовательские привязки маршрутов

Знаете ли вы, что в Laravel можно определить собственные привязки маршрутов?

В этом примере мне нужно реализовать портфолио с помощью слаг. Но слаг не уникален, потому что несколько пользователей могут иметь портфолио с именем «Foo».

Поэтому я определяю, как Laravel должен разрешать их, из параметра маршрута:

```php
class RouteServiceProvider extends ServiceProvider
{
    public const HOME = '/dashboard';

    public function boot()
    {
        Route::bind('portfolio', function (string $slug) {
            return Portfolio::query()
                ->whereBelongsto(request()->user())
                ->whereSlug($slug)
                ->firstOrFail();
        });
    }
}
```

```php
Route::get('portfolios/{portfolio}', function (Portfolio $portfolio) {
    /*
     * Портфолио будет результатом запроса, определенного RouteServiceProvider
     */
})
```

Совет от [@mmartin_joo](https://twitter.com/mmartin_joo/status/1496871240346509312)

[Вернуться к списку](#Routing)
___

### Два способа проверить имя маршрута

Вот два способа проверить имя маршрута в Laravel.

```php
// #1
<a
    href="{{ route('home') }}"
    @class="['navbar-link', 'active' => Route::current()->getName() === 'home']"
>
    Home
</a>
// #2
<a
    href="{{ route('home') }}"
    @class="['navbar-link', 'active' => request()->routeIs('home)]"
>
    Home
</a>
```

Совет от [@AndrewSavetchuk](https://twitter.com/AndrewSavetchuk/status/1510197418909999109)

[Вернуться к списку](#Routing)
___

### Привязка модели маршрута к мягко-удаленным моделям

By default, when using route model binding will not retrieve models that have been soft-deleted.
You can change that behavior by using `withTrashed` in your route.

```php
Route::get('/posts/{post}', function (Post $post) {
    return $post;
})->withTrashed();
```

Совет от [@cosmeescobedo](https://twitter.com/cosmeescobedo/status/1511154599255703553)

[Вернуться к списку](#Routing)
___

### Получить URL без параметров запроса

Если по какой-то причине ваш URL-адрес имеет параметры запроса, вы можете получить URL-адрес без параметров запроса, используя метод запроса `fullUrlWithoutQuery` следующим образом:

```php
// Исходный URL: https://www.amitmerchant.com?search=laravel&lang=en&sort=desc
$urlWithQueryString = $request->fullUrlWithoutQuery([
    'lang',
    'sort'
]);
echo $urlWithQueryString;
// Результат: https://www.amitmerchant.com?search=laravel
```

Совет от [@amit_merchant](https://twitter.com/amit_merchant/status/1510867527962066944)

[Вернуться к списку](#Routing)
___

### Настройка поведения отсутствующей модели в привязках модели маршрута

По умолчанию Laravel выдает ошибку 404, когда не может связать модель, но вы можете изменить это поведение, передав замыкание отсутствующему методу.

```php
Route::get('/users/{user}', [UsersController::class, 'show'])
    ->missing(function ($parameters) {
        return Redirect::route('users.index');
    });
```

Совет от [@cosmeescobedo](https://twitter.com/cosmeescobedo/status/1511322007576608769)

[Вернуться к списку](#Routing)
___

### Исключить посредники из роута

Вы можете исключить посредника на уровне маршрута в Laravel, используя метод без промежуточного ПО.

```php
Route::post('/some/route', SomeController::class)
    ->withoutMiddleware([VerifyCsrfToken::class]);
```

Совет от [@alexjgarrett](https://twitter.com/alexjgarrett/status/1512529798790320129)

[Вернуться к списку](#Routing)
___

### Группы контроллеров

Вместо использования контроллера в каждом маршруте рассмотрите возможность использования группы контроллеров маршрута. Добавлено в Laravel начиная с версии 8.80.

```php
// Было
Route::get('users', [UserController::class, 'index']);
Route::post('users', [UserController::class, 'store']);
Route::get('users/{user}', [UserController::class, 'show']);
Route::get('users/{user}/ban', [UserController::class, 'ban']);
// Стало
Route::controller(UsersController::class)->group(function () {
    Route::get('users', 'index');
    Route::post('users', 'store');
    Route::get('users/{user}', 'show');
    Route::get('users/{user}/ban', 'ban');
});
```

Совет от [@justsanjit](https://twitter.com/justsanjit/status/1514943541612527616)

[Вернуться к списку](#Routing)
___