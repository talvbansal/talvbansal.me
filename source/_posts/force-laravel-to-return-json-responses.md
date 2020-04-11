---
title: Force Laravel to return JSON responses
date: 2020-04-06 20:32:00
tags:
- Laravel
- JSON
- Javascript
- Middleware

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1586202198/posts/bagan-baloons.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1586202198/posts/bagan-baloons.jpg
thumbnailImagePosition: right
---
Today I was working on an API written in Laravel for a React Native app with another developer.

He was trying to make requests to the Laravel backend and told me he kept receiving a response with a http status code of 302. 3XX http status codes are redirection status codes. 

It turned out that he had not set an accept header on the requests to the server that the app was making.

By default if you dont set a requests accept headers they default to `Accept: */*`. With those set laravel responds with a `Content-Type` of `text/html`.

For the purpose of this project every api response needs to return JSON.

We can achieve this really easily by using middleware to overwrite the `Accept` headers on the incoming request and setting them to `application/json`.

<!-- more -->

Create a new middleware class:
```bash
php artisan make:middlware RespondWithJsonMiddleware
```

Now open the new file created in `app/Http/Middlware` and change the handle method to look like this:

```php
    public function handle($request, Closure $next)
    {
        $request->headers->set('Accept', 'application/json');

        return $next($request);
    }
```
And thats as simple as it gets, now we just need to apply that middleware to a given set of routes.

### Applying to all API routes

For my purposes I wanted every request to return JSON so I added this new class to api global middleware group in `app/Http/Kernel.php`

```php
...
        'api' => [
            'throttle:60,1',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
            App\Http\Middlware\RespondWithJsonMiddleware::class,
        ],
...
```

If we look at the `app/Providers/RouteServiceProvider.php` file we can see that this `api` middleware group is applied to all of the routes in ``routes/api.php`:

```php
    protected function mapApiRoutes()
    {
        Route::prefix('api')
            ->middleware('api')
            ->namespace($this->namespace)
            ->group(base_path('routes/api.php'));
    }
```

### Applying to specific routes

If we didn't want to apply the new middleware to every api route and to a given controller we can do so like this.

First remove the new middleware from the api group

```php
// app/Http/Kernel.php
...
        'api' => [
            'throttle:60,1',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
...
```

Now add the new middleware item to the `$routeMiddleware` array in the same file

```php
protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
...
        'respond.json' => App\Http\Middlware\RespondWithJsonMiddleware::class,
];
```

Then use the middleware as you normally would in your routes file

```php
// routes/api.php
Route::group(['middleware' => ['respond.json']], function () {
    // routes in here...
}
 
```

### Anonymously in a controller
We can also apply this code to a controller using the controller middleware function.

```php
class IndexController extends Controller
{

    public function __construct()
    {
        $this->middleware(function ($request, $next) {
            $request->headers->set('Accept', 'application/json');

            return $next($request);
        });
    }

...
```