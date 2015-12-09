With the mechanisms for registering and logging in/out built, you can now adapt your application to do different things based on whether the user is logged in not.




## Auth::check()
Let's start by adapting the Foobooks navigation menu

If the user *is* logged in they should these links:
+ Home
+ Add a book
+ Log out

If the user is *not* logged in, they should see these links:
+ Home
+ Register
+ Login

To accomplish this, we'll use the __`Auth::check()`__ method which returns `True` if the user is logged in or `False` if they are not.

In `resources/views/layouts/master.blade.php` the `<nav>` will be updated to look like this:

```php
<nav>
    <ul>
        @if(Auth::check())
            <li><a href='/'>Home</a></li>
            <li><a href='/books/create'>Add a book</a></li>
            <li><a href='/logout'>Log out</a></li>
        @else
            <li><a href='/'>Home</a></li>
            <li><a href='/login'>Log in</a></li>
            <li><a href='/register'>Register</a></li>
        @endif
    </ul>
</nav>
```

<img src='http://making-the-internet.s3.amazonaws.com/laravel-menu-for-guest-vs-user@2x.png' style='max-width:1008px; width:100%' alt=''>




## Manually restricting routes
One of the purposes of authentication is to keep non-registered users away from certain routes.

For Foobooks, we want to set it up so that any visitor can view books, but only logged in users can add books.

One way we could approach this is to update the `getCreate()` controller action so it redirects the user away from the page if they're not logged in like so:

```php
/**
 * Responds to requests to GET /books/create
 */
public function getCreate() {
    if(!\Auth::check() ) {
        \Session::flash('flash_message','You have to be logged in to create a new book');
        return redirect('/');
    }
    return view('books.create');
}
```

This works, but can become tedious if you have many actions that need to be restricted. An alternative method is to use Middleware...




## Restricting routes with Middleware
Laravel uses a feature called [Middleware](http://laravel.com/docs/middleware) which lets you filter requests entering your application. (Earlier, we saw an example of Middleware with the CSRF checking that happens when forms are submitted.)

Laravel ships with a default Middleware file for authentication which can be found at `/app/Http/Middleware/Authenticate.php`. When this Middleware filter is used, a check is done to see if a user is authenticated, and if not they're redirected to the login page.

This is all managed by the `handle()` method:

```php
# /app/Http/Middleware/Authenticate.php
public function handle($request, Closure $next)
{
    if ($this->auth->guest()) {
        if ($request->ajax()) {
            return response('Unauthorized.', 401);
        } else {
            return redirect()->guest('auth/login');
        }
    }

    return $next($request);
}
```

We want to alter this method with two changes:
1. Change the redirect destination from `/auth/login` to `/login` (because that's the route we used in our login setup)
2. Before doing the redirect, include a flash message so we can let the user know why they were redirected to the login page.

The updated `handle()` method looks like this:
```php
public function handle($request, Closure $next)
{
    if ($this->auth->guest()) {
        if ($request->ajax()) {
            return response('Unauthorized.', 401);
        } else {
            \Session::flash('flash_message','You must be logged in to access this page.');
            return redirect()->guest('/login');
        }
    }

    return $next($request);
}
```

Next, using this Middleware...

To understand how this Middleware is used, you should open `/app/Http/Kernel.php` and note that it's part of the `$routeMiddleWare` array with the key `auth`:

```php
protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
];
```

Knowing this, we'll adapt a route to use this Middleware.

For example, here's the existing route to view the `/books/create` page:

```php
Route::get('/books/create', 'BookController@getCreate');
```

We'll update this route to look like this:

```php
Route::get('/books/create', [
    'middleware' => 'auth',
    'uses' => 'BookController@getCreate'
]);
```


__Test it out__
If you are not logged in and you attempt to visit `http://localhost/books/create` you should be redirected to the login page, with a flash message telling you `You must be logged in to access this page.`.

The redirect that got you here was executed with this line: `redirect()->guest('/login');`

The [guest method](http://laravel.com/api/5.0/Illuminate/Routing/Redirector.html#method_guest) will store the current URL in the Session before doing the redirect. Because of this, when you do log in, it will know what URL you came from (`/books/create`) and it will send you back there.

This is a logical flow: If your user tries to access a page, but is denied access and asked to log in...upon logging in, they should automatically be sent to the page they were attempting to visit.




## Restricting multiple routes with Middleware
In the above example, we restricted a single route, but you can also restrict many routes using a route group like this:

```php
Route::group(['middleware' => 'auth'], function () {
    Route::get('/books/create', 'BookController@getCreate');
    Route::post('/books/create', 'BookController@postCreate');

    Route::get('/books/edit/{id?}', 'BookController@getEdit');
    Route::post('/books/edit', 'BookController@postEdit');
});
```


## Restricting routes to guests only
In the above examples we restricted routes to logged in users.

On the flip side, you may also want to restrict routes so that only &ldquo;guests&rdquo; (i.e. non-logged in users) can see them.

For example, if your user is already logged in, you don't want them to be able to visit `http://localhost/login` or `http://localhost/register`

Turns out, some of this functionality is already set up for you with the baked in authentication files Laravel gives you, so lets trace that.

First, look at the `AuthController.php` and note the `__construct()` method invokes a middleware `guest`:

```php
public function __construct()
{
    $this->middleware('guest', ['except' => 'getLogout']);
}
```

The `__construct()` method is a special method that gets invoked before any action in this controller. This means, that every action in AuthController will use the `guest` middleware, *except* for `getLogout` (as indicated).

So what does the `guest` Middleware do?
Again we can refer to the `Kernel.php`:

```php
protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
];
```

here we see that the `guest` Middleware is using `/app/Http/Middleware/RedirectIfAuthenticated.php` and in that file we see this `handle()` method:

```php
public function handle($request, Closure $next)
{
    if ($this->auth->check()) {
        return redirect('/home');
    }

    return $next($request);
}
```

This method is simple enough: If the user is authorized, redirect them to `/home`.

For our purposes, you should update the redirect to go to just `/`:

```php
# /app/Http/Middleware/RedirectIfAuthenticated.php Line 38
return redirect('/');
```

After saving this change, you can test if it's working by logging in (`http://localhost/login`) and then attempting to visit `http://localhost/login` or `http://localhost/register`.

If everything is working, it should redirect you to `http://localhost/`.



## Reference
+ [Docs: Authentication](http://laravel.com/docs/authentication)
+ [Docs: Middleware](http://laravel.com/docs/middleware)
