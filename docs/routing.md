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
- [Translate Resource Verbs](#translate-resource-verbs)
- [Custom Resource Route Names](#custom-resource-route-names)
- [Eager load relationship](#eager-load-relationship)
- [Localizing Resource URIs](#localizing-resource-uris)
- [Resource Controllers naming](#resource-controllers-naming)
- [Easily highlight your navbar menus](#easily-highlight-your-navbar-menus)
- [Generate absolute path using route() helper](#generate-absolute-path-using-route-helper)
- [Override the route binding resolver for each of your models](#override-the-route-binding-resolver-for-each-of-your-models)
- [If you need public URL, but you want them to be secured](#if-you-need-public-url-but-you-want-them-to-be-secured)
- [Using Gate in middleware method](#using-gate-in-middleware-method)
- [Simple route with arrow function](#simple-route-with-arrow-function)
- [Route view](#route-view)
- [Route directory instead of route file](#route-directory-instead-of-route-file)
- [Route resources grouping](#route-resources-grouping)
- [Custom route bindings](#custom-route-bindings)
- [Two ways to check the route name](#two-ways-to-check-the-route-name)
- [Route model binding soft-deleted models](#route-model-binding-soft-deleted-models)
- [Retrieve the URL without query parameters](#retrieve-the-url-without-query-parameters)
- [Customizing Missing Model Behavior in route model bindings](#customizing-missing-model-behavior-in-route-model-bindings)
- [Exclude middleware from a route](#exclude-middleware-from-a-route)
- [Controller groups](#controller-groups)

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

### Поддомены с подстановочными знаками

Вы можете создать группу маршрутов по имени динамического поддомена и передать его значение каждому маршруту.

```php
Route::domain('{username}.workspace.com')->group(function () {
    Route::get('user/{id}', function ($username, $id) {
        //
    });
});
```

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

### Привязка модели маршрута: вы можете определить ключ

Вы можете выполнить привязку модели в маршруте, например, `Route::get('api/users/{user}', function (User $user) { … }` - но не только по полю ID. Если вы хотите использовать `{user}` из названия `username` поля, поместите это в модель:

```php
public function getRouteKeyName() {
    return 'username';
}
```

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

### Параметры строки запроса для маршрутов

Если вы передадите дополнительные параметры маршруту в массиве, 
эти пары ключ / значение будут автоматически добавлены в строку запроса сгенерированного URL-адреса.

```php
Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']); // Результат: /user/1/profile?photos=yes
```

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

### Translate Resource Verbs

If you use resource controllers, but want to change URL verbs 
to non-English for SEO purposes, so instead of `/create` you want Spanish `/crear`, 
you can configure it by using `Route::resourceVerbs()` method in `App\Providers\RouteServiceProvider`:

```php
public function boot()
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);

    // ...
}
```

### Custom Resource Route Names

When using Resource Controllers, in `routes/web.php` you can specify `->names()` parameter, so the URL prefix in the browser and the route name prefix you use all over Laravel project may be different.

```php
Route::resource('p', ProductController::class)->names('products');
```

So this code above will generate URLs like `/p`, `/p/{id}`, `/p/{id}/edit`, etc.
But you would call them in the code by `route('products.index')`, `route('products.create')`, etc.

### Eager load relationship

If you use Route Model Binding and think you can't use Eager Loading for relationships, think again.

So you use Route Model Binding

```php
public function show(Product $product) {
    //
}
```

But you have a belongsTo relationship, and cannot use $product->with('category') eager loading?

You actually can! Load the relationship with `->load()`

```php
public function show(Product $product) {
    $product->load('category');
    //
}
```

### Localizing Resource URIs

If you use resource controllers, but want to change URL verbs to non-English, so instead of `/create` you want Spanish `/crear`, you can configure it with `Route::resourceVerbs()` method.

```php
public function boot()
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
    //
}
```

### Resource Controllers naming

In Resource Controllers, in `routes/web.php` you can specify `->names()` parameter, so the URL prefix and the route name prefix may be different.

This will generate URLs like `/p`, `/p/{id}`, `/p/{id}/edit` etc. But you would call them:

- route('products.index)
- route('products.create)
- etc

```php
Route::resource('p', \App\Http\Controllers\ProductController::class)->names('products');
```

### Easily highlight your navbar menus

Use `Route::is('route-name')` to easily highlight your navbar menus

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

Tip given by [@anwar_nairi](https://twitter.com/anwar_nairi/status/1443893957507747849)

### Generate absolute path using route() helper

```php
route('page.show', $page->id);
// http://laravel.test/pages/1

route('page.show', $page->id, false);
// /pages/1
```

Tip given by [@oliverds\_](https://twitter.com/oliverds_/status/1445796035742240770)

### Override the route binding resolver for each of your models

You can override the route binding resolver for each of your models. In this example, I have no control over the @ sign in the URL, so using the `resolveRouteBinding` method, I'm able to remove the @ sign and resolve the model.

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

Tip given by [@Philo01](https://twitter.com/Philo01/status/1447539300397195269)

### If you need public URL, but you want them to be secured

If you need public URL but you want them to be secured, use Laravel signed URL

```php
class AccountController extends Controller
{
    public function destroy(Request $request)
    {
        $confirmDeleteUrl = URL::signedRoute('confirm-destroy', [
            $user => $request->user()
        ]);
        // Send link by email...
    }

    public function confirmDestroy(Request $request, User $user)
    {
        if (! $request->hasValidSignature()) {
            abort(403);
        }

        // User confirmed by clicking on the email
        $user->delete();

        return redirect()->route('home');
    }
}
```

Tip given by [@anwar_nairi](https://twitter.com/anwar_nairi/status/1448239591467589633)

### Using Gate in middleware method

You can use the gates you specified in `App\Providers\AuthServiceProvider` in middleware method.

To do this, you just need to put inside the `can:` and the names of the necessary gates.

```php
Route::put('/post/{post}', function (Post $post) {
    // The current user may update the post...
})->middleware('can:update,post');
```

### Simple route with arrow function

You can use php arrow function in routing, without having to use anonymous function.

To do this, you can use `fn() =>`, it looks easier.

```php
// Instead of
Route::get('/example', function () {
    return User::all();
});

// You can
Route::get('/example', fn () => User::all());
```

### Route view

You can use `Route::view($uri , $bladePage)` to return a view directly, without having to use controller function.

```php
//this will return home.blade.php view
Route::view('/home', 'home');
```

### Route directory instead of route file

You can create a _/routes/web/_ directory and only fill _/routes/web.php_ with:

```php
foreach(glob(dirname(__FILE__).'/web/*', GLOB_NOSORT) as $route_file){
    include $route_file;
}
```

Now every file inside _/routes/web/_ act as a web router file and you can organize your routes into different files.

### Route resources grouping

If your routes have a lot of resource controllers, you can group them and call one Route::resources() instead of many single Route::resource() statements.

```php
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

### Custom route bindings

Did you know you can define custom route bindings in Laravel?

In this example, I need to resolve a portfolio by slug. But the slug is not unique, because multiple users can have a portfolio named 'Foo'

So I define how Laravel should resolve them from a route parameter

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
     * The $portfolio will be the result of the query defined in the RouteServiceProvider
     */
})
```

Tip given by [@mmartin_joo](https://twitter.com/mmartin_joo/status/1496871240346509312)

### Two ways to check the route name

Here are two ways to check the route name in Laravel.

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

Tip given by [@AndrewSavetchuk](https://twitter.com/AndrewSavetchuk/status/1510197418909999109)

### Route model binding soft-deleted models

By default, when using route model binding will not retrieve models that have been soft-deleted.
You can change that behavior by using `withTrashed` in your route.

```php
Route::get('/posts/{post}', function (Post $post) {
    return $post;
})->withTrashed();
```

Tip given by [@cosmeescobedo](https://twitter.com/cosmeescobedo/status/1511154599255703553)

### Retrieve the URL without query parameters

If for some reason, your URL is having query parameters, you can retrieve the URL without query parameters using the `fullUrlWithoutQuery` method of request like so.

```php
// Original URL: https://www.amitmerchant.com?search=laravel&lang=en&sort=desc
$urlWithQueryString = $request->fullUrlWithoutQuery([
    'lang',
    'sort'
]);
echo $urlWithQueryString;
// Outputs: https://www.amitmerchant.com?search=laravel
```

Tip given by [@amit_merchant](https://twitter.com/amit_merchant/status/1510867527962066944)

### Customizing Missing Model Behavior in route model bindings

By default, Laravel throws a 404 error when it can't bind the model, but you can change that behavior by passing a closure to the missing method.

```php
Route::get('/users/{user}', [UsersController::class, 'show'])
    ->missing(function ($parameters) {
        return Redirect::route('users.index');
    });
```

Tip given by [@cosmeescobedo](https://twitter.com/cosmeescobedo/status/1511322007576608769)

### Exclude middleware from a route

You can exclude middleware at the route level in Laravel using the withoutMiddleware method.

```php
Route::post('/some/route', SomeController::class)
    ->withoutMiddleware([VerifyCsrfToken::class]);
```

Tip given by [@alexjgarrett](https://twitter.com/alexjgarrett/status/1512529798790320129)

### Controller groups

Instead of using the controller in each route, consider using a route controller group. Added to Laravel since v8.80

```php
// Before
Route::get('users', [UserController::class, 'index']);
Route::post('users', [UserController::class, 'store']);
Route::get('users/{user}', [UserController::class, 'show']);
Route::get('users/{user}/ban', [UserController::class, 'ban']);
// After
Route::controller(UsersController::class)->group(function () {
    Route::get('users', 'index');
    Route::post('users', 'store');
    Route::get('users/{user}', 'show');
    Route::get('users/{user}/ban', 'ban');
});
```

Tip given by [@justsanjit](https://twitter.com/justsanjit/status/1514943541612527616)

